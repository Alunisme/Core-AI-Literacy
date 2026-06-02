<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'
import fake4Img from './assets/fake4.PNG'
import fake6Img from './assets/fake6.PNG'

const scrollTo = (id) => {
  const element = document.getElementById(id)
  if (element) {
    element.scrollIntoView({ behavior: 'smooth' })
  }
}

// --- 素養評測 (Quiz) 邏輯 ---
const isQuizStarted = ref(false)
const isQuizFinished = ref(false)
const currentQuestionIndex = ref(0)
const userAnswers = ref([])
const shuffledQuestions = ref([])

const questions = [
  {
    title: '深偽影像辨識',
    scenario: '情境：在社群網路上看到一段知名政治人物發表爭議言論的影片，畫面看起來非常真實，但你懷疑是 AI 變造的。',
    question: '問題：下列哪一個方法最不適合用來初步檢驗該影片的真偽？',
    options: [
      { value: 'A', text: '(A) 仔細觀察人物眨眼頻率是否異常、邊緣是否有毛邊' },
      { value: 'B', text: '(B) 尋找影片中是否有可疑的環境光影或陰影不連續' },
      { value: 'C', text: '(C) 觀察說話時的嘴型與聲音是否完全同步' },
      { value: 'D', text: '(D) 直接在影片下方留言詢問發文者這是不是真的。' }
    ],
    answer: 'D',
    explanation: '發文者本身可能是造假者，或者他也不知情，因此直接詢問並不能作為客觀檢驗真偽的可靠方法。應該優先尋找影像本身的異常痕跡。'
  },
  {
    title: '生成式 AI 的資訊思辨',
    scenario: '情境：你使用 AI 寫一份報告，AI 給出了一段看似非常專業的歷史文獻引用與數據。',
    question: '問題：為了展現良好的「AI 識讀能力」，你該採取什麼行動？',
    options: [
      { value: 'A', text: '(A) 獨立去搜尋引擎或圖書館查證該文獻與數據的真實性，因為 AI 可能會產生「幻覺」' },
      { value: 'B', text: '(B) 只要 AI 說得很有條理，就可以直接複製貼上' },
      { value: 'C', text: '(C) 換另外一家公司的 AI 再問一次，如果回答一樣就代表是真的' },
      { value: 'D', text: '(D) 檢查 AI 回覆的字數是否符合預期。' }
    ],
    answer: 'A',
    explanation: 'AI 模型偶爾會產生「幻覺」，也就是用極其自信的語氣給出虛構的資訊或不存在的文獻。因此，面對重要的數據與文獻，務必進行獨立的交叉查證。'
  },
  {
    title: 'AI 偏見與資訊識讀',
    scenario: '情境：某公司全面使用 AI 來篩選求職者履歷，但事後發現被錄取的人選具備高度的性別與地域單一性。',
    question: '問題：這反映了 AI 識讀中的哪一個核心觀念？',
    options: [
      { value: 'A', text: '(A) AI 的運算速度不夠快' },
      { value: 'B', text: '(B) AI 的產出取決於訓練數據，若數據帶有偏見，AI 就會放大偏見' },
      { value: 'C', text: '(C) 應該完全禁用 AI 篩選履歷' },
      { value: 'D', text: '(D) 求職者的履歷寫得不夠符合 AI 的關鍵字。' }
    ],
    answer: 'B',
    explanation: 'AI 模型是基於過往的歷史數據進行訓練的。如果過去的招募數據本身就偏好某個性別或地區，AI 就會學到這個偏見並將其放大，這正是 AI 倫理與偏見的核心議題。'
  }
]

const shuffleArray = (array) => {
  const newArray = [...array]
  for (let i = newArray.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [newArray[i], newArray[j]] = [newArray[j], newArray[i]];
  }
  return newArray
}

const startQuiz = () => {
  isQuizStarted.value = true
  isQuizFinished.value = false
  currentQuestionIndex.value = 0
  userAnswers.value = []
  shuffledQuestions.value = shuffleArray(questions)
}

const selectOption = (optionValue) => {
  userAnswers.value[currentQuestionIndex.value] = optionValue
}

const nextQuestion = () => {
  if (currentQuestionIndex.value < shuffledQuestions.value.length - 1) {
    currentQuestionIndex.value++
  } else {
    isQuizFinished.value = true
    isQuizStarted.value = false
  }
}

const score = computed(() => {
  return userAnswers.value.reduce((total, answer, index) => {
    return answer === shuffledQuestions.value[index].answer ? total + 1 : total
  }, 0)
})

