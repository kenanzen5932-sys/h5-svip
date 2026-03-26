<template>
  <div class="svip-container" 
       @touchstart="handleTouchStart" 
       @touchmove="handleTouchMove" 
       @touchend="handleTouchEnd"
       :style="{
         '--bg-bottom': currentTheme.bottom,
         '--accent': currentTheme.accent,
         '--benefit-bg': currentTheme.benefitBg,
         backgroundImage: `url(${currentBgImage})`
       }">
    
    <!-- 1. Header: AppBar ve Title -->
    <header class="svip-header">
      <div class="icon-btn" @click="goBack">
        <i class="lucide-chevron-left"></i>
        <span class="back-arrow">‹</span>
      </div>
      <div class="header-title">
        <img src="https://via.placeholder.com/100x40/103a27/f1dd96?text=LOGO" alt="SVIP" class="svip-logo-text" v-if="false" />
        <h1 class="svip-title-text">SVIP</h1>
      </div>
      <div class="header-actions">
        <div class="action-icon" style="border: none;">
          <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none">
            <path
              d="M11.1673 14.9169H12.834V16.5836H11.1673V14.9169ZM12.834 13.1291L12.834 14.0836H11.1673L11.1673 12.4169C11.1673 12.1959 11.2551 11.9839 11.4114 11.8276C11.5677 11.6713 11.7796 11.5836 12.0007 11.5836C12.2374 11.5836 12.4692 11.5164 12.6692 11.3898C12.8693 11.2632 13.0292 11.0824 13.1305 10.8684C13.2318 10.6545 13.2703 10.4162 13.2414 10.1812C13.2126 9.94628 13.1176 9.72435 12.9676 9.54126C12.8175 9.35817 12.6186 9.22144 12.3939 9.14699C12.1692 9.07254 11.928 9.06342 11.6983 9.12069C11.4686 9.17797 11.2599 9.29928 11.0965 9.47053C10.933 9.64177 10.7195 9.91688 10.7195 10.2436L9.08398 9.91688C9.08398 9.50022 9.47472 8.78384 9.81752 8.39718C10.1603 8.01052 10.5997 7.72183 11.0906 7.56064C11.5816 7.39944 12.1065 7.37152 12.6118 7.47973C13.1171 7.58794 13.5846 7.82839 13.9665 8.17651C14.3483 8.52462 14.6309 8.96791 14.7853 9.46105C14.9397 9.95418 14.9604 10.4795 14.8452 10.9832C14.73 11.4869 14.4831 11.9511 14.1297 12.3281C13.7764 12.7051 13.3292 12.9816 12.834 13.1291Z"
              fill="#E1B780" />
            <circle cx="12" cy="12" r="8.4" stroke="#E1B780" stroke-width="1.2" />
          </svg>
        </div>
      </div>
    </header>

    <!-- 2. Profil Çizgisi (Profil, Bar) -->
    <div class="top-info-section">
      <div class="top-info-content">
        <div class="user-card">
          <img :src="userAvatar" class="avatar" alt="Avatar" />
          <div class="user-info">
            <div class="username">{{ userName }}</div>
          </div>
        </div>

        <!-- Hareketli PAG Madalyası (Sağ Üst) -->
        <div class="medal-container">
          <canvas :key="currentLevel" id="pag-medal" class="pag-medal" width="400" height="400"></canvas>
        </div>
      </div>

      <div class="progress-bar-wrapper">
        <div class="progress-bar-bg">
          <div class="progress-bar-fill" :style="{ width: progressPercent + '%' }"></div>
        </div>
        <div class="progress-text">
          SVIP {{ currentLevel }} Olmak İçin <span class="highlight">{{ remainingCoins }}</span> Daha Yükle
        </div>
      </div>
      <!-- SVIP Geçiş Çubuğu (Noktalar) -->
      <div class="levels-slider-container">
        <div class="levels-track">
          <div class="level-indicator">
            <!-- Active Line + Dots -->
            <div class="progress-line">
              <div class="progress-fill" :style="{ width: ((currentLevel - 1) / 9) * 100 + '%' }"></div>
            </div>
            <div class="dots-container">
              <div 
                v-for="n in 10" 
                :key="n" 
                class="dot-wrapper"
                :class="{ 'active': currentLevel === n, 'past': n < currentLevel }"
                @click="goToLevel(n)"
              >
                <div class="dot"></div>
                <span class="dot-label">SVIP{{ n }}</span>
              </div>
            </div>
          </div>
        </div>
      </div>

    </div> <!-- top-info-section end -->

    <!-- 3. Alt Kısım: SVIP Ayrıcalıkları -->
    <div class="benefits-section" style="margin-bottom: 30px;">
      
      <div class="benefits-subtitle">
        <span class="deco">〰️</span> SVIP Ayrıcalıkları <span class="deco">〰️</span>
      </div>

      <!-- Yükleniyor -->
      <div v-if="isLoading" class="loading-state">
        <span>Yükleniyor...</span>
      </div>

      <!-- Ayrıcalık Listesi -->
      <div v-else-if="currentBenefits.length > 0" class="benefits-list">
        <div class="benefit-card" v-for="item in currentBenefits" :key="item.id">
          <div class="benefit-icon-wrapper">
            <!-- URL ise resim, emoji ise metin olarak göster -->
            <img v-if="item.icon && item.icon.startsWith('http')" :src="item.icon" alt="Benefit" />
            <span v-else class="benefit-emoji">{{ item.icon }}</span>
          </div>
          <div class="benefit-name">{{ item.name }}</div>
        </div>
      </div>

      <!-- Veri yoksa bilgi mesajı -->
      <div v-else class="empty-benefits">
        <span>Bu seviye için henüz ayrıcalık eklenmemiş.</span>
      </div>

    </div>

  </div>
