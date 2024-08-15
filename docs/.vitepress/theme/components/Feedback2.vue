<script setup lang="ts">
import { computed, ref, reactive } from 'vue'
import {
  feedbackOptions,
  type FeedbackType,
  getFeedbackOption
} from '../../types/Feedback'
import { useRouter } from 'vitepress'

const props = defineProps<{
  heading?: string
}>()

const prompts = [
  'Make it count!',
  'Leave some feedback for us!',
  `We're all ears 🐰`,
  'Tell us what is missing in FMHY',
  'Your thoughts matter to us 💡',
  'Feedback is a gift 🎁',
  'What do you think?',
  'We appreciate your support 🙏',
  'Help us make FMHY better 🤝',
  'We need your help 👋',
  'Your feedback is valuable 💯',
  'So... what do you think?',
  "I guess you don't need to say anything 😉"
]

function getPrompt() {
  return prompts[Math.floor(Math.random() * prompts.length)]
}

const messages = {
  bug: [
    "We're sorry to hear that!",
    'Please try to be as specific as possible and provide us with the steps to reproduce the bug.'
  ],
  suggestion: [
    "We're glad you want to share your ideas!",
    'Nix the fluff and just tell us what you think!',
    "We'll be happy to read your thoughts and incorporate them into our content."
  ],
  appreciation: [
    'We appreciate your support!',
    "We're always looking for ways to improve!.",
    'Your feedback is valuable and helps us make FMHY better.'
  ],
  other: [
    "We're always looking for ways to improve!",
    'Your feedback is valuable and helps us make FMHY better.'
  ]
}

function getMessage(type: FeedbackType['type']) {
  return messages[type][Math.floor(Math.random() * messages[type].length)]
}

const loading = ref<boolean>(false)
const error = ref<unknown>(null)
const success = ref<boolean>(false)

const isDisabled = computed(() => {
  return (
    !feedback.message.length ||
    feedback.message.length < 5 ||
    feedback.message.length > 1000
  )
})

const router = useRouter()
const feedback = reactive<
  Pick<FeedbackType, 'message' | 'page'> & Partial<Pick<FeedbackType, 'type'>>
>({
  page: router.route.path,
  message: ''
})

const selectedOption = ref(feedbackOptions[0])

async function handleSubmit(type?: FeedbackType['type']) {
  if (type) feedback.type = type
  loading.value = true

  const body: FeedbackType = {
    message: feedback.message,
    type: selectedOption.value.value,
    page: feedback.page,
    ...(props.heading && { heading: props.heading })
  }

  try {
    const response = await fetch('/feedback', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(body)
    })

    const data = await response.json()
    if (data.error) {
      error.value = data.error
      return
    }
    if (data.status === 'ok') {
      success.value = true
    }
  } catch (err) {
    error.value = err
  } finally {
    loading.value = false
  }
}

const showCard = ref<boolean>(false)
const helpfulText = props.heading ? 'section' : 'page'

const prompt = computed(() => getPrompt())
const message = computed(() => getMessage(feedback.type!))
</script>