const incorrectDetails = computed(() => {
  return shuffledQuestions.value.map((q, index) => {
    const userAnswer = userAnswers.value[index]
    if (userAnswer !== q.answer) {
      const userOpt = q.options.find(o => o.value === userAnswer)
      const correctOpt = q.options.find(o => o.value === q.answer)
      return {
        ...q,
        userAnswerText: userOpt ? userOpt.text : '未作答',
        correctAnswerText: correctOpt ? correctOpt.text : ''
      }
    }
    return null
  }).filter(item => item !== null)
})

const resultData = computed(() => {
  const s = score.value
  if (s === 3) {
    return { title: '👑 頂尖 AI 識讀大師', desc: '完美！您具備極高的批判性思考能力，完全能駕馭 AI 帶來的挑戰，是現代數位公民的典範！' }
  } else if (s === 2) {
    return { title: '🛡️ 科技防禦先鋒', desc: '很不錯！您具備良好的防範意識，只需要在特定情境下再多一層確認，就能避免落入 AI 陷阱。' }
  } else {
    return { title: '🌱 AI 探索新手', desc: '您正在建立基礎觀念！AI 發展日新月異，建議多多複習我們的「識讀指南」，提升對資訊真偽的敏銳度喔！' }
  }
})

const restartQuiz = () => {
  isQuizStarted.value = false
  isQuizFinished.value = false
  // 直接呼叫開始測驗，它會幫你重設所有變數並重新洗牌
  startQuiz()
}

// --- 圖片輪播邏輯 ---
const carouselIndex = ref(0)
let carouselTimer = null

const startCarouselTimer = () => {
  carouselTimer = setInterval(() => {
    carouselIndex.value = (carouselIndex.value + 1) % 2
  }, 5000) // 每 5 秒自動切換
}

const resetCarouselTimer = () => {
  if (carouselTimer) clearInterval(carouselTimer)
  startCarouselTimer()
}

const nextSlide = () => {
  carouselIndex.value = (carouselIndex.value + 1) % 2
  resetCarouselTimer()
}
const prevSlide = () => {
  carouselIndex.value = (carouselIndex.value - 1 + 2) % 2
  resetCarouselTimer()
}
const goToSlide = (index) => {
  carouselIndex.value = index
  resetCarouselTimer()
}

// --- 介面互動效果邏輯 ---
const showBackToTop = ref(false)
const activeSection = ref('about')

const handleScroll = () => {
  showBackToTop.value = window.scrollY > 300
}

const scrollToTop = () => {
  window.scrollTo({ top: 0, behavior: 'smooth' })
}

onMounted(() => {
  window.addEventListener('scroll', handleScroll)
  
  // 啟動輪播定時器
  startCarouselTimer()
  
  // 滾動進場動畫觀察器 (Intersection Observer)
  const observer = new IntersectionObserver((entries) => {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        entry.target.classList.add('is-revealed')
      }
    })
  }, { threshold: 0.15, rootMargin: '0px 0px -50px 0px' })

  // 監聽所有帶有 reveal-target 的元素
  document.querySelectorAll('.reveal-target').forEach(el => observer.observe(el))

  // 導覽列當前區塊觀察器 (Scrollspy)
  const sectionObserver = new IntersectionObserver((entries) => {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        activeSection.value = entry.target.id
      }
    })
  }, { rootMargin: '-20% 0px -70% 0px' }) // 當區塊進入畫面上半部時觸發
  document.querySelectorAll('.section').forEach(el => sectionObserver.observe(el))
})

onUnmounted(() => {
  window.removeEventListener('scroll', handleScroll)
  if (carouselTimer) clearInterval(carouselTimer)
})
</script>