</template>

<script setup>
import { ref, computed, watch, onMounted, onBeforeUnmount, nextTick } from 'vue'
import { PAGInit } from 'libpag'
import { supabase } from './supabase.js'

// ─── Mevcut Seviye (Swipe ile değişir) ───────────────────────────────────────
const currentLevel = ref(1)

// ─── Dinamik Renk Temaları ────────────────────────────────────────────────────
const svipThemes = {
  1: { top: '#1a4f3b', bottom: '#030e0a', accent: '#f1dd96', benefitBg: '#0f2d1e' },
  2: { top: '#013e2e', bottom: '#02120e', accent: '#6ee7b7', benefitBg: '#052a20' }, 
  3: { top: '#0c356a', bottom: '#041224', accent: '#60a5fa', benefitBg: '#082346' }, 
  4: { top: '#3b1c6b', bottom: '#150826', accent: '#c084fc', benefitBg: '#2a144e' }, 
  5: { top: '#45060b', bottom: '#140103', accent: '#fca5a5', benefitBg: '#300408' }, 
  6: { top: '#3b2f02', bottom: '#1a1401', accent: '#fde047', benefitBg: '#2a2002' }, 
  7: { top: '#4a1e05', bottom: '#1c0901', accent: '#fdbf84', benefitBg: '#331201' }, 
  8: { top: '#1c1b18', bottom: '#050403', accent: '#eab308', benefitBg: '#11100f' }, 
  9: { top: '#4a330a', bottom: '#1c1103', accent: '#fbbf24', benefitBg: '#211603' }, 
  10: { top: '#081c33', bottom: '#020912', accent: '#b0d1fb', benefitBg: '#051121' },
}

const currentTheme = computed(() => svipThemes[currentLevel.value] || svipThemes[1])

// ─── Arka Plan Görseli ────────────────────────────────────────────────────────
const currentBgImage = computed(() => {
  return new URL(`./assets/SVIP/Backrounds/SVIP${currentLevel.value}ARKAPLAN.png`, import.meta.url).href;
})

// ─── PAG Madalya Animasyonu ───────────────────────────────────────────────────

const currentPagSrc = computed(() => {
  const fileName = currentLevel.value === 9 ? 'SVIP9 MADALYA.pag' : `SVIP${currentLevel.value}MADALYA.pag`;
  return `/SVIPMEDALS/${fileName}`;
})

let PAG = null;
let currentPagView = null;
let pagLoadId = 0; // Her yükleme çağrısına benzersiz ID — race condition önleyici

const destroyCurrentPag = () => {
  if (currentPagView) {
    try { if (typeof currentPagView.pause === 'function') currentPagView.pause() } catch(e) {}
    try { if (typeof currentPagView.destroy === 'function') currentPagView.destroy() } catch(e) {}
    currentPagView = null;
  }
}

