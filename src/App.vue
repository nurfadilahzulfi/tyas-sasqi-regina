<template>
  <div class="birthday-container" @click="createHeart">
    <!-- MUSIC START OVERLAY (centered) -->
    <div v-if="showMusicPrompt" class="music-overlay" @click.stop>
      <div class="music-popup" role="dialog" aria-modal="true" aria-label="Putar musik ulang tahun">
        <p class="music-text">🎵 Musik spesial untuk Sasqi :3</p>
        <p class="music-sub">Klik tombol di bawah untuk memulai musik</p>
        <div class="music-actions">
          <button @click="startMusic" class="music-start-btn">Putar Musik 🎶</button>
        </div>
      </div>
    </div>

    <!-- Animated Background (stars) -->
    <div class="stars-bg" aria-hidden="true">
      <div v-for="i in 100" :key="'star-' + i" class="star" :style="getStarStyle(i)"></div>
    </div>

    <!-- Floating Hearts -->
    <div
      v-for="heart in floatingHearts"
      :key="heart.id"
      class="floating-heart"
      :style="{
        left: heart.x + 'px',
        top: heart.y + 'px',
        fontSize: heart.size + 'px',
        animationDuration: heart.duration + 's'
      }"
      aria-hidden="true"
    >
      {{ heart.emoji }}
    </div>

    <!-- Magical Particles -->
    <div
      v-for="particle in particles"
      :key="particle.id"
      class="particle"
      :style="{
        left: particle.left + '%',
        animationDuration: particle.duration + 's',
        animationDelay: particle.delay + 's'
      }"
      aria-hidden="true"
    ></div>

    <!-- Main Content -->
    <div :class="['content-wrapper', { show: showContent }]">
      <!-- Cake with Candles -->
      <div class="cake-container" @click="blowCandles" role="button" aria-pressed="false">
        <div class="cake" aria-hidden="true">
          <div v-for="i in 3" :key="'layer-' + i" :class="['cake-layer', 'layer-' + i]">
            <div class="frosting"></div>
          </div>
          <div class="candles">
            <div v-for="i in 5" :key="'candle-' + i" :class="['candle', { blown: candlesBlown }]">
              <div class="flame"></div>
            </div>
          </div>
        </div>
        <p v-if="!candlesBlown" class="instruction">✨ Klik kue untuk meniup lilin ✨</p>
      </div>

      <!-- Main Card -->
      <div class="card" v-if="candlesBlown">
        <div class="title-container">
          <h1 class="animated-title">
            <span v-for="(char, i) in titleText" :key="'char-' + i" :style="{ animationDelay: i * 0.08 + 's' }">
              {{ char }}
            </span>
          </h1>
        </div>

        <div class="name-container">
          <div class="name-glow">Tyas Sasqi Regina</div>
          <h2 class="name-main">Tyas Sasqi Regina</h2>
        </div>

        <div class="messages-grid">
          <div
            v-for="(msg, idx) in loveMessages"
            :key="'msg-' + idx"
            :class="['love-card', 'card-' + idx, { flipped: flippedCards.includes(idx) }]"
            @click="flipCard(idx)"
            role="button"
          >
            <div class="card-inner">
              <div class="card-front">
                <div class="card-icon">{{ msg.icon }}</div>
                <p class="card-hint">Klik untuk membaca</p>
              </div>
              <div class="card-back">
                <p class="card-message">{{ msg.text }}</p>
              </div>
            </div>
          </div>
        </div>

        <div class="quote-container">
          <div class="quote-mark">"</div>
          <p class="quote-text">
            "Di hari spesial ini, aku punya banyak harapan buat kamu. 
              Semoga kamu selalu dikelilingi orang-orang yang sayang dan support kamu. 
              Semoga setiap mimpi yang kamu punya bisa jadi kenyataan, satu per satu. 
              Semoga kamu selalu sehat, bahagia, dan nggak pernah kehilangan semangat buat ngejar apa yang kamu mau. 
              Dan yang paling penting, semoga kamu selalu merasa dicintai dan dihargai, karena kamu emang layak dapetin semua itu. 🌟💕"
          </p>
          <div class="quote-mark closing">"</div>
        </div>

        <div class="wish-section">
          <button @click="makeWish" :class="['wish-button', { wished: wishMade }]">
            <span v-if="!wishMade">💫 Buat Permohonan</span>
            <span v-else>✨ Permohonanmu Telah Dikirim ke Alam Semesta ✨</span>
          </button>
        </div>

        <div class="love-counter">
          <div class="counter-display">
            <span class="heart-pulse">❤️</span>
            <span class="counter-number">{{ loveCount }}</span>
            <span class="heart-pulse">❤️</span>
          </div>
          <p class="counter-text">Kasih Sayang untuk Tyas</p>
        </div>
      </div>
    </div>

    <!-- Fireworks -->
    <div v-if="showFireworks" class="fireworks" aria-hidden="true">
      <div v-for="i in 20" :key="'firework-' + i" class="firework" :style="getFireworkStyle(i)"></div>
    </div>

    <!-- Hidden audio element (looping) -->
    <audio ref="audioPlayer" loop preload="auto">
      <source src="/music/bergema.mp3" type="audio/mp3" />
    </audio>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