<template>
  <div class="app-container">
    <!-- 頂部固定導覽列 -->
    <nav class="navbar">
      <div class="logo">Core AI Literacy</div>
      <ul class="nav-links">
        <li><a href="#about" :class="{ active: activeSection === 'about' }" @click.prevent="scrollTo('about')">關於本站</a></li>
        <li><a href="#insights" :class="{ active: activeSection === 'insights' }" @click.prevent="scrollTo('insights')">識讀指南</a></li>
        <li><a href="#assessment" :class="{ active: activeSection === 'assessment' }" @click.prevent="scrollTo('assessment')">素養評測</a></li>
      </ul>
    </nav>

    <main>
      <!-- 關於本站 (About) -->
      <section id="about" class="section about-section">
        <h1 class="glow-text reveal-target">Core AI Literacy</h1>
        <p class="subtitle reveal-target" style="transition-delay: 0.1s;">建構您的 AI 核心素養，駕馭未來科技</p>
        
        <div class="card glass-panel reveal-target" style="transition-delay: 0.2s;">
          <div class="about-content">
            <h2>本站的使命：普及 AI 識讀</h2>
            <p class="lead-text">
              在人工智慧技術爆發、真假資訊交錯的時代，我們看見了一個迫切的需求。我們建立這個網站的終極目標，就是希望讓<strong>「所有人」</strong>都能夠認識並掌握<strong>「AI 識讀（AI Literacy）」</strong>這項關鍵能力。
            </p>
            <div class="highlight-box">
              💡 為什麼 AI 識讀如此重要？<br>
              因為在可見的未來，AI 將深度介入我們的生活。您對 AI 的認知與識讀素養，將直接影響您未來在面對海量資訊時的<strong>判斷力、決策品質，以及保護自身權益的能力</strong>。
            </div>
            <p>
              我們不只要探討如何使用科技，更要與您一起培養「批判性思考」的習慣。透過網站中的真實案例剖析與情境評測，我們期盼能帶您看穿 AI 的盲區與偏見，培育具備獨立思考能力的現代數位公民。
            </p>
            <p class="mission-statement">
              讓我們一起駕馭科技，而不被科技所駕馭！
            </p>
          </div>
        </div>
      </section>

      <!-- 識讀指南 (Insights) -->
      <section id="insights" class="section insights-section">
        <h2 class="glow-text reveal-target">識讀指南 (Insights)</h2>
        <p class="subtitle reveal-target" style="transition-delay: 0.1s;">透過真實案例，培養您的科技洞察力</p>
        
        <div class="insights-list">
          <!-- 案例 1 -->
          <div class="insight-card glass-panel reveal-target">
            <div class="insight-image-wrapper">
              <img src="./assets/fake1.PNG" alt="破綻百出的物理法則" />
            </div>
            <div class="insight-content">
              <h3>🚨 破綻百出的物理法則</h3>
              <p>畫面初始看似真實，但隨著延長線突兀且誇張地燃燒爆炸，便暴露出 AI 難以合理化物理情境與連貫邏輯的弱點。許多 AI 生成工具在處理複雜動態時，仍會出現明顯的邏輯斷層。</p>
            </div>
          </div>

          <!-- 案例 2 -->
          <div class="insight-card glass-panel reverse reveal-target">
            <div class="insight-image-wrapper">
              <img src="./assets/fake2.PNG" alt="負責任的自律標示" />
            </div>
            <div class="insight-content">
              <h3>✅ 負責任的自律標示</h3>
              <p>仔細觀察肢體動作即可察覺其為 AI 生成。此類內容若能主動於下方標註「AI 生成」，不僅有助於資訊透明，更是優良數位創作者建立 AI 倫理與防範誤導的好榜樣。</p>
            </div>
          </div>

          <!-- 案例 3 & 4 (對比) -->
          <div class="insight-card glass-panel column-card reveal-target">
            <div class="card-header">
              <h3>👁️ 專業知識的壁壘與盲區</h3>
            </div>
            <div class="comparison-grid">
              <!-- 上半部：專家視角 -->
              <div class="comparison-item">
                <img src="./assets/fake3.PNG" alt="專家視角" />
              </div>
              <div class="comparison-item flex-center">
                <div class="detail-section">
                  <h4>📌 專家的銳利之眼（左圖）</h4>
                  <p>如左方所展示的，對於具備相關專業（如建築、土木）背景的人而言，影片中不符合物理常理與實務邏輯的施工流程，便成為一眼識破 AI 的突破口。他們能馬上察覺破綻，指出這是不可能實現的危險工法。</p>
                </div>
              </div>
              
              <!-- 下半部：一般觀眾 (輪播圖) -->
              <div class="comparison-item">
                <div class="carousel-container">
                  <transition name="fade-carousel" mode="out-in">
                    <img :key="carouselIndex" :src="carouselIndex === 0 ? fake4Img : fake6Img" alt="資訊盲區留言" class="carousel-img" />
                  </transition>

                  <button class="carousel-btn prev-btn" @click="prevSlide" aria-label="上一張">
                    <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><path d="m15 18-6-6 6-6"/></svg>
                  </button>
                  <button class="carousel-btn next-btn" @click="nextSlide" aria-label="下一張">
                    <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><path d="m9 18 6-6-6-6"/></svg>
                  </button>
                  
                  <div class="carousel-indicators">
                    <span :class="{ active: carouselIndex === 0 }" @click="goToSlide(0)"></span>
                    <span :class="{ active: carouselIndex === 1 }" @click="goToSlide(1)"></span>
                  </div>
                </div>
              </div>
              <div class="comparison-item flex-center">
                <div class="detail-section">
                  <h4>📌 一般觀眾的資訊盲區（左方留言）</h4>
                  <p>然而，透過左方觀眾的留言區可以觀察到，當缺乏該領域知識的觀眾，面對這類主打「奇觀、紓壓」且「未主動標註 AI 生成」的影片時，便容易陷入迷惘與混淆。許多人開始詢問「這是真的嗎？」、「管線是牽給誰的？」、「電是從哪裡來的？」。</p>
                  <br>
                  <p>這強烈凸顯了：在跨領域的資訊汪洋中，我們不能僅仰賴個人既有的常識，更需要建立系統化的 AI 識讀能力，才能避免落入未知的認知盲區。</p>
                </div>
              </div>
            </div>
          </div>

          <!-- 案例 5 (超級大重點) -->
          <div class="insight-card glass-panel highlight-card reverse reveal-target">
            <div class="insight-image-wrapper">
              <img src="./assets/true.PNG" alt="真實作品的信任危機" />
            </div>
            <div class="insight-content">
              <h3>💔 虛實難辨下的信任危機</h3>
              <p>這是當代最諷刺的現象之一。當 AI 生成技術越發強大，許多創作者嘔心瀝血、在天時地利下捕捉的真實絕美作品，反而被大眾誤認為是 AI 產製，引發了對真實事物的信任危機。這也是為什麼我們需要提升識讀素養，將尊嚴與價值還給真實的創作者。</p>
            </div>
          </div>
        </div>
      </section>

      <!-- 素養評測 (Assessment) -->
      <section id="assessment" class="section assessment-section">
        <h2 class="glow-text reveal-target">素養評測 (Assessment)</h2>
        <p class="subtitle reveal-target" style="transition-delay: 0.1s;">準備好檢驗您的 AI 識讀能力了嗎？</p>
        
        <!-- 1. 測驗入口 -->
        <div v-if="!isQuizStarted && !isQuizFinished" class="card glass-panel text-center reveal-target" style="transition-delay: 0.2s;">
          <p>透過專業設計的情境題，快速了解您目前的 AI 核心素養等級，並獲得專屬的學習建議。</p>
          <button class="cta-button" @click="startQuiz">開始測驗</button>
        </div>

        <!-- 2. 測驗進行中 -->
        <div v-else-if="isQuizStarted" class="card glass-panel quiz-panel">
          <div class="quiz-header">
            <span>當前進度：{{ currentQuestionIndex + 1 }} / {{ shuffledQuestions.length }}</span>
          </div>
          <h3 class="quiz-title">第 {{ currentQuestionIndex + 1 }} 題：{{ shuffledQuestions[currentQuestionIndex].title }}</h3>
          <p class="quiz-scenario">{{ shuffledQuestions[currentQuestionIndex].scenario }}</p>
          <p class="quiz-question">{{ shuffledQuestions[currentQuestionIndex].question }}</p>
          
          <div class="options-container">
            <button 
              v-for="option in shuffledQuestions[currentQuestionIndex].options" 
              :key="option.value"
              class="option-button"
              :class="{ selected: userAnswers[currentQuestionIndex] === option.value }"
              @click="selectOption(option.value)"
            >
              {{ option.text }}
            </button>
          </div>
          
          <div class="quiz-footer">
            <button class="cta-button next-button" :disabled="!userAnswers[currentQuestionIndex]" @click="nextQuestion">
              {{ currentQuestionIndex < shuffledQuestions.length - 1 ? '下一題' : '查看結果' }}
            </button>
          </div>
        </div>

        <!-- 3. 測驗結果 -->
        <div v-else-if="isQuizFinished" class="card glass-panel text-center result-panel">
          <div class="score-display">答對 {{ score }} / {{ shuffledQuestions.length }} 題</div>
          <h2 class="result-title">{{ resultData.title }}</h2>
          <p class="result-desc">{{ resultData.desc }}</p>
          
          <!-- 創站初衷區塊 -->
          <div class="creator-message">
            <h3>✨ 創站初衷</h3>
            <p>
              在這個 AI 快速重塑世界的時代，我們深信「AI 識讀」不應只是少數專家的專利，而是每位數位公民導航未來的必備指南。期盼這個網站能成為您探索科技浪潮的燈塔，幫助每個人在享受便利的同時，也能保有清醒的思辨力——善用科技，而不被科技所迷惘。讓我們共同攜手，迎向更具智慧與溫度的未來。
            </p>
          </div>

          <!-- 錯題回顧區塊 -->
          <div v-if="incorrectDetails.length > 0" class="incorrect-reviews">
            <h3 class="review-title">錯題回顧與解析</h3>
            <div v-for="(item, index) in incorrectDetails" :key="index" class="review-item">
              <h4 class="review-q-title">{{ item.title }}</h4>
              <p class="review-question">{{ item.question }}</p>
              <div class="review-answers">
                <p class="wrong-answer">❌ 您的選擇：{{ item.userAnswerText }}</p>
                <p class="correct-answer">✅ 正確解答：{{ item.correctAnswerText }}</p>
              </div>
              <div class="explanation">
                <strong>💡 解析：</strong>{{ item.explanation }}
              </div>
            </div>
          </div>

          <button class="cta-button" @click="restartQuiz">重新測驗</button>
        </div>
      </section>
    </main>

    <!-- 懸浮回到頂部按鈕 -->
    <button class="back-to-top" :class="{ 'show': showBackToTop }" @click="scrollToTop">
      <svg xmlns="http://www.0000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><path d="m18 15-6-6-6 6"/></svg>
    </button>

    <!-- 頁尾 Footer -->
    <footer class="footer">
      <div class="footer-logo">Core AI Literacy</div>
      <p>致力於普及 AI 識讀，培育現代獨立思考的數位公民。</p>
    </footer>
  </div>
