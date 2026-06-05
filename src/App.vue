<template>
  <div class="nutrition-app-container">
    <header class="app-header">
      <div class="brand-badge">Food Education Tech</div>
      <h1 class="app-title">糖分特工隊</h1>
      <p class="app-subtitle">手搖飲成分與熱量透視鏡，解密日常飲品的健康代價</p>
    </header>

    <section class="info-section">
      <h2 class="group-title">點餐前的 3 個健康冷知識</h2>
      <div class="info-grid">
        <div 
          v-for="(tip, index) in knowledgeTips" 
          :key="index" 
          class="info-card"
          :class="{ active: tip.isOpen }"
          @click="tip.isOpen = !tip.isOpen"
        >
          <div class="info-card-header">
            <span class="info-icon">{{ tip.icon }}</span>
            <h3>{{ tip.title }}</h3>
            <span class="info-arrow"></span>
          </div>
          <div class="info-card-body" :class="{ show: tip.isOpen }">
            <p>{{ tip.content }}</p>
          </div>
        </div>
      </div>
    </section>

    <div class="workspace-layout">
      <section class="panel-card config-panel">
        <h2 class="group-title">點餐模擬器</h2>
        
        <div class="control-group">
          <label class="control-label">1. 選擇容量</label>
          <div class="segmented-control">
            <button 
              v-for="(val, key) in sizeOptions" 
              :key="key"
              :class="{ active: selectedSize === key }"
              @click="selectedSize = key"
            >
              {{ key }} <span class="volume-hint">({{ val }}ml)</span>
            </button>
          </div>
        </div>

        <div class="control-group">
          <label class="control-label">
            2. 選擇茶底 
            <span class="inline-tag warning">基底是否含奶精？</span>
          </label>
          <div class="custom-select-container">
            <select v-model="selectedBase" class="clean-select">
              <option v-for="base in drinkBases" :key="base.name" :value="base">
                {{ base.name }} (+{{ base.calories }} kcal)
              </option>
            </select>
          </div>
        </div>

        <div class="control-group">
          <label class="control-label">3. 選擇甜度 <span class="sub-label">（折算方糖數）</span></label>
          <div class="options-grid text-options">
            <button 
              v-for="sugar in sugarLevels" 
              :key="sugar.name"
              class="option-pill"
              :class="{ active: selectedSugar.name === sugar.name }"
              @click="selectedSugar = sugar"
            >
              <span class="check-mark" v-if="selectedSugar.name === sugar.name">✓</span>
              {{ sugar.name }}
            </button>
          </div>
          
          <div class="sugar-visualizer" v-if="selectedSugar.cubes > 0">
            <span class="visualizer-title">實體糖量對照：</span>
            <div class="sugar-cubes">
              <span v-for="n in selectedSugar.cubes" :key="n" class="sugar-cube">🍬</span>
            </div>
          </div>
        </div>

        <div class="control-group">
          <label class="control-label">
            4. 邪惡配料特區 <span class="sub-label">（可多選）</span>
            <span class="inline-tag success">透明配料熱量較低</span>
          </label>
          <div class="options-grid topping-options">
            <div 
              v-for="topping in toppingsList" 
              :key="topping.name"
              class="topping-tile"
              :class="{ checked: hasTopping(topping.name) }"
              @click="toggleTopping(topping)"
            >
              <span class="topping-emoji">{{ topping.emoji }}</span>
              <div class="topping-info">
                <span class="topping-name">{{ topping.name }}</span>
                <span class="topping-cal">+{{ topping.calories }} kcal</span>
              </div>
              <div class="tile-checkbox"></div>
            </div>
          </div>
        </div>
      </section>

      <section class="panel-card dashboard-panel">
        <h2 class="group-title">飲品熱量透視鏡</h2>
        
        <div class="visual-cup-section">
          <div class="cup-wrapper">
            <div class="cup-rim"></div>
            <div class="cup-body" :class="{ 'shake-alert': totalCalories > 500 }">
              <div 
                class="liquid-fill" 
                :style="{ height: liquidHeight }"
                :class="{ 'has-pearls': hasTopping('波霸珍珠') }"
              >
                <div class="liquid-overlay-text">
                  <span class="main-text">{{ selectedSize }} · {{ selectedBase.name }}</span>
                  <span class="sub-text">{{ selectedSugar.name }} / {{ toppingNamesSummary }}</span>
                </div>
              </div>
            </div>
          </div>
        </div>

        <div class="metrics-grid">
          <div class="metric-card calorie">
            <span class="metric-label">總估計熱量</span>
            <span class="metric-value">{{ totalCalories }} <span class="unit">kcal</span></span>
            <span class="metric-footer">佔每日建議熱量 <strong>{{ caloriePercentage }}%</strong></span>
          </div>
          <div class="metric-card sugar">
            <span class="metric-label">估計含糖量</span>
            <span class="metric-value">×{{ totalSugarCubes }} <span class="unit">顆糖</span></span>
            <span class="metric-footer" :class="{ 'danger-text': totalSugarCubes > 5 }">
              {{ totalSugarCubes > 5 ? '❌ 糖分攝取超標' : '✅ 符合每日上限' }}
            </span>
          </div>
        </div>

        <div class="report-box" :class="verdictClass">
          <h3 class="report-title">特工健康報告書</h3>
          <div class="report-row">
            <span class="row-icon">📊</span>
            <p>換算熱量：這杯飲料相當於吃下了 <strong class="highlight">{{ equivalentRice }}</strong> 碗白飯。</p>
          </div>
          <div class="report-row">
            <span class="row-icon">🏃‍♂️</span>
            <p>代謝成本：你需要慢跑 <strong class="highlight">{{ heavyExerciseTime }}</strong> 分鐘才能完全消耗。</p>
          </div>
          
          <div class="educational-tip" v-if="eduAdviceText">
            <div class="tip-title">成份科普觀點</div>
            <p class="tip-desc">{{ eduAdviceText }}</p>
          </div>
          
          <div class="verdict-banner">
            <span v-if="totalCalories >= 500">🚨 警報！這杯是熱量炸彈！配料或甜度太高囉，建議改成微糖或去掉一種配料！</span>
            <span v-else-if="totalCalories >= 250">⚠️ 輕鬆一下還行，但要注意下一餐的澱粉與油脂攝取量喔！</span>
            <span v-else>✅ 完美！這是一杯非常優秀的低負擔輕手搖！</span>
          </div>
        </div>
      </section>
    </div>

    <hr class="divider-line" />
    
    <section class="quiz-section">
      <div class="quiz-intro">
        <h2 class="group-title">特工隨堂測驗</h2>
        <p class="quiz-subtitle">檢驗你的抗糖戰鬥力，避開手搖點餐的隱形陷阱</p>
      </div>
      
      <div class="quiz-container">
        <div 
          v-for="(quiz, qIdx) in quizList" 
          :key="qIdx" 
          class="quiz-row-card"
          :class="{ 'answered': quiz.selectedAnswer !== null }"
        >
          <div class="quiz-meta">
            <span class="quiz-index">0{{ qIdx + 1 }}</span>
            <span class="quiz-category">{{ quiz.tag }}</span>
          </div>
          
          <div class="quiz-content-block">
            <h3 class="quiz-question-text">{{ quiz.question }}</h3>
            
            <div class="quiz-answers-stack">
              <button
                v-for="(option, oIdx) in quiz.options"
                :key="oIdx"
                class="answer-pill"
                :class="{
                  'selected': quiz.selectedAnswer === oIdx,
                  'is-correct': quiz.selectedAnswer !== null && oIdx === quiz.correctAnswer,
                  'is-wrong': quiz.selectedAnswer === oIdx && oIdx !== quiz.correctAnswer
                }"
                :disabled="quiz.selectedAnswer !== null"
                @click="quiz.selectedAnswer = oIdx"
              >
                <span class="option-index">{{ ['A', 'B', 'C', 'D'][oIdx] }}</span>
                <span class="option-text">{{ option }}</span>
              </button>
            </div>

            <div v-if="quiz.selectedAnswer !== null" class="quiz-explain-panel" :class="quiz.selectedAnswer === quiz.correctAnswer ? 'right' : 'wrong'">
              <span class="status-badge">
                {{ quiz.selectedAnswer === quiz.correctAnswer ? 'Correct' : 'Incorrect' }}
              </span>
              <p class="explain-text"><strong>解析：</strong>{{ quiz.explanation }}</p>
            </div>
          </div>
        </div>
      </div>

      <div v-if="isAllAnswered" class="score-summary-bar">
        <div class="score-badge">
          <span class="score-num">{{ totalScore }}</span>
          <span class="score-total">/100</span>
        </div>
        <div class="score-message">
          <h4>特工考核結算</h4>
          <p>{{ scoreComment }}</p>
        </div>
        <button class="btn-secondary" @click="resetQuiz">重新挑戰</button>
      </div>
    </section>
  </div>
