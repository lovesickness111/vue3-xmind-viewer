<template>
  <div class="embed-viewer-container">
    <div v-if="loading" class="viewer-overlay">
      <div class="spinner"></div>
      <p>Loading XMind viewer...</p>
    </div>
    <div v-if="error" class="viewer-overlay">
      <p class="err-msg">⚠️ {{ error }}</p>
      <p class="hint">This viewer requires an internet connection (renders from xmind.app)</p>
    </div>
    <iframe ref="iframeRef" class="xmind-iframe" :style="{ visibility: loading || error ? 'hidden' : 'visible' }"></iframe>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount, watch } from 'vue';
// @ts-ignore
import { XMindEmbedViewer } from 'xmind-embed-viewer';

const props = defineProps<{
  file: File | null;
}>();

const iframeRef = ref<HTMLIFrameElement | null>(null);
const loading = ref(false);
const error = ref<string | null>(null);
let viewerInstance: any = null;

const loadFile = async (file: File) => {
  if (!iframeRef.value) return;
  loading.value = true;
  error.value = null;

  try {
    viewerInstance = new XMindEmbedViewer({
      el: iframeRef.value,
    });

    // Keep the iframe at full container size
    viewerInstance.setStyles({
      'width': '100%',
      'height': '100%',
      'border': 'none',
    });

    viewerInstance.addEventListener('map-ready', () => {
      loading.value = false;
      // Fit the map to the available space after ready
      try { viewerInstance.setFitMap(); } catch (_) {}
    });

    const arrayBuffer = await file.arrayBuffer();
    viewerInstance.load(arrayBuffer);

    // Fallback: hide spinner after 10s regardless
    setTimeout(() => { loading.value = false; }, 10000);

  } catch (e: any) {
    console.error('xmind-embed-viewer error:', e);
    error.value = `Failed to load: ${e?.message || String(e)}`;
    loading.value = false;
  }
};

onMounted(() => {
  if (props.file) loadFile(props.file);
});

onBeforeUnmount(() => {
  viewerInstance = null;
});

watch(() => props.file, (newFile) => {
  if (newFile) loadFile(newFile);
});
</script>

<style scoped>
.embed-viewer-container {
  /* Fill the flexbox parent (modal-body) completely */
  flex: 1;
  min-height: 0;
  width: 100%;
  height: 100%;
  position: relative;
  background: #fff;
  display: flex;
  flex-direction: column;
}

.xmind-iframe {
  flex: 1;
  min-height: 0;
  width: 100%;
  border: none;
  display: block;
}

.viewer-overlay {
  position: absolute;
  inset: 0;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 12px;
  background: #fff;
  z-index: 1;
}

.err-msg {
  color: #dc2626;
  font-size: 1rem;
}

.hint {
  color: #64748b;
  font-size: 0.85rem;
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
