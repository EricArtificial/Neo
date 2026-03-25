<template>
  <div>
    <div class="background-glow"></div>

    <header class="top-nav">
      <div class="brand">
        <div class="brand-dot"></div>
        <span>NeoVision AI</span>
      </div>
      <nav>
        <a
          v-for="item in navItems"
          :key="item"
          href="#"
          :class="{ active: item === activeNav }"
          @click.prevent="activeNav = item"
        >
          {{ t(`nav.${item}`) }}
        </a>
      </nav>
      <button class="nav-btn" @click="toggleLang">{{ locale === 'EN' ? '中' : 'EN' }}</button>
    </header>

    <main class="container">
      <p v-if="apiError" class="api-error">{{ apiError }}</p>

      <template v-if="activeNav === 'Dashboard'">
        <h1>Dashboard</h1>

        <section class="card workflow-panel">
          <div class="panel-header">
            <h2>{{ t('dashboard.workflow') }}</h2>
            <span class="live-pill">{{ t('dashboard.status') }}</span>
          </div>

          <div class="workflow-bar">
            <template v-for="(step, index) in workflowSteps" :key="step.name">
              <article class="workflow-step" :class="step.state">
                <div class="step-title">
                  <span class="step-index">{{ index + 1 }}</span>
                  <h3>{{ step.name }}</h3>
                </div>

                <div class="step-stats">
                  <div>
                    <p>{{ t('dashboard.uploaded') }}</p>
                    <strong>{{ formatNumber(step.uploaded) }}</strong>
                  </div>
                  <div>
                    <p>{{ t('dashboard.tampered') }}</p>
                    <strong>{{ formatNumber(step.tampered) }}</strong>
                  </div>
                  <div>
                    <p>{{ t('dashboard.avgConfidence') }}</p>
                    <strong>{{ step.confidence }}%</strong>
                  </div>
                </div>
              </article>
              <div class="workflow-line" v-if="index < workflowSteps.length - 1"></div>
            </template>
          </div>
        </section>

        <section class="feature-grid">
          <article class="card feature-card">
            <div class="card-head">
              <h3>{{ t('dashboard.liveFeed') }}</h3>
              <span class="tag">{{ t('dashboard.streaming') }}</span>
            </div>
            <p>{{ t('dashboard.feedDesc') }}</p>
            <ul class="feed-list">
              <li v-for="item in recentUploads" :key="item.id">
                <div>
                  <strong>{{ item.id }}</strong>
                  <span>{{ item.device }} · {{ item.time }} · hash {{ item.hashValue }}</span>
                </div>
                <div class="case-actions">
                  <em>{{ item.status }}</em>
                  <button class="ghost-btn mini" @click="selectAnalysisById(item.id)">View</button>
                </div>
              </li>
            </ul>
          </article>

          <article class="card feature-card">
            <div class="card-head">
              <h3>{{ t('dashboard.detectionQueue') }}</h3>
              <span class="tag">{{ t('dashboard.pipeline') }}</span>
            </div>
            <p>{{ t('dashboard.queueDesc') }}</p>
            <div class="queue-list">
              <div class="queue-item" v-for="queue in detectionQueue" :key="queue.name">
                <div class="queue-meta">
                  <span>{{ queue.name }}</span>
                  <strong>{{ queue.pending }} pending</strong>
                </div>
                <div class="progress-track">
                  <div class="progress-fill" :style="{ width: queue.progress + '%' }"></div>
                </div>
              </div>
            </div>
          </article>

          <article class="card feature-card">
            <div class="card-head">
              <h3>{{ t('dashboard.reportExport') }}</h3>
              <span class="tag">{{ t('dashboard.pdf') }}</span>
            </div>
            <p>{{ t('dashboard.reportDesc') }}</p>
            <div class="report-list">
              <div class="report-btn" v-for="report in reports" :key="report.id">
                <strong>Document {{ report.id }}</strong>
                <span>{{ report.filename }}</span>
                <div class="case-actions">
                  <button class="ghost-btn mini" @click="generateReport(report.id)">Generate</button>
                  <button class="ghost-btn mini" @click="downloadReport(report.id)">Download</button>
                </div>
              </div>
            </div>
          </article>
        </section>

        <section class="card ai-full-width">
          <div class="panel-header">
            <h2>{{ t('dashboard.aiResults') }}</h2>
            <span class="tag">{{ t('dashboard.inference') }}</span>
          </div>
          <p>{{ t('dashboard.aiDesc') }}</p>
          <div class="analysis-preview larger">
            <div class="preview-box original box-visual" :style="analysisBackgroundStyle">
              <div class="box-overlay-layer">
                <div
                  v-for="(box, index) in visualizationBoxes"
                  :key="`analysis-${index}`"
                  class="tamper-box"
                  :style="box"
                ></div>
              </div>
            </div>
            <div class="preview-box heatmap box-visual" :style="analysisBackgroundStyle">
              <div class="box-overlay-layer">
                <div
                  v-for="(box, index) in visualizationBoxes"
                  :key="`heat-${index}`"
                  class="tamper-box"
                  :style="box"
                ></div>
              </div>
            </div>
            <div class="analysis-text">
              <span>{{ t('dashboard.lastResult') }}: {{ analysisResult.summary }}</span>
              <strong>{{ analysisResult.confidence }}% {{ t('dashboard.tamperProb') }}</strong>
              <span>is_tampered: {{ analysisResult.isTampered }}</span>
              <span>num_boxes: {{ analysisResult.numBoxes }}</span>
              <span>mask_shape: {{ analysisResult.maskShape }}</span>
              <span>boxes: {{ analysisResult.boxes }}</span>
            </div>
          </div>
        </section>
      </template>

      <template v-else>
        <section class="hero card page-hero">
          <p class="eyebrow">{{ t('settings.eyebrow') }}</p>
          <h1>{{ t('settings.title') }}</h1>
          <p class="hero-subtitle">{{ t('settings.subtitle') }}</p>
        </section>

        <section class="page-grid two-col">
          <article class="card page-card">
            <div class="panel-header">
              <h2>{{ t('settings.realtime') }}</h2>
              <span class="tag">{{ t('settings.security') }}</span>
            </div>
            <div class="settings-list">
              <div class="setting-row" v-for="(setting, index) in settingToggles" :key="setting.name">
                <div>
                  <strong>{{ setting.name }}</strong>
                  <p>{{ setting.desc }}</p>
                </div>
                <button class="switch" :class="{ on: setting.enabled }" @click="toggleSetting(index)">
                  <span></span>
                </button>
              </div>
            </div>
          </article>

          <article class="card page-card">
            <div class="panel-header">
              <h2>{{ t('settings.environment') }}</h2>
              <span class="tag">{{ t('settings.system') }}</span>
            </div>
            <ul class="simple-list compact">
              <li v-for="meta in settingsMeta" :key="meta.key">
                <div>
                  <strong>{{ meta.key }}</strong>
                  <span>{{ meta.value }}</span>
                </div>
              </li>
            </ul>
          </article>
        </section>
      </template>
    </main>
  </div>
