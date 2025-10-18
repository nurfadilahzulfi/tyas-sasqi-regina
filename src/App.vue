<template>
  <div class="birthday-container" @click="createHeart">
    <!-- MUSIC START OVERLAY -->
    <div v-if="showMusicPrompt" class="music-overlay" @click.stop>
      <div class="music-popup">
        <p class="music-text">🎵 Musik spesial untuk Sasqi :3</p>
        <p class="music-sub">Klik tombol di bawah untuk memulai musik</p>
        <button @click="startMusic" class="music-start-btn">Putar Musik 🎶</button>
      </div>
    </div>

    <!-- Animated Background -->
    <div class="stars-bg">
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
    >
      {{ heart.emoji }}
    </div>

    <!-- Particles -->
    <div
      v-for="particle in particles"
      :key="particle.id"
      class="particle"
      :style="{
        left: particle.left + '%',
        animationDuration: particle.duration + 's',
        animationDelay: particle.delay + 's'
      }"
    ></div>

    <!-- Main Content -->
    <div :class="['content-wrapper', { show: showContent }]">
      <!-- Cake -->
      <div class="cake-container" @click="blowCandles">
        <div class="cake">
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
            :class="['love-card', { flipped: flippedCards.includes(idx) }]"
            @click="flipCard(idx)"
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
    <div v-if="showFireworks" class="fireworks">
      <div v-for="i in 20" :key="'firework-' + i" class="firework" :style="getFireworkStyle(i)"></div>
    </div>

    <audio ref="audioPlayer" loop preload="auto">
      <source src="/music/bergema.mp3" type="audio/mp3" />
    </audio>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

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

const loveMessages = [
  { icon: '🌹', text: 'Semoga setiap hari kamu dipenuhi hal-hal baik yang bikin kamu tersenyum tanpa henti' },
  { icon: '⭐', text: 'Kamu tuh spesial dengan caramu sendiri, jadi diri sendiri aja terus ya!' },
  { icon: '🦋', text: 'Hidup cuma sekali, jadi lakuin aja yang bikin kamu bahagia' },
  { icon: '🌸', text: 'Apapun yang kamu lakuin itu ada artinya kok, terus maju aja dan jangan takut coba hal baru' },
  { icon: '💝', text: 'Kamu layak dapetin semua kebahagiaan di dunia ini, jangan pernah merasa kurang dari siapa pun' },
  { icon: '🌙', text: 'Di tengah hiruk pikuk dunia, semoga kamu selalu menemukan ketenangan dan jadi tempat pulang yang nyaman buat diri sendiri' }
]

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

const startMusic = async () => {
  try {
    await audioPlayer.value.play()
    showMusicPrompt.value = false
  } catch (err) {
    showMusicPrompt.value = true
  }
}

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
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

/* ===== CONTAINER ===== */
.birthday-container {
  min-height: 100vh;
  width: 100%;
  background: 
    radial-gradient(circle at 20% 80%, rgba(240,147,251,0.3) 0%, transparent 50%),
    radial-gradient(circle at 80% 20%, rgba(79,172,254,0.3) 0%, transparent 50%),
    linear-gradient(135deg, #1a1a2e 0%, #16213e 25%, #0f3460 50%, #533483 75%, #2d1b69 100%);
  background-size: 100% 100%, 100% 100%, 400% 400%;
  animation: gradientShift 20s ease infinite;
  overflow-x: hidden;
  position: relative;
  cursor: pointer;
  padding: 2rem 1rem;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
}

@keyframes gradientShift {
  0%, 100% { background-position: 0% 0%, 0% 0%, 0% 50%; }
  50% { background-position: 0% 0%, 0% 0%, 100% 50%; }
}

.content-wrapper {
  max-width: 900px;
  margin: 0 auto;
  position: relative;
  z-index: 10;
  transform: scale(0.9);
  opacity: 0;
  transition: all 0.9s cubic-bezier(0.34, 1.56, 0.64, 1);
}

.content-wrapper.show {
  transform: scale(1);
  opacity: 1;
}

/* ===== MUSIC OVERLAY ===== */
.music-overlay {
  position: fixed;
  inset: 0;
  background: rgba(4, 6, 15, 0.85);
  backdrop-filter: blur(10px);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 99999;
}

.music-popup {
  background: linear-gradient(135deg, rgba(79, 172, 254, 0.15), rgba(240, 147, 251, 0.15));
  backdrop-filter: blur(20px);
  padding: 2rem 2.5rem;
  border-radius: 28px;
  text-align: center;
  color: #fff;
  max-width: 90%;
  width: 400px;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.5);
  border: 1px solid rgba(255, 255, 255, 0.15);
}

