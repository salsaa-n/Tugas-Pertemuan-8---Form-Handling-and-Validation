<template>
  <div class="dashboard-page">

    <!-- Navbar -->
    <nav class="navbar">
      <div class="navbar-brand">
        <svg width="22" height="22" viewBox="0 0 48 48" aria-hidden="true">
          <path d="M24 4 C30 10, 30 16, 24 18 C18 16, 18 10, 24 4 Z" fill="#d4af37"/>
          <rect x="22" y="2" width="4" height="6" fill="#d4af37"/>
          <path d="M10 40 L10 26 C10 18, 16 14, 24 14 C32 14, 38 18, 38 26 L38 40 Z" fill="#d4af37"/>
          <rect x="6" y="40" width="36" height="4" fill="#d4af37"/>
        </svg>
        <span>Prayer Times</span>
      </div>
      <div class="navbar-right">
        <span class="navbar-user">{{ authStore.username || 'User' }}</span>
        <button type="button" class="logout-button" @click="handleLogout">Logout</button>
      </div>
    </nav>

    <!-- Main content -->
    <main class="main-content">

      <!-- Info bar -->
      <div class="info-bar">
        <div class="info-bar-left">
          <h1 class="info-title">Prayer Times</h1>
          <p class="info-date">{{ todayReadable }}</p>
          <p class="info-hijri" v-if="hijriDate">{{ hijriDate }}</p>
        </div>
        <div class="info-bar-right">
          <label for="city" class="city-label">City</label>
          <select id="city" v-model="selectedCity" @change="fetchPrayerTimes" class="city-select">
            <option v-for="c in cityList" :key="c.city" :value="c">{{ c.label }}</option>
          </select>
        </div>
      </div>

      <!-- Loading / Error -->
      <div v-if="loading" class="status-box">
        <p>Loading prayer schedule...</p>
      </div>

      <div v-else-if="errorMessage" class="status-box error">
        <p>{{ errorMessage }}</p>
        <button type="button" @click="fetchPrayerTimes">Try again</button>
      </div>

      <template v-else-if="timings">
        <!-- Next prayer banner -->
        <div class="next-banner">
          <span class="next-banner-label">Next Prayer Time</span>
          <span class="next-banner-name">{{ nextPrayer.label }}</span>
          <span class="next-banner-time">{{ nextPrayer.time }}</span>
          <span class="next-banner-countdown" v-if="countdownText">{{ countdownText }}</span>
        </div>

        <!-- Prayer grid 2 kolom -->
        <div class="prayer-grid">
          <div
            v-for="prayer in prayerList"
            :key="prayer.key"
            class="prayer-card"
            :class="{ active: prayer.key === nextPrayer.key, selected: prayer.key === selectedPrayer?.key }"
            @click="togglePrayerDetail(prayer)"
          >
            <p class="prayer-card-name">{{ prayer.label }}</p>
            <p class="prayer-card-time">{{ prayer.time }}</p>
          </div>
        </div>

        <!-- Detail panel -->
        <Transition name="detail-fade">
          <div v-if="selectedPrayer" class="detail-panel">
            <p class="detail-name">{{ selectedPrayer.label }}</p>
            <p class="detail-time">{{ selectedPrayer.time }}</p>
            <p class="detail-status">{{ selectedPrayerStatus }}</p>
          </div>
        </Transition>
      </template>

    </main>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue';
import { useRouter } from 'vue-router';
import { useAuthStore } from '../stores/auth';

const router = useRouter();
const authStore = useAuthStore();

const handleLogout = () => {
  authStore.logout();
  router.push('/login');
};

const ALADHAN_API_URL = 'https://api.aladhan.com/v1/timingsByCity';

const CITY_LIST = [
  { city: 'Jakarta', country: 'Indonesia', label: 'Jakarta' },
  { city: 'Bogor', country: 'Indonesia', label: 'Bogor' },
  { city: 'Bandung', country: 'Indonesia', label: 'Bandung' },
  { city: 'Surabaya', country: 'Indonesia', label: 'Surabaya' },
  { city: 'Yogyakarta', country: 'Indonesia', label: 'Yogyakarta' },
  { city: 'Medan', country: 'Indonesia', label: 'Medan' },
  { city: 'Semarang', country: 'Indonesia', label: 'Semarang' },
  { city: 'Makassar', country: 'Indonesia', label: 'Makassar' },
];

const PRAYER_LABELS = {
  Fajr: 'Fajr',
  Dhuhr: 'Dhuhr',
  Asr: 'Asr',
  Maghrib: 'Maghrib',
  Isha: 'Isha',
};