</template>

<style>
/* 全局重置與基礎設定 */
:root {
  ---color: #0b0f19; /* 深色科技背景 */
  --nav-bg: rgba(11, 15, 25, 0.75);
  --text-main: #f8fafc;
  --text-muted: #94a3b8;
  --accent-cyan: #06b6d4;
  --accent-purple: #8b5cf6;
}

html {
  scroll-behavior: smooth;
}

body {
  margin: 0;
  padding: 0;
  font-family: 'Inter', 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background-color: var(--bg-color);
  /* 使用漸層疊加深色遮罩，確保背景圖片不會太亮影響文字閱讀 */
  background-image: linear-gradient(rgba(11, 15, 25, 0.7), rgba(11, 15, 25, 0.9)), url('./assets/bg.png');
  background-size: cover;
  background-position: center;
  background-attachment: fixed; /* 背景固定，創造視差滾動感 */
  color: var(--text-main);
  overflow-x: hidden;
}

/* 頂部導覽列 */
.navbar {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 70px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0 8%;
  background: var(--nav-bg);
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  box-sizing: border-box;
  z-index: 1000;
  border-bottom: 1px solid rgba(255, 255, 255, 0.08);
}

.logo {
  font-size: 1.5rem;
  font-weight: 800;
  background: linear-gradient(90deg, var(--accent-cyan), var(--accent-purple));
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  letter-spacing: 1px;
}