<template>
  <template v-if="props.heading">
    <button
      v-if="!showCard"
      @click="showCard = true"
      class="ml-auto inline-flex h-8 items-center justify-center rounded-md whitespace-nowrap text-sm text-primary font-medium border-solid border-1 border-primary hover:border-dotted bg-bg-alt px-2 py-2 sm:h-7"
    >
      <span class="i-carbon-send-alt" />
      <span class="sr-only">Send Feedback</span>
    </button>
    <button
      v-if="showCard"
      class="ml-auto inline-flex h-8 items-center justify-center rounded-md whitespace-nowrap text-sm text-primary font-medium border-solid border-1 border-primary hover:border-dotted bg-bg-alt px-2 py-2 sm:h-7"
      @click="showCard = false"
    >
      <span class="i-carbon-close" />
      <span class="sr-only">Close Feedback</span>
    </button>
  </template>
  <template v-else>
    <button
      v-if="!showCard"
      class="mt-2 text-primary font-bold text-underline text-sm"
      @click="showCard = true"
    >
      <span class="i-carbon-send-alt mr-2" />
      <span>Send Feedback</span>
    </button>
    <button
      v-if="showCard"
      class="mt-2 text-primary font-bold text-underline text-sm"
      @click="showCard = false"
    >
      <span class="i-carbon-close mr-2" />
      <span>Close Feedback</span>
    </button>
  </template>

  <Transition name="fade" mode="out-in">
    <div v-if="showCard" class="wrapper step">
      <Transition name="fade" mode="out-in">
        <div v-if="!feedback.type" class="step">
          <div>
            <div>
              <p class="desc">{{ prompt }}</p>
              <p class="heading">How helpful was this {{ helpfulText }}?</p>
            </div>
          </div>
          <div class="flex flex-wrap gap-2">
            <button
              v-for="item in feedbackOptions"
              :key="item.value"
              class="btn"
              @click="handleSubmit(item.value)"
            >
              <span>{{ item.label }}</span>
            </button>
          </div>
        </div>
        <div v-else-if="feedback.type && !success" class="step">
          <div>
            <p class="desc">
              Let us know how helpful this {{ helpfulText }} is
            </p>
            <div>
              <span>{{ getFeedbackOption(feedback.type)?.label }}</span>
              <button
                style="margin-left: 0.5rem"
                class="btn"
                @click="feedback.type = undefined"
              >
                <span class="i-carbon-close-large">close</span>
              </button>
            </div>
          </div>
          <p class="heading">
            {{ message }}
          </p>
          <div v-if="feedback.type === 'suggestion'" class="text-sm mb-2">
            <details>
              <summary>
                <span class="i-carbon-warning ml-1 mb-1 bg-cerise-400" />
                Do not submit any of the following:
              </summary>
              <strong>🕹️ Emulators</strong>
              <p class="desc">
                They're already on the
                <a
                  class="text-primary font-bold text-underline"
                  href="https://emulation.gametechwiki.com/index.php/Main_Page"
                >
                  Game Tech Wiki.
                </a>
              </p>
              <strong>🔻 Leeches</strong>
              <p class="desc">
                They're already on the
                <a
                  class="text-primary font-bold text-underline"
                  href="https://filehostlist.miraheze.org/wiki/Free_Premium_Leeches"
                >
                  File Hosting Wiki.
                </a>
              </p>
              <strong>🐧 Distros</strong>
              <p class="desc">
                They're already on
                <a
                  class="text-primary font-bold text-underline"
                  href="https://distrowatch.com/"
                >
                  DistroWatch.
                </a>
              </p>
              <strong>🎲 Mining / Betting Sites</strong>
              <p class="desc">
                Don't post anything related to betting, mining, BINs, CCs, etc.
              </p>
              <strong>🎮 Multiplayer Game Hacks</strong>
              <p class="desc">
                Don't post any hacks/exploits that give unfair advantages in
                multiplayer games.
              </p>
            </details>
          </div>
          <textarea
            v-model="feedback.message"
            autofocus
            class="input"
            placeholder="What a lovely wiki!"
          />
          <p class="desc mb-2">
            If you want a reply to your feedback, feel free to mention a contact
            in the message or join our
            <a
              class="text-primary font-semibold text-underline"
              href="https://discord.gg/Stz6y6NgNg"
            >
              Discord.
            </a>
          </p>
          <button
            type="submit"
            class="btn btn-primary"
            :disabled="isDisabled"
            @click="handleSubmit()"
          >
            Send Feedback 📩
          </button>
        </div>
        <div v-else class="step">
          <p class="heading">Thanks for your feedback!</p>
        </div>
      </Transition>
    </div>
  </Transition>
</template>

<style scoped lang="css">
.step > * + * {
  margin-top: 1rem;
}

.btn {
  border: 1px solid var(--vp-c-divider);
  background-color: var(--vp-c-bg);
  border-radius: 8px;
  transition:
    border-color 0.25s,
    background-color 0.25s;
  display: inline-block;
  font-size: 14px;
  font-weight: 500;
  line-height: 1.5;
  margin: 0;
  padding: 0.375rem 0.75rem;
  text-align: center;
  vertical-align: middle;
  white-space: nowrap;
}

.btn:disabled {
  opacity: 0.5;
}

.btn:hover {
  border-color: var(--vp-c-brand);
}

.btn-primary {
  color: #fff;
  background-color: var(--vp-c-brand);
  border-color: var(--vp-c-brand);
}

.btn-primary:hover {
  background-color: var(--vp-c-brand-darker);
  border-color: var(--vp-c-brand-darker);
}

.heading {
  font-size: 1.2rem;
  font-weight: 700;
}

.wrapper {
  margin: 2rem 0;
  padding: 1.5rem;
  border: 1px solid var(--vp-c-divider);
  border-radius: 8px;
  background: var(--vp-c-bg-alt);
}

.input {
  background-color: var(--vp-c-bg-alt);
  color: var(--vp-c-text-2);
  width: 100%;
  height: 100px;
  border: 1px solid var(--vp-c-divider);
  border-radius: 4px;
  padding: 0.375rem 0.75rem;
}

.desc {
  display: block;
  line-height: 20px;
  font-size: 12px;
  font-weight: 500;
  color: var(--vp-c-text-2);
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.25s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
