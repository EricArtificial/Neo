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
          {{ item }}
        </a>
      </nav>
      <button class="nav-btn">Live</button>
    </header>

    <main class="container">
      <template v-if="activeNav === 'Dashboard'">
        <section class="hero card">
          <h1><span class="hero-gradient">AI Dashboard</span> for Image Authenticity Detection</h1>
          <p class="hero-subtitle">
            ESP camera streams are monitored in real time, tampering is detected by AI, and outputs are prepared for result visualization and report export.
          </p>
        </section>

        <section class="card workflow-panel">
          <div class="panel-header">
            <h2>Workflow Pipeline</h2>
            <span class="live-pill">Real-time Status</span>
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
                    <p>Uploaded</p>
                    <strong>{{ formatNumber(step.uploaded) }}</strong>
                  </div>
                  <div>
                    <p>Tampered</p>
                    <strong>{{ formatNumber(step.tampered) }}</strong>
                  </div>
                  <div>
                    <p>Avg Confidence</p>
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
              <h3>Live Image Feed</h3>
              <span class="tag">Streaming</span>
            </div>
            <p>Latest image frames uploaded from ESP devices with device ID, timestamp, and stream state.</p>
            <ul class="feed-list">
              <li v-for="item in recentUploads" :key="item.id">
                <div>
                  <strong>{{ item.id }}</strong>
                  <span>{{ item.device }} · {{ item.time }}</span>
                </div>
                <em>{{ item.status }}</em>
              </li>
            </ul>
          </article>

          <article class="card feature-card">
            <div class="card-head">
              <h3>Detection Queue</h3>
              <span class="tag">Pipeline</span>
            </div>
            <p>Real-time processing queue from uploaded frames to model inference readiness.</p>
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
              <h3>AI Analysis Results</h3>
              <span class="tag">Inference</span>
            </div>
            <p>Tampered regions and confidence scores generated with explainable AI outputs.</p>
            <div class="analysis-preview">
              <div class="preview-box original"></div>
              <div class="preview-box heatmap"></div>
              <div class="analysis-text">
                <span>Last Result: {{ analysisResult.summary }}</span>
                <strong>{{ analysisResult.confidence }}% tampering probability</strong>
              </div>
            </div>
          </article>

          <article class="card feature-card">
            <div class="card-head">
              <h3>Report Export Center</h3>
              <span class="tag">PDF</span>
            </div>
            <p>Generate and export forensic results into downloadable reports for audit and compliance.</p>
            <div class="report-list">
              <button class="report-btn" v-for="report in reports" :key="report.id">
                {{ report.name }}
              </button>
            </div>
          </article>
        </section>
      </template>

      <template v-else-if="activeNav === 'Image Uploads'">
        <section class="hero card page-hero">
          <p class="eyebrow">Capture & Ingestion</p>
          <h1>Image Upload Monitoring</h1>
          <p class="hero-subtitle">Track ESP camera upload throughput, transfer latency, and device stream quality in one minimal control view.</p>
        </section>

        <section class="page-grid two-col">
          <article class="card page-card">
            <div class="panel-header">
              <h2>Upload Overview</h2>
              <span class="tag">Live</span>
            </div>
            <div class="kpi-grid">
              <div class="kpi-item" v-for="item in uploadOverview" :key="item.label">
                <p>{{ item.label }}</p>
                <strong>{{ item.value }}</strong>
              </div>
            </div>
          </article>

          <article class="card page-card">
            <div class="panel-header">
              <h2>Camera Health</h2>
              <span class="tag">Fleet</span>
            </div>
            <ul class="simple-list">
              <li v-for="device in uploadDevices" :key="device.name">
                <div>
                  <strong>{{ device.name }}</strong>
                  <span>{{ device.note }}</span>
                </div>
                <span class="status-pill" :class="device.stateClass">{{ device.state }}</span>
              </li>
            </ul>
          </article>

          <article class="card page-card span-2">
            <div class="panel-header">
              <h2>Latest Upload Batches</h2>
              <button class="ghost-btn">Export CSV</button>
            </div>
            <div class="table-wrap">
              <table class="data-table">
                <thead>
                  <tr>
                    <th>Batch</th>
                    <th>Device</th>
                    <th>Frames</th>
                    <th>Tampered</th>
                    <th>Uploaded At</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="batch in uploadBatches" :key="batch.id">
                    <td>{{ batch.id }}</td>
                    <td>{{ batch.device }}</td>
                    <td>{{ batch.frames }}</td>
                    <td>{{ batch.tampered }}</td>
                    <td>{{ batch.time }}</td>
                  </tr>
                </tbody>
              </table>
            </div>
          </article>
        </section>
      </template>

      <template v-else-if="activeNav === 'Detection Results'">
        <section class="hero card page-hero">
          <p class="eyebrow">Model Inference</p>
          <h1>Detection Results</h1>
          <p class="hero-subtitle">Review model outputs, tampering confidence, and explainability artifacts from recent image inspections.</p>
        </section>

        <section class="page-grid two-col">
          <article class="card page-card">
            <div class="panel-header">
              <h2>Inference Summary</h2>
              <span class="tag">24h</span>
            </div>
            <div class="kpi-grid">
              <div class="kpi-item" v-for="item in resultSummary" :key="item.label">
                <p>{{ item.label }}</p>
                <strong>{{ item.value }}</strong>
              </div>
            </div>
          </article>

          <article class="card page-card">
            <div class="panel-header">
              <h2>Flagged Cases</h2>
              <span class="tag">Priority</span>
            </div>
            <ul class="simple-list">
              <li v-for="item in resultCases" :key="item.id">
                <div>
                  <strong>{{ item.id }} · {{ item.device }}</strong>
                  <span>{{ item.region }} · {{ item.confidence }}%</span>
                </div>
                <span class="status-pill" :class="item.stateClass">{{ item.state }}</span>
              </li>
            </ul>
          </article>

          <article class="card page-card span-2">
            <div class="panel-header">
              <h2>Visualization Panel</h2>
              <button class="ghost-btn">Open Full Viewer</button>
            </div>
            <div class="analysis-preview larger">
              <div class="preview-box original"></div>
              <div class="preview-box heatmap"></div>
              <div class="analysis-text">
                <span>Model Explanation</span>
                <strong>Region-level inconsistencies indicate compositing probability above 95% in the detected face area.</strong>
              </div>
            </div>
          </article>
        </section>
      </template>

      <template v-else-if="activeNav === 'Reports'">
        <section class="hero card page-hero">
          <p class="eyebrow">Compliance Workspace</p>
          <h1>Reports</h1>
          <p class="hero-subtitle">Generate, schedule, and export investigation-ready PDF reports for stakeholders and compliance teams.</p>
        </section>

        <section class="page-grid two-col">
          <article class="card page-card">
            <div class="panel-header">
              <h2>Report Templates</h2>
              <span class="tag">Library</span>
            </div>
            <div class="template-grid">
              <div class="template-item" v-for="tpl in reportTemplates" :key="tpl.name">
                <strong>{{ tpl.name }}</strong>
                <span>{{ tpl.desc }}</span>
                <em>{{ tpl.type }}</em>
              </div>
            </div>
          </article>

          <article class="card page-card">
            <div class="panel-header">
              <h2>Export Queue</h2>
              <span class="tag">Jobs</span>
            </div>
            <ul class="simple-list">
              <li v-for="job in reportJobs" :key="job.name">
                <div>
                  <strong>{{ job.name }}</strong>
                  <span>{{ job.time }}</span>
                </div>
                <span class="status-pill" :class="job.stateClass">{{ job.state }}</span>
              </li>
            </ul>
          </article>

          <article class="card page-card span-2">
            <div class="panel-header">
              <h2>Recent Exports</h2>
              <button class="ghost-btn">Download All</button>
            </div>
            <div class="report-list horizontal">
              <button class="report-btn" v-for="history in reportHistory" :key="history.id">
                <strong>{{ history.name }}</strong>
                <span>{{ history.meta }}</span>
              </button>
            </div>
          </article>
        </section>
      </template>

      <template v-else>
        <section class="hero card page-hero">
          <p class="eyebrow">Platform Controls</p>
          <h1>Settings</h1>
          <p class="hero-subtitle">Manage model behavior, notification policies, and IoT device security preferences.</p>
        </section>

        <section class="page-grid two-col">
          <article class="card page-card">
            <div class="panel-header">
              <h2>Realtime Preferences</h2>
              <span class="tag">Security</span>
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
              <h2>Environment</h2>
              <span class="tag">System</span>
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