</template>

<script setup>
import { ref, reactive, computed } from 'vue'

// --- 科普知識庫資料 ---
const knowledgeTips = reactive([
  {
    icon: '💡',
    title: '「微糖」其實不微量？',
    content: '一般飲料店的「微糖（3分糖）」通常指的是額外添加的蔗糖量，但如果茶底本身有加鮮奶或水果原汁，加上配料本身的蜜糖，整杯的總含糖量往往會逼近半糖甚至全糖！',
    isOpen: false
  },
  {
    icon: '🧪',
    title: '隱形熱量大魔王：奶蓋與椰果',
    content: '大家都知道珍珠熱量高，但其實「奶蓋」是由鮮奶油與起司粉打發而成，油脂含量極高，大杯加滿奶蓋直接飆破 200 大卡！而「椰果」雖然聽起來健康，但製作過程中會浸泡大量高濃度糖漿，熱量甚至超越布丁！',
    isOpen: false
  },
  {
    icon: '🌱',
    title: '想咬東西？選這些配料負擔最少',
    content: '如果嘴饞想滿足咀嚼肌，建議捨棄珍珠、芋圓等澱粉類配料，改選「寒天晶球、愛玉、仙草或蘆薈」。這些配料本身熱量極低（每份約 30-50 大卡），是減脂期的無痛救星！',
    isOpen: false
  }
])