</template>

<script setup>
import { computed, onBeforeUnmount, onMounted, ref } from 'vue';
import { useI18n } from 'vue-i18n';

const { t, locale } = useI18n();

const toggleLang = () => {
  locale.value = locale.value === 'EN' ? 'ZH' : 'EN';
};

const navItems = ['Dashboard', 'Settings'];
const activeNav = ref('Dashboard');

function formatNumber(value) {
  return new Intl.NumberFormat('en-US').format(value);
}

const API_BASE = 'http://47.121.198.34:8000';
const API_REFRESH_MS = 12000;

const apiError = ref('');
const documents = ref([]);
const selectedDocumentId = ref(null);

function apiUrl(path) {
  return `${API_BASE}${path}`;
}

function normalizeConfidence(value) {
  const numeric = Number(value ?? 0);
  if (Number.isNaN(numeric)) {
    return 0;
  }
  return numeric <= 1 ? Number((numeric * 100).toFixed(1)) : Number(numeric.toFixed(1));
}

function formatClock(value) {
  if (!value) {
    return '--:--:--';
  }
  const normalized = String(value).replace(' ', 'T');
  const date = new Date(normalized);
  if (Number.isNaN(date.getTime())) {
    return '--:--:--';
  }
  const hh = String(date.getHours()).padStart(2, '0');
  const mm = String(date.getMinutes()).padStart(2, '0');
  const ss = String(date.getSeconds()).padStart(2, '0');
  return `${hh}:${mm}:${ss}`;
}

