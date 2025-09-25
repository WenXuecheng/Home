<template>
  <div
    ref="wrapperRef"
    :class="['infinite-scroll-wrapper', className]"
  >
    <div
      ref="containerRef"
      class="infinite-scroll-container"
      :style="containerInlineStyle"
    >
      <div
        v-for="(item, index) in items"
        :key="itemKey(item, index)"
        class="infinite-scroll-item"
        :style="itemInlineStyle"
      >
        <slot :item="item" :index="index">
          {{ item }}
        </slot>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed, nextTick, onMounted, onUnmounted, ref, useTemplateRef, watch } from 'vue';
import { gsap } from 'gsap';
import { Observer } from 'gsap/Observer';

gsap.registerPlugin(Observer);

type TiltDirection = 'left' | 'right';

interface ScrollStackProps<T = unknown> {
  items?: T[];
  className?: string;
  width?: string;
  maxHeight?: string;
  itemMinHeight?: number;
  itemMinWidth?: number;
  itemMinWidthMobile?: number;
  itemGapMobile?: number;
  itemGap?: number;
  isTilted?: boolean;
  tiltDirection?: TiltDirection;
  autoplay?: boolean;
  autoplayHoldDuration?: number;
  autoplayCenterHoldDuration?: number;
  autoplayEdgeHoldDuration?: number;
  autoplayTransitionDuration?: number;
  autoplayDirection?: 'down' | 'up';
  pauseOnHover?: boolean;
}

const props = withDefaults(defineProps<ScrollStackProps>(), {
  items: () => [],
  className: '',
  width: '100%',
  maxHeight: '100%',
  itemMinHeight: 420,
  itemMinWidth: 420,
  itemMinWidthMobile: undefined,
  itemGapMobile: undefined,
  itemGap: 48,
  isTilted: false,
  tiltDirection: 'left',
  autoplay: true,
  autoplayHoldDuration: 2.5,
  autoplayTransitionDuration: 1.1,
  autoplayDirection: 'down',
  pauseOnHover: true
});

const wrapperRef = useTemplateRef<HTMLDivElement>('wrapperRef');
const containerRef = useTemplateRef<HTMLDivElement>('containerRef');
const observer = ref<Observer | null>(null);
let stopTicker: (() => void) | null = null;
let startTicker: (() => void) | null = null;
let velocity = 0;
let visibilityObserver: IntersectionObserver | null = null;
let startAutoplayCycle: (() => void) | null = null;
let stopAutoplayCycle: (() => void) | null = null;
let autoplayTween: gsap.core.Tween | null = null;
let autoplayDelay: gsap.core.DelayedCall | null = null;
let wrapFunction: ((n: number) => number) | null = null;
let elementsCache: HTMLDivElement[] = [];
let scrollSteps: number[] = [];
let currentIndex = 0;
let resizeHandler: (() => void) | null = null;
let nextHoldPhase: 'center' | 'edge' = 'center';
const viewportWidth = ref(typeof window !== 'undefined' ? window.innerWidth : 0);

const updateCardMetrics = (elements: HTMLDivElement[]) => {
  const container = containerRef.value;
  if (!container || !elements.length) {
    if (!elements.length) {
      currentIndex = 0;
    }
    return;
  }

  const containerRect = container.getBoundingClientRect();
  const centerX = containerRect.left + containerRect.width / 2;
  const halfWidth = containerRect.width / 2 || 1;

  let bestFocus = -Infinity;
  let focusIndex = currentIndex;

  elements.forEach((child, index) => {
    const card = child.firstElementChild as HTMLElement | null;
    if (!card) return;
    const rect = child.getBoundingClientRect();
    const cardCenter = rect.left + rect.width / 2;
    const distance = (cardCenter - centerX) / halfWidth;
    const clamped = Math.max(-1.2, Math.min(1.2, distance));
    const focus = Math.max(0, 1 - Math.abs(clamped));
    const eased = Math.pow(focus, 0.6);
    card.style.setProperty('--center-distance', clamped.toFixed(3));
    card.style.setProperty('--focus-progress', eased.toFixed(3));

    if (eased > bestFocus) {
      bestFocus = eased;
      focusIndex = index;
    }
  });

  currentIndex = focusIndex;
};

