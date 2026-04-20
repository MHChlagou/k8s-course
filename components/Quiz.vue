<script setup>
import { ref, computed } from 'vue'

const props = defineProps({
  questions: { type: Array, required: true }
})

const currentIdx = ref(0)
const selected = ref(props.questions.map(() => null))
const finished = ref(false)

const current = computed(() => props.questions[currentIdx.value])
const currentSelected = computed(() => selected.value[currentIdx.value])
const isAnswered = computed(() => currentSelected.value !== null)
const isCorrect = computed(() => {
  const sel = currentSelected.value
  return sel !== null && !!current.value.choices[sel].ok
})

const score = computed(() =>
  selected.value.reduce((acc, sel, i) => {
    if (sel === null) return acc
    return acc + (props.questions[i].choices[sel].ok ? 1 : 0)
  }, 0)
)

const verdict = computed(() => {
  const total = props.questions.length
  if (score.value === total) return '🎉 Sans faute !'
  if (score.value >= total - 1) return 'Très bien !'
  if (score.value >= total / 2) return 'Pas mal, mais on peut revoir.'
  return 'À revoir avant de passer au module suivant.'
})

function selectChoice(idx) {
  if (isAnswered.value) return
  selected.value[currentIdx.value] = idx
}

function next() {
  if (currentIdx.value < props.questions.length - 1) {
    currentIdx.value++
  } else {
    finished.value = true
  }
}

function restart() {
  currentIdx.value = 0
  selected.value = props.questions.map(() => null)
  finished.value = false
}

function choiceClass(idx) {
  if (!isAnswered.value) return 'quiz-choice'
  const choice = current.value.choices[idx]
  if (choice.ok) return 'quiz-choice quiz-choice-correct'
  if (idx === currentSelected.value) return 'quiz-choice quiz-choice-wrong'
  return 'quiz-choice quiz-choice-muted'
}

function letter(idx) {
  return String.fromCharCode(65 + idx)
}
</script>

<template>
  <div class="quiz">
    <div v-if="!finished" class="quiz-card">
      <div class="quiz-meta">
        <span>Question {{ currentIdx + 1 }} / {{ questions.length }}</span>
        <span class="quiz-progress">
          <span
            v-for="(_, i) in questions"
            :key="i"
            class="quiz-dot"
            :class="{
              'quiz-dot-active': i === currentIdx,
              'quiz-dot-done': selected[i] !== null
            }"
          />
        </span>
      </div>

      <h3 class="quiz-q">{{ current.q }}</h3>

      <div class="quiz-choices">
        <button
          v-for="(choice, idx) in current.choices"
          :key="idx"
          :class="choiceClass(idx)"
          :disabled="isAnswered"
          @click="selectChoice(idx)"
        >
          <span class="quiz-bullet">{{ letter(idx) }}</span>
          <span class="quiz-text">{{ choice.t }}</span>
          <span v-if="isAnswered && choice.ok" class="quiz-icon">✓</span>
          <span v-else-if="isAnswered && idx === currentSelected" class="quiz-icon">✗</span>
        </button>
      </div>

      <div v-if="isAnswered" class="quiz-feedback" :class="isCorrect ? 'ok' : 'ko'">
        <strong>{{ isCorrect ? 'Bonne réponse.' : 'Pas tout à fait.' }}</strong>
        <span v-if="current.explain"> {{ current.explain }}</span>
      </div>

      <div v-if="isAnswered" class="quiz-actions">
        <button class="quiz-next" @click="next">
          {{ currentIdx < questions.length - 1 ? 'Question suivante →' : 'Voir le score' }}
        </button>
      </div>
    </div>

    <div v-else class="quiz-card quiz-results">
      <div class="quiz-score">{{ score }} / {{ questions.length }}</div>
      <div class="quiz-verdict">{{ verdict }}</div>
      <button class="quiz-next" @click="restart">↻ Recommencer</button>
    </div>
  </div>
</template>

<style scoped>
.quiz {
  max-width: 720px;
  margin: 0 auto;
  text-align: left;
}

