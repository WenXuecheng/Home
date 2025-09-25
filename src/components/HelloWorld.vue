<script setup lang="ts">
import BeamsBackground from './BeamsBackground.vue';
import ScrollStack from './ScrollStack.vue';

type CardLinkTarget = '_self' | '_blank';

interface CardContent {
  id: number;
  eyebrow: string;
  title: string;
  description: string;
  cta: string;
  href: string;
  target?: CardLinkTarget;
}

const cards: CardContent[] = [
  {
    id: 1,
    eyebrow: 'Featured',
    title: 'AI 智能制造解决方案',
    description: '利用多模态模型实时监控生产效率，构建智能预警系统。',
    cta: '了解详情',
    href: 'https://cv.wen-xc.site',
    target: '_blank'
  },
  {
    id: 2,
    eyebrow: 'Case Study',
    title: '自动化营销增长 300%',
    description: '通过行为画像与自动化工作流实现精准触达，助力客户获取。',
    cta: '查看案例',
    href: 'https://eat.wen-xc.site',
    target: '_blank'
  },
  {
    id: 3,
    eyebrow: 'Insight',
    title: '下一代人机协同趋势',
    description: '解析黑金属视觉背后的技术逻辑与交互未来。',
    cta: '阅读洞察',
    href: 'https://order.wen-xc.site',
    target: '_blank'
  },
  {
    id: 4,
    eyebrow: 'Insight',
    title: '现代智能微电网',
    description: '探索智能电网中的核心技术与未来应用场景。',
    cta: '了解更多',
    href: 'https://order.wen-xc.site',
    target: '_blank'
  }
];

const cardClassName = 'experience-card';

const handleCardClick = (card: CardContent) => {
  if (!card.href) return;
  const target = card.target ?? '_self';
  window.open(card.href, target);
};

const handleKeyPress = (card: CardContent, event: KeyboardEvent) => {
  if (event.key === 'Enter' || event.key === ' ') {
    event.preventDefault();
    handleCardClick(card);
  }
};
</script>

<template>
  <section class="experience-wrapper">
    <BeamsBackground
      class="experience-background"
      :beam-number="28"
      :beam-width="0.95"
      :beam-height="24"
      :beam-spacing="0.55"
      :scale="0.14"
      :rotation="32"
      :tilt="-16"
      :speed="1.6"
    />

    <div class="experience-overlay">
      <header class="experience-header">
        <p class="experience-tag">NEXT-GEN CONTROL SYSTEMS</p>
        <h1 class="experience-title">智能控制与多智能体未来</h1>
        <p class="experience-subtitle">
          研究与工程实践中，探索系统分析、AI方法与协同控制的边界。
        </p>
      </header>

      <ScrollStack
        class-name="experience-scroll"
        :items="cards"
        :item-gap="0"
        :item-min-height="320"
        :item-min-width="360"
        :item-min-width-mobile="280"
        :item-gap-mobile="-140"
        :autoplay-center-hold-duration="3"
        :autoplay-edge-hold-duration="0"
        :autoplay-transition-duration="1"
        autoplay-direction="up"
        pause-on-hover
      >
        <template #default="{ item }">
          <article
            :class="cardClassName"
            role="link"
            tabindex="0"
            @click="() => handleCardClick(item)"
            @keydown="handleKeyPress(item, $event)"
          >
            <div class="experience-card__content">
              <div class="card-eyebrow">{{ item.eyebrow }}</div>
              <h2 class="card-title">{{ item.title }}</h2>
              <p class="card-description">{{ item.description }}</p>
              <span class="card-cta">{{ item.cta }}</span>
            </div>
          </article>
        </template>
      </ScrollStack>
    </div>
  </section>
</template>

<style scoped>
.experience-wrapper {
  position: relative;
  width: 100vw;
  height: 100vh;
  overflow: hidden;
  color: #fff;
  background-color: #050505;
}

.experience-background {
  pointer-events: none;
}

.experience-overlay {
  position: relative;
  z-index: 1;
  display: flex;
  flex-direction: column;
  justify-content: center;
  gap: 3vh;
  height: 100%;
  max-width: 1280px;
  margin: 0 auto;
  padding: 4vh 0 1vh;
}

.experience-header {
  padding: 0 4rem 2.4rem 4rem;
  max-width: 720px;
}

.experience-tag {
  display: inline-block;
  padding: 0.35rem 0.75rem;
  margin-bottom: 0.75rem;
  border: 1px solid rgba(255, 255, 255, 0.12);
  border-radius: 999px;
  font-size: 0.75rem;
  letter-spacing: 0.12em;
  text-transform: uppercase;
  background: linear-gradient(120deg, rgba(255, 255, 255, 0.08), rgba(255, 255, 255, 0));
}

.experience-title {
  margin: 0;
  font-size: clamp(2.5rem, 5vw, 3.75rem);
  line-height: 1.1;
  font-weight: 600;
}

.experience-subtitle {
  margin: 1rem 0 0;
  font-size: 1rem;
  color: rgba(255, 255, 255, 0.72);
  line-height: 1.7;
}