const itemKey = (item: unknown, index: number) => {
  if (item && typeof item === 'object' && 'id' in (item as Record<string, unknown>)) {
    return (item as { id?: string | number }).id ?? index;
  }
  return index;
};

const getTiltTransform = () => {
  if (!props.isTilted) return 'none';
  return props.tiltDirection === 'left'
    ? 'rotateX(20deg) rotateZ(-16deg) skewX(16deg)'
    : 'rotateX(20deg) rotateZ(16deg) skewX(-16deg)';
};

const containerInlineStyle = computed(() => ({
  width: props.width,
  maxHeight: props.maxHeight,
  transform: getTiltTransform(),
  transformOrigin: 'center center',
  transformStyle: 'preserve-3d'
}));

const resolveItemMinWidth = () => {
  const mobileThreshold = 768;
  if (viewportWidth.value <= mobileThreshold) {
    if (typeof props.itemMinWidthMobile === 'number') {
      return props.itemMinWidthMobile;
    }
  }
  return props.itemMinWidth;
};

const resolveItemGap = () => {
  const mobileThreshold = 768;
  if (viewportWidth.value <= mobileThreshold) {
    if (typeof props.itemGapMobile === 'number') {
      return props.itemGapMobile;
    }
  }
  return props.itemGap;
};

const itemInlineStyle = computed(() => ({
  minHeight: `${props.itemMinHeight}px`,
  minWidth: `${resolveItemMinWidth()}px`,
  marginRight: `${resolveItemGap()}px`
}));

const centerHoldDuration = computed(() =>
  props.autoplayCenterHoldDuration ?? props.autoplayHoldDuration
);

const edgeHoldDuration = computed(() =>
  props.autoplayEdgeHoldDuration ?? props.autoplayHoldDuration
);

const hasEdgeHold = computed(() => (edgeHoldDuration.value ?? 0) > 0);

const cleanup = () => {
  const container = containerRef.value;

  if (observer.value) {
    observer.value.kill();
    observer.value = null;
  }

  if (visibilityObserver) {
    visibilityObserver.disconnect();
    visibilityObserver = null;
  }

  stopAutoplayCycle?.();

  if (container) {
    gsap.killTweensOf(Array.from(container.children));
    if (props.pauseOnHover && stopTicker && startTicker) {
      container.removeEventListener('mouseenter', stopTicker);
      container.removeEventListener('mouseleave', startTicker);
    }
  }

  if (resizeHandler) {
    window.removeEventListener('resize', resizeHandler);
    resizeHandler = null;
  }

  stopTicker = null;
  startTicker = null;
  velocity = 0;
  autoplayTween = null;
  autoplayDelay = null;
  startAutoplayCycle = null;
  stopAutoplayCycle = null;
  wrapFunction = null;
  elementsCache = [];
  scrollSteps = [];
  currentIndex = 0;
  nextHoldPhase = 'center';
};

