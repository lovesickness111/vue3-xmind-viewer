<template>
  <Teleport to="body">
    <Transition name="modal">
      <div v-if="visible" class="modal-overlay" @click.self="close">
        <div class="modal-content">
          <header class="modal-header">
            <h3>XMind Preview: {{ file?.name || 'Untitled' }}</h3>
            <div class="actions">
              <button
                v-if="viewerType === 'simple'"
                class="btn secondary"
                @click="downloadPNG"
                :disabled="!file"
              >⬇️ PNG</button>
              <button
                v-if="viewerType === 'simple'"
                class="btn secondary"
                @click="downloadXMind"
                :disabled="!file"
              >⬇️ XMind</button>
              <button
                v-if="viewerType === 'elixir'"
                class="btn secondary"
                @click="elixirExportSVG"
                :disabled="!file"
              >⬇️ SVG</button>
              <button
                v-if="viewerType === 'elixir'"
                class="btn secondary"
                @click="elixirExportXMind"
                :disabled="!file"
              >⬇️ XMind</button>
              <button class="close-btn" @click="close">&times;</button>
            </div>
          </header>
          <div class="modal-body">
            <XMindViewer v-if="file && viewerType === 'simple'" :file="file" ref="viewerRef" />
            <MindElixirViewer v-else-if="file && viewerType === 'elixir'" :file="file" ref="elixirRef" />
            <EmbedViewer v-else-if="file && viewerType === 'embed'" :file="file" />
            <div v-else class="empty-state">No file selected</div>
          </div>
        </div>
      </div>
    </Transition>
  </Teleport>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import XMindViewer from './XMindViewer.vue';
import MindElixirViewer from './MindElixirViewer.vue';
import EmbedViewer from './EmbedViewer.vue';

const viewerRef = ref<InstanceType<typeof XMindViewer> | null>(null);
const elixirRef = ref<InstanceType<typeof MindElixirViewer> | null>(null);

defineProps<{
  visible: boolean;
  file: File | null;
  viewerType: 'simple' | 'elixir' | 'embed';
}>();

const emit = defineEmits<{
  (e: 'update:visible', value: boolean): void;
}>();

const close = () => {
  emit('update:visible', false);
};

const downloadPNG = () => {
  viewerRef.value?.exportPNG();
};

const downloadXMind = () => {
  viewerRef.value?.exportXMind();
};

const elixirExportSVG = () => {
  elixirRef.value?.exportToPNG();
};

const elixirExportXMind = () => {
  elixirRef.value?.exportToXmind();
};
</script>

<style scoped>
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background-color: rgba(0, 0, 0, 0.6);
  backdrop-filter: blur(5px);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal-content {
  background: white;
  width: 95vw;
  height: 90vh;
  border-radius: 12px;
  display: flex;
  flex-direction: column;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.3);
  overflow: hidden;
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 16px 24px;
  border-bottom: 1px solid #eee;
  background-color: #fafafa;
  flex-shrink: 0;
}

.modal-header h3 {
  margin: 0;
  font-size: 1.25rem;
  color: #333;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.close-btn {
  background: none;
  border: none;
  font-size: 2rem;
  line-height: 1;
  cursor: pointer;
  color: #666;
  transition: color 0.2s;
  padding: 0;
}

.close-btn:hover {
  color: #000;
}

.actions {
  display: flex;
  align-items: center;
  gap: 12px;
}

.btn.secondary {
  background-color: #fff;
  border: 1px solid #d1d5db;
  color: #374151;
  padding: 8px 16px;
  font-size: 0.9rem;
  border-radius: 8px;
  cursor: pointer;
  font-weight: 500;
  transition: all 0.2s;
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.05);
  display: flex;
  align-items: center;
  gap: 6px;
}

.btn.secondary:hover:not(:disabled) {
  background-color: #f3f4f6;
  border-color: #9ca3af;
}

.btn.secondary:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.modal-body {
  flex: 1;
  overflow: hidden;
  background-color: #f0f2f5;
  display: flex;
  flex-direction: column;
  min-height: 0;
}

.empty-state {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100%;
  color: #888;
  font-size: 1.2rem;
}

/* Transitions */
.modal-enter-active,
.modal-leave-active {
  transition: opacity 0.3s ease;
}

.modal-enter-from,
.modal-leave-to {
  opacity: 0;
}

.modal-enter-active .modal-content,
.modal-leave-active .modal-content {
  transition: transform 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275);
}

.modal-enter-from .modal-content,
.modal-leave-to .modal-content {
  transform: scale(0.95) translateY(20px);
}
</style>
