<template>
  <div
    class="relative overflow-hidden transition-all duration-500 h-12"
    :class="[isSwiping ? 'select-none' : '', closeItem && 'h-0 ']"
  >
    <div
      class="absolute top-0 left-0 w-full h-full bg-green-500"
      :class="[
        chosenDirection === 'RIGHT' ? 'z-0' : '-z-1',
        fullSwipeChoice && 'text-white',
      ]"
    >
      <button
        ref="left-button"
        class="h-12 w-12 select-none"
        @click="processIrreversibleChoice"
      >
        ✓
      </button>
    </div>
    <div
      class="absolute top-0 left-0 w-full h-full bg-red-500 flex justify-end"
      :class="[
        chosenDirection === 'LEFT' ? 'z-0' : '-z-1',
        fullSwipeChoice && 'text-white',
      ]"
    >
      <button
        ref="right-button"
        class="h-12 w-12 select-none"
        @click="processIrreversibleChoice"
      >
        ✘
      </button>
    </div>
    <div
      ref="el"
      class="
        relative
        h-12
        bg-white
        border-b
        flex
        items-center
        select-none
        truncate
      "
      :class="[
        !isHorizontalSwiping && 'transition-all duration-500',
        chosenDirection === 'RIGHT'
          ? 'ml-auto'
          : chosenDirection === 'LEFT'
          ? 'mr-auto'
          : '',
      ]"
      :style="{
        width: `calc(100% - ${widthCalc}px)`,
      }"
    >
      <div class="px-4">
        <slot></slot>
        <span>{{ confirmChoice }} </span>
      </div>
    </div>
  </div>
</template>

<script>
import { defineComponent, ref, watch } from '@nuxtjs/composition-api'
import { useSwipe, useWindowSize } from '@vueuse/core'

export default defineComponent({
  setup(props, context) {
    const mockControlsData = ref({
      LEFT: {
        reversible: true,
        state: false,
      },
      RIGHT: {
        reversible: false,
        state: false,
      },
    })

    const el = ref(null)
    const chosenDirection = ref('NONE')
    const isHorizontalSwiping = ref(false)
    const fullSwipeChoice = ref(false)
    const closeItem = ref(false)
    const confirmChoice = ref(false)
    const widthCalc = ref(0)
    const buttonWidth = 48

    const { width: viewportWidth } = useWindowSize()
    const { direction, lengthX, isSwiping } = useSwipe(el, {
      threshold: 1,
      onSwipeStart(e) {
        console.log('onSwipeStart')
        // confirmChoice.value = false
      },
      onSwipeEnd(e) {
        console.log('onSwipeEnd')
        // confirmChoice.value = false
        if (fullSwipeChoice.value === true) {
          processIrreversibleChoice()
        } else if (chosenDirection.value === 'LEFT') {
          context.refs['right-button'].focus()
        } else if (chosenDirection.value === 'RIGHT') {
          context.refs['left-button'].focus()
        }

        isHorizontalSwiping.value = false

        if (!fullSwipeChoice.value && widthCalc.value) {
          if (!confirmChoice.value) {
            confirmChoice.value = true
          } else {
            confirmChoice.value = false
          }
          widthCalc.value = buttonWidth
        }
        resetBodyLock()
      },
    })

    watch(lengthX, (n, o) => {
      if (
        isHorizontalSwiping.value === true ||
        direction.value === 'LEFT' ||
        direction.value === 'RIGHT' ||
        direction.value === 'NONE'
      ) {
        isHorizontalSwiping.value = true
        const sign = Math.sign(n)
        chosenDirection.value = sign > 0 ? 'LEFT' : sign < 0 ? 'RIGHT' : 'NONE'
        widthCalc.value = Math.abs(lengthX.value)
        if (Math.abs(lengthX.value) >= viewportWidth.value / 2) {
          fullSwipeChoice.value = true
        } else {
          fullSwipeChoice.value = false
        }

        // disable confirmation button
        if (chosenDirection.value === 'NONE') {
          confirmChoice.value = false
        }
      }
    })

    watch(isHorizontalSwiping, (n, o) => {
      if (n === true) {
        bodyLock()
      }
    })

    watch(confirmChoice, (n, o) => {
      if (n === false) {
        resetConfirmChoice()
      }
    })

    function processIrreversibleChoice() {
      isHorizontalSwiping.value = false
      widthCalc.value = viewportWidth.value
      closeItem.value = true
    }

    function resetConfirmChoice() {
      widthCalc.value = 0
    }

    function bodyLock() {
      const scrollBarGap =
        window.innerWidth - document.documentElement.clientWidth
      document.body.style.overflow = 'hidden'
      document.body.style.paddingRight = `${scrollBarGap}px`
    }

    function resetBodyLock() {
      document.body.style.overflow = null
      document.body.style.paddingRight = null
    }

    return {
      el,
      direction,
      lengthX,
      isSwiping,
      chosenDirection,
      isHorizontalSwiping,
      fullSwipeChoice,
      confirmChoice,
      widthCalc,
      processIrreversibleChoice,
      closeItem,
    }
  },
})
</script>
