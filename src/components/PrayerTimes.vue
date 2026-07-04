<template>
  <div class="prayer-times">
    <div class="prayer-times-header">
      <div>
        <h2 class="section-title">Prayer Times</h2>
        <p class="section-subtitle" v-if="hijriDate">{{ hijriDate }}</p>
      </div>
      <div class="city-picker">
        <label for="city">City</label>
        <select id="city" v-model="selectedCity" @change="fetchPrayerTimes">
          <option v-for="c in CITY_LIST" :key="c.city" :value="c">{{ c.label }}</option>
        </select>
      </div>
    </div>

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
        <span class="next-label">Next Prayer Time</span>
        <span class="next-name">{{ nextPrayer.label }}</span>
        <span class="next-time">{{ nextPrayer.time }}</span>
        <span class="next-countdown" v-if="countdownText">{{ countdownText }}</span>
      </div>

      <!-- 5 Waktu Sholat Utama -->
      <p class="grid-label">Daily Prayers Time</p>
      <div class="prayer-grid">
        <div
          v-for="prayer in prayerList"
          :key="prayer.key"
          class="prayer-card"
          :class="{ active: prayer.key === nextPrayer.key, selected: prayer.key === selectedPrayer?.key }"
          @click="toggleDetail(prayer)"
        >
          <p class="prayer-card-name">{{ prayer.label }}</p>
          <p class="prayer-card-time">{{ prayer.time }}</p>
        </div>
      </div>

      <!-- Detail panel -->
      <Transition name="fade">
        <div v-if="selectedPrayer" class="detail-panel">
          <span class="detail-name">{{ selectedPrayer.label }}</span>
          <span class="detail-time">{{ selectedPrayer.time }}</span>
          <span class="detail-status">{{ selectedPrayerStatus }}</span>
        </div>
      </Transition>

      <!-- Waktu Tambahan -->
      <p class="grid-label" style="margin-top: 1.25rem;">Additional Times</p>
      <div class="extra-grid">
        <div v-for="extra in extraList" :key="extra.key" class="extra-card">
          <p class="extra-name">{{ extra.label }}</p>
          <p class="extra-time">{{ extra.time }}</p>
        </div>
      </div>
    </template>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue';

const API_BASE_URL = 'http://localhost:3001/api';

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

const EXTRA_LABELS = {
  Imsak: 'Imsak',
  Sunrise: 'Sunrise',
  Midnight: 'Midnight',
  Lastthird: 'Tahajjud',
};

const selectedCity = ref(CITY_LIST[0]);
const timings = ref(null);
const hijriDate = ref('');
const loading = ref(false);
const errorMessage = ref('');
const now = ref(new Date());
const selectedPrayer = ref(null);
let clockInterval = null;

const toggleDetail = (prayer) => {
  selectedPrayer.value = selectedPrayer.value?.key === prayer.key ? null : prayer;
};

const prayerList = computed(() => {
  if (!timings.value) return [];
  return Object.keys(PRAYER_LABELS).map((key) => ({
    key,
    label: PRAYER_LABELS[key],
    time: timings.value[key]?.substring(0, 5) || '--:--',
  }));
});


