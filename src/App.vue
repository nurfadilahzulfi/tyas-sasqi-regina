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
const showMusicPrompt = ref(false) // popup hanya muncul kalau autoplay gagal

/* constants */
const MUSIC_KEY = 'tyas_music_played'

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
    showMusicPrompt.value = true // show popup if autoplay blocked
  }
}

/* LIFECYCLE */
onMounted(() => {
  setTimeout(() => (showContent.value = true), 450)

  // init particles
  particles.value = Array.from({ length: 30 }, (_, i) => ({
    id: i,
    left: Math.random() * 100,
    delay: Math.random() * 5,
    duration: 3 + Math.random() * 4
  }))

  // attempt autoplay every time page loads
  audioPlayer.value = document.querySelector('audio')
  if (audioPlayer.value) {
    audioPlayer.value.currentTime = 0 // start from beginning
    audioPlayer.value.loop = true
    startMusic()
  }
})
</script>


<style scoped>
/* ---------- Base & layout ---------- */
* { box-sizing: border-box; margin: 0; padding: 0; }
:root { --glass: rgba(255,255,255,0.12); --accent1: #f093fb; --accent2:#667eea; --accent3:#4facfe; }

.birthday-container {
  min-height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 25%, #f093fb 50%, #4facfe 75%, #00f2fe 100%);
  background-size: 400% 400%;
  animation: gradientShift 15s ease infinite;
  overflow-x: hidden;
  position: relative;
  cursor: pointer;
  padding: 2rem 1rem;
  font-family: 'Segoe UI Emoji', 'Noto Color Emoji', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
}

@keyframes gradientShift { 0% { background-position: 0% 50%; } 50% { background-position: 100% 50%; } 100% { background-position: 0% 50%; } }

.content-wrapper { max-width: 900px; margin: 0 auto; position: relative; z-index: 10; transform: scale(0.92); opacity: 0; transition: all 900ms cubic-bezier(.34,1.56,.64,1); }
.content-wrapper.show { transform: scale(1); opacity: 1; }

/* ---------- Music overlay (centered) ---------- */
.music-overlay {
  position: fixed;
  inset: 0;
  background: rgba(4,6,15,0.75);
  backdrop-filter: blur(6px) saturate(120%);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 99999;
}

.music-popup {
  background: linear-gradient(180deg, rgba(255,255,255,0.06), rgba(255,255,255,0.03));
  padding: 2.0rem 2.2rem;
  border-radius: 16px;
  text-align: center;
  color: #fff;
  width: min(520px, 92%);
  box-shadow: 0 10px 30px rgba(0,0,0,0.45);
  border: 1px solid rgba(255,255,255,0.08);
  transform: translateY(0);
  animation: popupIn .32s ease;
}

@keyframes popupIn { from { transform: translateY(8px) scale(.99); opacity: 0 } to { transform: translateY(0) scale(1); opacity: 1 } }

.music-text { font-size: 1.25rem; font-weight: 700; margin-bottom: 6px; letter-spacing: 0.2px; }
.music-sub { font-size: 0.95rem; color: rgba(255,255,255,0.85); margin-bottom: 14px; }

.music-actions { display:flex; justify-content:center; gap:12px; }

/* start button */
.music-start-btn {
  background: linear-gradient(90deg, var(--accent1), var(--accent2));
  color: white;
  border: none;
  padding: 12px 20px;
  border-radius: 999px;
  cursor: pointer;
  font-weight: 700;
  box-shadow: 0 8px 24px rgba(102,126,234,0.18);
  transition: transform .15s ease, box-shadow .15s ease;
}
.music-start-btn:hover { transform: translateY(-4px); box-shadow: 0 12px 32px rgba(102,126,234,0.22); }

/* ---------- Stars background ---------- */
.stars-bg { position: fixed; top: 0; left: 0; width: 100%; height: 100%; pointer-events: none; z-index: 1; }
.star { position: absolute; width: 2px; height: 2px; background: white; border-radius: 50%; animation: twinkle 3s infinite ease-in-out; box-shadow: 0 0 8px rgba(255,255,255,0.85); opacity: .6; }
@keyframes twinkle { 0%,100% { opacity: 0.25; transform: scale(1) } 50% { opacity: 1; transform: scale(1.6) } }

/* ---------- Floating hearts ---------- */
.floating-heart { position: fixed; pointer-events: none; animation: floatUp linear; z-index: 1000; will-change: transform, opacity; }
@keyframes floatUp { 0% { transform: translateY(0) rotate(0deg); opacity: 1 } 100% { transform: translateY(-220px) rotate(360deg); opacity: 0 } }

/* ---------- Particles ---------- */
.particle { position: absolute; width: 4px; height: 4px; background: radial-gradient(circle, #fff, #f093fb); border-radius: 50%; top: -10px; animation: particleFall linear infinite; box-shadow: 0 0 10px rgba(240,147,251,0.8); }
@keyframes particleFall { to { transform: translateY(100vh) rotate(360deg); opacity: 0 } }

/* ---------- Cake & candles ---------- */
.cake-container { text-align:center; margin-bottom: 3rem; animation: slideDown 900ms ease-out; z-index: 10; }
@keyframes slideDown { from { transform: translateY(-80px); opacity:0 } to { transform: translateY(0); opacity:1 } }

.cake { display:inline-block; position:relative; margin-bottom: 1rem; }
.cake-layer { width: 200px; height: 40px; margin: 6px auto; border-radius: 10px; position: relative; box-shadow: 0 10px 30px rgba(0,0,0,0.28); }
.layer-1 { background: linear-gradient(to bottom, #ff6b9d, #c44569); width:240px; }
.layer-2 { background: linear-gradient(to bottom, #feca57, #ee5a6f); width:220px; }
.layer-3 { background: linear-gradient(to bottom, #48dbfb, #0abde3); }

.frosting { position:absolute; top:0; left:0; right:0; height:8px; background:#fff; border-radius:10px 10px 0 0; opacity:.95; }

.candles { display:flex; justify-content:space-around; position:absolute; top:-40px; left:50%; transform:translateX(-50%); width:150px; }
.candle { width:8px; height:30px; background: linear-gradient(to bottom,#ffeaa7,#fdcb6e); border-radius:3px 3px 0 0; position:relative; box-shadow: 0 5px 15px rgba(0,0,0,.18); }
.flame { position:absolute; top:-15px; left:50%; transform:translateX(-50%); width:12px; height:20px; background: radial-gradient(ellipse at center,#fff,#ff6348); border-radius:50%/60% 60% 40% 40%; animation:flicker .3s infinite alternate; box-shadow: 0 0 20px #ff6348, 0 0 30px #ff6348; }
@keyframes flicker { 0% { transform: translateX(-50%) scale(1) } 100% { transform: translateX(-50%) scale(1.1) translateY(-2px) } }
.candle.blown .flame { animation: blowOut .5s forwards; }
@keyframes blowOut { 0% { opacity:1; transform: translateX(-50%) scale(1) } 50% { opacity:.5; transform: translateX(-50%) scale(1.4) translateX(20px) } 100% { opacity:0; transform: translateX(-50%) scale(0) } }

/* ---------- Card / title / name ---------- */
.card { background: rgba(255,255,255,0.12); backdrop-filter: blur(14px); border-radius: 24px; padding: 2.6rem; box-shadow: 0 20px 60px rgba(0,0,0,0.3); border: 1px solid rgba(255,255,255,0.06); animation: fadeInUp .9s ease-out; }
@keyframes fadeInUp { from { opacity:0; transform:translateY(30px) } to { opacity:1; transform:translateY(0) } }

.title-container { text-align:center; margin-bottom:1.6rem; perspective:1000px; }
.animated-title { font-size:2.4rem; font-weight:800; color:#fff; text-shadow: 0 0 24px rgba(255,255,255,0.8), 0 0 40px rgba(240,147,251,0.45); display:inline-block; }
.animated-title span { display:inline-block; animation: bounceIn .8s ease-out both; }
@keyframes bounceIn { 0% { opacity:0; transform:scale(0) rotateY(180deg) } 50% { transform:scale(1.1) rotateY(90deg) } 100% { opacity:1; transform:scale(1) rotateY(0deg) } }

.name-container { position:relative; text-align:center; margin-bottom:1.5rem; }
.name-glow { position:absolute; top:50%; left:50%; transform:translate(-50%,-50%); font-size:2.8rem; font-weight:800; color:transparent; text-shadow:0 0 40px rgba(255,255,255,0.85), 0 0 70px rgba(240,147,251,0.6); animation: glow 2s ease-in-out infinite; z-index:0; }
.name-main { font-size:2.8rem; font-weight:800; background: linear-gradient(45deg,#fff,#f093fb,#fff); background-size:200% auto; -webkit-background-clip:text; background-clip:text; -webkit-text-fill-color:transparent; animation: shine 3s linear infinite; position:relative; z-index:1; }
@keyframes glow { 0%,100% { opacity:.55; transform:translate(-50%,-50%) scale(1) } 50% { opacity:1; transform:translate(-50%,-50%) scale(1.06) } }
@keyframes shine { to { background-position:200% center } }

/* ---------- Messages grid & cards ---------- */
.messages-grid { display:grid; grid-template-columns: repeat(auto-fit, minmax(240px,1fr)); gap:1.2rem; margin-bottom:1.6rem; }
.love-card { height:200px; perspective:1000px; cursor:pointer; }
.card-inner { position:relative; width:100%; height:100%; transition: transform .8s; transform-style:preserve-3d; }
.love-card.flipped .card-inner { transform: rotateY(180deg); }
.card-front, .card-back { position:absolute; width:100%; height:100%; backface-visibility:hidden; border-radius:12px; display:flex; flex-direction:column; align-items:center; justify-content:center; padding:1rem; }
.card-front { background: linear-gradient(135deg, rgba(255,255,255,0.06), rgba(255,255,255,0.03)); border:1px solid rgba(255,255,255,0.04); box-shadow: 0 8px 20px rgba(0,0,0,0.2); }
.card-back { background: linear-gradient(135deg, rgba(240,147,251,0.12), rgba(102,126,234,0.12)); transform: rotateY(180deg); border:1px solid rgba(255,255,255,0.04); }
.card-icon { font-size:3.6rem; margin-bottom: .6rem; animation: float 3s ease-in-out infinite; }
@keyframes float { 0%,100% { transform: translateY(0) } 50% { transform: translateY(-10px) } }
.card-hint { color: white; font-size: .9rem; font-style: italic; }
.card-message { color: white; font-size: 1.02rem; text-align:center; line-height:1.5; }

/* ---------- Quote ---------- */
.quote-container { position:relative; padding:1.4rem; margin:1.4rem 0; background: linear-gradient(135deg, rgba(255,255,255,0.06), rgba(255,255,255,0.02)); border-radius:12px; border:1px solid rgba(255,255,255,0.04); }
.quote-mark { font-size:3rem; color: rgba(255,255,255,0.6); font-family: Georgia, serif; position:absolute; top:8px; left:10px; }
.quote-mark.closing { top:auto; bottom:8px; left:auto; right:10px; transform: rotate(180deg); }
.quote-text { color:white; font-size:1.05rem; font-style:italic; line-height:1.6; text-align:center; margin: .6rem 1rem; }

/* ---------- Wish & counter ---------- */
.wish-button { padding: 12px 28px; font-size:1.05rem; font-weight:700; color:white; background: linear-gradient(45deg,var(--accent1),var(--accent2)); border:none; border-radius:999px; cursor:pointer; box-shadow: 0 10px 30px rgba(102,126,234,0.18); }
.wish-button.wished { background: linear-gradient(90deg,#00f2fe,#4facfe); animation: pulse 1.8s infinite; }
@keyframes pulse { 0%,100% { transform:scale(1) } 50% { transform:scale(1.05) } }

.love-counter { text-align:center; margin-top:1.4rem; padding:1.4rem; background: linear-gradient(135deg, rgba(255,255,255,0.06), rgba(255,255,255,0.02)); border-radius:12px; border:1px solid rgba(255,255,255,0.04); }
.counter-number { font-size:2.4rem; font-weight:800; color:white; text-shadow:0 0 20px rgba(255,255,255,0.6); }
.heart-pulse { font-size:1.6rem; animation: pulse 1s infinite; }

/* ---------- Fireworks ---------- */
.fireworks { position:fixed; top:0; left:0; width:100%; height:100%; pointer-events:none; z-index:9999; }
.firework { position:absolute; width:4px; height:4px; border-radius:50%; animation: explode 1.5s ease-out forwards; box-shadow:0 0 20px currentColor; }
@keyframes explode { 0% { opacity:1; transform:scale(1) } 100% { opacity:0; transform:scale(0) } }
.firework:nth-child(odd) { background:#ff6b9d } .firework:nth-child(even) { background:#4facfe } .firework:nth-child(3n) { background:#feca57 }

/* ---------- Responsive ---------- */
@media (max-width: 768px) {
  .animated-title { font-size:1.8rem }
  .name-main, .name-glow { font-size:1.8rem }
  .card { padding:1.4rem }
  .messages-grid { gap:.8rem }
  .music-popup { padding:1.2rem 1rem }
  .music-text { font-size:1rem }
}

/* small helpers */
.instruction { color: white; font-size:1.05rem; font-weight:600; text-shadow:0 0 14px rgba(255,255,255,0.6) }
.player-container { display:none } /* hide any legacy player UI; audio controlled programmatically */
</style>