// --- 選項資料集 ---
const sizeOptions = { '大杯': 700, '中杯': 500 }

const drinkBases = [
  { name: '四季春青茶/紅茶 (無奶)', calories: 0 },
  { name: '經典奶茶 (奶精)', calories: 250 },
  { name: '鮮奶拿鐵 (鮮奶)', calories: 150 },
  { name: '黑糖鮮奶', calories: 320 }
]

const sugarLevels = [
  { name: '無糖 (0分糖)', cubes: 0, calories: 0 },
  { name: '微糖 (3分糖)', cubes: 3, calories: 60 },
  { name: '半糖 (5分糖)', cubes: 5, calories: 100 },
  { name: '全糖 (10分糖)', cubes: 10, calories: 200 }
]

const toppingsList = [
  { name: '波霸珍珠', calories: 220, emoji: '⚫' },
  { name: '布丁 (一顆)', calories: 110, emoji: '🍮' },
  { name: '椰果', calories: 140, emoji: '🥥' },
  { name: '奶蓋', calories: 200, emoji: '☁️' },
  { name: '蘆薈/寒天', calories: 40, emoji: '💎' }
]

// --- 狀態綁定 ---
const selectedSize = ref('大杯')
const selectedBase = ref(drinkBases[0])
const selectedSugar = ref(sugarLevels[1]) 
const selectedToppings = ref([])

// --- 互動方法 ---
const toggleTopping = (topping) => {
  const index = selectedToppings.value.findIndex(t => t.name === topping.name)
  if (index === -1) {
    selectedToppings.value.push(topping)
  } else {
    selectedToppings.value.splice(index, 1)
  }
}

const hasTopping = (name) => selectedToppings.value.some(t => t.name === name)

// --- 計算屬性 ---
const totalCalories = computed(() => {
  let baseCal = selectedBase.value.calories
  let sugarCal = selectedSugar.value.calories
  let toppingCal = selectedToppings.value.reduce((sum, t) => sum + t.calories, 0)
  let multiplier = selectedSize.value === '中杯' ? 0.8 : 1
  return Math.round((baseCal + sugarCal) * multiplier + toppingCal)
})

const totalSugarCubes = computed(() => selectedSugar.value.cubes)
const equivalentRice = computed(() => (totalCalories.value / 280).toFixed(1))
const heavyExerciseTime = computed(() => Math.round(totalCalories.value / 8))
const caloriePercentage = computed(() => ((totalCalories.value / 2000) * 100).toFixed(0))