.nav-links {
  list-style: none;
  display: flex;
  gap: 2.5rem;
  margin: 0;
  padding: 0;
}

.nav-links a {
  color: var(--text-main);
  text-decoration: none;
  font-size: 1.05rem;
  font-weight: 500;
  transition: color 0.3s ease;
}

.nav-links a:hover {
  color: var(--accent-cyan);
}

.nav-links a.active {
  color: var(--accent-cyan);
  font-weight: 700;
  position: relative;
}

.nav-links a.active::after {
  content: '';
  position: absolute;
  bottom: -6px;
  left: 0;
  width: 100%;
  height: 2px;
  background: var(--accent-cyan);
  border-radius: 2px;
  box-shadow: 0 0 10px var(--accent-cyan);
}

/* 區塊排版 */
.section {
  min-height: 100vh;
  padding: 120px 5% 60px; /* 縮小左右內邊距，讓畫面更寬更滿 */
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.text-center {
  text-align: center;
}

/* 文字與漸層發光特效 */
.glow-text {
  font-size: 3.5rem;
  line-height: 1.3; /* 增加行高避免字體上下被裁切 */
  margin: 0 0 1rem 0;
  padding: 10px; /* 增加邊距避免漸層文字背景在部分瀏覽器被截斷 */
  background: linear-gradient(135deg, #ffffff, var(--accent-cyan));
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  text-shadow: 0 0 40px rgba(6, 182, 212, 0.4);
  text-align: center;
}

.subtitle {
  font-size: 1.25rem;
  color: var(--text-muted);
  margin-bottom: 3.5rem;
  text-align: center;
}

/* 卡片與毛玻璃特效 (Glassmorphism) */
.glass-panel {
  background: rgba(255, 255, 255, 0.03);
  border: 1px solid rgba(255, 255, 255, 0.08);
  backdrop-filter: blur(16px);
  -webkit-backdrop-filter: blur(16px);
  border-radius: 20px;
  box-shadow: 0 15px 35px rgba(0, 0, 0, 0.4);
}

.card {
  padding: 3rem;
  max-width: 1200px; /* 再次放寬卡片的最大寬度，提升整體氣勢 */
  width: 100%;
  box-sizing: border-box;
}

.card h2 {
  margin-top: 0;
  color: var(--accent-purple);
  font-size: 2rem;
}

.card p {
  line-height: 1.8;
  font-size: 1.15rem;
  color: var(--text-muted);
  margin-bottom: 0;
}

/* 網格系統 (針對 Insights 區塊) */
.grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 2rem;
  width: 100%;
  max-width: 1400px; /* 放寬網格的最大寬度 */
}

.grid-item {
  padding: 2.5rem 2rem;
  transition: transform 0.4s ease, border-color 0.4s ease, box-shadow 0.4s ease;
}

.grid-item:hover {
  transform: translateY(-8px);
  border-color: rgba(6, 182, 212, 0.4);
  box-shadow: 0 15px 30px rgba(6, 182, 212, 0.15);
}

.grid-item h3 {
  color: var(--accent-cyan);
  font-size: 1.5rem;
  margin-top: 0;
  margin-bottom: 1rem;
}

.grid-item p {
  color: var(--text-muted);
  line-height: 1.7;
  margin: 0;
}

/* Insights 案例列表樣式 */
.insights-list {
  display: flex;
  flex-direction: column;
  gap: 3rem;
  width: 100%;
  max-width: 1400px; /* 大幅放寬指南卡片的寬度，避免圖片縮太小 */
}

.insight-card {
  display: flex;
  flex-direction: column;
  gap: 2.5rem;
  padding: 2.5rem;
  transition: transform 0.4s ease, border-color 0.4s ease, box-shadow 0.4s ease;
}

@media (min-width: 768px) {
  .insight-card {
    flex-direction: row;
    align-items: center;
  }
  .insight-card.reverse {
    flex-direction: row-reverse;
  }
  .insight-image-wrapper, .insight-content {
    flex: 1;
    min-width: 0; /* 關鍵修正：防止圖片過大撐破 flex 容器導致與文字重疊 */
  }
}

.insight-image-wrapper {
  overflow: hidden;
  border-radius: 12px;
}

.insight-image-wrapper img {
  width: 100%;
  border-radius: 12px;
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.3);
  border: 1px solid rgba(255, 255, 255, 0.1);
  object-fit: cover;
  transition: transform 0.6s cubic-bezier(0.25, 0.46, 0.45, 0.94); /* 平滑圖片縮放過渡 */
}

