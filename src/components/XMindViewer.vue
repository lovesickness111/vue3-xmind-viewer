<template>
  <div class="xmind-viewer-container" ref="containerRef"></div>
</template>

<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount, watch } from 'vue';
// @ts-ignore
import MindMap from 'simple-mind-map';
// @ts-ignore
import xmind from 'simple-mind-map/src/parse/xmind.js';
// @ts-ignore
import Export from 'simple-mind-map/src/plugins/Export.js';

MindMap.usePlugin(Export);

const props = defineProps<{
  file: File | null;
}>();

const containerRef = ref<HTMLElement | null>(null);
let mindMapInstance: MindMap | null = null;

const initMindMap = () => {
  if (!containerRef.value) return;
  mindMapInstance = new MindMap({
    el: containerRef.value,
    data: { data: { text: 'Loading...' }, children: [] },
  } as any);
  
  // Listen for resize events to refit
  window.addEventListener('resize', () => {
    // @ts-ignore
    mindMapInstance?.resize?.();
  });
};

const loadFile = async (file: File) => {
  if (!mindMapInstance) return;
  try {
    const data = await xmind.parseXmindFile(file);
    mindMapInstance.setData(data);
    // Wait for the modal transition to complete before fitting
    setTimeout(() => {
      // @ts-ignore
      mindMapInstance?.resize?.();
      // @ts-ignore
      mindMapInstance?.fit?.() || mindMapInstance?.view?.fit?.();
    }, 400);
  } catch (error) {
    console.error('Failed to parse XMind file:', error);
    alert('Failed to parse XMind file. Check console for details.');
  }
};

onMounted(() => {
  initMindMap();
  if (props.file) {
    loadFile(props.file);
  }
});

watch(() => props.file, (newFile, oldFile) => {
  // Only handle actual file changes after initial mount
  if (newFile && newFile !== oldFile && mindMapInstance) {
    loadFile(newFile);
  }
});

onBeforeUnmount(() => {
  if (mindMapInstance) {
    mindMapInstance.destroy();
  }
});

const exportPNG = async () => {
  if (!mindMapInstance) return;
  const name = props.file?.name?.replace('.xmind', '') || 'mindmap';
  try {
    // @ts-ignore
    await mindMapInstance.doExport.export('png', true, name);
  } catch (error) {
    console.error('Failed to export PNG:', error);
  }
};

const exportXMind = () => {
  if (!props.file) return;
  // Create a blob URL directly from the original file for a perfect copy
  const url = URL.createObjectURL(props.file);
  const a = document.createElement('a');
  a.href = url;
  a.download = props.file.name || 'mindmap.xmind';
  document.body.appendChild(a);
  a.click();
  document.body.removeChild(a);
  URL.revokeObjectURL(url);
};

defineExpose({
  exportPNG,
  exportXMind
});
</script>

<style scoped>
.xmind-viewer-container {
  width: 100%;
  height: 100%;
  position: relative;
  background-color: #ffffff;
  border-radius: inherit;
  overflow: hidden;
}
/* Ensure simple-mind-map container fills relative parent */
:deep(.smm-mind-map-container) {
  width: 100% !important;
  height: 100% !important;
  position: absolute !important;
  left: 0 !important;
  top: 0 !important;
}
</style>