/* STATE */
const showContent = ref(false)
const candlesBlown = ref(false)
const flippedCards = ref([])
const wishMade = ref(false)
const showFireworks = ref(false)
const loveCount = ref(0)
const floatingHearts = ref([])
const particles = ref([])
const titleText = '✨ Selamat Ulang Tahun ✨'
const audioPlayer = ref(null)
const showMusicPrompt = ref(false)

/* messages */
const loveMessages = [
  { icon: '🌹', text: 'Semoga setiap hari kamu dipenuhi hal-hal baik yang bikin kamu tersenyum tanpa henti' },
  { icon: '⭐', text: 'Kamu tuh spesial dengan caramu sendiri, jadi diri sendiri aja terus ya!' },
  { icon: '🦋', text: 'Hidup cuma sekali, jadi lakuin aja yang bikin kamu bahagia' },
  { icon: '🌸', text: 'Apapun yang kamu lakuin itu ada artinya kok, terus maju aja dan jangan takut coba hal baru' },
  { icon: '💝', text: 'Kamu layak dapetin semua kebahagiaan di dunia ini, jangan pernah merasa kurang dari siapa pun' },
  { icon: '🌙', text: 'Di tengah hiruk pikuk dunia, semoga kamu selalu menemukan ketenangan dan jadi tempat pulang yang nyaman buat diri sendiri' }
]

/* helpers */
const getStarStyle = (i) => ({
  left: Math.random() * 100 + '%',
  top: Math.random() * 100 + '%',
  animationDelay: Math.random() * 3 + 's',
  animationDuration: (2 + Math.random() * 3) + 's'
})

const getFireworkStyle = (i) => ({
  left: Math.random() * 100 + '%',
  top: Math.random() * 60 + '%',
  animationDelay: (i * 0.15) + 's'
})

/* interactions */
const createHeart = (e) => {
  const heart = {
    id: Date.now() + Math.random(),
    x: e.clientX,
    y: e.clientY,
    emoji: ['❤️', '💖', '💕', '💗', '💝'][Math.floor(Math.random() * 5)],
    size: 18 + Math.random() * 36,
    duration: 2 + Math.random()
  }
  floatingHearts.value.push(heart)
  setTimeout(() => {
    floatingHearts.value = floatingHearts.value.filter(h => h.id !== heart.id)
  }, 3000)
}

const blowCandles = () => {
  candlesBlown.value = true
  showFireworks.value = true
  setTimeout(() => (showFireworks.value = false), 3000)

  const interval = setInterval(() => {
    if (loveCount.value < 100) loveCount.value += Math.floor(Math.random() * 5) + 1
    else clearInterval(interval)
  }, 50)
}

const flipCard = (idx) => {
  if (!flippedCards.value.includes(idx)) flippedCards.value.push(idx)
  else flippedCards.value = flippedCards.value.filter(i => i !== idx)
}

const makeWish = () => {
  wishMade.value = true
  showFireworks.value = true
  for (let i = 0; i < 20; i++) {
    setTimeout(() => {
      const heart = {
        id: Date.now() + Math.random(),
        x: window.innerWidth / 2 + (Math.random() - 0.5) * 200,
        y: window.innerHeight / 2 + (Math.random() - 0.5) * 200,
        emoji: ['❤️', '💖', '💕', '💗', '💝', '✨', '⭐'][Math.floor(Math.random() * 7)],
        size: 28 + Math.random() * 20,
        duration: 2
      }
      floatingHearts.value.push(heart)
    }, i * 40)
  }
  setTimeout(() => (showFireworks.value = false), 3000)
}

/* MUSIC */
const startMusic = async () => {
  try {
    await audioPlayer.value.play()
    showMusicPrompt.value = false
  } catch (err) {
    showMusicPrompt.value = true
  }
}