const cityList = CITY_LIST;
const selectedCity = ref(CITY_LIST[0]);
const timings = ref(null);
const hijriDate = ref('');
const loading = ref(false);
const errorMessage = ref('');
const now = ref(new Date());
const selectedPrayer = ref(null);
let clockInterval = null;

const todayReadable = computed(() =>
  now.value.toLocaleDateString('en-GB', { day: 'numeric', month: 'long', year: 'numeric' })
);

const togglePrayerDetail = (prayer) => {
  selectedPrayer.value = selectedPrayer.value?.key === prayer.key ? null : prayer;
};

const selectedPrayerStatus = computed(() => {
  if (!selectedPrayer.value || selectedPrayer.value.time === '--:--') return '';
  const [h, m] = selectedPrayer.value.time.split(':').map(Number);
  const target = new Date(now.value);
  target.setHours(h, m, 0, 0);
  const diffMinutes = Math.round((target - now.value) / 60000);
  if (diffMinutes === 0) return 'Ongoing now';
  if (diffMinutes > 0) {
    const hours = Math.floor(diffMinutes / 60);
    const mins = diffMinutes % 60;
    return hours > 0 ? `${hours}h ${mins}m left` : `${mins} minutes left`;
  }
  const passed = Math.abs(diffMinutes);
  const hours = Math.floor(passed / 60);
  const mins = passed % 60;
  return hours > 0 ? `${hours}h ${mins}m ago` : `${mins} minutes ago`;
});

const prayerList = computed(() => {
  if (!timings.value) return [];
  return Object.keys(PRAYER_LABELS).map((key) => ({
    key,
    label: PRAYER_LABELS[key],
    time: timings.value[key]?.substring(0, 5) || '--:--',
  }));
});

const nextPrayer = computed(() => {
  if (!prayerList.value.length) return { key: '', label: '-', time: '-' };
  const nowMinutes = now.value.getHours() * 60 + now.value.getMinutes();
  for (const prayer of prayerList.value) {
    const [h, m] = prayer.time.split(':').map(Number);
    if (h * 60 + m > nowMinutes) return prayer;
  }
  return prayerList.value[0];
});

const countdownText = computed(() => {
  if (!nextPrayer.value.time || nextPrayer.value.time === '-') return '';
  const [h, m] = nextPrayer.value.time.split(':').map(Number);
  const target = new Date(now.value);
  target.setHours(h, m, 0, 0);
  if (target <= now.value) target.setDate(target.getDate() + 1);
  const diffMinutes = Math.floor((target - now.value) / 60000);
  const hours = Math.floor(diffMinutes / 60);
  const mins = diffMinutes % 60;
  return hours > 0 ? `${hours}h ${mins}m left` : `${mins} minutes left`;
});

const fetchPrayerTimes = async () => {
  loading.value = true;
  errorMessage.value = '';
  selectedPrayer.value = null;
  try {
    const { city, country } = selectedCity.value;
    const response = await fetch(
      `${ALADHAN_API_URL}?city=${encodeURIComponent(city)}&country=${encodeURIComponent(country)}&method=20`
    );
    if (!response.ok) throw new Error('Failed to fetch prayer times.');
    const result = await response.json();
    if (result.code !== 200) throw new Error('Prayer times data unavailable.');
    timings.value = result.data.timings;
    const hijri = result.data.date.hijri;
    hijriDate.value = `${hijri.day} ${hijri.month.en} ${hijri.year} AH in ${selectedCity.value.label}, Indonesia`;
  } catch (error) {
    errorMessage.value = error.message || 'An error occurred while loading prayer times.';
  } finally {
    loading.value = false;
  }
};

onMounted(() => {
  fetchPrayerTimes();
  clockInterval = setInterval(() => { now.value = new Date(); }, 30000);
});

onUnmounted(() => { if (clockInterval) clearInterval(clockInterval); });
</script>

<style scoped>
/* ── Page ── */
.dashboard-page {
  min-height: 100vh;
  background: linear-gradient(160deg, #04241c 0%, #0a4536 60%, #0d5a45 100%);
  display: flex;
  flex-direction: column;
}

/* ── Navbar ── */
.navbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.9rem 2rem;
  background-color: rgba(4, 36, 28, 0.85);
  backdrop-filter: blur(8px);
  border-bottom: 1px solid #1c5945;
  position: sticky;
  top: 0;
  z-index: 10;
}

.navbar-brand {
  display: flex;
  align-items: center;
  gap: 10px;
  font-size: 1.1rem;
  font-weight: 600;
  color: #f4e9c1;
}

.navbar-right {
  display: flex;
  align-items: center;
  gap: 1rem;
}

.navbar-user {
  font-size: 0.9rem;
  color: #9fcdb8;
}