const toppingNamesSummary = computed(() => {
  if (selectedToppings.value.length === 0) return '不加料'
  return selectedToppings.value.map(t => t.name).join('+')
})

const verdictClass = computed(() => {
  if (totalCalories.value >= 500) return 'danger'
  if (totalCalories.value >= 250) return 'warning'
  return 'safe'
})

const liquidHeight = computed(() => {
  let baseHeight = selectedSize.value === '中杯' ? 65 : 75
  let toppingBonus = selectedToppings.value.length * 6
  return `${Math.min(baseHeight + toppingBonus, 95)}%`
})

const eduAdviceText = computed(() => {
  let advices = []
  if (selectedBase.value.name.includes('奶精')) {
    advices.push('【奶精警告】奶精主成分為植物油與糊精，飽和脂肪較高，多喝容易增加心血管負擔。')
  }
  if (selectedBase.value.name.includes('黑糖鮮奶')) {
    advices.push('【黑糖陷阱】黑糖含糖量極高，這類茶底通常無法調整甜度，容易在不知不覺中攝取過多糖分。')
  }
  if (hasTopping('奶蓋')) {
    advices.push('【奶蓋知識】奶蓋是由鮮奶油、煉乳與起司打發而成，屬於高油脂配料，是造成脂肪囤積的隱形殺手。')
  }
  if (hasTopping('波霸珍珠') && selectedSugar.value.cubes >= 5) {
    advices.push('【雙重糖分】珍珠在製程中會使用大量蜜糖熬煮，若同時搭配半糖以上飲料，整杯含糖量將嚴重超標。')
  }
  return advices.length > 0 ? advices.join(' ') : '目前搭配組合配置優良，繼續保持健康的飲水點餐習慣！'
})

// --- 隨堂測驗資料集 ---
const quizList = reactive([
  {
    tag: '糖分觀念',
    question: '當你點了微糖（3分糖）的水果茶，這杯飲料的總含糖量通常會如何？',
    options: [
      '只有原本全糖的 30%，非常安全',
      '可能接近半糖甚至全糖，因為水果原汁與配料自帶蜜糖',
      '完全沒有任何糖分，純屬心理安慰',
      '比點全糖還要甜'
    ],
    correctAnswer: 1,
    selectedAnswer: null,
    explanation: '水果茶通常會加入果醬、濃縮汁或新鮮水果，這些成分本身含糖量極高，再加上配料可能用糖蜜過，所以即使點微糖，總糖量還是容易爆表喔！'
  },
  {
    tag: '油脂陷阱',
    question: '小明正在實行減脂計畫，嘴饞想喝飲料，下列哪一個茶底或配料的組合脂肪量最高、最該避開？',
    options: [
      '四季春青茶 + 蘆薈',
      '鮮奶拿鐵 + 寒天晶球',
      '黑糖鮮奶 + 波霸珍珠',
      '經典奶茶 + 厚厚一層奶蓋'
    ],
    correctAnswer: 3,
    selectedAnswer: null,
    explanation: '經典奶茶的奶精主成分是植物油，而奶蓋更是由鮮奶油與起司粉打發而成，兩者都是純油脂大魔王，加在一起直接讓體脂肪飆升！'
  },
  {
    tag: '無痛減脂',
    question: '如果今天下午茶真的很想享受「有東西咬」的咀嚼感，選哪一組配料對身體負擔最小？',
    options: [
      '波霸珍珠 + 布丁',
      '寒天晶球 + 仙草凍',
      '椰果 + 芋圓',
      '奶蓋 + 雙倍珍珠'
    ],
    correctAnswer: 1,
    selectedAnswer: null,
    explanation: '寒天與仙草、愛玉等配料水分含量高、本身熱量極低（每份約 30-50 大卡）。而珍珠、芋圓屬於澱粉類，椰果則吸飽了高濃度糖漿，熱量都高出許多。'
  }
])

const isAllAnswered = computed(() => quizList.every(q => q.selectedAnswer !== null))

const totalScore = computed(() => {
  let correctCount = quizList.filter(q => q.selectedAnswer === q.correctAnswer).length
  if (correctCount === 0) return 0
  if (correctCount === 1) return 33
  if (correctCount === 2) return 66
  return 100
})