/* LIFECYCLE */
onMounted(() => {
  setTimeout(() => (showContent.value = true), 450)

  particles.value = Array.from({ length: 30 }, (_, i) => ({
    id: i,
    left: Math.random() * 100,
    delay: Math.random() * 5,
    duration: 3 + Math.random() * 4
  }))

  audioPlayer.value = document.querySelector('audio')
  if (audioPlayer.value) {
    audioPlayer.value.currentTime = 0
    audioPlayer.value.loop = true
    startMusic()
  }
})
</script>

<style scoped>
/* ---------- Base & layout ---------- */
* { box-sizing: border-box; margin: 0; padding: 0; }
:root { 
  --glass: rgba(255,255,255,0.12); 
  --accent1: #f093fb; 
  --accent2:#667eea; 
  --accent3:#4facfe;
  --shadow-lg: 0 20px 60px rgba(0,0,0,0.35), 0 0 100px rgba(240,147,251,0.15);
  --shadow-xl: 0 30px 80px rgba(0,0,0,0.4), 0 0 120px rgba(102,126,234,0.2);
}

.birthday-container {
  min-height: 100vh;
  background: 
    radial-gradient(circle at 20% 80%, rgba(240,147,251,0.3) 0%, transparent 50%),
    radial-gradient(circle at 80% 20%, rgba(79,172,254,0.3) 0%, transparent 50%),
    radial-gradient(circle at 40% 40%, rgba(102,126,234,0.2) 0%, transparent 50%),
    linear-gradient(135deg, #1a1a2e 0%, #16213e 25%, #0f3460 50%, #533483 75%, #2d1b69 100%);
  background-size: 100% 100%, 100% 100%, 100% 100%, 400% 400%;
  animation: gradientShift 20s ease infinite;
  overflow-x: hidden;
  position: relative;
  cursor: pointer;
  padding: clamp(1rem, 3vw, 2.5rem) clamp(0.75rem, 2vw, 1.5rem);
  font-family: 'Segoe UI Emoji', 'Noto Color Emoji', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
}

@keyframes gradientShift { 
  0%, 100% { background-position: 0% 0%, 0% 0%, 0% 0%, 0% 50%; } 
  50% { background-position: 0% 0%, 0% 0%, 0% 0%, 100% 50%; } 
}

.content-wrapper { 
  max-width: min(920px, 95%);
  margin: 0 auto; 
  position: relative; 
  z-index: 10; 
  transform: scale(0.92); 
  opacity: 0; 
  transition: all 900ms cubic-bezier(.34,1.56,.64,1); 
}
.content-wrapper.show { transform: scale(1); opacity: 1; }

/* ---------- Music overlay (centered) ---------- */
.music-overlay {
  position: fixed;
  inset: 0;
  background: rgba(4,6,15,0.85);
  backdrop-filter: blur(12px) saturate(140%);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 99999;
}

.music-popup {
  background: linear-gradient(135deg, rgba(79,172,254,0.15), rgba(240,147,251,0.15));
  backdrop-filter: blur(20px);
  padding: clamp(1.5rem, 4vw, 2.5rem) clamp(1.8rem, 5vw, 3rem);
  border-radius: 24px;
  text-align: center;
  color: #fff;
  width: min(480px, 90%);
  box-shadow: 
    0 20px 60px rgba(0,0,0,0.5),
    inset 0 1px 0 rgba(255,255,255,0.1),
    0 0 80px rgba(79,172,254,0.2);
  border: 1px solid rgba(255,255,255,0.15);
  transform: translateY(0);
  animation: popupIn .4s cubic-bezier(.34,1.56,.64,1);
}

@keyframes popupIn { 
  from { transform: translateY(20px) scale(.95); opacity: 0; filter: blur(4px); } 
  to { transform: translateY(0) scale(1); opacity: 1; filter: blur(0); } 
}

.music-text { 
  font-size: clamp(1.1rem, 3vw, 1.4rem);
  font-weight: 700; 
  margin-bottom: 8px; 
  letter-spacing: 0.3px;
  text-shadow: 0 2px 12px rgba(79,172,254,0.4);
}
.music-sub { 
  font-size: clamp(0.9rem, 2.5vw, 1rem);
  color: rgba(255,255,255,0.85); 
  margin-bottom: 18px; 
}

.music-actions { display:flex; justify-content:center; gap:12px; flex-wrap: wrap; }

.music-start-btn {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 50%, #f093fb 100%);
  color: white;
  border: none;
  padding: clamp(10px, 2.5vw, 14px) clamp(20px, 5vw, 28px);
  border-radius: 999px;
  cursor: pointer;
  font-weight: 700;
  font-size: clamp(0.95rem, 2.5vw, 1.05rem);
  box-shadow: 
    0 10px 30px rgba(102,126,234,0.35),
    inset 0 1px 0 rgba(255,255,255,0.2);
  transition: all .3s cubic-bezier(.34,1.56,.64,1);
  position: relative;
  overflow: hidden;
}
.music-start-btn::before {
  content: '';
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255,255,255,0.3), transparent);
  transition: left 0.5s;
}
.music-start-btn:hover::before {
  left: 100%;
}
.music-start-btn:hover { 
  transform: translateY(-3px) scale(1.05); 
  box-shadow: 
    0 15px 40px rgba(102,126,234,0.45),
    inset 0 1px 0 rgba(255,255,255,0.3);
}
.music-start-btn:active {
  transform: translateY(-1px) scale(1.02);
}

