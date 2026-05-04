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
      </div>
      <div class="header-actions">
        <div class="action-icon" style="border: none; cursor: pointer;" @click="showInfo = true">
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

      <!-- Kilit Aç / Zaten Açık Durumu -->
      <div class="unlock-section">
        <div v-if="userSvipLevel >= currentLevel" class="unlock-status unlocked">
          <span>SVIP {{ currentLevel }} Açık</span>
        </div>
        <div v-else class="unlock-status locked">
          <div class="unlock-info">
            <span class="unlock-price">{{ formatCoins(diffCost) }} Coin</span>
            <span class="unlock-label">ile SVIP {{ currentLevel }} Aç</span>
          </div>
          <button 
            class="unlock-btn" 
            :class="{ 'disabled': userCoins < diffCost || isUnlocking }"
            :disabled="userCoins < diffCost || isUnlocking"
            @click="unlockLevel"
          >
            <span v-if="isUnlocking">Açılıyor...</span>
            <span v-else-if="userCoins < diffCost">Yetersiz Bakiye</span>
            <span v-else>Kilidini Aç</span>
          </button>
          <div class="coin-balance">Bakiyen: {{ formatCoins(userCoins) }} Coin</div>
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

  <!-- Bilgilendirme Tam Sayfa -->
  <div v-if="showInfo" class="info-page">
    <header class="info-page-header">
      <div class="info-back-btn" @click="showInfo = false">
        <span class="back-arrow">&#8249;</span>
      </div>
      <span class="info-page-title">SVIP Nedir?</span>
      <div style="width: 40px;"></div>
    </header>
    <div class="info-page-body">
      <p class="info-section-title">SVIP Nasil Acilir?</p>
      <p>SVIP seviyelerini Coin harcayarak kalici olarak acabilirsin. Her seviye bir onceki seviyenin ustune ek ucret gerektirir.</p>

      <p class="info-section-title">Fiyatlar</p>
      <div class="info-price-list">
        <div class="info-price-row" v-for="lvl in svipLevelData" :key="lvl.level">
          <span class="info-level">SVIP {{ lvl.level }}</span>
          <span class="info-cost">{{ formatCoins(lvl.required_points) }} Coin</span>
        </div>
      </div>

      <p class="info-section-title">Onemli Bilgiler</p>
      <ul class="info-list">
        <li>SVIP seviyeleri kalicidir, suresi dolmaz.</li>
        <li>Sadece bir ust seviyeye gecis yapilabilir.</li>
        <li>Her seviyede ozel hediyeler ve ayricaliklar kazanirsin.</li>
        <li>Coin bakiyen yetersizse once Coin yuklemen gerekir.</li>
        <li>Seviye dusme sistemi yoktur, kazandigin seviye senindir.</li>
      </ul>

      <p class="info-section-title">SVIP Ayricaliklari</p>
      <p>Her SVIP seviyesinde ozel cerceveler, giris efektleri, renkli isim ve daha fazla ozellik acilir. Seviye arttikca ayricaliklar da artar.</p>
    </div>
  </div>

  <transition name="toast-fade">
    <div v-if="toastVisible" class="svip-toast">{{ toastMessage }}</div>
  </transition>
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
      PAG = await PAGInit({ locateFile: (file) => `https://cdn.jsdelivr.net/npm/libpag@4.5.42/lib/${file}` });
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
const userCoins = ref(0)           // Kullanıcının coin bakiyesi
const userSvipLevel = ref(0)       // Kullanıcının gerçek SVIP seviyesi
const isUnlocking = ref(false)     // Kilit açma işlemi devam ediyor mu
const showInfo = ref(false)        // Bilgilendirme modal'ı açık mı

// ─── SVIP Seviye Tablosu (Supabase'den) ──────────────────────────────────────
const svipLevelData = ref([])  // [{level, required_points, ...}]
const isLoading = ref(true)

