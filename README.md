# Automatica Home

A single-page marketing experience for Automatica, built with Vue 3 and Vite. The page showcases AI-driven control systems with a kinetic carousel, animated background beams, and responsive layouts that adapt from widescreen desktops down to stacked mobile cards.

## Features

- **ScrollStack carousel** powered by GSAP for smooth, looping card transitions with keyboard and pointer support.
- **Adaptive layout**: cards resize, overlap, and remain centered on every viewport, avoiding clipping on small screens.
- **Beams background**: animated Three.js beams create a premium, motion-rich hero backdrop.
- **Accessible interactions**: link-like cards support keyboard activation and focus outlines.
- **Easy content editing**: card data lives in `src/components/HelloWorld.vue`, ready to update without touching animation logic.

## Tech Stack

- [Vue 3](https://vuejs.org/) with `<script setup>` single-file components
- [Vite](https://vitejs.dev/) for development server and builds
- [GSAP](https://greensock.com/gsap/) + `Observer` plugin for scroll/drag/autoplay animations
- [Three.js](https://threejs.org/) for background beam rendering

## Getting Started

```bash
# Install dependencies
npm install

# Start the local dev server
npm run dev

# Build for production
npm run build

# Preview the production build locally
npm run preview
```

Vite outputs the compiled site to `dist/`.

## Project Structure

```
├── public/                # Static assets copied as-is
├── src/
│   ├── components/
│   │   ├── HelloWorld.vue # Main experience section and card definitions
│   │   ├── ScrollStack.vue# Infinite carousel component with GSAP logic
│   │   └── BeamsBackground.vue # Animated background
│   └── App.vue            # Entrypoint wiring components together
├── index.html             # Vite HTML entry
├── vite.config.js         # Build configuration
└── package.json
```

## Customization Tips

- Update `cards` inside `HelloWorld.vue` to change copy, links, or add new experiences.
- Tweak autoplay timings (`autoplayCenterHoldDuration`, `autoplayTransitionDuration`) on the `ScrollStack` instance to adjust rhythm.
- Modify mobile spacing (`itemMinWidthMobile`, `itemGapMobile`) to refine the stacked effect on phones.
- Background beam parameters live on the `BeamsBackground` component props for quick experimentation.

## Deployment

Run `npm run build` and serve the `dist/` folder via any static host (Vercel, Netlify, GitHub Pages, etc.).

## License

This project inherits the licensing of your repository. Update this section if you plan to open-source the code with a specific license.