/* ---------- Stars background ---------- */
.stars-bg { position: fixed; top: 0; left: 0; width: 100%; height: 100%; pointer-events: none; z-index: 1; }
.star { 
  position: absolute; 
  width: clamp(1.5px, 0.3vw, 3px);
  height: clamp(1.5px, 0.3vw, 3px);
  background: white; 
  border-radius: 50%; 
  animation: twinkle 3s infinite ease-in-out; 
  box-shadow: 0 0 clamp(6px, 1.5vw, 12px) rgba(255,255,255,0.9); 
  opacity: .7; 
}
@keyframes twinkle { 
  0%,100% { opacity: 0.3; transform: scale(0.8) } 
  50% { opacity: 1; transform: scale(1.8) } 
}

/* ---------- Floating hearts ---------- */
.floating-heart { position: fixed; pointer-events: none; animation: floatUp linear; z-index: 1000; will-change: transform, opacity; }
@keyframes floatUp { 0% { transform: translateY(0) rotate(0deg); opacity: 1 } 100% { transform: translateY(-220px) rotate(360deg); opacity: 0 } }

/* ---------- Particles ---------- */
.particle { position: absolute; width: 4px; height: 4px; background: radial-gradient(circle, #fff, #f093fb); border-radius: 50%; top: -10px; animation: particleFall linear infinite; box-shadow: 0 0 10px rgba(240,147,251,0.8); }
@keyframes particleFall { to { transform: translateY(100vh) rotate(360deg); opacity: 0 } }

/* ---------- Cake & candles ---------- */
.cake-container { 
  text-align:center; 
  margin-bottom: clamp(2rem, 5vw, 3.5rem);
  animation: slideDown 1.2s cubic-bezier(.34,1.56,.64,1);
  z-index: 10; 
}
@keyframes slideDown { 
  from { transform: translateY(-100px) rotate(-5deg); opacity:0; filter: blur(10px); } 
  to { transform: translateY(0) rotate(0deg); opacity:1; filter: blur(0); } 
}

.cake { 
  display:inline-block; 
  position:relative; 
  margin-bottom: 1.5rem;
  filter: drop-shadow(0 15px 35px rgba(0,0,0,0.4));
  transform-style: preserve-3d;
}
.cake-layer { 
  width: clamp(160px, 35vw, 200px);
  height: clamp(32px, 7vw, 40px);
  margin: clamp(4px, 1vw, 6px) auto; 
  border-radius: 12px; 
  position: relative; 
  box-shadow: 
    0 8px 20px rgba(0,0,0,0.35),
    inset 0 2px 0 rgba(255,255,255,0.3),
    inset 0 -2px 4px rgba(0,0,0,0.2);
  transform: translateZ(20px);
}
.layer-1 { 
  background: linear-gradient(135deg, #ff6b9d 0%, #f06292 50%, #c44569 100%);
  width: clamp(192px, 42vw, 240px);
  box-shadow: 
    0 10px 30px rgba(255,107,157,0.4),
    inset 0 2px 0 rgba(255,255,255,0.3);
}
.layer-2 { 
  background: linear-gradient(135deg, #feca57 0%, #ff9ff3 50%, #ee5a6f 100%);
  width: clamp(176px, 38.5vw, 220px);
  box-shadow: 
    0 10px 25px rgba(254,202,87,0.4),
    inset 0 2px 0 rgba(255,255,255,0.3);
}
.layer-3 { 
  background: linear-gradient(135deg, #48dbfb 0%, #0abde3 50%, #00b8d4 100%);
  box-shadow: 
    0 10px 25px rgba(72,219,251,0.4),
    inset 0 2px 0 rgba(255,255,255,0.3);
}

.frosting { 
  position:absolute; 
  top:0; 
  left:0; 
  right:0; 
  height: clamp(6px, 1.5vw, 10px);
  background: linear-gradient(180deg, #fff 0%, rgba(255,255,255,0.8) 100%);
  border-radius: 12px 12px 0 0; 
  opacity:.95;
  box-shadow: 0 2px 8px rgba(255,255,255,0.5);
}

.candles { 
  display:flex; 
  justify-content:space-around; 
  position:absolute; 
  top: clamp(-32px, -7vw, -40px);
  left:50%; 
  transform:translateX(-50%); 
  width: clamp(120px, 26vw, 150px);
}
.candle { 
  width: clamp(6px, 1.5vw, 8px);
  height: clamp(24px, 5vw, 30px);
  background: linear-gradient(135deg, #ffeaa7 0%, #fdcb6e 100%);
  border-radius: 4px 4px 0 0; 
  position:relative; 
  box-shadow: 
    0 5px 15px rgba(253,203,110,0.3),
    inset 1px 0 0 rgba(255,255,255,0.4),
    inset -1px 0 0 rgba(0,0,0,0.1);
}
.flame { 
  position:absolute; 
  top: clamp(-12px, -2.5vw, -15px);
  left:50%; 
  transform:translateX(-50%); 
  width: clamp(10px, 2vw, 12px);
  height: clamp(16px, 3.5vw, 20px);
  background: radial-gradient(ellipse at center, #fff 0%, #ffeb3b 30%, #ff6348 100%);
  border-radius:50%/60% 60% 40% 40%; 
  animation: flicker .3s infinite alternate; 
  box-shadow: 
    0 0 15px #ffeb3b, 
    0 0 25px #ff6348,
    0 0 35px rgba(255,99,72,0.3);
  filter: blur(0.5px);
}
@keyframes flicker { 
  0% { transform: translateX(-50%) scale(1) } 
  100% { transform: translateX(-50%) scale(1.15) translateY(-2px) } 
}
.candle.blown .flame { animation: blowOut .6s cubic-bezier(.34,1.56,.64,1) forwards; }
@keyframes blowOut { 
  0% { opacity:1; transform: translateX(-50%) scale(1) } 
  50% { opacity:.5; transform: translateX(-50%) scale(1.6) translateX(30px) rotate(45deg) } 
  100% { opacity:0; transform: translateX(-50%) scale(0) translateY(-20px) } 
}

.instruction { 
  color: white; 
  font-size: clamp(0.95rem, 2.5vw, 1.1rem);
  font-weight:600; 
  text-shadow: 
    0 0 20px rgba(255,255,255,0.8),
    0 2px 10px rgba(79,172,254,0.6);
  animation: pulse 2s ease-in-out infinite;
}

/* ---------- Card / title / name ---------- */
.card { 
  background: linear-gradient(135deg, rgba(79,172,254,0.08), rgba(240,147,251,0.08));
  backdrop-filter: blur(20px) saturate(180%);
  border-radius: 28px; 
  padding: clamp(1.8rem, 5vw, 3rem);
  box-shadow: 
    var(--shadow-lg),
    inset 0 1px 0 rgba(255,255,255,0.1);
  border: 1px solid rgba(255,255,255,0.1);
  animation: fadeInUp 1.2s cubic-bezier(.34,1.56,.64,1);
  position: relative;
  overflow: hidden;
}
.card::before {
  content: '';
  position: absolute;
  top: -50%;
  left: -50%;
  width: 200%;
  height: 200%;
  background: radial-gradient(circle, rgba(255,255,255,0.03) 0%, transparent 70%);
  animation: rotate 20s linear infinite;
  pointer-events: none;
}
@keyframes rotate {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}
@keyframes fadeInUp { 
  from { opacity:0; transform:translateY(40px) scale(0.95); filter: blur(8px); } 
  to { opacity:1; transform:translateY(0) scale(1); filter: blur(0); } 
}

.title-container { 
  text-align:center; 
  margin-bottom: clamp(1.2rem, 3vw, 1.8rem);
  perspective:1000px; 
}
.animated-title { 
  font-size: clamp(1.6rem, 5vw, 2.6rem);
  font-weight:800; 
  color:#fff; 
  text-shadow: 
    0 0 30px rgba(255,255,255,0.9), 
    0 0 50px rgba(240,147,251,0.6),
    0 4px 20px rgba(0,0,0,0.3);
  display:inline-block; 
  letter-spacing: 1px;
}
.animated-title span { 
  display:inline-block; 
  animation: bounceIn .8s ease-out both; 
}
@keyframes bounceIn { 
  0% { opacity:0; transform:scale(0) rotateY(180deg) translateY(-30px) } 
  50% { transform:scale(1.15) rotateY(90deg) } 
  100% { opacity:1; transform:scale(1) rotateY(0deg) translateY(0) } 
}

.name-container { 
  position:relative; 
  text-align:center; 
  margin-bottom: clamp(1.2rem, 3vw, 1.8rem);
  padding: clamp(0.5rem, 2vw, 1rem) 0;
}
.name-glow { 
  position:absolute; 
  top:50%; 
  left:50%; 
  transform:translate(-50%,-50%); 
  font-size: clamp(1.8rem, 6vw, 3rem);
  font-weight:800; 
  color:transparent; 
  text-shadow:
    0 0 50px rgba(255,255,255,0.9), 
    0 0 90px rgba(240,147,251,0.7),
    0 0 120px rgba(79,172,254,0.5);
  animation: glow 2.5s ease-in-out infinite; 
  z-index:0; 
  letter-spacing: 2px;
}
.name-main { 
  font-size: clamp(1.8rem, 6vw, 3rem);
  font-weight:800; 
  background: linear-gradient(135deg, #fff 0%, #f093fb 25%, #4facfe 50%, #fff 75%, #f093fb 100%);
  background-size:300% auto; 
  -webkit-background-clip:text; 
  background-clip:text; 
  -webkit-text-fill-color:transparent; 
  animation: shine 4s linear infinite; 
  position:relative; 
  z-index:1;
  letter-spacing: 2px;
  filter: drop-shadow(0 4px 12px rgba(240,147,251,0.4));
}
@keyframes glow { 
  0%,100% { opacity:.6; transform:translate(-50%,-50%) scale(1) } 
  50% { opacity:1; transform:translate(-50%,-50%) scale(1.08) } 
}
@keyframes shine { to { background-position:300% center } }

/* ---------- Messages grid & cards ---------- */
.messages-grid { 
  display:grid; 
  grid-template-columns: repeat(auto-fit, minmax(min(220px, 100%), 1fr));
  gap: clamp(0.9rem, 2.5vw, 1.4rem);
  margin-bottom: clamp(1.2rem, 3vw, 1.8rem);
}
.love-card { 
  height: clamp(180px, 40vw, 220px);
  perspective:1200px; 
  cursor:pointer;
  transition: transform 0.3s ease;
}
.love-card:hover {
  transform: translateY(-5px);
}
.card-inner { 
  position:relative; 
  width:100%; 
  height:100%; 
  transition: transform .8s cubic-bezier(.34,1.56,.64,1);
  transform-style:preserve-3d; 
}
.love-card.flipped .card-inner { transform: rotateY(180deg); }
.card-front, .card-back { 
  position:absolute; 
  width:100%; 
  height:100%; 
  backface-visibility:hidden; 
  border-radius:16px; 
  display:flex; 
  flex-direction:column; 
  align-items:center; 
  justify-content:center; 
  padding: clamp(0.8rem, 3vw, 1.2rem);
}
.card-front { 
  background: linear-gradient(135deg, rgba(79,172,254,0.12), rgba(240,147,251,0.08));
  backdrop-filter: blur(10px);
  border:1px solid rgba(255,255,255,0.15);
  box-shadow: 
    0 10px 30px rgba(0,0,0,0.3),
    inset 0 1px 0 rgba(255,255,255,0.1);
}
.card-back { 
  background: linear-gradient(135deg, rgba(240,147,251,0.18), rgba(102,126,234,0.18));
  backdrop-filter: blur(15px);
  transform: rotateY(180deg); 
  border:1px solid rgba(255,255,255,0.15);
  box-shadow: 
    0 10px 30px rgba(0,0,0,0.3),
    inset 0 1px 0 rgba(255,255,255,0.1);
}
.card-icon { 
  font-size: clamp(2.8rem, 8vw, 4rem);
  margin-bottom: clamp(0.4rem, 1.5vw, 0.8rem);
  animation: float 3s ease-in-out infinite; 
  filter: drop-shadow(0 4px 12px rgba(255,255,255,0.3));
}
@keyframes float { 
  0%,100% { transform: translateY(0) rotate(0deg) } 
  50% { transform: translateY(-12px) rotate(5deg) } 
}
.card-hint { 
  color: rgba(255,255,255,0.9);
  font-size: clamp(0.85rem, 2.2vw, 0.95rem);
  font-style: italic; 
  text-shadow: 0 2px 8px rgba(0,0,0,0.3);
}
.card-message { 
  color: white; 
  font-size: clamp(0.9rem, 2.5vw, 1.05rem);
  text-align:center; 
  line-height:1.6;
  text-shadow: 0 2px 10px rgba(0,0,0,0.3);
  padding: 0 clamp(0.3rem, 1vw, 0.5rem);
}

/* ---------- Quote ---------- */
.quote-container { 
  position:relative; 
  padding: clamp(1.2rem, 3vw, 1.8rem);
  margin: clamp(1.2rem, 3vw, 1.8rem) 0;
  background: linear-gradient(135deg, rgba(79,172,254,0.1), rgba(240,147,251,0.08));
  backdrop-filter: blur(10px);
  border-radius:16px; 
  border:1px solid rgba(255,255,255,0.12);
  box-shadow: 
    0 8px 25px rgba(0,0,0,0.25),
    inset 0 1px 0 rgba(255,255,255,0.1);
}
.quote-mark { 
  font-size: clamp(2.2rem, 6vw, 3.5rem);
  color: rgba(255,255,255,0.3);
  font-family: Georgia, serif; 
  position:absolute; 
  top: clamp(4px, 1vw, 8px);
  left: clamp(6px, 1.5vw, 10px);
  text-shadow: 0 0 20px rgba(79,172,254,0.4);
  line-height: 1;
}
.quote-mark.closing { 
  top:auto; 
  bottom: clamp(4px, 1vw, 8px);
  left:auto; 
  right: clamp(6px, 1.5vw, 10px);
  transform: rotate(180deg); 
}
.quote-text { 
  color:white; 
  font-size: clamp(0.95rem, 2.5vw, 1.1rem);
  font-style:italic; 
  line-height:1.7;
  text-align:center; 
  margin: clamp(0.4rem, 1.5vw, 0.8rem) clamp(0.8rem, 3vw, 1.5rem);
  text-shadow: 0 2px 10px rgba(0,0,0,0.3);
}

/* ---------- Wish & counter ---------- */
.wish-section {
  text-align: center;
  margin: clamp(1.2rem, 3vw, 1.6rem) 0;
}
.wish-button { 
  padding: clamp(10px, 2.5vw, 14px) clamp(22px, 5vw, 32px);
  font-size: clamp(0.95rem, 2.5vw, 1.1rem);
  font-weight:700; 
  color:white; 
  background: linear-gradient(135deg, #667eea 0%, #764ba2 50%, #f093fb 100%);
  border:none; 
  border-radius:999px; 
  cursor:pointer; 
  box-shadow: 
    0 12px 35px rgba(102,126,234,0.35),
    inset 0 1px 0 rgba(255,255,255,0.2);
  transition: all 0.3s cubic-bezier(.34,1.56,.64,1);
  position: relative;
  overflow: hidden;
}
.wish-button::before {
  content: '';
  position: absolute;
  top: 50%;
  left: 50%;
  width: 0;
  height: 0;
  border-radius: 50%;
  background: rgba(255,255,255,0.3);
  transform: translate(-50%, -50%);
  transition: width 0.6s, height 0.6s;
}
.wish-button:hover::before {
  width: 300px;
  height: 300px;
}
.wish-button:hover {
  transform: translateY(-3px) scale(1.05);
  box-shadow: 
    0 15px 45px rgba(102,126,234,0.45),
    inset 0 1px 0 rgba(255,255,255,0.3);
}
.wish-button.wished { 
  background: linear-gradient(135deg, #00f2fe 0%, #4facfe 50%, #00b8d4 100%);
  animation: wishPulse 2s infinite; 
  box-shadow: 
    0 12px 40px rgba(79,172,254,0.5),
    0 0 60px rgba(79,172,254,0.3);
}
@keyframes wishPulse { 
  0%,100% { transform:scale(1); box-shadow: 0 12px 40px rgba(79,172,254,0.5); } 
  50% { transform:scale(1.05); box-shadow: 0 15px 50px rgba(79,172,254,0.6), 0 0 80px rgba(79,172,254,0.4); } 
}

.love-counter { 
  text-align:center; 
  margin-top: clamp(1.2rem, 3vw, 1.6rem);
  padding: clamp(1.2rem, 3vw, 1.6rem);
  background: linear-gradient(135deg, rgba(79,172,254,0.1), rgba(240,147,251,0.08));
  backdrop-filter: blur(10px);
  border-radius:16px; 
  border:1px solid rgba(255,255,255,0.12);
  box-shadow: 
    0 8px 25px rgba(0,0,0,0.25),
    inset 0 1px 0 rgba(255,255,255,0.1);
}
.counter-display {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: clamp(0.6rem, 2vw, 1rem);
  margin-bottom: clamp(0.4rem, 1.5vw, 0.6rem);
}
.counter-number { 
  font-size: clamp(2rem, 6vw, 2.8rem);
  font-weight:800; 
  color:white; 
  text-shadow:
    0 0 25px rgba(255,255,255,0.8),
    0 4px 15px rgba(240,147,251,0.5);
  animation: counterGlow 2s ease-in-out infinite;
}
@keyframes counterGlow {
  0%, 100% { text-shadow: 0 0 25px rgba(255,255,255,0.8), 0 4px 15px rgba(240,147,251,0.5); }
  50% { text-shadow: 0 0 35px rgba(255,255,255,1), 0 4px 20px rgba(240,147,251,0.8); }
}
.heart-pulse { 
  font-size: clamp(1.4rem, 4vw, 1.8rem);
  animation: heartbeat 1.2s infinite; 
  display: inline-block;
}
@keyframes heartbeat {
  0%, 100% { transform: scale(1); }
  25% { transform: scale(1.2); }
  50% { transform: scale(1); }
}
.counter-text {
  color: rgba(255,255,255,0.9);
  font-size: clamp(0.9rem, 2.5vw, 1.05rem);
  font-weight: 600;
  text-shadow: 0 2px 8px rgba(0,0,0,0.3);
}

/* ---------- Fireworks ---------- */
.fireworks { 
  position:fixed; 
  top:0; 
  left:0; 
  width:100%; 
  height:100%; 
  pointer-events:none; 
  z-index:9999; 
}
.firework { 
  position:absolute; 
  width: clamp(3px, 0.8vw, 6px);
  height: clamp(3px, 0.8vw, 6px);
  border-radius:50%; 
  animation: explode 1.8s ease-out forwards; 
  box-shadow:
    0 0 clamp(15px, 3vw, 25px) currentColor,
    0 0 clamp(30px, 5vw, 45px) currentColor;
}
@keyframes explode { 
  0% { 
    opacity:1; 
    transform:scale(1) translate(0, 0); 
  } 
  50% {
    opacity: 0.8;
    transform: scale(20) translate(var(--tx, 20px), var(--ty, 20px));
  }
  100% { 
    opacity:0; 
    transform:scale(0) translate(var(--tx, 40px), var(--ty, 40px)); 
  } 
}
.firework:nth-child(odd) { 
  background:#ff6b9d;
  --tx: 30px;
  --ty: -30px;
} 
.firework:nth-child(even) { 
  background:#4facfe;
  --tx: -30px;
  --ty: 30px;
} 
.firework:nth-child(3n) { 
  background:#feca57;
  --tx: 40px;
  --ty: 40px;
}
.firework:nth-child(4n) {
  background: #48dbfb;
  --tx: -40px;
  --ty: -40px;
}
.firework:nth-child(5n) {
  background: #f093fb;
  --tx: 25px;
  --ty: -45px;
}

/* ---------- Responsive ---------- */
@media (max-width: 768px) {
  .birthday-container {
    padding: clamp(1rem, 3vw, 1.5rem) clamp(0.75rem, 2vw, 1rem);
  }
  
  .messages-grid { 
    grid-template-columns: 1fr;
    gap: 1rem;
  }
  
  .love-card {
    height: 200px;
  }
  
  .card {
    border-radius: 20px;
  }
}

@media (max-width: 480px) {
  .birthday-container {
    padding: 1rem 0.75rem;
  }
  
  .content-wrapper {
    max-width: 100%;
  }
  
  .card {
    padding: 1.5rem 1rem;
    border-radius: 18px;
  }
  
  .cake-layer {
    margin: 4px auto;
  }
  
  .messages-grid {
    gap: 0.85rem;
  }
  
  .love-card {
    height: 180px;
  }
  
  .quote-container {
    padding: 1rem;
  }
  
  .quote-text {
    margin: 0.4rem 0.6rem;
  }
}

@media (min-width: 1024px) {
  .messages-grid {
    grid-template-columns: repeat(3, 1fr);
  }
}

/* Landscape orientation fixes */
@media (max-height: 600px) and (orientation: landscape) {
  .birthday-container {
    padding: 1rem 2rem;
  }
  
  .cake-container {
    margin-bottom: 1.5rem;
  }
  
  .card {
    padding: 1.5rem 2rem;
  }
  
  .title-container,
  .name-container {
    margin-bottom: 1rem;
  }
}

/* small helpers */
.player-container { display:none; }

</style>