// Fark ücreti: hedef seviye fiyatı - mevcut seviye fiyatı
const diffCost = computed(() => {
  const targetData = svipLevelData.value.find(l => l.level === currentLevel.value)
  const currentData = svipLevelData.value.find(l => l.level === userSvipLevel.value)
  const targetCost = targetData ? targetData.required_points : 0
  const currentCost = currentData ? currentData.required_points : 0
  return Math.max(0, targetCost - currentCost)
})

// Toast sistemi
const toastMessage = ref('')
const toastVisible = ref(false)
let toastTimer = null
const showToast = (msg) => {
  toastMessage.value = msg
  toastVisible.value = true
  if (toastTimer) clearTimeout(toastTimer)
  toastTimer = setTimeout(() => { toastVisible.value = false }, 2500)
}

// Coin formatlama
const formatCoins = (num) => {
  if (num >= 1000000000) return (num / 1000000000).toFixed(1) + 'B'
  if (num >= 1000000) return (num / 1000000).toFixed(0) + 'M'
  if (num >= 1000) return (num / 1000).toFixed(0) + 'K'
  return num.toString()
}

// Kilit açma fonksiyonu
const unlockLevel = async () => {
  if (isUnlocking.value || userCoins.value < diffCost.value) return
  if (currentLevel.value !== userSvipLevel.value + 1) {
    showToast(`Önce SVIP ${userSvipLevel.value + 1} seviyesini açmalısın.`)
    return
  }
  isUnlocking.value = true
  try {
    const { data, error } = await supabase.rpc('unlock_svip_level', { p_target_level: currentLevel.value })
    if (error) throw error
    if (data && data.success) {
      userSvipLevel.value = data.new_level
      userCoins.value = data.remaining_coins
      showToast(`SVIP ${data.new_level} başarıyla açıldı!`)
    } else {
      showToast(data?.message || 'Bir hata oluştu.')
    }
  } catch (err) {
    console.error('Kilit açma hatası:', err)
    showToast('İşlem başarısız oldu.')
  } finally {
    isUnlocking.value = false
  }
}

// ─── SVIP Ayrıcalıkları (Supabase'den) ────────────────────────────────────────
const allLevelBenefits = ref({})

const currentBenefits = computed(() => allLevelBenefits.value[currentLevel.value] || [])

const loadSvipData = async () => {
  isLoading.value = true
  try {
    const { data: levels, error: levelsErr } = await supabase
      .from('svip_levels').select('*').order('level')
    if (levelsErr) throw levelsErr
    svipLevelData.value = levels

    // Hediye eşyalar - svip_rewards_full view RLS bypass eder, gizli eşyalar da gelir
    const { data: rewards, error: rewardsErr } = await supabase
      .from('svip_rewards_full')
      .select('level, item_id, item_name, thumbnail_url, item_type')
    if (rewardsErr) throw rewardsErr

    // Dinamik ayrıcalıklar - svip_benefits tablosundan
    const { data: svipBenefits } = await supabase
      .from('svip_benefits')
      .select('level, name, icon_url')
      .order('id')

    const benefitsMap = {}
    for (const levelRow of levels) {
      const lvl = levelRow.level
      benefitsMap[lvl] = []

      // 1) Hediye eşyalar (svip_rewards) — ÖNCE ÜSTTE
      const levelRewards = (rewards || []).filter(r => r.level === lvl)
      for (const reward of levelRewards) {
        if (reward.item_id) {
          benefitsMap[lvl].push({
            id: reward.item_id,
            name: reward.item_name,
            icon: reward.thumbnail_url || '',
            type: reward.item_type
          })
        }
      }

      // 2) Ayrıcalıklar (svip_benefits) — ALTTA SONDA
      const lvlBenefits = (svipBenefits || []).filter(b => b.level === lvl)
      for (const b of lvlBenefits) {
        benefitsMap[lvl].push({ id: `b_${b.name}`, name: b.name, icon: b.icon_url || '✨', type: 'feature' })
      }
    }
    allLevelBenefits.value = benefitsMap
  } catch (err) {
    console.error('SVIP veri hatası:', err)
  } finally {
    isLoading.value = false
  }
}