const scoreComment = computed(() => {
  if (totalScore.value === 100) return '太完美了！你已經掌握了手搖飲點餐的精髓，是高能的健康控糖特工！'
  if (totalScore.value >= 60) return '表現不錯！對隱形熱量有基本概念，點餐前記得多複習一下避坑指南唷！'
  return '特工警報！你很容易掉入手搖飲的糖分陷阱中，快重新對照左側的科普標籤多玩幾次模擬器吧！'
})

const resetQuiz = () => {
  quizList.forEach(q => q.selectedAnswer = null)
}
</script>

<style scoped>
/* ==========================================================================
   1. 全域現代化樣式重設 (高級中性暖色調)
   ========================================================================== */
.nutrition-app-container {
  max-width: 1140px;
  margin: 50px auto;
  padding: 40px;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
  background-color: #FBF9F6;
  color: #2B1F17;
  border-radius: 24px;
  border: 1px solid #ECE7E1;
  box-shadow: 0 30px 60px rgba(43, 31, 23, 0.04);
}

/* 頂部排版優化 */
.app-header {
  text-align: center;
  margin-bottom: 48px;
}
.brand-badge {
  display: inline-block;
  font-size: 11px;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 2px;
  color: #A39081;
  border: 1px solid #E3DCD5;
  padding: 4px 12px;
  border-radius: 30px;
  margin-bottom: 16px;
}
.app-title {
  font-size: 42px;
  font-weight: 800;
  letter-spacing: -1px;
  color: #1F140E;
  margin: 0 0 8px 0;
}
.app-subtitle {
  font-size: 16px;
  color: #706053;
  margin: 0;
}

/* ==========================================================================
   2. 科普知識區 (現代折疊式手風琴排版)
   ========================================================================== */
.group-title {
  font-size: 20px;
  font-weight: 700;
  letter-spacing: -0.5px;
  color: #1F140E;
  margin: 0 0 24px 0;
}

.info-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 16px;
  margin-bottom: 48px;
}
@media (max-width: 900px) { .info-grid { grid-template-columns: 1fr; } }

.info-card {
  background: #FFFFFF;
  border: 1px solid #EBE6E0;
  border-radius: 16px;
  padding: 20px;
  cursor: pointer;
  transition: all 0.25s cubic-bezier(0.4, 0, 0.2, 1);
}
.info-card:hover {
  border-color: #D4B296;
  transform: translateY(-2px);
  box-shadow: 0 10px 20px rgba(43, 31, 23, 0.03);
}
.info-card-header {
  display: flex;
  align-items: center;
  gap: 12px;
  position: relative;
}
.info-icon { font-size: 20px; }
.info-card h3 {
  margin: 0;
  font-size: 15px;
  font-weight: 700;
  color: #2B1F17;
  flex-grow: 1;
}
.info-arrow {
  width: 8px;
  height: 8px;
  border-right: 2px solid #A39081;
  border-bottom: 2px solid #A39081;
  transform: rotate(45deg);
  transition: transform 0.2s ease;
}
.info-card.active .info-arrow {
  transform: rotate(-135deg);
}

.info-card-body {
  max-height: 0;
  overflow: hidden;
  transition: max-height 0.25s ease-out;
}
.info-card-body.show {
  max-height: 200px;
  margin-top: 12px;
  border-top: 1px solid #F5F2EB;
  padding-top: 12px;
}
.info-card-body p {
  margin: 0;
  font-size: 13.5px;
  line-height: 1.6;
  color: #615043;
}

/* ==========================================================================
   3. 主工作區版面與控制元件
   ========================================================================== */
.workspace-layout {
  display: grid;
  grid-template-columns: 1.2fr 1fr;
  gap: 40px;
}
@media (max-width: 900px) { .workspace-layout { grid-template-columns: 1fr; } }

.panel-card {
  background: #FFFFFF;
  border: 1px solid #ECE7E1;
  border-radius: 20px;
  padding: 32px;
}

.control-group {
  margin-bottom: 28px;
}
.control-label {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 14.5px;
  font-weight: 700;
  color: #1F140E;
  margin-bottom: 12px;
}
.sub-label {
  font-weight: 400;
  color: #8A7A6E;
  font-size: 13px;
}