.music-text {
  font-size: 1.25rem;
  font-weight: 700;
  margin-bottom: 0.5rem;
}

.music-sub {
  font-size: 0.95rem;
  color: rgba(255, 255, 255, 0.85);
  margin-bottom: 1.5rem;
}

.music-start-btn {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 50%, #f093fb 100%);
  color: white;
  border: none;
  padding: 12px 28px;
  border-radius: 50px;
  cursor: pointer;
  font-weight: 700;
  font-size: 1rem;
  box-shadow: 0 10px 30px rgba(102, 126, 234, 0.35);
  transition: all 0.3s;
}

.music-start-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 15px 40px rgba(102, 126, 234, 0.45);
}

/* ===== STARS ===== */
.stars-bg {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  z-index: 1;
}

.star {
  position: absolute;
  width: 2px;
  height: 2px;
  background: white;
  border-radius: 50%;
  animation: twinkle 3s infinite ease-in-out;
  box-shadow: 0 0 8px rgba(255, 255, 255, 0.9);
}

@keyframes twinkle {
  0%, 100% { opacity: 0.3; transform: scale(0.8); }
  50% { opacity: 1; transform: scale(1.5); }
}

/* ===== HEARTS & PARTICLES ===== */
.floating-heart {
  position: fixed;
  pointer-events: none;
  animation: floatUp linear;
  z-index: 1000;
}

@keyframes floatUp {
  0% { transform: translateY(0) rotate(0deg); opacity: 1; }
  100% { transform: translateY(-220px) rotate(360deg); opacity: 0; }
}

