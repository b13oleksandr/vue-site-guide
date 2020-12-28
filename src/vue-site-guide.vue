<template>
  <div class="guide" ref="guide">
    <div v-if="innerValue" class="guide__backdrop" @click="onBackdropClick" />

    <div>
      <slot />
    </div>

    <div
      v-if="innerValue"
      class="guide__step-backdrop"
      :style="stepBackdropStyle"
      ref="tooltip"
    >
      <div
        ref="guideControl"
        class="guide__tooltip"
        :class="
          `guide__tooltip--${innerConfig.tooltipAlign} guide__tooltip--arrow-${innerConfig.tooltipAlign}`
        "
          :style="tooltipStyle"
      >
        <slot v-if="$scopedSlots[`${step}Title`]" :name="`${step}Title`" />
        <template v-else>
          <div
            v-if="currentStepData.title"
            class="guide__title"
            :class="innerConfig.titleClass"
            :style="innerConfig.titleStyle"
          >
            {{ currentStepData.title }}
          </div>
        </template>

        <slot v-if="$scopedSlots[`${step}Message`]" :name="`${step}Message`" />
        <template v-else>
          <div
            v-if="currentStepData.message"
            class="guide__message"
            :class="innerConfig.messageClass"
            :style="innerConfig.messageStyle"
          >
            {{ currentStepData.message }}
          </div>
        </template>

        <div class="guide__buttons">
          <slot
            v-if="$scopedSlots.buttonSkip"
            :skip="close"
            name="buttonSkip"
          />
          <div
            v-else
            class="guide__skip-button"
            @click="close"
            :class="innerConfig.skipButtonClass"
            :style="innerConfig.skipButtonStyle"
          >
            {{ innerConfig.skipButtonText }}
          </div>

          <div class="guide__nav-buttons">
            <slot
              v-if="$scopedSlots.buttonPrev"
              :prev="onPrevStep"
              :canPrev="canPrev"
              name="buttonPrev"
            />
            <button
              v-else
              @click="onPrevStep"
              :disabled="!canPrev"
              class="guide__nav-button"
              :class="innerConfig.prevButtonClass"
              :style="innerConfig.prevButtonStyle"
            >
              {{ innerConfig.prevButtonText }}
            </button>

            <slot
              v-if="$scopedSlots.buttonNext"
              :next="onNextStep"
              :canNext="canNext"
              name="buttonNext"
            />
            <button
              v-else
              @click="onNextStep"
              :disabled="!canNext"
              class="guide__nav-button"
              :class="innerConfig.nextButtonClass"
              :style="innerConfig.nextButtonStyle"
            >
              {{ innerConfig.nextButtonText }}
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
const TOOLTIP_ALIGN = {
  left: 'left',
  center: 'center',
  right: 'right'
}
const FIRST_STEP = 0
const defaultConfig = {
  steps: [],
  initialStep: FIRST_STEP,
  backdropClickToClose: true,
  tooltipAlign: TOOLTIP_ALIGN.center,
  titleClass: '',
  titleStyle: {},
  messageClass: '',
  messageStyle: {},
  skipButtonText: 'Skip',
  skipButtonClass: '',
  skipButtonStyle: {},
  prevButtonText: 'Prev',
  prevButtonClass: '',
  prevButtonStyle: {},
  nextButtonText: 'Next',
  nextButtonClass: '',
  nextButtonStyle: {},
  tooltipBgColor: '',
  tooltipRadius: '',
  tooltipPadding: '',
  tooltipShadow: '',
  stepBackdropColor: '',
  stepBackdropRadius: '',
  stepBackdropShadow: ''
}

