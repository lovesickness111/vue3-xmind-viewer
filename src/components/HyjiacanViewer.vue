<template>
  <div class="hyjiacan-viewer-container" ref="containerRef">
    <div v-if="loading" class="viewer-overlay">
      <div class="spinner"></div>
      <p>Loading mind map...</p>
    </div>
    <div v-if="error" class="viewer-overlay">
      <p class="err-msg">⚠️ {{ error }}</p>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount, watch } from 'vue';
// Static import to avoid Vite dynamic module resolution issues
// @ts-ignore
import xmindViewerLib from '@hyjiacan/xmind-viewer/dist/xmind-viewer.umd.js';

const props = defineProps<{
  file: File | null;
}>();

const containerRef = ref<HTMLElement | null>(null);
const loading = ref(false);
const error = ref<string | null>(null);

const loadFile = async (file: File) => {
  if (!containerRef.value) return;
  loading.value = true;
  error.value = null;

  try {
    const { viewer } = xmindViewerLib;
    if (!viewer || typeof viewer.render !== 'function') {
      throw new Error('Cannot find viewer.render() in @hyjiacan/xmind-viewer');
    }

    const el = containerRef.value;
    // Destroy previous G6 graph instance if any
    if ((el as any).__graph) {
      (el as any).__graph.destroy?.();
      (el as any).__graph = null;
    }
    el.innerHTML = '';

    const arrayBuffer = await file.arrayBuffer();
    await viewer.render(el, arrayBuffer);
  } catch (e: any) {
    console.error('@hyjiacan/xmind-viewer error:', e);
    error.value = e?.message || String(e);
  } finally {
    loading.value = false;
  }
};

onMounted(() => {
  if (props.file) loadFile(props.file);
});

onBeforeUnmount(() => {
  if (containerRef.value) {
    const el = containerRef.value as any;
    el.__graph?.destroy?.();
    el.innerHTML = '';
  }
});

watch(() => props.file, (newFile) => {
  if (newFile) loadFile(newFile);
});
</script>

<style scoped>
.hyjiacan-viewer-container {
  width: 100%;
  height: 100%;
  position: relative;
  background: #fff;
  overflow: hidden;
}

.viewer-overlay {
  position: absolute;
  inset: 0;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 12px;
  background: rgba(255, 255, 255, 0.9);
  z-index: 2;
}

.err-msg {
  color: #dc2626;
  font-size: 1rem;
  padding: 0 24px;
  text-align: center;
}

.spinner {
  width: 40px;
  height: 40px;
  border: 4px solid #e2e8f0;
  border-top-color: #3b82f6;
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}
</style>
