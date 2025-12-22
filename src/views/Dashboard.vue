<template>
  <div class="container-fluid py-3 min-vh-100">
    <div class="row h-100" ref="dashboardRow">
      <!-- Left sidebar menu -->
      <aside class="col-12 col-md-2 d-flex flex-column" style="position:relative;height:100%" ref="sidebarRef">
        <div class="d-flex gap-2 mb-3">
          <button class="btn btn-sm btn-outline-secondary w-100" @click="noop('save')">Save</button>
          <button class="btn btn-sm btn-outline-secondary w-100" @click="noop('import')">Import</button>
          <button class="btn btn-sm btn-outline-secondary w-100" @click="noop('export')">Export</button>
        </div>
        <div class="menu-flex p-2 mb-3 flex-grow-1">
          <button v-for="item in menuItems" :key="item.key" class="btn btn-light w-100 my-2 text-start" @click="noop(item.key)">
            {{ item.label }}
          </button>
        </div>

        <div ref="prefWrap" style="position:absolute; left:0; width:100%;">
          <button ref="prefBtn" class="btn btn-outline-primary w-100 mt-2" @click="noop('preferences')">Preferences</button>
        </div>
      </aside>

      <!-- Main content -->
      <main class="col-12 col-md-10 d-flex flex-column">
        <div class="row g-3 flex-grow-1" style="overflow:auto;">
          <div class="col-12 col-lg-3">
            <div class="card viz-card h-100 p-3 d-flex flex-column">
              <h6 class="mb-2 text-center w-100">Tasks</h6>
              <!-- Chart.js donut for tasks -->
              <div class="w-100 d-flex flex-column" style="flex:1; position:relative;">
                <div class="flex-grow-1 d-flex align-items-center justify-content-center">
                  <canvas ref="taskCanvas" class="task-canvas" width="220" height="220" style="width:220px; height:220px; max-width:100%; max-height:220px;"></canvas>
                </div>

                <div class="d-flex justify-content-center gap-4 mt-4 text-muted counts-row">
                  <div class="text-center"><div class="small text-muted">To-Do</div><div><strong>{{ taskBreakdown[0] }}</strong></div></div>
                  <div class="text-center"><div class="small text-muted">Done</div><div><strong>{{ taskBreakdown[1] }}</strong></div></div>
                </div>

              </div>
            </div>
          </div>

          <div class="col-12 col-lg-6">
            <div class="card viz-card h-100 p-4 d-flex flex-column">
              <h5 class="text-muted text-center w-100">Your Special Moment Is Coming In:</h5>
              <div class="d-flex align-items-center justify-content-center flex-grow-1">
                <div class="text-center">
                  <div v-if="hasWeddingDate">
                    <h2 class="display-6 mt-3">{{ countdownText }}</h2>
                  </div>
                  <div v-else class="small text-muted mt-3">no wedding date set yet • <a href="#" @click.prevent="goToPreferences">add wedding date</a></div>
                </div>
              </div>
            </div>
          </div>

          <div class="col-12 col-lg-3">
            <div class="card viz-card h-100 p-3 d-flex flex-column">
              <h6 class="mb-2 text-center w-100">Invitation Photo</h6>
              <div class="d-flex align-items-center justify-content-center flex-grow-1">
                <div class="text-center">
                  <div v-if="hasInvitationImage">
                    <img :src="state.value.settings.invitationImage" alt="invitation" class="invitation-placeholder mx-auto mb-2" />
                  </div>
                  <div v-else class="my-2 small text-muted">no image added yet • <a href="#" @click.prevent="goToPreferences">add invitation image</a></div>
                </div>
              </div>
            </div>
          </div>

          <div class="col-12 col-lg-6" ref="venueCol">
            <div class="card p-3 thin-card" ref="venueCard" style="position:relative">
              <div class="venue-inner" style="width:100%">
                  <h6 class="venue-header">Venue Location</h6>
                  <div class="venue-body">
                    <div v-if="hasVenue" class="map-placeholder my-2"></div>
                    <div v-else class="venue-placeholder small text-muted text-center">no venue location added yet • <a href="#" @click.prevent="goToPreferences">add venue location</a></div>
                  </div>
                </div>
            </div>
          </div>

          <div class="col-12 col-lg-6">
            <div class="card p-3 thin-card d-flex align-items-center">
              <div style="width:100%; display:flex; flex-direction:column; height:100%">
                <h6 class="mb-2">Guests</h6>
                <div class="mb-2">
                  <canvas ref="guestCanvas" class="guest-canvas" width="400" height="40" style="width:100%; height:40px"></canvas>
                </div>

                <div class="mt-auto guest-counts-wrap">
                  <div class="text-center"><div class="small text-muted">Unlikely</div><div class="guest-count-number"><strong :style="{color: '#e74c3c'}">{{ guestCounts[0] }}</strong></div></div>
                  <div class="text-center"><div class="small text-muted">Likely</div><div class="guest-count-number"><strong :style="{color: '#f1c40f'}">{{ guestCounts[1] }}</strong></div></div>
                  <div class="text-center"><div class="small text-muted">Definitely</div><div class="guest-count-number"><strong :style="{color: '#196f3d'}">{{ guestCounts[2] }}</strong></div></div>
                  <div class="text-center"><div class="small text-muted">Confirmed</div><div class="guest-count-number"><strong :style="{color: '#27ae60'}">{{ guestCounts[3] }}</strong></div></div>
                </div>
                <div class="text-center mt-2 total-guests">Total Guests Invited: <strong class="total-guests-number">{{ totalGuests }}</strong></div>
              </div>
            </div>
          </div>

          <!-- bottom spacer (removed Data & tools) -->
        </div>
      </main>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted, nextTick } from 'vue'