export default {
  name: 'Guide',

  props: {
    config: {
      type: Object,
      default: () => defaultConfig
    },
    value: {
      type: Boolean,
      default: false
    }
  },

  data: vm => ({
    innerConfig: {
      ...defaultConfig,
      ...vm.config
    },
    step: vm.config.initialStep || defaultConfig.initialStep,
    tooltip: {},
    currentStepData: {
      message: ''
    },
    observeFlag: false,
    innerValue: vm.value,
    isBeginning: false,
    isEnd: false
  }),

  mounted() {
    if (this.value) {
      this.init()
    }
  },

  computed: {
    canPrev() {
      return !(this.isBeginning || this.reachBeginning)
    },

    canNext() {
      return !(this.isEnd || this.reachEnd)
    },

    reachBeginning() {
      return !this.isBeginning && this.step === FIRST_STEP
    },

    reachEnd() {
      return !this.isEnd && this.step >= this.stepsCount
    },

    stepsCount() {
      return this.innerConfig.steps.length - 1
    },

    noSteps() {
      return this.stepsCount <= 0
    },

    tooltipStyle() {
      return {
        ...(this.innerConfig.tooltipBgColor
            ? {
              'background-color': this.innerConfig.tooltipBgColor,
              'border-color': this.innerConfig.tooltipBgColor
            }
            : {}),
        ...(this.innerConfig.tooltipRadius
            ? { 'border-radius': this.innerConfig.tooltipRadius }
            : {}),
        ...(this.innerConfig.tooltipPadding
            ? { padding: this.innerConfig.tooltipPadding }
            : {}),
        ...(this.innerConfig.tooltipShadow
            ? { 'box-shadow': this.innerConfig.tooltipShadow }
            : {})
      }
    },

    stepBackdropStyle() {
      return {
        ...(this.innerConfig.stepBackdropColor
            ? {
              'background-color': this.innerConfig.stepBackdropColor
            }
            : {}),
        ...(this.innerConfig.stepBackdropRadius
            ? { 'border-radius': this.innerConfig.stepBackdropRadius }
            : {}),
        ...(this.innerConfig.stepBackdropShadow
            ? { 'box-shadow': this.innerConfig.stepBackdropShadow }
            : {})
      }
    }
  },

  watch: {
    step(step) {
      this.toStep(step)
    },

    value(value) {
      if (this.noSteps) {
        return
      }

      if (value && !this.innerValue) {
        this.init()
      }

      this.innerValue = value
    },

    reachBeginning(begin) {
      if (begin) {
        this.$emit('reachBeginning')
      }
    },

    reachEnd(end) {
      if (end) {
        this.$emit('reachEnd')
      }
    }
  },

  methods: {
    init() {
      if (!this.stepsCount) {
        return
      }

      this.innerValue = true

      this.$nextTick(() => {
        this.step = this.innerConfig.initialStep

        this.correctConfigRelativeToDomSteps()

        this.toStep(this.innerConfig.initialStep, false)

        this.observer = new IntersectionObserver(entries => {
          entries.forEach(entry => {
            if (!entry.isIntersecting && this.observeFlag) {
              this.$refs.tooltip.scrollIntoView()
              this.observeFlag = false
            }
          })
        })
        this.observer.observe(this.$refs.tooltip)

        if (this.innerConfig.initialStep === FIRST_STEP) {
          this.isBeginning = true
          this.$emit('isBeginning')
        } else if (this.innerConfig.initialStep >= this.stepsCount) {
          this.isEnd = true
          this.$emit('isEnd')
        }

        this.$emit('shown')
      })
    },

    onPrevStep() {
      if (this.step > FIRST_STEP) {
        this.$emit('prevStep', this.step)

        this.step--
      }
    },

    onNextStep() {
      if (this.step < this.stepsCount) {
        this.$emit('nextStep', this.step)

        this.step++
      }
    },

    toStep(stepNumber, needObserve = true) {
      if (
          !this.innerConfig.steps[stepNumber] ||
          !this.innerConfig.steps[stepNumber].el
      ) {
        return
      }

      this.observeFlag = needObserve
      this.currentStepData = this.innerConfig.steps[stepNumber] || {}

      this.resetStepsStyle()
      this.applyStyles(this.innerConfig.steps[stepNumber].el, {
        zIndex: '999999',
        position: 'relative'
      })

      const stepCoords = this.getStepCoords(stepNumber)
      const guideCords = this.$refs.guide.getBoundingClientRect()
      this.setStepBackdropPosition(
          stepCoords.top - guideCords.top,
          guideCords.right - stepCoords.right,
          guideCords.bottom - stepCoords.bottom,
          stepCoords.left - guideCords.left
      )

      this.isBeginning = false
      this.isEnd = false

      this.$emit('stepChanged', stepNumber)
    },

    setStepBackdropPosition(top, right, bottom, left) {
      this.applyStyles(this.$refs.tooltip, {
        top: `${top}px`,
        right: `${right}px`,
        bottom: `${bottom}px`,
        left: `${left}px`
      })

      this.correctTooltipPosition()
    },

    correctTooltipPosition() {
      const guideCords = this.$refs.guide.getBoundingClientRect()
      const tooltip = this.$refs.guideControl
      const tooltipCords = tooltip.getBoundingClientRect()

      this.applyStyles(tooltip, null)
      this.applyStyles(tooltip, this.tooltipStyle)
      this.setTooltipArrowPosition(this.innerConfig.tooltipAlign)

      const rightOffset = tooltipCords.right - guideCords.right
      if (rightOffset > 0) {
        this.applyStyles(tooltip, {
          transform: 'unset',
          right: 0,
          left: 'unset'
        })

        if (this.innerConfig.tooltipAlign !== TOOLTIP_ALIGN.right) {
          this.setTooltipArrowPosition(TOOLTIP_ALIGN.right)
        }
      }

      const leftOffset = guideCords.left - tooltipCords.left
      if (leftOffset > 0) {
        this.applyStyles(tooltip, {
          transform: 'unset',
          left: 0
        })

        if (this.innerConfig.tooltipAlign !== TOOLTIP_ALIGN.left) {
          this.setTooltipArrowPosition(TOOLTIP_ALIGN.left)
        }
      }
    },

    setTooltipArrowPosition(align) {
      this.$refs.guideControl.classList.remove('guide__tooltip--arrow-left')
      this.$refs.guideControl.classList.remove('guide__tooltip--arrow-center')
      this.$refs.guideControl.classList.remove('guide__tooltip--arrow-right')
      this.$refs.guideControl.classList.add(`guide__tooltip--arrow-${align}`)
    },

    applyStyles(selector, styles) {
      if (!styles) {
        selector.style = {}

        return
      }

      for (let [prop, value] of Object.entries(styles)) {
        selector.style[prop] = value
      }
    },

    onBackdropClick() {
      if (!this.innerConfig.backdropClickToClose) {
        return
      }

      this.close()
    },

    close() {
      this.$emit('beforeClose')

      this.destroyObserver()
      this.innerValue = false

      this.$emit('input', this.innerValue)
      this.$emit('close')
    },

    destroyObserver() {
      if (this.observer) {
        this.observer.disconnect()
      }
    },

    correctConfigRelativeToDomSteps() {
      this.innerConfig.steps = this.innerConfig.steps.reduce((steps, step) => {
        const domStep = this.$refs.guide.querySelector(step.selector)

        if (domStep) {
          this.$set(step, 'el', domStep)
          steps.push(step)
        }

        return steps
      }, [])
    },

    getStepCoords(stepNumber) {
      return this.innerConfig.steps[stepNumber].el.getBoundingClientRect()
    },

    resetStepsStyle() {
      this.innerConfig.steps.forEach(step => (step.el.style = {}))
    }
  },

  beforeDestroy() {
    this.destroyObserver()
  }
}
</script>

