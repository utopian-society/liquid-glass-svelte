<script lang="ts">
  let {
    children,
    class: className = '',
    style: userStyle = '',
    roundness = 16,
    accent = '#bef264',
    contrast = 'light',
  } = $props();

  let rootEl = $state<HTMLElement>();
  let isHovering = $state(false);
  let filterId = $state('');

  $effect(() => {
    filterId = `lg-dist-${Math.random().toString(36).slice(2, 9)}`;
    if (rootEl) {
      rootEl.style.setProperty('--roundness', `${roundness}px`);
      rootEl.style.setProperty('--lg-hover-time', '400ms');
      rootEl.style.setProperty('--lg-hover-ease', 'cubic-bezier(0.25, 1, 0.5, 1)');
      rootEl.style.setProperty('--angle-1', '-75deg');
      rootEl.style.setProperty('--angle-2', '-45deg');
    }
  });

  $effect(() => {
    if (rootEl) {
      rootEl.style.setProperty('--roundness', `${roundness}px`);
    }
  });

  let isLight = $derived(contrast === 'light' || contrast === 'light-contrast');
  const shadowColor = $derived(isLight ? 'rgba(0,0,0,0.2)' : 'rgba(254,254,254,0.2)');
  const shadowColor2 = $derived(isLight ? 'rgba(0,0,0,0.1)' : 'rgba(254,254,254,0.1)');
</script>

<div
  bind:this={rootEl}
  class="liquid-glass-wrap {className}"
  style="{userStyle} --lg-filter: url(#{filterId});"
  onmouseenter={() => (isHovering = true)}
  onmouseleave={() => (isHovering = false)}
>
  {#if isHovering}
    <div class="lg-hoverstyle absolute inset-0 opacity-20" style="background: rgba(255, 255, 255, 0.7);">
      <div
        class="lg-rotating-gradient pointer-events-none absolute inset-0"
        style="border-radius: inherit; mix-blend-mode: lighten; opacity: 0.25; background: conic-gradient(from 0deg, #ffffff 0%, rgba(190, 242, 100, 0.3) 25%, #ffffff 50%, rgba(190, 242, 100, 0.3) 75%, #ffffff 100%); animation: rotate-gradient 4s ease-in-out infinite;"
      />
  </div>
  {/if}

  <div class="lg-tint absolute inset-0 opacity-30 z-5 pointer-events-none" style="background-color: rgba(255, 255, 255, 0.15);" />

  <div class="lg-content relative z-10">
    {@render children?.()}
  </div>

  <div
    class="lg-shadow absolute pointer-events-none"
    style="background: linear-gradient(180deg, {shadowColor}, {shadowColor2});"
  />
  <div
    class="lg-glass-filter"
    style="border-radius: {roundness}px; filter: url(#{filterId}) saturate(140%) brightness(1.0);"
  />

  <svg style="display: none; border-radius: {roundness}px">
    <filter id={filterId} x="0%" y="0%" width="100%" height="100%">
      <feTurbulence
        type="fractalNoise"
        baseFrequency="0.008 0.008"
        numOctaves="2"
        seed="92"
        result="noise"
      />
      <feGaussianBlur in="noise" stdDeviation="2" result="blurred" />
      <feDisplacementMap
        in="SourceGraphic"
        in2="blurred"
        scale="80"
        xChannelSelector="R"
        yChannelSelector="G"
      />
    </filter>
  </svg>
</div>

<style>
  @property --angle-1 {
    syntax: '<angle>';
    inherits: false;
    initial-value: -75deg;
  }
  @property --angle-2 {
    syntax: '<angle>';
    inherits: false;
    initial-value: -45deg;
  }

  @keyframes rotate-gradient {
    from { transform: rotate(0deg); }
    to { transform: rotate(360deg); }
  }

  :global(.lg-hover-time) {
    --lg-hover-time: 400ms;
  }
  :global(.lg-hover-ease) {
    --lg-hover-ease: cubic-bezier(0.25, 1, 0.5, 1);
  }

  .liquid-glass-wrap {
    position: relative;
    overflow: hidden;
    border-radius: var(--roundness, 16px);
    background: transparent;
    pointer-events: none;
    transition: all var(--lg-hover-time, 400ms) var(--lg-hover-ease, cubic-bezier(0.25, 1, 0.5, 1));
  }

  .lg-glass-filter {
    position: absolute;
    inset: 0;
    z-index: 0;
    -webkit-backdrop-filter: blur(var(--uto-glass-blur, 24px));
    backdrop-filter: blur(var(--uto-glass-blur, 24px));
    isolation: isolate;
    pointer-events: none;
  }

  .lg-shadow {
    --shadow-cuttoff-fix: 2em;
    position: absolute;
    width: calc(100% + var(--shadow-cuttoff-fix));
    height: calc(100% + var(--shadow-cuttoff-fix));
    top: calc(0% - var(--shadow-cuttoff-fix) / 2);
    left: calc(0% - var(--shadow-cuttoff-fix) / 2);
    filter: blur(clamp(2px, 0.125em, 12px));
    -webkit-filter: blur(clamp(2px, 0.125em, 12px));
    overflow: visible;
    pointer-events: none;
  }

  .lg-shadow::after {
    content: '';
    position: absolute;
    z-index: 0;
    inset: 0;
    border-radius: var(--roundness, 16px);
    width: calc(100% - var(--shadow-cuttoff-fix) - 0.25em);
    height: calc(100% - var(--shadow-cuttoff-fix) - 0.25em);
    top: calc(var(--shadow-cuttoff-fix) - 0.5em);
    left: calc(var(--shadow-cuttoff-fix) - 0.875em);
    padding: 0.125em;
    box-sizing: border-box;
    mask: linear-gradient(#000 0 0) content-box, linear-gradient(#000 0 0);
    mask-composite: exclude;
    -webkit-mask: linear-gradient(#000 0 0) content-box, linear-gradient(#000 0 0);
    -webkit-mask-composite: xor;
    transition: all var(--lg-hover-time, 400ms) var(--lg-hover-ease, cubic-bezier(0.25, 1, 0.5, 1));
    overflow: visible;
    opacity: 1;
  }

  .liquid-glass-wrap:has(.lg-content:hover) .lg-shadow {
    filter: blur(clamp(2px, 0.0625em, 6px));
    -webkit-filter: blur(clamp(2px, 0.0625em, 6px));
    transition: filter var(--lg-hover-time, 400ms) var(--lg-hover-ease, cubic-bezier(0.25, 1, 0.5, 1));
  }

  .liquid-glass-wrap:has(.lg-content:active) .lg-shadow {
    filter: blur(clamp(2px, 0.125em, 12px));
    -webkit-filter: blur(clamp(2px, 0.125em, 12px));
  }

  .liquid-glass-wrap:has(.lg-content:active) .lg-shadow::after {
    top: calc(var(--shadow-cuttoff-fix) - 0.5em);
    opacity: 0.75;
  }

  .lg-tint {
    z-index: 5;
    pointer-events: none;
  }

  .lg-content {
    pointer-events: auto;
    position: relative;
  }
</style>
