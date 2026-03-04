<template>
  <div class="mind-elixir-wrapper" ref="containerRef">
    <div v-if="loading" class="viewer-overlay">
      <div class="spinner"></div>
      <p>Parsing XMind file...</p>
    </div>
    <div v-if="error" class="viewer-overlay">
      <p class="err-msg">⚠️ {{ error }}</p>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount, watch } from 'vue';
import MindElixir from 'mind-elixir';
import 'mind-elixir/style.css';
// @ts-ignore – no type declarations available
import exportXmind from '@mind-elixir/export-xmind';
// @ts-ignore
import xmindParse from 'simple-mind-map/src/parse/xmind.js';
// @ts-ignore
import JSZip from 'jszip';

const { parseXmindFile } = xmindParse;

const props = defineProps<{
  file: File | null;
}>();

const containerRef = ref<HTMLElement | null>(null);
const loading = ref(false);
const error = ref<string | null>(null);
let mindInstance: InstanceType<typeof MindElixir> | null = null;

// ─── Direction mappings ───────────────────────────────────────────────────────

/** XMind structureClass → Mind Elixir direction */
const XMIND_TO_ME_DIRECTION: Record<string, number> = {
  'org.xmind.ui.logic.right': MindElixir.RIGHT,
  'org.xmind.ui.logic.left':  MindElixir.LEFT,
  'org.xmind.ui.map.unbalanced': MindElixir.SIDE,
  'org.xmind.ui.map.clockwise': MindElixir.SIDE,
};

/** Mind Elixir direction → XMind structureClass */
const ME_TO_XMIND_STRUCTURE: Record<number, string> = {
  [MindElixir.RIGHT]: 'org.xmind.ui.logic.right',
  [MindElixir.LEFT]:  'org.xmind.ui.logic.left',
  [MindElixir.SIDE]:  'org.xmind.ui.map.unbalanced',
};

/** Mind Elixir direction → XMind extensions.mainTopic */
const ME_TO_XMIND_MAIN_TOPIC: Record<number, string> = {
  [MindElixir.RIGHT]: 'org.xmind.ui.logic.right',
  [MindElixir.LEFT]:  'org.xmind.ui.logic.left',
  [MindElixir.SIDE]:  'org.xmind.ui.map.unbalanced',
};

// ─── Detect direction from raw XMind content.json ────────────────────────────
async function detectDirectionFromXmind(file: File): Promise<number> {
  try {
    const zip = await JSZip.loadAsync(file);
    const jsonFile = zip.files['content.json'];
    if (!jsonFile) return MindElixir.SIDE;

    const json = JSON.parse(await jsonFile.async('string'));
    const sheet = json?.[0];

    // rootTopic.structureClass is the authoritative source for overall layout.
    // extensions[].content.mainTopic is a branch-style hint, NOT the map direction —
    // an "unbalanced" map can have mainTopic="logic.left" in extensions but that
    // doesn't mean the map is left-only. Only trust rootTopic.structureClass.
    const structureClass: string = sheet?.rootTopic?.structureClass ?? '';
    console.log('[MindElixir] rootTopic.structureClass:', structureClass || '(none)');

    return XMIND_TO_ME_DIRECTION[structureClass] ?? MindElixir.SIDE;
  } catch {
    return MindElixir.SIDE;
  }
}

// ─── Convert SMM node tree { data: { text }, children: [] } → Mind Elixir ──
function smmNodeToME(smmNode: any): any {
  const text = smmNode?.data?.text || smmNode?.text || '';
  return {
    id: (smmNode?.data?.uid || crypto.randomUUID()).toString(),
    topic: text,
    expanded: true,
    children: (smmNode?.children || []).map(smmNodeToME),
  };
}