/* 滑鼠懸停時圖片放大特效 */
.insight-card:hover .insight-image-wrapper img {
  transform: scale(1.05);
}

.insight-content h3 {
  font-size: 1.6rem;
  color: var(--accent-cyan);
  margin-top: 0;
  margin-bottom: 1.2rem;
}

.insight-content p {
  color: var(--text-muted);
  font-size: 1.2rem; /* 微微調大文字以平衡放大的版面 */
  line-height: 1.8;
  margin: 0;
}

/* --- 案例 3 & 4 網格排版專用 --- */
.column-card {
  flex-direction: column !important; /* 強制覆蓋，讓這張卡片主軸為垂直 */
  align-items: stretch !important;
}

.card-header h3 {
  font-size: 1.8rem;
  color: var(--accent-cyan);
  margin: 0 0 1.5rem 0;
}

.comparison-grid {
  display: grid;
  grid-template-columns: 1fr;
  gap: 2rem;
}

@media (min-width: 768px) {
  .comparison-grid {
    grid-template-columns: 1fr 1fr; /* 左右分兩欄 */
    gap: 3rem;
  }
}

.comparison-item img {
  width: 100%;
  border-radius: 12px;
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.3);
  border: 1px solid rgba(255, 255, 255, 0.1);
}

/* --- 圖片輪播樣式 --- */
.carousel-container {
  position: relative;
  width: 100%;
  border-radius: 12px;
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
  background: rgba(0, 0, 0, 0.2); /* 載入動畫時的深色底 */
}

.carousel-img {
  width: 100%;
  border-radius: 12px;
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.3);
  border: 1px solid rgba(255, 255, 255, 0.1);
}

.carousel-btn {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  background: rgba(11, 15, 25, 0.75);
  color: white;
  border: 1px solid rgba(255, 255, 255, 0.15);
  width: 40px;
  height: 40px;
  border-radius: 50%;
  display: flex;
  justify-content: center;
  align-items: center;
  cursor: pointer;
  backdrop-filter: blur(8px);
  transition: all 0.3s ease;
  z-index: 10;
}

.carousel-btn:hover {
  background: var(--accent-cyan);
  transform: translateY(-50%) scale(1.1);
  box-shadow: 0 0 15px rgba(6, 182, 212, 0.5);
}

.prev-btn { left: 15px; }
.next-btn { right: 15px; }

.carousel-indicators {
  position: absolute;
  bottom: 15px;
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  gap: 10px;
  z-index: 10;
  background: rgba(0, 0, 0, 0.5);
  padding: 6px 12px;
  border-radius: 20px;
  backdrop-filter: blur(4px);
}

.carousel-indicators span {
  width: 10px;
  height: 10px;
  background: rgba(255, 255, 255, 0.3);
  border-radius: 50%;
  cursor: pointer;
  transition: all 0.3s ease;
}

.carousel-indicators span.active {
  background: var(--accent-cyan);
  transform: scale(1.2);
}