.logout-button {
  padding: 0.45rem 1rem;
  border: 1px solid #e53e3e;
  border-radius: 6px;
  background-color: transparent;
  color: #e53e3e;
  font-size: 0.85rem;
  cursor: pointer;
  transition: background-color 0.2s, color 0.2s;
}

.logout-button:hover {
  background-color: #e53e3e;
  color: #fff;
}

/* ── Main content ── */
.main-content {
  max-width: 860px;
  width: 100%;
  margin: 2rem auto;
  padding: 0 1.5rem;
}

/* ── Info bar ── */
.info-bar {
  display: flex;
  justify-content: space-between;
  align-items: flex-end;
  margin-bottom: 1.5rem;
  flex-wrap: wrap;
  gap: 1rem;
}

.info-title {
  margin: 0 0 0.25rem;
  font-size: 1.6rem;
  font-weight: 600;
  color: #f4e9c1;
}

.info-date {
  margin: 0;
  font-size: 0.9rem;
  color: #cfe8db;
}

.info-hijri {
  margin: 2px 0 0;
  font-size: 0.85rem;
  color: #9fcdb8;
}

.city-label {
  display: block;
  font-size: 0.8rem;
  color: #9fcdb8;
  margin-bottom: 4px;
}

.city-select {
  padding: 0.5rem 0.75rem;
  border: 1px solid #2e6b56;
  border-radius: 8px;
  background-color: #0d4a3a;
  color: #f4e9c1;
  font-size: 0.9rem;
  min-width: 160px;
}

/* ── Status ── */
.status-box {
  text-align: center;
  padding: 3rem 0;
  color: #cfe8db;
}

.status-box.error { color: #f0a3a3; }

.status-box button {
  margin-top: 0.75rem;
  padding: 0.5rem 1.25rem;
  border: none;
  border-radius: 8px;
  background-color: #d4af37;
  color: #07382c;
  font-weight: 600;
  cursor: pointer;
}

/* ── Next prayer banner ── */
.next-banner {
  display: flex;
  align-items: center;
  gap: 1rem;
  background-color: #0d4a3a;
  border: 1px solid #1c5945;
  border-left: 4px solid #d4af37;
  border-radius: 10px;
  padding: 1rem 1.5rem;
  margin-bottom: 1.25rem;
  flex-wrap: wrap;
}

.next-banner-label {
  font-size: 0.8rem;
  color: #d4af37;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

.next-banner-name {
  font-size: 1.2rem;
  font-weight: 600;
  color: #f4e9c1;
}

.next-banner-time {
  font-size: 1.2rem;
  color: #cfe8db;
}

.next-banner-countdown {
  margin-left: auto;
  font-size: 0.85rem;
  color: #9fcdb8;
}

/* ── Prayer grid 2 kolom ── */
.prayer-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 12px;
  margin-bottom: 1.25rem;
}

@media (min-width: 600px) {
  .prayer-grid {
    grid-template-columns: repeat(3, 1fr);
  }
}

.prayer-card {
  background-color: #0d4a3a;
  border: 1px solid #1c5945;
  border-radius: 10px;
  padding: 1.25rem 1rem;
  cursor: pointer;
  transition: transform 0.15s, box-shadow 0.15s, border-color 0.15s;
}

.prayer-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.3);
  border-color: #2e6b56;
}

.prayer-card.active {
  background-color: #d4af37;
  border-color: #d4af37;
}

.prayer-card.selected {
  border-color: #f4e9c1;
}

.prayer-card-name {
  margin: 0 0 6px;
  font-size: 0.8rem;
  color: #9fcdb8;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

.prayer-card.active .prayer-card-name { color: #4a3b0c; }

.prayer-card-time {
  margin: 0;
  font-size: 1.5rem;
  font-weight: 600;
  color: #f4e9c1;
}

.prayer-card.active .prayer-card-time { color: #07382c; }

/* ── Detail panel ── */
.detail-panel {
  background-color: #0d4a3a;
  border: 1px solid #d4af37;
  border-radius: 10px;
  padding: 1rem 1.5rem;
  display: flex;
  align-items: center;
  gap: 1rem;
  flex-wrap: wrap;
}

.detail-name {
  margin: 0;
  font-size: 1rem;
  font-weight: 600;
  color: #f4e9c1;
}

.detail-time {
  margin: 0;
  font-size: 1rem;
  color: #cfe8db;
}

.detail-status {
  margin: 0 0 0 auto;
  font-size: 0.85rem;
  color: #d4af37;
}

/* ── Transition ── */
.detail-fade-enter-active,
.detail-fade-leave-active {
  transition: opacity 0.2s, transform 0.2s;
}

.detail-fade-enter-from,
.detail-fade-leave-to {
  opacity: 0;
  transform: translateY(-6px);
}
</style>