.particle {
  position: absolute;
  width: 4px;
  height: 4px;
  background: radial-gradient(circle, #fff, #f093fb);
  border-radius: 50%;
  top: -10px;
  animation: particleFall linear infinite;
  box-shadow: 0 0 10px rgba(240, 147, 251, 0.8);
}

@keyframes particleFall {
  to { transform: translateY(100vh) rotate(360deg); opacity: 0; }
}

/* ===== CAKE ===== */
.cake-container {
  text-align: center;
  margin-bottom: 3rem;
  animation: slideDown 1s ease-out;
}

@keyframes slideDown {
  from { transform: translateY(-80px); opacity: 0; }
  to { transform: translateY(0); opacity: 1; }
}

.cake {
  display: inline-block;
  position: relative;
  margin-bottom: 1rem;
  filter: drop-shadow(0 15px 35px rgba(0, 0, 0, 0.4));
}

.cake-layer {
  width: 200px;
  height: 40px;
  margin: 6px auto;
  border-radius: 12px;
  position: relative;
  box-shadow: 
    0 8px 20px rgba(0, 0, 0, 0.35),
    inset 0 2px 0 rgba(255, 255, 255, 0.3);
}

.layer-1 {
  background: linear-gradient(135deg, #ff6b9d 0%, #c44569 100%);
  width: 240px;
}

.layer-2 {
  background: linear-gradient(135deg, #feca57 0%, #ee5a6f 100%);
  width: 220px;
}

.layer-3 {
  background: linear-gradient(135deg, #48dbfb 0%, #0abde3 100%);
}

.frosting {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 8px;
  background: linear-gradient(180deg, #fff 0%, rgba(255, 255, 255, 0.8) 100%);
  border-radius: 12px 12px 0 0;
  opacity: 0.95;
}

.candles {
  display: flex;
  justify-content: space-around;
  position: absolute;
  top: -40px;
  left: 50%;
  transform: translateX(-50%);
  width: 150px;
}

.candle {
  width: 8px;
  height: 30px;
  background: linear-gradient(135deg, #ffeaa7 0%, #fdcb6e 100%);
  border-radius: 4px 4px 0 0;
  position: relative;
  box-shadow: 0 5px 15px rgba(253, 203, 110, 0.3);
}

.flame {
  position: absolute;
  top: -15px;
  left: 50%;
  transform: translateX(-50%);
  width: 12px;
  height: 20px;
  background: radial-gradient(ellipse at center, #fff 0%, #ffeb3b 30%, #ff6348 100%);
  border-radius: 50%/60% 60% 40% 40%;
  animation: flicker 0.3s infinite alternate;
  box-shadow: 0 0 15px #ffeb3b, 0 0 25px #ff6348;
}

@keyframes flicker {
  0% { transform: translateX(-50%) scale(1); }
  100% { transform: translateX(-50%) scale(1.15) translateY(-2px); }
}

.candle.blown .flame {
  animation: blowOut 0.6s forwards;
}

@keyframes blowOut {
  0% { opacity: 1; transform: translateX(-50%) scale(1); }
  50% { opacity: 0.5; transform: translateX(-50%) scale(1.6) translateX(30px) rotate(45deg); }
  100% { opacity: 0; transform: translateX(-50%) scale(0) translateY(-20px); }
}

.instruction {
  color: white;
  font-size: 1.05rem;
  font-weight: 600;
  text-shadow: 0 0 20px rgba(255, 255, 255, 0.8);
  animation: pulse 2s ease-in-out infinite;
}

/* ===== CARD ===== */
.card {
  background: linear-gradient(135deg, rgba(79, 172, 254, 0.1), rgba(240, 147, 251, 0.1));
  backdrop-filter: blur(20px);
  border-radius: 24px;
  padding: 2.5rem;
  box-shadow: 
    0 20px 60px rgba(0, 0, 0, 0.35),
    inset 0 1px 0 rgba(255, 255, 255, 0.1);
  border: 1px solid rgba(255, 255, 255, 0.1);
  animation: fadeInUp 1s ease-out;
}

@keyframes fadeInUp {
  from { opacity: 0; transform: translateY(40px); }
  to { opacity: 1; transform: translateY(0); }
}

.title-container {
  text-align: center;
  margin-bottom: 1.5rem;
}

.animated-title {
  font-size: 2.2rem;
  font-weight: 800;
  color: #fff;
  text-shadow: 0 0 30px rgba(255, 255, 255, 0.9), 0 0 50px rgba(240, 147, 251, 0.6);
  display: inline-block;
}

.animated-title span {
  display: inline-block;
  animation: bounceIn 0.8s ease-out both;
}

@keyframes bounceIn {
  0% { opacity: 0; transform: scale(0) rotateY(180deg); }
  50% { transform: scale(1.15) rotateY(90deg); }
  100% { opacity: 1; transform: scale(1) rotateY(0deg); }
}

.name-container {
  position: relative;
  text-align: center;
  margin-bottom: 1.5rem;
  padding: 1rem 0;
}

.name-glow {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  font-size: 2.5rem;
  font-weight: 800;
  color: transparent;
  text-shadow: 
    0 0 50px rgba(255, 255, 255, 0.9),
    0 0 90px rgba(240, 147, 251, 0.7);
  animation: glow 2.5s ease-in-out infinite;
  z-index: 0;
}

.name-main {
  font-size: 2.5rem;
  font-weight: 800;
  background: linear-gradient(135deg, #fff 0%, #f093fb 25%, #4facfe 50%, #fff 75%, #f093fb 100%);
  background-size: 300% auto;
  -webkit-background-clip: text;
  background-clip: text;
  -webkit-text-fill-color: transparent;
  animation: shine 4s linear infinite;
  position: relative;
  z-index: 1;
  filter: drop-shadow(0 4px 12px rgba(240, 147, 251, 0.4));
}

@keyframes glow {
  0%, 100% { opacity: 0.6; transform: translate(-50%, -50%) scale(1); }
  50% { opacity: 1; transform: translate(-50%, -50%) scale(1.08); }
}

@keyframes shine {
  to { background-position: 300% center; }
}

/* ===== MESSAGE CARDS ===== */
.messages-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  gap: 1.2rem;
  margin-bottom: 1.5rem;
}

.love-card {
  height: 200px;
  perspective: 1200px;
  cursor: pointer;
  transition: transform 0.3s;
}

.love-card:hover {
  transform: translateY(-5px);
}

.card-inner {
  position: relative;
  width: 100%;
  height: 100%;
  transition: transform 0.8s cubic-bezier(0.34, 1.56, 0.64, 1);
  transform-style: preserve-3d;
}

.love-card.flipped .card-inner {
  transform: rotateY(180deg);
}

.card-front,
.card-back {
  position: absolute;
  width: 100%;
  height: 100%;
  backface-visibility: hidden;
  border-radius: 16px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 1rem;
}

.card-front {
  background: linear-gradient(135deg, rgba(79, 172, 254, 0.12), rgba(240, 147, 251, 0.08));
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.15);
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
}

.card-back {
  background: linear-gradient(135deg, rgba(240, 147, 251, 0.18), rgba(102, 126, 234, 0.18));
  backdrop-filter: blur(15px);
  transform: rotateY(180deg);
  border: 1px solid rgba(255, 255, 255, 0.15);
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
}

.card-icon {
  font-size: 3.5rem;
  margin-bottom: 0.5rem;
  animation: float 3s ease-in-out infinite;
  filter: drop-shadow(0 4px 12px rgba(255, 255, 255, 0.3));
}

@keyframes float {
  0%, 100% { transform: translateY(0) rotate(0deg); }
  50% { transform: translateY(-12px) rotate(5deg); }
}

.card-hint {
  color: rgba(255, 255, 255, 0.9);
  font-size: 0.9rem;
  font-style: italic;
}

.card-message {
  color: white;
  font-size: 1rem;
  text-align: center;
  line-height: 1.6;
  text-shadow: 0 2px 10px rgba(0, 0, 0, 0.3);
}

/* ===== QUOTE ===== */
.quote-container {
  position: relative;
  padding: 1.5rem;
  margin: 1.5rem 0;
  background: linear-gradient(135deg, rgba(79, 172, 254, 0.1), rgba(240, 147, 251, 0.08));
  backdrop-filter: blur(10px);
  border-radius: 16px;
  border: 1px solid rgba(255, 255, 255, 0.12);
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.25);
}

.quote-mark {
  font-size: 3rem;
  color: rgba(255, 255, 255, 0.3);
  font-family: Georgia, serif;
  position: absolute;
  top: 8px;
  left: 10px;
  line-height: 1;
}

.quote-mark.closing {
  top: auto;
  bottom: 8px;
  left: auto;
  right: 10px;
  transform: rotate(180deg);
}

.quote-text {
  color: white;
  font-size: 1.05rem;
  font-style: italic;
  line-height: 1.7;
  text-align: center;
  margin: 0.5rem 1rem;
  text-shadow: 0 2px 10px rgba(0, 0, 0, 0.3);
}

/* ===== WISH BUTTON ===== */
.wish-section {
  text-align: center;
  margin: 1.5rem 0;
}

.wish-button {
  padding: 14px 32px;
  font-size: 1.05rem;
  font-weight: 700;
  color: white;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 50%, #f093fb 100%);
  border: none;
  border-radius: 50px;
  cursor: pointer;
  box-shadow: 0 12px 35px rgba(102, 126, 234, 0.35);
  transition: all 0.3s;
}

.wish-button:hover {
  transform: translateY(-3px) scale(1.05);
  box-shadow: 0 15px 45px rgba(102, 126, 234, 0.45);
}

.wish-button.wished {
  background: linear-gradient(135deg, #00f2fe 0%, #4facfe 50%, #00b8d4 100%);
  animation: wishPulse 2s infinite;
}

@keyframes wishPulse {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.05); }
}

/* ===== LOVE COUNTER ===== */
.love-counter {
  text-align: center;
  margin-top: 1.5rem;
  padding: 1.5rem;
  background: linear-gradient(135deg, rgba(79, 172, 254, 0.1), rgba(240, 147, 251, 0.08));
  backdrop-filter: blur(10px);
  border-radius: 16px;
  border: 1px solid rgba(255, 255, 255, 0.12);
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.25);
}

.counter-display {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 1rem;
  margin-bottom: 0.5rem;
}

.counter-number {
  font-size: 2.5rem;
  font-weight: 800;
  color: white;
  text-shadow: 0 0 25px rgba(255, 255, 255, 0.8);
}

.heart-pulse {
  font-size: 1.5rem;
  animation: heartbeat 1.2s infinite;
  display: inline-block;
}

@keyframes heartbeat {
  0%, 100% { transform: scale(1); }
  25% { transform: scale(1.2); }
  50% { transform: scale(1); }
}

.counter-text {
  color: rgba(255, 255, 255, 0.9);
  font-size: 1rem;
  font-weight: 600;
  text-shadow: 0 2px 8px rgba(0, 0, 0, 0.3);
}

/* ===== FIREWORKS ===== */
.fireworks {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  z-index: 9999;
}

.firework {
  position: absolute;
  width: 4px;
  height: 4px;
  border-radius: 50%;
  animation: explode 1.8s ease-out forwards;
  box-shadow: 0 0 20px currentColor;
}

@keyframes explode {
  0% { opacity: 1; transform: scale(1); }
  50% { opacity: 0.8; transform: scale(20); }
  100% { opacity: 0; transform: scale(0); }
}

.firework:nth-child(odd) { background: #ff6b9d; }
.firework:nth-child(even) { background: #4facfe; }
.firework:nth-child(3n) { background: #feca57; }
.firework:nth-child(4n) { background: #48dbfb; }
.firework:nth-child(5n) { background: #f093fb; }

/* ===== RESPONSIVE ===== */
/* Tablet */
@media (max-width: 768px) {
  .birthday-container {
    padding: 1.5rem 1rem;
  }

  .card {
    padding: 2rem 1.5rem;
  }

  .animated-title {
    font-size: 1.8rem;
  }

  .name-main,
  .name-glow {
    font-size: 2rem;
  }

  .messages-grid {
    grid-template-columns: 1fr;
    gap: 1rem;
  }

  .love-card {
    height: 180px;
  }

  .card-icon {
    font-size: 3rem;
  }

  .quote-text {
    font-size: 0.95rem;
  }
}

/* Mobile (iPhone, Android) */
@media (max-width: 480px) {
  .birthday-container {
    padding: 1rem 0.75rem;
  }

  .content-wrapper {
    padding: 0;
  }

  .card {
    padding: 1.5rem 1rem;
    border-radius: 20px;
    margin-bottom: 1rem;
  }

  .animated-title {
    font-size: 1.5rem;
  }

  .name-main,
  .name-glow {
    font-size: 1.6rem;
    letter-spacing: 1px;
  }

  .messages-grid {
    gap: 0.85rem;
  }

  .love-card {
    height: 170px;
  }

  .card-icon {
    font-size: 2.5rem;
  }

  .card-message {
    font-size: 0.9rem;
    line-height: 1.5;
  }

  .quote-container {
    padding: 1.2rem 1rem;
    margin: 1.2rem 0;
    border-radius: 14px;
  }

  .quote-text {
    font-size: 0.9rem;
    line-height: 1.6;
    margin: 0.5rem 0.75rem;
  }

  .quote-mark {
    font-size: 2.2rem;
  }

  .wish-button {
    padding: 12px 24px;
    font-size: 0.95rem;
  }

  .love-counter {
    padding: 1.2rem;
    margin-top: 1.2rem;
  }

  .counter-number {
    font-size: 2rem;
  }

  .heart-pulse {
    font-size: 1.3rem;
  }

  .counter-text {
    font-size: 0.9rem;
  }

  .cake-layer {
    width: 160px;
    height: 32px;
    margin: 5px auto;
  }

  .layer-1 {
    width: 192px;
  }

  .layer-2 {
    width: 176px;
  }

  .candles {
    width: 120px;
    top: -32px;
  }

  .candle {
    width: 6px;
    height: 24px;
  }

  .flame {
    width: 10px;
    height: 16px;
    top: -12px;
  }

  .instruction {
    font-size: 0.95rem;
  }

  .cake-container {
    margin-bottom: 2rem;
  }

  .music-popup {
    padding: 1.5rem 1.75rem;
    width: 90%;
  }

  .music-text {
    font-size: 1.1rem;
  }

  .music-sub {
    font-size: 0.875rem;
  }

  .music-start-btn {
    padding: 10px 24px;
    font-size: 0.95rem;
  }
}

/* Extra small phones */
@media (max-width: 360px) {
  .card {
    padding: 1.25rem 0.85rem;
  }

  .animated-title {
    font-size: 1.3rem;
  }

  .name-main,
  .name-glow {
    font-size: 1.4rem;
  }

  .love-card {
    height: 160px;
  }

  .card-icon {
    font-size: 2.2rem;
  }

  .card-message {
    font-size: 0.85rem;
  }

  .quote-text {
    font-size: 0.85rem;
  }

  .cake-layer {
    width: 140px;
    height: 28px;
  }

  .layer-1 {
    width: 168px;
  }

  .layer-2 {
    width: 154px;
  }
}

/* Landscape mode */
@media (max-height: 500px) and (orientation: landscape) {
  .birthday-container {
    padding: 1rem 1.5rem;
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

  .messages-grid {
    grid-template-columns: repeat(2, 1fr);
    gap: 1rem;
  }

  .love-card {
    height: 140px;
  }

  .quote-container,
  .love-counter {
    padding: 1rem;
    margin: 1rem 0;
  }
}

/* Desktop */
@media (min-width: 1024px) {
  .messages-grid {
    grid-template-columns: repeat(3, 1fr);
  }

  .content-wrapper {
    max-width: 900px;
  }
}

/* Large desktop */
@media (min-width: 1440px) {
  .content-wrapper {
    max-width: 1000px;
  }

  .card {
    padding: 3rem;
  }
}
</style>