/* 輪播切換動畫 */
.fade-carousel-enter-active,
.fade-carousel-leave-active {
  transition: opacity 0.3s ease;
}

.fade-carousel-enter-from,
.fade-carousel-leave-to {
  opacity: 0;
}

.flex-center {
  display: flex;
  flex-direction: column;
  justify-content: center;
}

/* 詳細解說區塊 (針對案例 3 & 4) */
.detail-section {
  margin-bottom: 1.5rem;
}

.detail-section:last-child {
  margin-bottom: 0;
}

.detail-section h4 {
  color: #e2e8f0;
  font-size: 1.15rem;
  margin-top: 0;
  margin-bottom: 0.5rem;
}

.highlight-card {
  border: 1px solid rgba(139, 92, 246, 0.5);
  background: linear-gradient(145deg, rgba(139, 92, 246, 0.05) 0%, rgba(0,0,0,0) 100%);
  box-shadow: 0 15px 40px rgba(139, 92, 246, 0.15);
}

.highlight-card h3 {
  color: var(--accent-purple);
  font-size: 1.8rem;
}

/* 互動按鈕 */
.cta-button {
  margin-top: 2.5rem;
  padding: 15px 40px;
  font-size: 1.2rem;
  font-weight: 700;
  color: #ffffff;
  background: linear-gradient(135deg, var(--accent-cyan), var(--accent-purple));
  border: none;
  border-radius: 50px;
  cursor: pointer;
  transition: opacity 0.3s ease, transform 0.3s ease, box-shadow 0.3s ease;
  box-shadow: 0 8px 25px rgba(139, 92, 246, 0.4);
}

.cta-button:hover {
  opacity: 0.95;
  transform: scale(1.05) translateY(-2px);
  box-shadow: 0 12px 30px rgba(139, 92, 246, 0.6);
}

/* --- 測驗區塊專用樣式 --- */
.quiz-panel {
  text-align: left;
  animation: fadeUp 0.6s ease forwards; /* 加入動態渲染動畫 */
}

.result-panel {
  animation: fadeUp 0.6s ease forwards; /* 加入動態渲染動畫 */
}

@keyframes fadeUp {
  0% { opacity: 0; transform: translateY(20px); }
  100% { opacity: 1; transform: translateY(0); }
}

.quiz-header {
  font-size: 1rem;
  color: var(--accent-cyan);
  font-weight: 600;
  margin-bottom: 1.5rem;
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
  padding-bottom: 1rem;
}

.quiz-title {
  font-size: 1.5rem;
  color: var(--text-main);
  margin-bottom: 1rem;
  margin-top: 0;
}

.quiz-scenario {
  color: var(--text-muted);
  margin-bottom: 0.8rem !important;
}

.quiz-question {
  color: var(--text-main);
  font-weight: bold;
  margin-bottom: 1.5rem !important;
}

.options-container {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.option-button {
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.15);
  border-radius: 12px;
  padding: 1.2rem;
  color: var(--text-main);
  font-size: 1.1rem;
  text-align: left;
  cursor: pointer;
  transition: all 0.3s ease;
  line-height: 1.5;
}

.option-button:hover {
  background: rgba(6, 182, 212, 0.1);
  border-color: rgba(6, 182, 212, 0.5);
}

.option-button.selected {
  background: rgba(139, 92, 246, 0.2);
  border-color: var(--accent-purple);
  box-shadow: 0 0 15px rgba(139, 92, 246, 0.3);
}

.quiz-footer {
  display: flex;
  justify-content: flex-end;
  margin-top: 1rem;
}

.next-button:disabled {
  opacity: 0.4;
  cursor: not-allowed;
  transform: none;
  box-shadow: none;
}

/* --- 結果區塊專用樣式 --- */
.score-display {
  font-size: 1.4rem;
  color: var(--text-muted);
  margin-bottom: 1rem;
}

.result-title {
  font-size: 2.8rem !important;
  background: linear-gradient(135deg, var(--accent-cyan), var(--accent-purple));
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  margin-bottom: 1.5rem;
}

.result-desc {
  font-size: 1.2rem;
  line-height: 1.8;
  color: var(--text-main);
  margin-bottom: 2.5rem !important;
}

/* --- 錯題回顧樣式 --- */
.incorrect-reviews {
  text-align: left;
  margin-top: 2.5rem;
  border-top: 1px solid rgba(255, 255, 255, 0.1);
  padding-top: 2rem;
  margin-bottom: 2rem;
}

.review-title {
  color: var(--accent-cyan);
  font-size: 1.5rem;
  text-align: center;
  margin-bottom: 1.5rem;
}

