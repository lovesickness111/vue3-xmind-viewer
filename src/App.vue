<template>
  <div class="app-container">
    <header class="hero">
      <h1>XMind Web Preview</h1>
      <p>View your mind maps instantly in the browser securely and beautifully.</p>
    </header>

    <main class="content">
      <!-- Viewer Selector -->
      <div class="viewer-selector">
        <p class="selector-label">Choose a viewer to compare:</p>
        <div class="tabs">
          <button
            class="tab"
            :class="{ active: selectedViewer === 'elixir' }"
            @click="selectedViewer = 'elixir'"
          >
            <span class="tab-num">①</span>
            <span class="tab-info">
              <strong>mind-elixir</strong>
              <small>Modern editor · Edit · Direction-aware import/export</small>
            </span>
          </button>
          <button
            class="tab"
            :class="{ active: selectedViewer === 'simple' }"
            @click="selectedViewer = 'simple'"
          >
            <span class="tab-num">②</span>
            <span class="tab-info">
              <strong>simple-mind-map</strong>
              <small>Canvas · Edit · Export PNG</small>
            </span>
          </button>
          <button
            class="tab"
            :class="{ active: selectedViewer === 'embed' }"
            @click="selectedViewer = 'embed'"
          >
            <span class="tab-num">③</span>
            <span class="tab-info">
              <strong>xmind-embed-viewer</strong>
              <small>Official by XMind Ltd · Requires internet</small>
            </span>
          </button>
        </div>
      </div>

      <div
        class="upload-box"
        @dragover.prevent="isDragging = true"
        @dragleave.prevent="isDragging = false"
        @drop.prevent="handleDrop"
        :class="{ 'is-dragging': isDragging }"
        @click="triggerFileInput"
      >
        <div class="icon">📁</div>
        <h3>Drop your .xmind file here</h3>
        <p>or click to browse</p>
        <input type="file" ref="fileInput" accept=".xmind" @change="handleFileSelect" hidden />
      </div>

      <div class="sample-section">
        <p>Don't have a file?</p>
        <button class="btn primary" @click="loadSample">Preview Sample Map</button>
      </div>
    </main>

    <XMindModal
      :visible="isModalVisible"
      :file="selectedFile"
      :viewerType="selectedViewer"
      @update:visible="isModalVisible = $event"
    />
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import XMindModal from './components/XMindModal.vue';

const isDragging = ref(false);
const isModalVisible = ref(false);
const selectedFile = ref<File | null>(null);
const fileInput = ref<HTMLInputElement | null>(null);
const selectedViewer = ref<'simple' | 'elixir' | 'embed'>('elixir');

const triggerFileInput = () => {
  fileInput.value?.click();
};

const openPreview = (file: File) => {
  if (!file.name.endsWith('.xmind')) {
    alert('Please select a valid .xmind file');
    return;
  }
  selectedFile.value = file;
  isModalVisible.value = true;
};

const handleFileSelect = (event: Event) => {
  const target = event.target as HTMLInputElement;
  if (target.files && target.files.length > 0) {
    const file = target.files.item(0);
    if (file) openPreview(file);
    target.value = '';
  }
};

const handleDrop = (event: DragEvent) => {
  isDragging.value = false;
  if (event.dataTransfer?.files && event.dataTransfer.files.length > 0) {
    const file = event.dataTransfer.files.item(0);
    if (file) openPreview(file);
  }
};

const loadSample = async () => {
  try {
    const response = await fetch('/co_cau_to_chuc.xmind');
    const blob = await response.blob();
    const file = new File([blob], 'co_cau_to_chuc.xmind', { type: 'application/xmind' });
    openPreview(file);
  } catch (error) {
    console.error('Failed to load sample:', error);
    alert('Failed to load sample XMind file. Make sure it exists in the public directory.');
  }
};
</script>

<style scoped>
.app-container {
  max-width: 860px;
  margin: 0 auto;
  padding: 60px 20px;
  text-align: center;
}

.hero {
  margin-bottom: 48px;
}

.hero h1 {
  font-size: 3rem;
  color: #1e293b;
  margin-bottom: 16px;
  font-weight: 800;
  letter-spacing: -0.025em;
}

.hero p {
  font-size: 1.2rem;
  color: #64748b;
}

/* Viewer Selector */
.viewer-selector {
  margin-bottom: 32px;
}

.selector-label {
  font-size: 0.95rem;
  color: #64748b;
  font-weight: 500;
  margin-bottom: 12px;
}

.tabs {
  display: flex;
  gap: 12px;
  justify-content: center;
  flex-wrap: wrap;
}

.tab {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 14px 20px;
  border: 2px solid #e2e8f0;
  border-radius: 12px;
  background: #fff;
  cursor: pointer;
  transition: all 0.2s;
  text-align: left;
  min-width: 220px;
  box-shadow: 0 1px 3px rgba(0,0,0,0.05);
}

.tab:hover {
  border-color: #93c5fd;
  background: #eff6ff;
}

.tab.active {
  border-color: #3b82f6;
  background: #eff6ff;
  box-shadow: 0 0 0 3px rgba(59,130,246,0.15);
}

.tab-num {
  font-size: 1.5rem;
  line-height: 1;
}

.tab-info {
  display: flex;
  flex-direction: column;
  gap: 3px;
}

.tab-info strong {
  font-size: 0.9rem;
  color: #1e293b;
}

.tab-info small {
  font-size: 0.78rem;
  color: #64748b;
}

/* Upload Box */
.upload-box {
  border: 3px dashed #cbd5e1;
  border-radius: 20px;
  padding: 60px 20px;
  background-color: #f8fafc;
  cursor: pointer;
  transition: all 0.3s ease;
  margin-bottom: 40px;
}

.upload-box:hover, .upload-box.is-dragging {
  border-color: #3b82f6;
  background-color: #eff6ff;
  transform: translateY(-2px);
  box-shadow: 0 10px 20px rgba(59, 130, 246, 0.1);
}

.upload-box .icon {
  font-size: 4rem;
  margin-bottom: 20px;
  display: block;
}

.upload-box h3 {
  font-size: 1.75rem;
  color: #334155;
  margin-bottom: 10px;
}

.upload-box p {
  color: #64748b;
  font-size: 1.1rem;
}

.sample-section {
  padding: 30px;
  background: #ffffff;
  border-radius: 16px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.05);
  display: inline-block;
}

.sample-section p {
  margin-bottom: 16px;
  color: #475569;
  font-weight: 500;
}

.btn.primary {
  background-color: #3b82f6;
  color: white;
  border: none;
  padding: 14px 28px;
  font-size: 1.1rem;
  border-radius: 10px;
  cursor: pointer;
  font-weight: 600;
  transition: all 0.2s;
  box-shadow: 0 4px 6px rgba(59, 130, 246, 0.3);
}

.btn.primary:hover {
  background-color: #2563eb;
  transform: translateY(-1px);
  box-shadow: 0 6px 12px rgba(59, 130, 246, 0.4);
}
</style>