const loadPagAnimation = async (src) => {
  if (!src) return;
  const thisLoadId = ++pagLoadId; // Bu çağrının ID'si
  try {
    if (!PAG) {
      PAG = await PAGInit({ locateFile: (file) => `https://unpkg.com/libpag@latest/lib/${file}` });
    }
    if (thisLoadId !== pagLoadId) return; // Eski çağrı — iptal
    const response = await fetch(src);
    if (!response.ok || thisLoadId !== pagLoadId) return;
    const buffer = await response.arrayBuffer();
    if (thisLoadId !== pagLoadId) return; // Hızlı geçişte hâlâ geçerli mi?
    const pagFile = await PAG.PAGFile.load(buffer);
    if (thisLoadId !== pagLoadId) return;
    const canvas = document.getElementById('pag-medal');
    if (!canvas || thisLoadId !== pagLoadId) return;
    destroyCurrentPag();
    currentPagView = await PAG.PAGView.init(pagFile, canvas);
    if (thisLoadId !== pagLoadId) { destroyCurrentPag(); return; }
    currentPagView.setRepeatCount(0);
    await currentPagView.play();
  } catch (error) {
    if (thisLoadId === pagLoadId) console.error('PAG yükleme hatası:', error);
  }
}

watch(currentLevel, async () => {
  destroyCurrentPag();
  await nextTick();
  loadPagAnimation(currentPagSrc.value);
})

// ─── Kullanıcı Verileri ───────────────────────────────────────────────────────
const userName = ref('...')
const userAvatar = ref('https://i.pravatar.cc/150?img=11')
const userSvipPoints = ref(0)      // Kullanıcının toplam SVIP puanı
const userSvipLevel = ref(1)       // Kullanıcının gerçek SVIP seviyesi

// ─── SVIP Seviye Tablosu (Supabase'den) ──────────────────────────────────────
const svipLevelData = ref([])  // [{level, required_points, return_points, has_colored_name, ...}]
const isLoading = ref(true)

// Şu anki seviyenin gereken puanı
const currentLevelRequired = computed(() => {
  const data = svipLevelData.value.find(l => l.level === currentLevel.value)
  return data ? data.required_points : 0
})

// Kullanıcının bu seviyeye olan ilerleme yüzdesi
const progressPercent = computed(() => {
  if (!currentLevelRequired.value) return 0
  const pct = (userSvipPoints.value / currentLevelRequired.value) * 100
  return Math.min(pct, 100)
})

// Kalan puan (kısaltılmış)
const remainingCoins = computed(() => {
  const rem = Math.max(0, currentLevelRequired.value - userSvipPoints.value)
  if (rem >= 1000000000) return (rem / 1000000000).toFixed(1) + 'B'
  if (rem >= 1000000) return (rem / 1000000).toFixed(0) + 'M'
  if (rem >= 1000) return (rem / 1000).toFixed(0) + 'K'
  return rem.toString()
})

// ─── SVIP Ayrıcalıkları (Supabase'den) ────────────────────────────────────────
const allLevelBenefits = ref({})

const currentBenefits = computed(() => allLevelBenefits.value[currentLevel.value] || [])

const levelToBenefits = (levelRow) => {
  const benefits = []
  if (levelRow.has_colored_name) benefits.push({ id: 'col_name', name: 'Renkli İsim', icon: '🎨', type: 'feature' })
  if (levelRow.has_anti_kick)    benefits.push({ id: 'anti_kick', name: 'Ban Koruması', icon: '🛡️', type: 'feature' })
  if (levelRow.can_upload_gif)   benefits.push({ id: 'gif', name: 'GIF Profil', icon: '🎬', type: 'feature' })
  if (levelRow.can_hide_online)  benefits.push({ id: 'hide', name: 'Gizli Giriş', icon: '🕵️', type: 'feature' })
  return benefits
}