const navItems = ['Dashboard', 'Image Uploads', 'Detection Results', 'Reports', 'Settings'];
const activeNav = ref('Dashboard');

function formatNumber(value) {
  return new Intl.NumberFormat('en-US').format(value);
}

const pipelineNames = [
  'ESP Camera',
  'Image Upload',
  'AI Tampering Detection',
  'Result Visualization',
  'Report Export'
];

const activeStep = ref(2);
const systemStatus = ref({
  uploaded: 4982,
  tampered: 386,
  confidence: 93.1
});

const recentUploads = ref([
  { id: 'IMG_23911', device: 'ESP-CAM-01', time: '09:21:14', status: 'Uploaded' },
  { id: 'IMG_23908', device: 'ESP-CAM-12', time: '09:20:58', status: 'Uploaded' },
  { id: 'IMG_23902', device: 'ESP-CAM-05', time: '09:19:40', status: 'Queued' },
  { id: 'IMG_23896', device: 'ESP-CAM-23', time: '09:18:33', status: 'Uploaded' }
]);

const detectionQueue = ref([
  { name: 'Ingestion', pending: 22, progress: 76 },
  { name: 'Pre-processing', pending: 14, progress: 64 },
  { name: 'AI Inference', pending: 11, progress: 58 },
  { name: 'Post-validation', pending: 6, progress: 83 }
]);