let userId = null

const loadUserData = async () => {
  if (!userId) {
    userName.value = 'GÜVENLİK HATASI'
    userCoins.value = 0
    userSvipLevel.value = 0
    return
  }
  const { data, error } = await supabase
    .from('profiles')
    .select('username, avatar_url, coins, vip_level, vip_type')
    .eq('id', userId).single()
  if (error || !data) return
  userName.value = data.username || 'NEVER'
  userAvatar.value = data.avatar_url || 'https://i.pravatar.cc/150?img=11'
  userCoins.value = data.coins || 0
  userSvipLevel.value = (data.vip_type === 'svip') ? (data.vip_level || 0) : 0
  currentLevel.value = Math.max(1, userSvipLevel.value > 0 ? userSvipLevel.value : 1)
}

onMounted(async () => {
  // GÜVENLİK: URL param yerine Flutter InAppWebView içinden gizlice alınmalı
  if (!window.flutter_inappwebview) {
    alert("🔒 Güvenlik İhlali: Lütfen bu sayfayı Zabu Uygulaması içinden açın.")
    return
  }

  try {
    const auth = await window.flutter_inappwebview.callHandler('getSupabaseAuth')
    if (auth && auth.uuid) {
      userId = auth.uuid
      if (auth.token) {
        await supabase.auth.setSession({ access_token: auth.token, refresh_token: auth.token })
      }
    } else {
      alert("Oturum bilgileri alınamadı.")
      return
    }
  } catch (e) {
    console.error("Auth bridge hatası:", e)
  }

  await loadSvipData()
  await loadUserData()
  loadPagAnimation(currentPagSrc.value)

  // REALTIME: Coin ve SVIP seviye güncellemelerini anında ekrana yansıt
  supabase.channel('svip_sub_' + userId)
    .on('postgres_changes', { 
        event: 'UPDATE', 
        schema: 'public', 
        table: 'profiles', 
        filter: `id=eq.${userId}` 
      }, (payload) => {
        const newCoins = payload.new.coins
        const newLevel = payload.new.vip_level
        const newType = payload.new.vip_type
        if (newCoins !== undefined) userCoins.value = newCoins
        if (newLevel !== undefined && newType === 'svip') {
          userSvipLevel.value = newLevel
          currentLevel.value = Math.max(1, newLevel)
        }
    }).subscribe()
})