// ─── Main loader ─────────────────────────────────────────────────────────────
const loadFile = async (file: File) => {
  if (!containerRef.value) return;
  loading.value = true;
  error.value = null;

  try {
    // 1. Detect direction from raw XMind content before SMM transforms it
    const direction = await detectDirectionFromXmind(file);

    // 2. Parse .xmind → root node directly: { data: { text }, children: [] }
    const smmData = await parseXmindFile(file);
    if (!smmData || (!smmData.data && !smmData.text)) {
      throw new Error('Empty or unsupported XMind file structure');
    }

    // 3. Convert to Mind Elixir format
    const meRootNode = smmNodeToME(smmData);
    const data = { nodeData: meRootNode, linkData: {} };

    // 4. Destroy previous instance if any
    if (mindInstance) {
      mindInstance.destroy?.();
      mindInstance = null;
    }

    // 5. Initialise Mind Elixir with the detected direction
    mindInstance = new MindElixir({
      el: containerRef.value,
      direction: direction as 0 | 1 | 2,
      draggable: true,
      editable: true,
      contextMenu: true,
      toolBar: true,
      keypress: true,
      locale: 'en',
    });

    mindInstance.install(exportXmind);
    mindInstance.init(data);

    setTimeout(() => { mindInstance?.toCenter(); }, 200);

  } catch (e: any) {
    console.error('MindElixirViewer error:', e);
    error.value = e?.message || String(e);
  } finally {
    loading.value = false;
  }
};

// ─── Patch exported XMind blob with current direction ────────────────────────
async function patchXmindDirection(blob: Blob, direction: number): Promise<Blob> {
  const structureClass = ME_TO_XMIND_STRUCTURE[direction] ?? 'org.xmind.ui.logic.right';
  const mainTopic      = ME_TO_XMIND_MAIN_TOPIC[direction]  ?? 'org.xmind.ui.logic.right';

  const zip = await JSZip.loadAsync(blob);
  const contentFile = zip.files['content.json'];
  if (!contentFile) return blob;

  const content = JSON.parse(await contentFile.async('string'));

  // Patch rootTopic.structureClass
  if (content[0]?.rootTopic) {
    content[0].rootTopic.structureClass = structureClass;
  }

  // Patch extensions.mainTopic (used by XMind to set the diagram layout style)
  const extensions: any[] = content[0]?.extensions ?? [];
  const skeletonExt = extensions.find(
    (e: any) => e.provider === 'org.xmind.ui.skeleton.structure.style'
  );
  if (skeletonExt?.content) {
    skeletonExt.content.mainTopic = mainTopic;
  }

  zip.file('content.json', JSON.stringify(content));
  return zip.generateAsync({ type: 'blob' });
}

// ─── Expose export ────────────────────────────────────────────────────────────
const exportToXmind = async () => {
  if (!mindInstance) return;
  try {
    // @ts-ignore
    const blob: Blob = await mindInstance.exportXmind();
    const direction: number = (mindInstance as any).direction ?? MindElixir.RIGHT;
    const patchedBlob = await patchXmindDirection(blob, direction);

    const rootTopic: string = (mindInstance as any).nodeData?.topic ?? 'mind-elixir';
    const url = URL.createObjectURL(patchedBlob);
    const a = document.createElement('a');
    a.href = url;
    a.download = `${rootTopic}.xmind`;
    a.click();
    URL.revokeObjectURL(url);
  } catch (e: any) {
    alert('Export XMind failed: ' + e?.message);
  }
};

const exportToSvg = async () => {
  if (!mindInstance) return;
  try {
    // @ts-ignore – exportSvg() returns a Blob with <?xml...> header included
    const blob: Blob = mindInstance.exportSvg();
    const rootTopic: string = (mindInstance as any).nodeData?.topic ?? 'mind-elixir';
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = `${rootTopic}.svg`;
    a.click();
    URL.revokeObjectURL(url);
  } catch (e: any) {
    alert('Export SVG failed: ' + e?.message);
  }
};

defineExpose({ exportToXmind, exportToPNG: exportToSvg });

// ─── Lifecycle ────────────────────────────────────────────────────────────────
onMounted(() => {
  if (props.file) loadFile(props.file);
});

onBeforeUnmount(() => {
  mindInstance?.destroy?.();
  mindInstance = null;
});

watch(() => props.file, (newFile) => {
  if (newFile) loadFile(newFile);
});
</script>

<style scoped>
.mind-elixir-wrapper {
  width: 100%;
  height: 100%;
  position: relative;
  background: #f8fafc;
}

.viewer-overlay {
  position: absolute;
  inset: 0;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 12px;
  background: rgba(248, 250, 252, 0.92);
  z-index: 10;
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