import Chart from 'chart.js/auto'
import { useRouter } from 'vue-router'
import store from '../utils/store'

const state = ref(store.getState())

function refresh(s) {
  state.value = JSON.parse(JSON.stringify(s))
}

let unsub
onMounted(() => {
  store.init()
  refresh(store.getState())
  unsub = store.onChange((s) => { refresh(s); nextTick(() => { alignPreferences(); updateTaskChart(); updateGuestChart() }) })
})
onUnmounted(() => { if (unsub) unsub() })

const router = useRouter()

function goToPreferences() {
  router.push('/preferences')
}

// refs for alignment
const dashboardRow = ref(null)
const sidebarRef = ref(null)
const venueCard = ref(null)
const prefWrap = ref(null)
const prefBtn = ref(null)

function alignPreferences() {
  // align the bottom of the preferences button with bottom of the venue card
  if (!sidebarRef.value || !venueCard.value || !prefWrap.value || !prefBtn.value) return
  const sidebarRect = sidebarRef.value.getBoundingClientRect()
  const venueRect = venueCard.value.getBoundingClientRect()
  const prefRect = prefBtn.value.getBoundingClientRect()
  // compute top relative to sidebar (so absolute positioning aligns correctly)
  const EXTRA_OFFSET = 12 // nudge up a bit to visually align
  const top = (venueRect.bottom - sidebarRect.top) - prefRect.height - EXTRA_OFFSET
  prefWrap.value.style.top = Math.max(0, top) + 'px'
}

function onResize() {
  nextTick(() => alignPreferences())
}

const tasksDue = computed(() => {
  const now = new Date()
  const limit = new Date(now.getTime() + 7 * 24 * 60 * 60 * 1000)
  const tasks = state.value.tasks || []
  return tasks.filter((t) => {
    if (!t || t.status === 'done') return false
    if (!t.due) return true
    const d = new Date(t.due)
    return d >= now && d <= limit
  }).length
})

const rsvpPercent = computed(() => {
  const guests = state.value.guests || []
  const invited = guests.length
  if (invited === 0) return 0
  const yes = guests.filter(g => g && g.rsvp === 'yes').length
  return Math.round((yes / invited) * 100)
})

const budgetSummary = computed(() => {
  const items = state.value.budget || []
  const total = items.reduce((s, it) => s + (Number(it.cost) || 0), 0)
  const spent = items.reduce((s, it) => s + ((it.paid) ? (Number(it.cost) || 0) : 0), 0)
  return { total, spent }
})

const pendingPayments = computed(() => {
  const items = state.value.budget || []
  return items.filter(it => !it.paid).length
})

// guest counts (support four types: unlikely, likely, definitely, confirmed)
const guestCounts = computed(() => {
  const guests = state.value.guests || []
  if (guests.length === 0) return [0,0,0,0]
  const unlikely = guests.filter(g => g && g.rsvp === 'unlikely').length
  const likely = guests.filter(g => g && g.rsvp === 'likely').length
  const definitely = guests.filter(g => g && g.rsvp === 'definitely').length
  const confirmed = guests.filter(g => g && (g.rsvp === 'confirmed' || g.rsvp === 'yes')).length
  return [unlikely, likely, definitely, confirmed]
})

const guestPercent = computed(() => {
  const counts = guestCounts.value
  const total = counts.reduce((s,n)=>s+n,0) || 1
  return {
    unlikely: Math.round((counts[0]/total)*100),
    likely: Math.round((counts[1]/total)*100),
    definitely: Math.round((counts[2]/total)*100),
    confirmed: Math.round((counts[3]/total)*100)
  }
})