onBeforeUnmount(() => {
  if (currentPagView && typeof currentPagView.destroy === 'function') currentPagView.destroy()
  if (toastTimer) clearTimeout(toastTimer)
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
  min-height: 80px;
}
.user-card {
  display: flex;
  align-items: center;
  gap: 12px;
  min-width: 0;
  flex: 1;
  overflow: hidden;
}
.medal-container {
  width: clamp(120px, 30vw, 200px);
  height: clamp(120px, 30vw, 200px);
  display: flex;
  justify-content: center;
  align-items: center;
  margin-top: -40px;
  margin-right: -10px;
  flex-shrink: 0;
}
.pag-medal {
  width: 100% !important;
  height: 100% !important;
  object-fit: contain;
}
.avatar {
  width: 56px;
  height: 56px;
  border-radius: 50%;
  border: none;
  flex-shrink: 0;
}
.user-info {
  display: flex;
  flex-direction: column;
  min-width: 0;
  overflow: hidden;
}
.username {
  font-weight: bold;
  font-size: 18px;
  letter-spacing: 0.5px;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  max-width: 100%;
}

/* Kilit Açma Bölümü */
.unlock-section {
  margin-top: 30px;
}
.unlock-status {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;
  padding: 16px;
  border-radius: 16px;
  border: 1px solid rgba(255, 255, 255, 0.1);
  background: rgba(255, 255, 255, 0.06);
  backdrop-filter: blur(16px);
  -webkit-backdrop-filter: blur(16px);
}
.unlock-status.unlocked {
  border-color: rgba(255, 255, 255, 0.15);
  background: rgba(255, 255, 255, 0.08);
  backdrop-filter: blur(16px);
  -webkit-backdrop-filter: blur(16px);
  flex-direction: row;
  justify-content: center;
  font-size: 16px;
  font-weight: bold;
  color: var(--accent);
}
.unlock-info {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 2px;
}
.unlock-price {
  font-size: 22px;
  font-weight: 900;
  color: var(--accent);
  text-shadow: 0 0 10px rgba(255, 215, 0, 0.3);
}
.unlock-label {
  font-size: 12px;
  color: #aaa;
}
.unlock-btn {
  width: 100%;
  padding: 14px;
  border: none;
  border-radius: 12px;
  background: linear-gradient(135deg, var(--accent), #f59e0b);
  color: #000;
  font-size: 16px;
  font-weight: 800;
  cursor: pointer;
  transition: transform 0.2s, opacity 0.2s;
  letter-spacing: 0.5px;
}
.unlock-btn:active {
  transform: scale(0.97);
}
.unlock-btn.disabled {
  opacity: 0.4;
  cursor: not-allowed;
  background: #555;
  color: #999;
}
.coin-balance {
  font-size: 11px;
  color: #888;
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

/* Bilgilendirme Tam Sayfa */
.info-page {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: #fff;
  z-index: 1000;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}
.info-page-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: max(44px, env(safe-area-inset-top)) 16px 12px 16px;
  background: #fff;
  border-bottom: 1px solid #f0f0f0;
  flex-shrink: 0;
}
.info-back-btn {
  width: 40px;
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  font-size: 32px;
  color: #e91e8a;
  line-height: 1;
}
.info-page-title {
  font-size: 18px;
  font-weight: 800;
  color: #1a1a1a;
}
.info-page-body {
  flex: 1;
  overflow-y: auto;
  padding: 24px 20px;
  padding-bottom: calc(24px + env(safe-area-inset-bottom, 20px));
  color: #333;
  font-size: 14px;
  line-height: 1.7;
  -webkit-overflow-scrolling: touch;
}
.info-page-body p {
  margin: 0 0 12px 0;
}
.info-section-title {
  font-size: 16px;
  font-weight: 700;
  color: #e91e8a;
  margin: 20px 0 8px 0 !important;
}
.info-section-title:first-child {
  margin-top: 0 !important;
}
.info-price-list {
  background: #fdf2f8;
  border-radius: 12px;
  padding: 12px;
  margin-bottom: 12px;
}
.info-price-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 6px 4px;
  border-bottom: 1px solid rgba(233, 30, 138, 0.08);
}
.info-price-row:last-child {
  border-bottom: none;
}
.info-level {
  font-weight: 600;
  color: #e91e8a;
  font-size: 13px;
}
.info-cost {
  font-weight: 700;
  color: #333;
  font-size: 13px;
}
.info-list {
  list-style: none;
  padding: 0;
  margin: 0 0 12px 0;
}
.info-list li {
  padding: 6px 0 6px 16px;
  position: relative;
  color: #555;
  font-size: 13px;
}
.info-list li::before {
  content: '';
  position: absolute;
  left: 0;
  top: 13px;
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background: #e91e8a;
}

.svip-toast {
  position: fixed;
  left: 50%;
  bottom: calc(24px + env(safe-area-inset-bottom, 0px));
  transform: translateX(-50%);
  z-index: 1200;
  max-width: calc(100% - 32px);
  padding: 12px 16px;
  border-radius: 14px;
  background: rgba(28, 28, 30, 0.92);
  color: #fff;
  font-size: 14px;
  font-weight: 600;
  text-align: center;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.28);
  backdrop-filter: blur(6px);
  -webkit-backdrop-filter: blur(6px);
}

.toast-fade-enter-active,
.toast-fade-leave-active {
  transition: opacity 0.2s ease, transform 0.2s ease;
}

.toast-fade-enter-from,
.toast-fade-leave-to {
  opacity: 0;
  transform: translateX(-50%) translateY(8px);
}
</style>