.review-item {
  background: rgba(0, 0, 0, 0.25);
  border-radius: 12px;
  padding: 1.5rem;
  margin-bottom: 1.5rem;
  border: 1px solid rgba(255, 255, 255, 0.05);
}

.review-q-title {
  color: var(--accent-purple);
  margin-top: 0;
  font-size: 1.2rem;
}

.review-question {
  font-weight: bold;
  margin-bottom: 1.5rem !important;
}

.review-answers p {
  margin: 0.5rem 0;
  font-size: 1rem;
}

.wrong-answer {
  color: #fca5a5;
}

.correct-answer {
  color: #86efac;
}

.explanation {
  margin-top: 1.2rem;
  padding-top: 1.2rem;
  border-top: 1px dashed rgba(255, 255, 255, 0.1);
  font-size: 1rem;
  line-height: 1.6;
  color: var(--text-muted);
}

/* --- 創站初衷樣式 --- */
.creator-message {
  margin: 2rem 0;
  padding: 1.8rem;
  background: rgba(139, 92, 246, 0.1);
  border-left: 4px solid var(--accent-purple);
  border-radius: 0 16px 16px 0;
  text-align: left;
}

.creator-message h3 {
  color: var(--accent-purple);
  margin-top: 0;
  font-size: 1.3rem;
  margin-bottom: 0.8rem;
}

.creator-message p {
  color: var(--text-main);
  font-size: 1.1rem;
  line-height: 1.8;
  margin: 0;
}

/* --- 關於本站專用樣式 --- */
.about-content {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

.lead-text {
  font-size: 1.25rem !important;
  color: var(--text-main) !important;
  line-height: 1.8;
}

.lead-text strong {
  color: var(--accent-cyan);
  font-weight: 700;
}

.highlight-box {
  background: rgba(6, 182, 212, 0.1);
  border-left: 4px solid var(--accent-cyan);
  padding: 1.5rem;
  border-radius: 0 12px 12px 0;
  font-size: 1.15rem;
  line-height: 1.8;
  color: var(--text-main);
}

.highlight-box strong {
  color: var(--accent-purple);
}

.mission-statement {
  font-size: 1.4rem !important;
  font-weight: 700;
  text-align: center;
  margin-top: 1rem !important;
  background: linear-gradient(135deg, var(--accent-cyan), var(--accent-purple));
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

/* --- 滾動進場動畫 (Scroll Reveal) --- */
.reveal-target {
  opacity: 0;
  transform: translateY(40px);
  transition: opacity 0.8s ease-out, transform 0.8s cubic-bezier(0.25, 0.46, 0.45, 0.94);
  will-change: opacity, transform;
}

.reveal-target.is-revealed {
  opacity: 1;
  transform: translateY(0);
}

/* --- 回到頂部按鈕 --- */
.back-to-top {
  position: fixed;
  bottom: 40px;
  right: 40px;
  width: 50px;
  height: 50px;
  background: linear-gradient(135deg, var(--accent-cyan), var(--accent-purple));
  color: white;
  border: none;
  border-radius: 50%;
  display: flex;
  justify-content: center;
  align-items: center;
  cursor: pointer;
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.4);
  opacity: 0;
  visibility: hidden;
  transform: translateY(20px);
  transition: all 0.4s ease;
  z-index: 999;
}

.back-to-top.show {
  opacity: 1;
  visibility: visible;
  transform: translateY(0);
}

.back-to-top:hover {
  transform: translateY(-5px) scale(1.1);
  box-shadow: 0 12px 25px rgba(139, 92, 246, 0.6);
}

/* --- 頁尾 Footer --- */
.footer {
  text-align: center;
  padding: 3rem 2rem;
  background: rgba(11, 15, 25, 0.9);
  border-top: 1px solid rgba(255, 255, 255, 0.05);
  margin-top: 4rem;
}

.footer-logo {
  font-size: 1.5rem;
  font-weight: 800;
  color: var(--text-main);
  margin-bottom: 1rem;
  letter-spacing: 1px;
}

.footer p {
  color: var(--text-muted);
  margin: 0.5rem 0;
  font-size: 1rem;
}

.copyright {
  font-size: 0.9rem !important;
  opacity: 0.6;
  margin-top: 1.5rem !important;
}

/* --- 手機版響應式微調 (Mobile RWD) --- */
@media (max-width: 768px) {
  .navbar {
    padding: 0 5%;
  }
  
  .logo {
    font-size: 1.2rem;
  }
  
  .nav-links {
    gap: 1rem;
  }
  
  .nav-links a {
    font-size: 0.95rem;
  }
  
  .glow-text {
    font-size: 2.2rem;
    line-height: 1.4;
  }
  
  .subtitle {
    font-size: 1.05rem;
  }
  
  .card {
    padding: 2rem 1.5rem;
  }
}
</style>