const extraList = computed(() => {
  if (!timings.value) return [];
  return Object.keys(EXTRA_LABELS).map((key) => ({
    key,
    label: EXTRA_LABELS[key],
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

const fetchPrayerTimes = async () => {
  loading.value = true;
  errorMessage.value = '';
  selectedPrayer.value = null;
  try {
    const { city, country } = selectedCity.value;
    const response = await fetch(
      `${API_BASE_URL}/prayer-times?city=${encodeURIComponent(city)}&country=${encodeURIComponent(country)}`
    );
    if (!response.ok) throw new Error('Failed to fetch prayer times.');
    const data = await response.json();
    timings.value = data.timings;
    hijriDate.value = `${data.hijri.day} ${data.hijri.month} ${data.hijri.year} AH — ${selectedCity.value.label}, Indonesia`;
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
.prayer-times {
  background-color: #07382c;
  border: 1px solid #1c5945;
  border-radius: 16px;
  padding: 1.5rem;
}

.prayer-times-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 1.25rem;
  flex-wrap: wrap;
  gap: 1rem;
}

.section-title {
  margin: 0 0 4px;
  font-size: 1.1rem;
  font-weight: 600;
  color: #f4e9c1;
}

.section-subtitle {
  margin: 0;
  font-size: 0.8rem;
  color: #9fcdb8;
}

.city-picker label {
  display: block;
  font-size: 0.75rem;
  color: #9fcdb8;
  margin-bottom: 4px;
}

.city-picker select {
  padding: 0.45rem 0.75rem;
  border: 1px solid #2e6b56;
  border-radius: 8px;
  background-color: #0d4a3a;
  color: #f4e9c1;
  font-size: 0.85rem;
  min-width: 140px;
}

.status-box {
  text-align: center;
  padding: 2rem 0;
  color: #cfe8db;
}

.status-box.error { color: #f0a3a3; }

.status-box button {
  margin-top: 0.5rem;
  padding: 0.4rem 1rem;
  border: none;
  border-radius: 8px;
  background-color: #d4af37;
  color: #07382c;
  font-weight: 600;
  cursor: pointer;
}

.next-banner {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  background-color: #0d4a3a;
  border: 1px solid #1c5945;
  border-left: 4px solid #d4af37;
  border-radius: 10px;
  padding: 0.9rem 1.25rem;
  margin-bottom: 1rem;
  flex-wrap: wrap;
}

.next-label {
  font-size: 0.7rem;
  color: #d4af37;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

.next-name {
  font-size: 1rem;
  font-weight: 600;
  color: #f4e9c1;
}

.next-time {
  font-size: 1rem;
  color: #cfe8db;
}

.next-countdown {
  margin-left: auto;
  font-size: 0.8rem;
  color: #9fcdb8;
}

.grid-label {
  margin: 0 0 0.6rem;
  font-size: 0.75rem;
  color: #9fcdb8;
  text-transform: uppercase;
  letter-spacing: 0.06em;
}

/* Grid 5 kolom untuk 5 waktu sholat utama */
.prayer-grid {
  display: grid;
  grid-template-columns: repeat(5, 1fr);
  gap: 8px;
  margin-bottom: 0.75rem;
}

.prayer-card {
  background-color: #0d4a3a;
  border: 1px solid #1c5945;
  border-radius: 10px;
  padding: 0.9rem 0.5rem;
  text-align: center;
  cursor: pointer;
  transition: transform 0.15s, border-color 0.15s;
}

.prayer-card:hover {
  transform: translateY(-2px);
  border-color: #2e6b56;
}

.prayer-card.active { background-color: #d4af37; border-color: #d4af37; }
.prayer-card.selected { border-color: #f4e9c1; }

.prayer-card-name {
  margin: 0 0 5px;
  font-size: 0.65rem;
  color: #9fcdb8;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

.prayer-card.active .prayer-card-name { color: #4a3b0c; }

.prayer-card-time {
  margin: 0;
  font-size: 1rem;
  font-weight: 600;
  color: #f4e9c1;
}

.prayer-card.active .prayer-card-time { color: #07382c; }

.detail-panel {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  background-color: #0d4a3a;
  border: 1px solid #d4af37;
  border-radius: 10px;
  padding: 0.75rem 1.25rem;
  flex-wrap: wrap;
  margin-bottom: 0.5rem;
}

.detail-name { font-size: 0.9rem; font-weight: 600; color: #f4e9c1; }
.detail-time { font-size: 0.9rem; color: #cfe8db; }
.detail-status { margin-left: auto; font-size: 0.8rem; color: #d4af37; }

/* Grid 4 kolom untuk waktu tambahan */
.extra-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 8px;
}

.extra-card {
  background-color: #0d4a3a;
  border: 1px solid #1c5945;
  border-radius: 10px;
  padding: 0.75rem 0.5rem;
  text-align: center;
}

.extra-name {
  margin: 0 0 4px;
  font-size: 0.65rem;
  color: #9fcdb8;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

.extra-time {
  margin: 0;
  font-size: 0.95rem;
  font-weight: 600;
  color: #cfe8db;
}

.fade-enter-active, .fade-leave-active { transition: opacity 0.2s, transform 0.2s; }
.fade-enter-from, .fade-leave-to { opacity: 0; transform: translateY(-4px); }
</style>