const loadSvipData = async () => {
  isLoading.value = true
  try {
    const { data: levels, error: levelsErr } = await supabase
      .from('svip_levels').select('*').order('level')
    if (levelsErr) throw levelsErr
    svipLevelData.value = levels

    const { data: rewards, error: rewardsErr } = await supabase
      .from('svip_rewards')
      .select('level, store_items(id, name, thumbnail_url, type)')
    if (rewardsErr) throw rewardsErr

    const benefitsMap = {}
    for (const levelRow of levels) {
      const lvl = levelRow.level
      benefitsMap[lvl] = levelToBenefits(levelRow)
      const levelRewards = (rewards || []).filter(r => r.level === lvl)
      for (const reward of levelRewards) {
        if (reward.store_items) {
          benefitsMap[lvl].push({
            id: reward.store_items.id,
            name: reward.store_items.name,
            icon: reward.store_items.thumbnail_url || '',
            type: reward.store_items.type
          })
        }
      }
      // Sadece veritabanındaki verileri kullan, fake item ekleme
    }
    allLevelBenefits.value = benefitsMap
  } catch (err) {
    console.error('SVIP veri hatası:', err)
  } finally {
    isLoading.value = false
  }
}

const loadUserData = async () => {
  const urlParams = new URLSearchParams(window.location.search)
  const userId = urlParams.get('user_id')
  if (!userId) {
    userName.value = 'NEVER'
    userSvipPoints.value = 0
    userSvipLevel.value = 1
    return
  }
  const { data, error } = await supabase
    .from('profiles')
    .select('username, avatar_url, svip_points')
    .eq('id', userId).single()
  if (error || !data) return
  userName.value = data.username || 'NEVER'
  userAvatar.value = data.avatar_url || 'https://i.pravatar.cc/150?img=11'
  userSvipPoints.value = data.svip_points || 0
  let computedLevel = 1
  for (const lvl of svipLevelData.value) {
    if (userSvipPoints.value >= lvl.required_points) computedLevel = lvl.level
  }
  userSvipLevel.value = computedLevel
  currentLevel.value = computedLevel
}

onMounted(async () => {
  await loadSvipData()
  await loadUserData()
  loadPagAnimation(currentPagSrc.value)
})

onBeforeUnmount(() => {
  if (currentPagView && typeof currentPagView.destroy === 'function') currentPagView.destroy()
})

// ─── Swipe Algoritması ────────────────────────────────────────────────────────
let touchStartX = 0
let touchEndX = 0
let isSwiping = false

const handleTouchStart = (e) => {
  touchStartX = e.touches[0].clientX
  touchEndX = touchStartX  // Aynı noktada başlat — kısa dokunuşta delta=0 olur
  isSwiping = false
}
const handleTouchMove = (e) => {
  touchEndX = e.touches[0].clientX
  // En az 20px hareket ettiyse gerçek bir swipe
  if (Math.abs(touchEndX - touchStartX) > 20) isSwiping = true
}
const handleTouchEnd = () => {
  if (!isSwiping) return  // Kısa dokunuş — hiçbir şey yapma
  const delta = touchStartX - touchEndX
  if (delta > 50) { if (currentLevel.value < 10) currentLevel.value++ }
  if (delta < -50) { if (currentLevel.value > 1) currentLevel.value-- }
}

const goToLevel = (lvl) => { currentLevel.value = lvl }

const goBack = () => {
  if (window.flutter_inappwebview) {
    window.flutter_inappwebview.callHandler('H5Message', 'close_h5')
  } else {
    console.log('Back button clicked')
  }
}
</script>

<style scoped>
.svip-container {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  width: 100%;
  position: relative;
  background-size: cover;
  background-position: top center;
  background-attachment: fixed;
  transition: background-image 0.4s ease-in-out;
}

/* Header */
.svip-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: max(30px, env(safe-area-inset-top)) 20px 10px 20px;
  z-index: 10;
}
.icon-btn {
  font-size: 24px;
  cursor: pointer;
  padding: 5px;
}
.back-arrow {
  font-size: 32px;
  line-height: 1;
}
.header-title {
  display: flex;
  align-items: center;
  justify-content: center;
}
.svip-title-text {
  font-style: italic;
  font-weight: 900;
  font-size: 28px;
  color: var(--accent);
  text-shadow: 2px 2px 4px rgba(0,0,0,0.5);
  letter-spacing: 2px;
}
.header-actions {
  display: flex;
  gap: 15px;
  font-size: 20px;
}
.action-icon {
  width: 30px;
  height: 30px;
  border-radius: 50%;
  border: 1px solid rgba(255, 255, 255, 0.2);
  display: flex;
  align-items: center;
  justify-content: center;
  color: var(--accent);
}