/* 分段選擇器 (Segmented Control) */
.segmented-control {
  display: flex;
  background: #F5F2EB;
  padding: 4px;
  border-radius: 12px;
}
.segmented-control button {
  flex: 1;
  border: none;
  background: transparent;
  padding: 12px;
  font-size: 14px;
  font-weight: 600;
  color: #615043;
  border-radius: 9px;
  cursor: pointer;
  transition: all 0.2s;
}
.segmented-control button.active {
  background: #FFFFFF;
  color: #2B1F17;
  box-shadow: 0 4px 10px rgba(43, 31, 23, 0.05);
}
.volume-hint {
  font-size: 12px;
  font-weight: 400;
  color: #8A7A6E;
}

/* 下拉選單 */
.clean-select {
  width: 100%;
  padding: 14px 16px;
  font-size: 14.5px;
  font-weight: 600;
  color: #2B1F17;
  background: #FFFFFF;
  border: 1px solid #DED8D1;
  border-radius: 12px;
  outline: none;
  cursor: pointer;
  appearance: none;
  background-image: url("data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='24' height='24' viewBox='0 0 24 24' fill='none' stroke='%238A7A6E' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'><polyline points='6 9 12 15 18 9'></polyline></svg>");
  background-repeat: no-repeat;
  background-position: right 16px center;
  background-size: 16px;
}
.clean-select:focus {
  border-color: #C49A78;
}