const initializeScroll = () => {
  const container = containerRef.value;
  if (!container) return;
  if (typeof window !== 'undefined') {
    viewportWidth.value = window.innerWidth;
  }
  const elements = gsap.utils.toArray<HTMLDivElement>(container.children);
  if (!elements.length) return;

  elementsCache = elements;

  if (visibilityObserver) {
    visibilityObserver.disconnect();
    visibilityObserver = null;
  }

  if (typeof window !== 'undefined' && 'IntersectionObserver' in window) {
    visibilityObserver = new IntersectionObserver(
      entries => {
        entries.forEach(entry => {
          const card = (entry.target as HTMLElement).firstElementChild as HTMLElement | null;
          if (!card) return;
          const visibility = Math.max(entry.intersectionRatio, 0);
          card.style.setProperty('--scroll-visibility', visibility.toFixed(3));
          const isVisible = entry.isIntersecting && visibility > 0.1;
          card.classList.toggle('is-visible', isVisible);
        });
        updateCardMetrics(elementsCache);
      },
      {
        root: container,
        threshold: [0, 0.05, 0.1, 0.15, 0.25, 0.35, 0.5, 0.65, 0.8, 0.95, 1]
      }
    );
  } else {
    elements.forEach(child => {
      const card = child.firstElementChild as HTMLElement | null;
      if (!card) return;
      card.style.setProperty('--scroll-visibility', '1');
      card.classList.add('is-visible');
    });
  }

  const sizes = elements.map(child => {
    const style = getComputedStyle(child);
    const width = child.offsetWidth;
    const marginLeft = parseFloat(style.marginLeft || '0');
    const marginRight = parseFloat(style.marginRight || '0');
    return {
      width,
      marginLeft,
      marginRight,
      total: width + marginLeft + marginRight
    };
  });

  const totalWidth = sizes.reduce((acc, size) => acc + size.total, 0);
  const wrapFn = gsap.utils.wrap(-totalWidth, totalWidth);
  wrapFunction = wrapFn;

  const positions: number[] = [];
  let offset = 0;
  elements.forEach((child, index) => {
    const size = sizes[index];
    offset += size.marginLeft;
    positions[index] = offset;
    offset += size.width + size.marginRight;
    if (visibilityObserver) {
      visibilityObserver.observe(child);
    } else {
      const card = child.firstElementChild as HTMLElement | null;
      card?.style.setProperty('--scroll-visibility', '1');
    }
  });

  const fallbackStep = resolveItemMinWidth() + resolveItemGap();
  scrollSteps = positions.map((position, index) => {
    if (positions.length === 1) {
      return sizes[index]?.total || fallbackStep;
    }
    const nextIndex = (index + 1) % positions.length;
    let distance = (positions[nextIndex] ?? 0) - position;
    if (distance <= 0) {
      distance += totalWidth;
    }
    return distance || fallbackStep;
  });
  if (!scrollSteps.length) {
    scrollSteps = [fallbackStep];
  }

  const containerWidth = container.offsetWidth || 1;
  const firstSize = sizes[0];
  const firstPosition = positions[0] ?? 0;
  const initialShift = firstSize
    ? containerWidth / 2 - (firstPosition + firstSize.width / 2)
    : 0;

  elements.forEach((child, index) => {
    const x = (positions[index] ?? 0) + initialShift;
    gsap.set(child, { x });
  });

  updateCardMetrics(elementsCache);

  let stopMomentum = () => {};

  observer.value = Observer.create({
    target: container,
    type: 'wheel,touch,pointer',
    preventDefault: true,
    onPress: () => {
      if (stopMomentum) stopMomentum();
    },
    onRelease: () => {
      const momentum = velocity * 0.8;
      if (Math.abs(momentum) > 0.1) {
        const tween = gsap.to(elements, {
          duration: 1.3,
          ease: 'power2.out',
          x: `+=${momentum}`,
          modifiers: { x: gsap.utils.unitize(wrapFn) },
          onUpdate: () => updateCardMetrics(elementsCache)
        });
        stopMomentum = () => tween.kill();
      }
      velocity = 0;
    },
    onChange: ({ deltaX, deltaY, isDragging, event }) => {
      const wheelDelta = -deltaY || -deltaX;
      const pointerDelta = deltaX || deltaY;
      const direction = event.type === 'wheel' ? wheelDelta : pointerDelta;
      const distance = isDragging ? direction * 5 : direction * 1.2;
      velocity = distance * 0.45;

      elements.forEach(child => {
        gsap.to(child, {
          duration: isDragging ? 0.28 : 0.8,
          ease: isDragging ? 'power1.out' : 'power3.out',
          x: `+=${distance}`,
          modifiers: { x: gsap.utils.unitize(wrapFn) },
          onUpdate: () => updateCardMetrics(elementsCache)
        });
      });
    }
  });

  container.style.touchAction = 'none';

  const getDirectionFactor = () => (props.autoplayDirection === 'down' ? 1 : -1);

  const resolveStepDistance = () => {
    if (!elementsCache.length || !scrollSteps.length) return 0;
    const stepsCount = scrollSteps.length;
    if (!stepsCount) return 0;
    const dir = getDirectionFactor();
    if (dir === 1) {
      const index = ((currentIndex % stepsCount) + stepsCount) % stepsCount;
      return scrollSteps[index];
    }
    const prevIndex = (currentIndex - 1 + stepsCount) % stepsCount;
    return -scrollSteps[prevIndex];
  };

  const runTransition = () => {
    if (!props.autoplay || !elementsCache.length) return;
    const distance = resolveStepDistance();
    stopAutoplayCycle?.();
    if (Math.abs(distance) < 0.5) {
      scheduleHold();
      return;
    }
    autoplayTween = gsap.to(elementsCache, {
      duration: props.autoplayTransitionDuration,
      ease: 'power3.inOut',
      x: `+=${distance}`,
      modifiers: wrapFunction ? { x: gsap.utils.unitize(wrapFunction) } : undefined,
      onUpdate: () => updateCardMetrics(elementsCache),
      onComplete: () => {
        autoplayTween = null;
        scheduleHold();
      }
    });
  };

  function scheduleHold() {
    if (!props.autoplay || !elementsCache.length) return;

    const isCenterPhase = !hasEdgeHold.value || nextHoldPhase === 'center';
    const rawDuration = isCenterPhase
      ? centerHoldDuration.value
      : edgeHoldDuration.value;
    const duration = Math.max(rawDuration ?? 0, 0);

    nextHoldPhase = hasEdgeHold.value && isCenterPhase ? 'edge' : 'center';

    if (duration <= 0) {
      runTransition();
      return;
    }

    autoplayDelay = gsap.delayedCall(duration, () => {
      autoplayDelay = null;
      runTransition();
    });
  }

  stopAutoplayCycle = () => {
    if (autoplayTween) {
      autoplayTween.kill();
      autoplayTween = null;
    }
    if (autoplayDelay) {
      autoplayDelay.kill();
      autoplayDelay = null;
    }
  };

  startAutoplayCycle = () => {
    if (!props.autoplay || !elementsCache.length) return;
    stopAutoplayCycle?.();
    updateCardMetrics(elementsCache);
    nextHoldPhase = 'center';
    scheduleHold();
  };

  if (props.autoplay) {
    startAutoplayCycle();

    if (props.pauseOnHover) {
      stopTicker = () => {
        stopAutoplayCycle?.();
      };
      startTicker = () => {
        startAutoplayCycle?.();
      };

      container.addEventListener('mouseenter', stopTicker);
      container.addEventListener('mouseleave', startTicker);
    }
  }

  resizeHandler = () => {
    if (typeof window !== 'undefined') {
      viewportWidth.value = window.innerWidth;
    }
    updateCardMetrics(elementsCache);
  };
  window.addEventListener('resize', resizeHandler);
};