.quiz-card {
  background: #ffffff;
  border: 1px solid var(--k8s-hairline, #e4e9f2);
  border-radius: 12px;
  padding: 1.4rem 1.6rem;
  box-shadow: 0 6px 24px rgba(50, 108, 229, 0.08);
}

.quiz-meta {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 0.72rem;
  text-transform: uppercase;
  letter-spacing: 0.1em;
  color: var(--k8s-blue, #326ce5);
  font-weight: 700;
  margin-bottom: 0.6rem;
}

.quiz-progress { display: flex; gap: 0.3rem; }

.quiz-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: #e4e9f2;
  display: inline-block;
}
.quiz-dot-done { background: var(--k8s-blue, #326ce5); opacity: 0.5; }
.quiz-dot-active { background: var(--k8s-blue, #326ce5); opacity: 1; transform: scale(1.3); }

.quiz-q {
  font-size: 1.25rem;
  color: var(--k8s-ink, #1a2038);
  margin: 0 0 1.1rem 0;
  font-weight: 600;
  line-height: 1.4;
}

.quiz-choices {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.quiz-choice {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  width: 100%;
  text-align: left;
  padding: 0.65rem 0.95rem;
  background: #f7f9fc;
  border: 1px solid var(--k8s-hairline, #e4e9f2);
  border-radius: 8px;
  color: var(--k8s-ink, #1a2038);
  font-size: 0.92rem;
  font-family: inherit;
  cursor: pointer;
  transition: all 0.15s ease;
}

.quiz-choice:not(:disabled):hover {
  background: var(--k8s-blue-soft, rgba(50, 108, 229, 0.08));
  border-color: var(--k8s-blue, #326ce5);
  transform: translateX(2px);
}

.quiz-choice:disabled { cursor: default; }

.quiz-bullet {
  width: 1.6rem;
  height: 1.6rem;
  border-radius: 50%;
  background: rgba(50, 108, 229, 0.12);
  color: var(--k8s-blue, #326ce5);
  display: inline-flex;
  align-items: center;
  justify-content: center;
  font-weight: 700;
  font-size: 0.78rem;
  flex-shrink: 0;
}

.quiz-text { flex: 1; }

.quiz-icon {
  margin-left: auto;
  font-size: 1.05rem;
  font-weight: 700;
  flex-shrink: 0;
}

.quiz-choice-correct {
  background: rgba(46, 158, 91, 0.10);
  border-color: #2e9e5b;
  color: #1d5f38;
}
.quiz-choice-correct .quiz-bullet { background: #2e9e5b; color: #fff; }
.quiz-choice-correct .quiz-icon { color: #2e9e5b; }

.quiz-choice-wrong {
  background: rgba(220, 53, 69, 0.08);
  border-color: #dc3545;
  color: #8b1e29;
}
.quiz-choice-wrong .quiz-bullet { background: #dc3545; color: #fff; }
.quiz-choice-wrong .quiz-icon { color: #dc3545; }

.quiz-choice-muted { opacity: 0.45; }

.quiz-feedback {
  margin-top: 1rem;
  padding: 0.7rem 0.95rem;
  border-radius: 8px;
  font-size: 0.9rem;
  line-height: 1.5;
}
.quiz-feedback.ok {
  background: rgba(46, 158, 91, 0.08);
  border-left: 3px solid #2e9e5b;
  color: #1d5f38;
}
.quiz-feedback.ko {
  background: rgba(220, 53, 69, 0.06);
  border-left: 3px solid #dc3545;
  color: #6e1722;
}

.quiz-actions {
  margin-top: 1rem;
  display: flex;
  justify-content: flex-end;
}

.quiz-next {
  background: var(--k8s-blue, #326ce5);
  color: white;
  border: none;
  border-radius: 6px;
  padding: 0.55rem 1.1rem;
  font-size: 0.88rem;
  font-weight: 600;
  font-family: inherit;
  cursor: pointer;
  transition: background 0.15s;
}
.quiz-next:hover { background: var(--k8s-blue-dark, #2656c8); }

.quiz-results { text-align: center; padding: 2.5rem 1.6rem; }
.quiz-score {
  font-size: 4rem;
  font-weight: 800;
  color: var(--k8s-blue, #326ce5);
  line-height: 1;
  margin-bottom: 0.5rem;
}
.quiz-verdict {
  font-size: 1.1rem;
  color: var(--k8s-ink, #1a2038);
  margin-bottom: 1.5rem;
}
</style>