const totalGuests = computed(() => {
  const counts = guestCounts.value || [0,0,0,0]
  return counts.reduce((s,n) => s + (Number(n) || 0), 0)
})

// countdown
import { ref as _ref } from 'vue'
const countdownText = _ref('')

function updateCountdown() {
  const settings = state.value.settings || {}
  const target = settings.weddingDate ? new Date(settings.weddingDate) : new Date(Date.now() + 1000 * 60 * 60 * 24 * 120)
  const diff = target - Date.now()
  if (diff <= 0) {
    countdownText.value = 'Wedding date reached'
    return
  }
  const days = Math.floor(diff / (1000*60*60*24))
  const months = Math.floor(days / 30)
  const weeks = Math.floor((days % 30) / 7)
  const remDays = Math.floor((days % 30) % 7)
  const hours = Math.floor((diff / (1000*60*60)) % 24)
  const minutes = Math.floor((diff / (1000*60)) % 60)
  const seconds = Math.floor((diff / 1000) % 60)
  countdownText.value = `${months}months ${weeks} weeks ${remDays} days ${hours} hours ${minutes} minutes ${seconds} seconds`
}

let countdownInterval = null
onMounted(() => {
  updateCountdown()
  countdownInterval = setInterval(updateCountdown, 1000)
  // align preferences after mount and on resize
  alignPreferences()
  window.addEventListener('resize', onResize)
  // create charts after DOM ready
  nextTick(() => {
    createTaskChart()
    createGuestChart()
  })
})
onUnmounted(() => { if (countdownInterval) clearInterval(countdownInterval) })
onUnmounted(() => { window.removeEventListener('resize', onResize) })

function noop(name) {
  // placeholder clickable action
  // eslint-disable-next-line no-console
  console.log('noop click', name)
}

// pie path helper
function piePath(startFrac, endFrac, cx, cy, r) {
  // simple conversion to arc path in percentage of circle
  const startAngle = startFrac * Math.PI * 2 - Math.PI/2
  const endAngle = endFrac * Math.PI * 2 - Math.PI/2
  const x1 = cx + Math.cos(startAngle) * (r*30)
  const y1 = cy + Math.sin(startAngle) * (r*30)
  const x2 = cx + Math.cos(endAngle) * (r*30)
  const y2 = cy + Math.sin(endAngle) * (r*30)
  const large = endFrac - startFrac > 0.5 ? 1 : 0
  return `M ${cx} ${cy} L ${x1} ${y1} A ${r*30} ${r*30} 0 ${large} 1 ${x2} ${y2} Z`
}

const menuItems = [
  { key: 'dashboard', label: 'Dashboard' },
  { key: 'todo', label: 'To Do List' },
  { key: 'notes', label: 'Notes' },
  { key: 'guests', label: 'Guest List' },
  { key: 'seating', label: 'Seating Chart' },
  { key: 'contacts', label: 'Contacts & Vendors' },
  { key: 'budget', label: 'Budget' }
]

// has venue and invitation helpers
const hasVenue = computed(() => !!(state.value.settings && state.value.settings.venue))
const hasInvitationImage = computed(() => !!(state.value.settings && state.value.settings.invitationImage))
const hasWeddingDate = computed(() => !!(state.value.settings && state.value.settings.weddingDate))

// tasks breakdown
const tasksCount = computed(() => (state.value.tasks || []).length)
const taskBreakdown = computed(() => {
  const tasks = state.value.tasks || []
  if (tasks.length === 0) return [0, 0]
  const todo = tasks.filter(t => t && t.status !== 'done').length
  const done = tasks.filter(t => t && t.status === 'done').length
  return [todo, done]
})

// Chart.js instances
const taskCanvas = ref(null)
const guestCanvas = ref(null)
let taskChart = null
let guestChart = null

// plugin to draw the center number inside the donut
const centerTextPlugin = {
  id: 'centerText',
  afterDraw(chart) {
    if (!chart || !chart.ctx) return
    const ctx = chart.ctx
    const txt = String(tasksCount.value || 0)
    ctx.save()
    const fontSize = Math.round((chart.height || 220) / 8)
    ctx.font = `bold ${fontSize}px sans-serif`
    ctx.fillStyle = '#495057'
    ctx.textAlign = 'center'
    ctx.textBaseline = 'middle'
    const cx = (chart.chartArea.left + chart.chartArea.right) / 2
    const cy = (chart.chartArea.top + chart.chartArea.bottom) / 2
    ctx.fillText(txt, cx, cy)
    ctx.restore()
  }
}