const analysisResult = ref({
  summary: 'Edge blending anomaly in lower-right region',
  confidence: 96.4
});

const reports = ref([
  { id: 1, name: 'Daily Security Report.pdf' },
  { id: 2, name: 'Tampering Incident Log.pdf' },
  { id: 3, name: 'Device Confidence Summary.pdf' }
]);

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

const resultCases = ref([
  { id: 'CASE-1192', device: 'ESP-CAM-07', region: 'Gate A', confidence: 96.4, state: 'Tampered', stateClass: 'critical' },
  { id: 'CASE-1188', device: 'ESP-CAM-14', region: 'Dock 3', confidence: 91.2, state: 'Review', stateClass: 'warn' },
  { id: 'CASE-1182', device: 'ESP-CAM-02', region: 'Hallway', confidence: 84.9, state: 'Observed', stateClass: 'neutral' }
]);

const reportTemplates = [
  { name: 'Daily Executive Summary', desc: 'Overview of anomalies, confidence trend, and critical events', type: 'PDF Template' },
  { name: 'Device Forensic Report', desc: 'Per-device anomaly timeline with visual evidence snapshots', type: 'PDF Template' },
  { name: 'Compliance Export Pack', desc: 'Structured report bundle for monthly audit reviews', type: 'ZIP + PDF' }
];

const reportJobs = [
  { name: 'Daily Security Digest', time: 'Running · 10:15', state: 'Processing', stateClass: 'neutral' },
  { name: 'Incident #217 Package', time: 'Queued · 10:09', state: 'Queued', stateClass: 'warn' },
  { name: 'Board Summary', time: 'Completed · 09:52', state: 'Ready', stateClass: 'ok' }
];

const reportHistory = [
  { id: 1, name: 'Executive Summary · March 07', meta: 'Generated 09:45 · 2.4 MB' },
  { id: 2, name: 'Tampering Casebook · Week 10', meta: 'Generated 08:10 · 5.8 MB' },
  { id: 3, name: 'Fleet Confidence Audit', meta: 'Generated 07:50 · 1.7 MB' }
];

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
  const uploadDelta = Math.floor(3 + Math.random() * 8);
  const tamperedDelta = Math.floor(Math.random() * 2);
  const confidenceDrift = (Math.random() - 0.45) * 0.3;

  systemStatus.value.uploaded += uploadDelta;
  systemStatus.value.tampered += tamperedDelta;
  systemStatus.value.confidence = Math.min(
    99.4,
    Math.max(88.5, Number((systemStatus.value.confidence + confidenceDrift).toFixed(1)))
  );

  activeStep.value = (activeStep.value + 1) % pipelineNames.length;

  const now = new Date();
  const hh = String(now.getHours()).padStart(2, '0');
  const mm = String(now.getMinutes()).padStart(2, '0');
  const ss = String(now.getSeconds()).padStart(2, '0');
  const device = `ESP-CAM-${String(1 + Math.floor(Math.random() * 24)).padStart(2, '0')}`;
  const statusPool = ['Uploaded', 'Uploaded', 'Queued', 'Synced'];

  recentUploads.value.unshift({
    id: `IMG_${Math.floor(23000 + Math.random() * 2000)}`,
    device,
    time: `${hh}:${mm}:${ss}`,
    status: statusPool[Math.floor(Math.random() * statusPool.length)]
  });

  if (recentUploads.value.length > 4) {
    recentUploads.value.pop();
  }

  detectionQueue.value = detectionQueue.value.map((item) => {
    const pendingDelta = Math.floor(Math.random() * 3) - 1;
    const progressDelta = Math.floor(Math.random() * 9) - 4;

    return {
      ...item,
      pending: Math.max(0, item.pending + pendingDelta),
      progress: Math.min(99, Math.max(25, item.progress + progressDelta))
    };
  });

  analysisResult.value.confidence = Number((94 + Math.random() * 4.8).toFixed(1));
  resultCases.value[0].confidence = Number((94 + Math.random() * 4).toFixed(1));
};

let timerId;

onMounted(() => {
  timerId = window.setInterval(tickRealtime, 2600);
});

onBeforeUnmount(() => {
  if (timerId) {
    window.clearInterval(timerId);
  }
});
</script>