onMounted(async () => {
  await nextTick();
  initializeScroll();
});

onUnmounted(() => {
  cleanup();
});

watch(
  () => ({
    items: props.items,
    autoplay: props.autoplay,
    autoplayHoldDuration: props.autoplayHoldDuration,
    autoplayCenterHoldDuration: props.autoplayCenterHoldDuration,
    autoplayEdgeHoldDuration: props.autoplayEdgeHoldDuration,
    autoplayTransitionDuration: props.autoplayTransitionDuration,
    autoplayDirection: props.autoplayDirection,
    pauseOnHover: props.pauseOnHover,
    itemMinHeight: props.itemMinHeight,
    itemMinWidth: props.itemMinWidth,
    itemMinWidthMobile: props.itemMinWidthMobile,
    itemGapMobile: props.itemGapMobile,
    itemGap: props.itemGap,
    isTilted: props.isTilted,
    tiltDirection: props.tiltDirection
  }),
  async () => {
    cleanup();
    await nextTick();
    initializeScroll();
  },
  { deep: true }
);
</script>

<style scoped>
.infinite-scroll-wrapper {
  position: relative;
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  pointer-events: auto;
}

.infinite-scroll-wrapper::before,
.infinite-scroll-wrapper::after {
  content: none;
}

.infinite-scroll-container {
  display: flex;
  flex-direction: row;
  align-items: center;
  padding: 0 clamp(1.5rem, 4vw, 3rem);
  box-sizing: border-box;
  backface-visibility: hidden;
  -webkit-backface-visibility: hidden;
  -moz-backface-visibility: hidden;
  -ms-backface-visibility: hidden;
  transform-style: preserve-3d;
}

.infinite-scroll-item {
  position: relative;
  flex: 0 0 auto;
  will-change: transform;
  backface-visibility: hidden;
  -webkit-backface-visibility: hidden;
  -moz-backface-visibility: hidden;
  -ms-backface-visibility: hidden;
  transform: translateZ(0);
}
</style>