function mapDocument(item) {
  const analysis = item.analysis || {};
  const id = Number(item.id || 0);
  const rawImageUrl = item.image_url || '';

  return {
    id,
    device: item.device_id || 'unknown',
    filename: item.filename || `document_${id}.jpg`,
    hashValue: item.hash_value || '',
    uploadTime: item.upload_time || '',
    imageUrl: rawImageUrl.startsWith('http') ? rawImageUrl : rawImageUrl ? apiUrl(rawImageUrl) : '',
    isTampered: Boolean(analysis.is_tampered),
    confidence: normalizeConfidence(analysis.confidence),
    numBoxes: Number(analysis.num_boxes || 0),
    boxes: Array.isArray(analysis.boxes) ? analysis.boxes : [],
    maskShape: Array.isArray(analysis.mask_shape) ? analysis.mask_shape : []
  };
}

async function fetchDocuments() {
  const response = await fetch(apiUrl('/documents'));
  if (!response.ok) {
    throw new Error(`${response.status} ${response.statusText}`);
  }
  const payload = await response.json();
  const list = Array.isArray(payload) ? payload : [];
  documents.value = list
    .map(mapDocument)
    .sort((a, b) => new Date(b.uploadTime).getTime() - new Date(a.uploadTime).getTime());

  if (!selectedDocumentId.value && documents.value.length) {
    selectedDocumentId.value = documents.value[0].id;
  }
}

function selectAnalysisById(id) {
  selectedDocumentId.value = id;
}

async function generateReport(id) {
  try {
    const response = await fetch(apiUrl(`/generate_report/${id}?background=true`), { method: 'POST' });
    if (!response.ok) {
      throw new Error(`${response.status} ${response.statusText}`);
    }
  } catch (error) {
    apiError.value = `Generate report failed: ${error.message}`;
  }
}

function downloadReport(id) {
  window.open(apiUrl(`/download_report/${id}`), '_blank', 'noopener,noreferrer');
}

const pipelineNames = [
  'ESP Camera',
  'Image Upload',
  'AI Tampering Detection',
  'Result Visualization',
  'Report Export'
];

const activeStep = ref(2);
const systemStatus = computed(() => {
  const total = documents.value.length;
  const tampered = documents.value.filter((doc) => doc.isTampered).length;
  const confidence = total
    ? Number((documents.value.reduce((sum, doc) => sum + doc.confidence, 0) / total).toFixed(1))
    : 0;

  return {
    uploaded: total,
    tampered,
    confidence
  };
});

const recentUploads = computed(() =>
  documents.value.slice(0, 6).map((doc) => ({
    id: doc.id,
    device: doc.device,
    time: formatClock(doc.uploadTime),
    hashValue: doc.hashValue,
    status: doc.isTampered ? 'Tampered' : 'Clean'
  }))
);

const detectionQueue = ref([
  { name: 'Ingestion', pending: 22, progress: 76 },
  { name: 'Pre-processing', pending: 14, progress: 64 },
  { name: 'AI Inference', pending: 11, progress: 58 },
  { name: 'Post-validation', pending: 6, progress: 83 }
]);

const selectedDocument = computed(
  () => documents.value.find((doc) => doc.id === selectedDocumentId.value) || documents.value[0] || null
);

const analysisResult = computed(() => {
  const doc = selectedDocument.value;
  if (!doc) {
    return {
      summary: 'No document selected',
      confidence: 0,
      isTampered: '-',
      numBoxes: 0,
      maskShape: '-',
      boxes: '[]'
    };
  }

  return {
    summary: `Document ${doc.id} · ${doc.filename}`,
    confidence: doc.confidence,
    isTampered: doc.isTampered,
    numBoxes: doc.numBoxes,
    maskShape: doc.maskShape.length ? doc.maskShape.join(' x ') : '-',
    boxes: JSON.stringify(doc.boxes)
  };
});

const reports = computed(() => documents.value.slice(0, 5));

const analysisBackgroundStyle = computed(() => {
  const url = selectedDocument.value?.imageUrl;
  if (!url) {
    return {};
  }
  return {
    backgroundImage: `url(${url})`,
    backgroundSize: 'cover',
    backgroundPosition: 'center'
  };
});