/* Üst Profil Alanı */
.top-info-section {
  padding: 0 20px;
  margin-top: 40px; 
}
.top-info-content {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
.user-card {
  display: flex;
  align-items: center;
  gap: 15px;
}
.medal-container {
  width: 200px;
  height: 200px;
  display: flex;
  justify-content: center;
  align-items: center;
  margin-top: -80px;
  margin-right: -10px;
}
.pag-medal {
  width: 100% !important;
  height: 100% !important;
  object-fit: contain;
}
.avatar {
  width: 70px;
  height: 70px;
  border-radius: 50%;
  border: none; /* Mavi çerçeve kökünden yok edildi */
}
.user-info {
  display: flex;
  flex-direction: column;
}
.username {
  font-weight: bold;
  font-size: 20px;
  letter-spacing: 0.5px;
}

.progress-bar-wrapper {
  margin-top: 65px; /* 2 tık (30px) daha aşağı alındı */
}
.progress-bar-bg {
  width: 100%;
  height: 4px;
  background: rgba(255,255,255,0.2);
  border-radius: 2px;
  overflow: hidden;
}
.progress-bar-fill {
  height: 100%;
  background: var(--accent);
  border-radius: 2px;
  transition: width 0.3s ease, background 0.3s ease;
}
.progress-text {
  font-size: 11px;
  color: #aaa;
  margin-top: 8px;
}
.highlight {
  color: white;
  font-weight: bold;
  font-size: 13px;
}

/* Seviye Çizgisi */
.levels-slider-container {
  padding: 10px 20px 20px 20px;
  position: relative;
  width: 100%;
}
.levels-track {
  width: 100%;
}
.level-indicator {
  position: relative;
  width: 100%;
  height: 40px;
  display: flex;
  align-items: center;
}
.progress-line {
  position: absolute;
  top: 50%;
  left: 20px;
  right: 20px;
  height: 2px;
  background: rgba(255, 255, 255, 0.2);
  transform: translateY(-50%);
  z-index: 1;
}
.progress-fill {
  height: 100%;
  background: var(--accent);
  transition: width 0.3s ease, background 0.3s ease;
}
.dots-container {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0 20px;
  z-index: 2;
}
.dot-wrapper {
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  cursor: pointer;
}
.dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: #666;
  transition: 0.3s;
}
.dot-wrapper.active .dot {
  background: var(--accent);
  box-shadow: 0 0 10px var(--accent);
  transform: scale(1.5);
}
.dot-wrapper.past .dot {
  background: var(--accent);
}
.dot-label {
  position: absolute;
  top: 15px;
  font-size: 10px;
  font-weight: bold;
  opacity: 0;
  color: var(--accent);
  transition: 0.3s;
  white-space: nowrap;
}
.dot-wrapper.active .dot-label {
  opacity: 1;
  top: 20px;
  font-size: 14px;
}

/* Özellikler Alt Bölüm */
.benefits-section {
  background: transparent;
  padding: 10px 20px 20px 20px;
  padding-bottom: env(safe-area-inset-bottom, 40px);
  width: 100%;
}

/* Açıklama vs */
.benefits-subtitle {
  text-align: center;
  font-size: 13px;
  color: #a0a0a0;
  margin-bottom: 20px;
}
.deco { font-size: 10px; color: #555; }

.benefits-list {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(100px, 1fr));
  gap: 15px;
  padding-bottom: 20px;
}
.benefit-card {
  display: flex;
  flex-direction: column;
  align-items: center;
  background: transparent;
  padding: 15px;
  border-radius: 12px;
  border: 1px solid rgba(255, 255, 255, 0.2);
  transition: transform 0.2s, background 0.3s;
}
.benefit-card:hover {
  background: rgba(255, 255, 255, 0.1);
}
.benefit-card:active {
  transform: scale(0.95);
}
.benefit-icon-wrapper {
  width: 60px;
  height: 60px;
  margin-bottom: 10px;
}
.benefit-icon-wrapper img {
  width: 100%;
  height: 100%;
  object-fit: contain;
}
.benefit-name {
  font-size: 12px;
  color: #ccc;
  text-align: center;
  line-height: 1.3;
}
.benefit-emoji {
  font-size: 40px;
  line-height: 60px;
  display: block;
  text-align: center;
}
.loading-state, .empty-benefits {
  text-align: center;
  padding: 30px;
  color: #ccc;
  font-size: 14px;
  font-style: italic;
  width: 100%;
}
</style>