.experience-scroll {
  flex: 1 1 auto;
  min-height: 0;
  height: 100%;
  margin-top: -1vh;
  width: 100vw;
  margin-left: calc(50% - 50vw);
  margin-right: calc(50% - 50vw);
}


.experience-scroll :deep(.infinite-scroll-container) {
  width: 100%;
  align-items: center;
  gap: 0;
  padding: 0;
}

.experience-scroll :deep(.infinite-scroll-item) {
  pointer-events: none;
}

.experience-scroll :deep(.infinite-scroll-item > *) {
  pointer-events: auto;
}

.experience-card {
  position: relative;
  display: flex;
  flex-direction: column;
  justify-content: center;
  gap: 1.2rem;
  width: 100%;
  min-height: clamp(18rem, 46vh, 28rem);
  padding: clamp(2rem, 3.8vw, 3.1rem);
  max-width: clamp(18rem, 82vw, 30rem);
  margin: 0 auto;
  --scroll-visibility: 0;
  --focus-progress: 0;
  --center-distance: 0;
  border-radius: 40px;
  background:
    linear-gradient(150deg, rgba(20, 20, 24, 0.82), rgba(10, 10, 14, 0.5)),
    radial-gradient(120% 140% at calc(50% + var(--center-distance, 0) * 18%) 0%, rgba(120, 168, 255, 0.18), rgba(120, 168, 255, 0));
  background-position:
    calc(50% + var(--center-distance, 0) * 10%) 50%,
    50% 0%;
  border: 1px solid rgba(132, 150, 255, 0.12);
  box-shadow:
    0 25px 60px rgba(0, 0, 0, 0.45),
    inset 0 0 28px rgba(152, 200, 255, 0.05);
  backdrop-filter: blur(22px) saturate(140%);
  overflow: hidden;
  transition:
    opacity 0.45s ease,
    transform 0.65s cubic-bezier(0.22, 1, 0.36, 1),
    filter 0.9s ease,
    box-shadow 0.7s ease;
  color: #ffffff;
  opacity: calc(0.35 * var(--scroll-visibility, 0) + 0.65 * var(--focus-progress, 0));
  transform:
    translateY(calc((1 - var(--focus-progress, 0)) * 32px))
    scale(calc(0.6 + var(--focus-progress, 0) * 0.5));
  filter: saturate(calc(0.6 + var(--focus-progress, 0) * 0.4));
  pointer-events: none;
  cursor: pointer;
}

.experience-card:hover {
  box-shadow: 0 28px 70px rgba(0, 0, 0, 0.7);
}

.experience-card.is-visible:hover {
  transform:
    translateY(calc((0.92 - var(--focus-progress, 0)) * 12px))
    scale(calc(0.63 + var(--focus-progress, 0) * 0.52));
}

.experience-card:focus-visible {
  outline: 2px solid rgba(123, 97, 255, 0.85);
  outline-offset: 6px;
}

.experience-card.is-visible {
  transform:
    translateY(calc((0.85 - var(--focus-progress, 0)) * 16px))
    scale(calc(0.68 + var(--focus-progress, 0) * 0.45));
  filter: saturate(calc(0.75 + var(--focus-progress, 0) * 0.35));
  pointer-events: auto;
}

.experience-card__content {
  position: relative;
  z-index: 1;
}


.card-eyebrow {
  margin-bottom: 1rem;
  font-size: 0.85rem;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  color: rgba(255, 255, 255, 0.65);
}

.card-title {
  margin: 0 0 1rem;
  font-size: clamp(1.8rem, 3.2vw, 2.4rem);
  line-height: 1.2;
}

.card-description {
  margin: 0 0 2.5rem;
  color: rgba(255, 255, 255, 0.78);
  font-size: 1.02rem;
  line-height: 1.6;
}

.card-cta {
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.5rem 1.2rem;
  border-radius: 999px;
  background: linear-gradient(135deg, rgba(123, 97, 255, 0.76), rgba(0, 168, 255, 0.72));
  font-weight: 600;
  letter-spacing: 0.03em;
  transition: background 0.35s ease, box-shadow 0.35s ease;
}

.card-cta:hover {
  box-shadow: 0 0 26px rgba(71, 137, 255, 0.45);
  background: linear-gradient(135deg, rgba(123, 97, 255, 0.92), rgba(0, 168, 255, 0.9));
}

@media (max-width: 768px) {
  .experience-overlay {
    padding: 1vh 0;
  }

  .experience-header {
    padding: 0 1.25rem 1.5rem;
  }

  .experience-scroll {
    height: 100%;
  }

  .experience-card {
    min-height: clamp(16rem, 64vh, 24rem);
    padding: clamp(1.6rem, 7vw, 2.2rem);
  }

  .card-title {
    font-size: clamp(1.6rem, 8vw, 2.2rem);
  }

  .card-description {
    margin-bottom: 1.8rem;
    font-size: 0.98rem;
  }
}

 
</style>