/* 標籤小徽章 */
.inline-tag {
  font-size: 11px;
  font-weight: 700;
  padding: 2px 8px;
  border-radius: 4px;
}
.inline-tag.warning { background: #FFF1E6; color: #D96B27; }
.inline-tag.success { background: #E6F4EA; color: #137333; }

/* 網格選項組 */
.options-grid {
  display: grid;
  gap: 10px;
}
.text-options { grid-template-columns: repeat(2, 1fr); }

.option-pill {
  border: 1px solid #DED8D1;
  background: #FFFFFF;
  padding: 14px;
  border-radius: 12px;
  font-size: 14px;
  font-weight: 600;
  color: #4A3B30;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
  transition: all 0.15s;
}
.option-pill.active {
  border-color: #2B1F17;
  background: #2B1F17;
  color: #FFFFFF;
}
.check-mark {
  font-size: 12px;
}

/* 方糖實體化面板 */
.sugar-visualizer {
  margin-top: 14px;
  padding: 14px;
  background: #FAF8F5;
  border-radius: 10px;
  border: 1px dashed #DED8D1;
}
.visualizer-title { font-size: 12px; font-weight: 700; color: #8A7A6E; display: block; margin-bottom: 8px; }
.sugar-cubes { display: flex; flex-wrap: wrap; gap: 6px; }
.sugar-cube { font-size: 18px; }

/* 配料大卡片設計（去 AI 罐頭感的關鍵：改為細緻清爽的橫向排版） */
.topping-options { grid-template-columns: 1fr; }
.topping-tile {
  display: flex;
  align-items: center;
  padding: 14px 16px;
  border: 1px solid #EBE6E0;
  background: #FFFFFF;
  border-radius: 12px;
  cursor: pointer;
  position: relative;
  transition: all 0.2s;
}
.topping-tile:hover {
  border-color: #C49A78;
}
.topping-emoji { font-size: 24px; margin-right: 14px; }
.topping-info { flex-grow: 1; display: flex; flex-direction: column; }
.topping-name { font-size: 14px; font-weight: 700; color: #2B1F17; }
.topping-cal { font-size: 12px; color: #8A7A6E; margin-top: 2px; }

.tile-checkbox {
  width: 18px;
  height: 18px;
  border: 2px solid #DED8D1;
  border-radius: 50%;
  position: relative;
}
.topping-tile.checked {
  border-color: #C49A78;
  background: #FAF4EE;
}
.topping-tile.checked .tile-checkbox {
  border-color: #C49A78;
  background: #C49A78;
}
.topping-tile.checked .tile-checkbox::after {
  content: '';
  position: absolute;
  width: 5px;
  height: 9px;
  border-right: 2px solid white;
  border-bottom: 2px solid white;
  top: 2px;
  left: 5px;
  transform: rotate(45deg);
}

/* ==========================================================================
   4. 右側擬物化數據面板
   ========================================================================== */
.visual-cup-section {
  display: flex;
  justify-content: center;
  margin-bottom: 32px;
  padding: 24px 0;
  background: #FAF8F5;
  border-radius: 16px;
}
.cup-wrapper {
  position: relative;
  width: 110px;
}
.cup-rim {
  height: 5px;
  background: #2B1F17;
  border-radius: 4px;
  margin-bottom: 2px;
}
.cup-body {
  height: 170px;
  background: rgba(255,255,255,0.8);
  border: 3px solid #2B1F17;
  border-top: none;
  border-bottom-left-radius: 24px;
  border-bottom-right-radius: 24px;
  overflow: hidden;
  position: relative;
}
.liquid-fill {
  position: absolute;
  bottom: 0;
  width: 100%;
  background: #EED8C5;
  transition: height 0.35s cubic-bezier(0.4, 0, 0.2, 1);
  display: flex;
  align-items: center;
  justify-content: center;
}
.liquid-fill.has-pearls {
  background: linear-gradient(to top, #3A281A 30%, #EED8C5 45%);
}
.liquid-overlay-text {
  text-align: center;
  padding: 0 8px;
  display: flex;
  flex-direction: column;
}
.liquid-overlay-text .main-text { font-size: 11px; font-weight: 800; color: #2B1F17; line-height: 1.3; }
.liquid-overlay-text .sub-text { font-size: 9px; color: #5E4B3C; margin-top: 4px; font-weight: 600; }

.shake-alert { animation: micro-shake 0.5s infinite; }
@keyframes micro-shake {
  0% { transform: translateX(0); }
  25% { transform: translateX(1px); }
  75% { transform: translateX(-1px); }
  100% { transform: translateX(0); }
}

/* 數據看板網格 */
.metrics-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 16px;
  margin-bottom: 32px;
}
.metric-card {
  border-radius: 14px;
  padding: 20px;
  border: 1px solid #ECE7E1;
}
.metric-card.calorie { background: #FFF5F5; border-color: #FADBD8; }
.metric-card.sugar { background: #F4FAF7; border-color: #D1EAE0; }

.metric-label { font-size: 12px; font-weight: 700; color: #706053; display: block; margin-bottom: 6px; }
.metric-value { font-size: 28px; font-weight: 800; color: #1F140E; display: block; }
.metric-value .unit { font-size: 14px; font-weight: 500; color: #706053; }
.metric-footer { font-size: 12px; color: #615043; display: block; margin-top: 8px; }
.danger-text { color: #C0392b; font-weight: 700; }

/* 報告書樣式優化 */
.report-box {
  padding: 24px;
  border-radius: 16px;
  border: 1px solid #ECE7E1;
}
.report-box.danger { background: #FFF8F8; border-left: 4px solid #E74C3C; color: #7F221E; }
.report-box.warning { background: #FFFDF0; border-left: 4px solid #F39C12; color: #7F541E; }
.report-box.safe { background: #F6FCF8; border-left: 4px solid #2ECC71; color: #1E5033; }

.report-title { margin: 0 0 16px 0; font-size: 16px; font-weight: 700; }
.report-row { display: flex; align-items: flex-start; gap: 10px; margin-bottom: 10px; font-size: 14px; }
.report-row p { margin: 0; line-height: 1.5; font-weight: 500; }
.highlight { font-size: 16px; font-weight: 700; text-decoration: underline; }

.educational-tip {
  margin-top: 16px;
  padding-top: 16px;
  border-top: 1px dashed rgba(43, 31, 23, 0.1);
}
.tip-title { font-size: 12px; font-weight: 700; text-transform: uppercase; letter-spacing: 0.5px; opacity: 0.8; margin-bottom: 4px; }
.tip-desc { font-size: 12.5px; line-height: 1.5; margin: 0; opacity: 0.9; }

.verdict-banner {
  margin-top: 16px;
  font-weight: 700;
  font-size: 13px;
}

/* ==========================================================================
   5. 底部隨堂測驗區 (去 AI 罐頭感的全新線性流線設計)
   ========================================================================== */
.divider-line {
  border: none;
  border-top: 1px solid #ECE7E1;
  margin: 56px 0;
}

.quiz-intro {
  margin-bottom: 32px;
}
.quiz-subtitle {
  font-size: 14.5px;
  color: #706053;
  margin: -16px 0 0 0;
}

.quiz-container {
  display: flex;
  flex-direction: column;
  gap: 20px;
  margin-bottom: 32px;
}

.quiz-row-card {
  background: #FFFFFF;
  border: 1px solid #ECE7E1;
  border-radius: 16px;
  padding: 28px;
  display: flex;
  gap: 24px;
}
@media (max-width: 768px) { .quiz-row-card { flex-direction: column; gap: 16px; } }

.quiz-meta {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  gap: 8px;
  width: 90px;
  flex-shrink: 0;
}
.quiz-index { font-size: 24px; font-weight: 800; color: #DED8D1; line-height: 1; }
.quiz-category { font-size: 11px; font-weight: 700; color: #8A7A6E; background: #F5F2EB; padding: 2px 6px; border-radius: 4px; }

.quiz-content-block { flex-grow: 1; }
.quiz-question-text { font-size: 15.5px; font-weight: 700; color: #1F140E; margin: 0 0 16px 0; line-height: 1.5; }

/* 問答按鈕垂直堆疊 */
.quiz-answers-stack {
  display: flex;
  flex-direction: column;
  gap: 8px;
}
.answer-pill {
  width: 100%;
  text-align: left;
  padding: 12px 16px;
  background: #FFFFFF;
  border: 1px solid #DED8D1;
  border-radius: 8px;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 12px;
  transition: all 0.15s ease;
}
.answer-pill:hover:not(:disabled) {
  background: #FAF8F5;
  border-color: #A39081;
}
.option-index {
  font-size: 11px;
  font-weight: 700;
  color: #8A7A6E;
  background: #F5F2EB;
  width: 20px;
  height: 20px;
  border-radius: 4px;
  display: flex;
  align-items: center;
  justify-content: center;
}
.option-text { font-size: 13.5px; font-weight: 500; color: #4A3B30; }

/* 答題後的正確/錯誤精緻狀態（告別亮色大背板） */
.answer-pill.is-correct {
  border-color: #2ECC71;
  background: #F4FCF7;
}
.answer-pill.is-correct .option-text { color: #1E5033; font-weight: 700; }
.answer-pill.is-correct .option-index { background: #2ECC71; color: white; }

.answer-pill.is-wrong {
  border-color: #E74C3C;
  background: #FFF8F8;
}
.answer-pill.is-wrong .option-text { color: #7F221E; }
.answer-pill.is-wrong .option-index { background: #E74C3C; color: white; }

/* 解析反饋小面板 */
.quiz-explain-panel {
  margin-top: 14px;
  padding: 14px 16px;
  border-radius: 8px;
  font-size: 13px;
  line-height: 1.6;
  animation: slideDown 0.2s ease-out;
}
.quiz-explain-panel.right { background: #F4FCF7; color: #1E5033; border-left: 3px solid #2ECC71; }
.quiz-explain-panel.wrong { background: #FFF8F8; color: #7F221E; border-left: 3px solid #E74C3C; }

.status-badge { font-weight: 800; font-size: 11px; text-transform: uppercase; display: block; margin-bottom: 2px; letter-spacing: 0.5px; }
.explain-text { margin: 0; color: #514235; }

/* 分數結算（現代簡約長條看板） */
.score-summary-bar {
  display: flex;
  align-items: center;
  background: #2B1F17;
  color: #FFFFFF;
  padding: 24px 32px;
  border-radius: 16px;
  gap: 24px;
  margin-top: 32px;
}
@media (max-width: 600px) { .score-summary-bar { flex-direction: column; text-align: center; } }

.score-badge { display: flex; align-items: baseline; }
.score-num { font-size: 36px; font-weight: 800; color: #F0B27A; }
.score-total { font-size: 14px; opacity: 0.6; margin-left: 2px; }

.score-message { flex-grow: 1; }
.score-message h4 { margin: 0 0 4px 0; font-size: 16px; font-weight: 700; color: #FFFFFF; }
.score-message p { margin: 0; font-size: 13.5px; color: #DED8D1; }

.btn-secondary {
  background: transparent;
  color: #FFFFFF;
  border: 1px solid rgba(255,255,255,0.3);
  padding: 12px 20px;
  font-size: 13.5px;
  font-weight: 600;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.2s;
}
.btn-secondary:hover {
  background: rgba(255,255,255,0.08);
  border-color: #FFFFFF;
}

@keyframes slideDown {
  from { opacity: 0; transform: translateY(-4px); }
  to { opacity: 1; transform: translateY(0); }
}
</style>