function createTaskChart() {
  if (!taskCanvas.value) return
  const ctx = taskCanvas.value.getContext('2d')
  const data = taskBreakdown.value
  // if all zeros, show an empty gray donut visually but keep breakdown values as 0
  const total = data.reduce((s, n) => s + n, 0)
  const displayData = total === 0 ? [1, 0] : data
  const colors = total === 0 ? ['#e9ecef', '#f8f9fA'] : ['#ff6384', '#36a2eb']
  taskChart = new Chart(ctx, {
    type: 'doughnut',
    data: {
      labels: ['Todo', 'Done'],
      datasets: [{ data: displayData, backgroundColor: colors, borderWidth: 2, borderColor: '#dee2e6' }]
    },
    options: {
      responsive: true,
      maintainAspectRatio: false,
      cutout: '68%',
      plugins: { legend: { display: false }, tooltip: { enabled: true } }
    }
    ,
    plugins: [centerTextPlugin]
  })
}

function updateTaskChart() {
  if (!taskChart) return
  const data = taskBreakdown.value
  const total = data.reduce((s, n) => s + n, 0)
  if (total === 0) {
    taskChart.data.datasets[0].data = [1, 0]
    taskChart.data.datasets[0].backgroundColor = ['#e9ecef', '#f8f9fA']
  } else {
    taskChart.data.datasets[0].data = data
    taskChart.data.datasets[0].backgroundColor = ['#ff6384', '#36a2eb']
  }
  taskChart.update()
}

function createGuestChart() {
  if (!guestCanvas.value) return
  const ctx = guestCanvas.value.getContext('2d')
  const counts = guestCounts.value
  // create stacked horizontal bar with four types: unlikely, likely, definitely, confirmed
  guestChart = new Chart(ctx, {
    type: 'bar',
    data: {
      labels: [''],
      datasets: [
        { label: 'Unlikely', data: [counts[0]], backgroundColor: '#e74c3c' },
        { label: 'Likely', data: [counts[1]], backgroundColor: '#f1c40f' },
        { label: 'Definitely', data: [counts[2]], backgroundColor: '#196f3d' },
        { label: 'Confirmed', data: [counts[3]], backgroundColor: '#27ae60' }
      ]
    },
    options: {
      indexAxis: 'y',
      responsive: true,
      maintainAspectRatio: false,
      scales: { x: { stacked: true, ticks: { display: false }, grid: { display: false } }, y: { stacked: true, grid: { display: false }, ticks: { display: false } } },
      plugins: { legend: { display: false }, tooltip: { enabled: true } }
    }
  })
}

function updateGuestChart() {
  if (!guestChart) return
  const counts = guestCounts.value
  guestChart.data.datasets[0].data = [counts[0]]
  guestChart.data.datasets[1].data = [counts[1]]
  guestChart.data.datasets[2].data = [counts[2]]
  guestChart.data.datasets[3].data = [counts[3]]
  guestChart.update()
}


</script>

<style scoped>
.viz-card{min-height:340px}
@media (min-width: 992px) {
  .viz-card{min-height:680px}
}
.invitation-placeholder{width:120px;height:120px;border:2px dashed #ccc;margin:0 auto}
.map-placeholder{height:140px;border:2px dashed #ccc}
.thin-card{min-height:200px}
.menu-flex{max-height:60vh;overflow:auto}
/* ensure thin cards align */
.thin-card{display:flex;align-items:center}
.thin-card .map-placeholder{height:120px}
.viz-card svg{height:100%;max-height:520px}

/* Counts row below the pie: larger numbers and more spacing */
.counts-row{margin-top:1.5rem}
.counts-row strong{font-size:1.4rem;line-height:1}
.task-canvas{display:block}

/* Guests card: center bottom counts */
.guest-counts-wrap{margin:0 auto 8px; display:flex; justify-content:space-between; align-items:center; width:100%; max-width:520px; padding:0 12px}
.guest-counts-wrap > .text-center{flex:1}
.guest-counts{font-size:1.1rem}
.guest-count-number strong{font-size:1.2rem}
.guest-canvas{display:block}

/* Total guests styling */
.total-guests{font-size:0.95rem;color:#495057;line-height:1}
.total-guests-number{font-size:1rem;margin-left:8px;color:#2b2b2b;display:inline-block;line-height:1;transform:scale(1.45);transform-origin:center}

/* Venue centering */
.venue-inner{position:relative;display:flex;flex-direction:column;padding:8px}
.venue-header{position:absolute;left:0;right:0;top:8px;text-align:center;margin:0;padding:4px 0;font-weight:600}
.venue-body{position:absolute;left:12px;right:12px;top:44px;bottom:12px;display:flex;align-items:center;justify-content:center}
.venue-placeholder{padding:6px}
</style>