const visualizationBoxes = computed(() => {
  const doc = selectedDocument.value;
  if (!doc?.boxes?.length || !doc.maskShape?.length) {
    return [];
  }

  const sourceHeight = Number(doc.maskShape[0] || 1);
  const sourceWidth = Number(doc.maskShape[1] || sourceHeight || 1);

  return doc.boxes
    .filter((box) => Array.isArray(box) && box.length === 4)
    .map((box) => {
      const [x1, y1, x2, y2] = box.map((value) => Number(value || 0));
      const left = Math.max(0, Math.min(100, (x1 / sourceWidth) * 100));
      const top = Math.max(0, Math.min(100, (y1 / sourceHeight) * 100));
      const width = Math.max(1, ((x2 - x1) / sourceWidth) * 100);
      const height = Math.max(1, ((y2 - y1) / sourceHeight) * 100);

      return {
        left: `${left}%`,
        top: `${top}%`,
        width: `${Math.min(100 - left, width)}%`,
        height: `${Math.min(100 - top, height)}%`
      };
    });
});

const uploadOverview = [
  { label: 'Images Today', value: '12,481' },
  { label: 'Avg Upload Latency', value: '286 ms' },
  { label: 'Online Cameras', value: '24 / 26' },
  { label: 'Transfer Success', value: '99.2%' }
];

const uploadDevices = [
  { name: 'ESP-CAM-01', note: 'North Gate · 31 fps', state: 'Healthy', stateClass: 'ok' },
  { name: 'ESP-CAM-12', note: 'Lobby · 28 fps', state: 'Stable', stateClass: 'neutral' },
  { name: 'ESP-CAM-19', note: 'Storage · packet loss 2.3%', state: 'Warning', stateClass: 'warn' }
];

const uploadBatches = [
  { id: 'BATCH-912', device: 'ESP-CAM-03', frames: 324, tampered: 18, time: '10:12:04' },
  { id: 'BATCH-911', device: 'ESP-CAM-11', frames: 295, tampered: 9, time: '10:09:41' },
  { id: 'BATCH-910', device: 'ESP-CAM-20', frames: 308, tampered: 12, time: '10:06:15' },
  { id: 'BATCH-909', device: 'ESP-CAM-08', frames: 277, tampered: 7, time: '10:03:50' }
];

const resultSummary = computed(() => [
  { label: 'Analyzed Frames', value: formatNumber(systemStatus.value.uploaded) },
  { label: 'Tampered Findings', value: formatNumber(systemStatus.value.tampered) },
  { label: 'Model Precision', value: '97.1%' },
  { label: 'Avg Confidence', value: `${systemStatus.value.confidence}%` }
]);


const settingToggles = ref([
  { name: 'Realtime Detection Alerts', desc: 'Push critical tampering alerts to incident channel.', enabled: true },
  { name: 'Automatic Report Export', desc: 'Generate end-of-day compliance reports automatically.', enabled: true },
  { name: 'Low Confidence Escalation', desc: 'Require manual review when confidence is between 60%-80%.', enabled: false }
]);

const settingsMeta = [
  { key: 'Model Version', value: 'TamperNet v3.4.1' },
  { key: 'Active Region', value: 'Asia Pacific · Singapore' },
  { key: 'Data Retention', value: '90 days (encrypted)' },
  { key: 'Last Policy Sync', value: '2026-03-07 09:58 UTC+8' }
];

function toggleSetting(index) {
  settingToggles.value[index].enabled = !settingToggles.value[index].enabled;
}

const workflowSteps = computed(() => {
  const { uploaded, tampered, confidence } = systemStatus.value;
  const factors = [1, 0.98, 0.95, 0.92, 0.89];

  return pipelineNames.map((name, index) => {
    const factor = factors[index];
    const confidenceOffset = index * 0.2;

    return {
      name,
      uploaded: Math.max(0, Math.floor(uploaded * factor)),
      tampered: Math.max(0, Math.floor(tampered * factor)),
      confidence: Math.max(70, Number((confidence - confidenceOffset).toFixed(1))),
      state: index < activeStep.value ? 'done' : index === activeStep.value ? 'active' : 'pending'
    };
  });
});

const tickRealtime = () => {
  activeStep.value = (activeStep.value + 1) % pipelineNames.length;
};

let timerId;

onMounted(() => {
  fetchDocuments().catch((error) => {
    apiError.value = `API request failed: ${error.message}`;
  });
  window.setInterval(() => {
    fetchDocuments().catch((error) => {
      apiError.value = `API request failed: ${error.message}`;
    });
  }, API_REFRESH_MS);

  timerId = window.setInterval(tickRealtime, 2600);
});

onBeforeUnmount(() => {
  if (timerId) {
    window.clearInterval(timerId);
  }
});
</script>
