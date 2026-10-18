<script setup lang="ts">
type Status = 'idle' | 'copied' | 'failed';

const status = ref<Status>('idle');
let resetTimer: ReturnType<typeof setTimeout> | undefined;

const label = computed(() => {
  if (status.value === 'copied') return 'Copied!';
  if (status.value === 'failed') return "Couldn't copy";
  return 'Copy link';
});

const icon = computed(() => {
  if (status.value === 'copied') return 'check';
  if (status.value === 'failed') return 'error';
  return 'link';
});

function legacyCopy(text: string): boolean {
  try {
    const el = document.createElement('textarea');
    el.value = text;
    el.setAttribute('readonly', '');
    el.style.position = 'fixed';
    el.style.top = '0';
    el.style.left = '0';
    el.style.opacity = '0';
    document.body.appendChild(el);
    el.select();
    el.setSelectionRange(0, text.length);
    const ok = document.execCommand('copy');
    document.body.removeChild(el);
    return ok;
  } catch {
    return false;
  }
}

async function copyLink() {
  const url = window.location.href;
  let ok = false;

  try {
    if (navigator.clipboard?.writeText) {
      await navigator.clipboard.writeText(url);
      ok = true;
    }
  } catch {
    ok = false;
  }

  if (!ok) ok = legacyCopy(url);

  status.value = ok ? 'copied' : 'failed';
  if (resetTimer) clearTimeout(resetTimer);
  resetTimer = setTimeout(() => {
    status.value = 'idle';
  }, 2000);
}

onBeforeUnmount(() => {
  if (resetTimer) clearTimeout(resetTimer);
});
</script>

<template>
  <button
    type="button"
    class="page-nav__button"
    :class="{ 'page-nav__button--success': status === 'copied' }"
    title="Copy a link to this page"
    aria-live="polite"
    @click="copyLink"
  >
    <i class="icon" aria-hidden="true">{{ icon }}</i>
    <span>{{ label }}</span>
  </button>
</template>