<style scoped>
.guide {
  position: relative;
}

.guide__backdrop {
  position: fixed;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  background-color: rgba(0, 0, 0, .5);
  z-index: 9999;
}

.guide__step-backdrop,
.guide__tooltip {
  z-index: 999998;
  position: absolute;
}

.guide__tooltip {
  top: 100%;
  margin-top: 16px;
  background: #fff;
  border-radius: 4px;
  padding: 16px;
  box-shadow: 0 3px 5px -1px rgba(0, 0, 0, .2), 0 5px 8px rgba(0, 0, 0, .14), 0 1px 14px rgba(0, 0, 0, .12);
  max-width: 300px;
  min-width: 300px;
  border-color: #fff;
}

.guide__tooltip::before {
  content: '';
  display: block;
  position: absolute;
  width: 0;
  height: 0;
  top: -8px;
  border-bottom: 8px solid #fff;
  border-left: 8px solid transparent !important;
  border-right: 8px solid transparent !important;;
  border-top: none;
  border-color: inherit;
}

.guide__tooltip--left {
  left: 0;
}

.guide__tooltip--center {
  transform: translateX(-50%);
  left: 50%;
}

.guide__tooltip--right {
  right: 0;
}

.guide__tooltip--arrow-right::before {
  right: 20px;
}

.guide__tooltip--arrow-left ::before {
  left: 20px;
}

.guide__tooltip--arrow-center::before {
  left: 50%;
  transform: translateX(-50%);
}

.guide__step-backdrop {
  display: block;
  background: #fff;
  position: absolute;
  z-index: 99999;
  box-shadow: 0 3px 5px -1px rgba(0, 0, 0, 0.2), 0 5px 8px rgba(0, 0, 0, 0.14), 0 1px 14px rgba(0, 0, 0, 0.12);
}

.guide__title {
  font-weight: bold;
  font-size: 17px;
  line-height: 21px;
  margin-bottom: 8px;
}

.guide__message {
  font-size: 14px;
  line-height: 18px;
  margin-bottom: 24px;
}

.guide__buttons {
  display: flex;
  justify-content: space-between;
  align-items: flex-end;
}

.guide__nav-buttons {
  display: flex;
  align-items: center;
}

.guide__nav-button {
  height: 32px;
  outline: none;
  background-color: #fff;
  border-radius: 4px;
  border: 2px solid #ccc;
  padding: 0 16px;
  font-size: 12px;
}

.guide__nav-button:not([disabled]) {
  cursor: pointer;
}

.guide__nav-button:first-child {
  margin-right: 8px;
}

.guide__skip-button {
  font-size: 12px;
  line-height: 18px;
  color: #777777;
  cursor: pointer;
}
</style>
