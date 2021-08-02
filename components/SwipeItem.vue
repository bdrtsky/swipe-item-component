<template>
  <div
    class="relative overflow-hidden transition-all duration-500 h-12"
    :class="[isSwiping ? 'select-none' : '', fullSwipeChoice && 'h-0 ']"
    @touchstart="processTouchStart"
    @touchend="processTouchEnd"
  >
    <div
      class="absolute top-0 left-0 w-full h-full bg-green-500"
      :class="[initDirection === 'RIGHT' ? 'z-0' : '-z-1']"
    >
      <button class="h-12 w-12 select-none" @click="processChoice">✓</button>
    </div>
    <div
      class="absolute top-0 left-0 w-full h-full bg-red-500 flex justify-end"
      :class="[initDirection === 'LEFT' ? 'z-0' : '-z-1']"
    >
      <button class="h-12 w-12 select-none" @click="processChoice">✘</button>
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
        !initIsSwiping && 'transition-all duration-500',
        initDirection === 'RIGHT'
          ? 'ml-auto'
          : initDirection === 'LEFT'
          ? 'mr-auto'
          : '',
      ]"
      :style="{
        width: `calc(100% - ${widthCalc}px)`,
      }"
    >
      <div class="px-4">
        <slot></slot>
        <span>{{ direction }} :: {{ initIsSwiping }} </span>
      </div>
    </div>
  </div>
</template>

<script>
import { defineComponent, ref, watch } from '@nuxtjs/composition-api'
import { useSwipe, useWindowSize } from '@vueuse/core'

export default defineComponent({
  setup(props, context) {
    const el = ref(null)
    const initDirection = ref(null)
    const initIsSwiping = ref(false)
    const fullSwipeChoice = ref(false)
    const confirmChoice = ref(false)
    const widthCalc = ref(0)
    const buttonWidth = 48

    const { width: viewportWidth } = useWindowSize()
    const { direction, lengthX, isSwiping } = useSwipe(el, {
      threshold: 1,
      onSwipeStart(e) {
        confirmChoice.value = false
      },
      onSwipeEnd(e) {
        // fullSwipeChoice.value = true
      },
    })

    watch(lengthX, (n, o) => {
      if (
        initIsSwiping.value === true ||
        direction.value === 'LEFT' ||
        direction.value === 'RIGHT'
      ) {
        initIsSwiping.value = true
        const sign = Math.sign(n)
        initDirection.value = sign > 0 ? 'LEFT' : sign < 0 ? 'RIGHT' : null
        widthCalc.value = Math.abs(lengthX.value)
        if (Math.abs(lengthX.value) >= viewportWidth.value / 2) {
          processChoice()
        }
      }
    })

    // watch(isSwiping, (n, o) => {
    //   if (n === false) {
    //     initIsSwiping.value = false // ???
    //   }
    // })

    watch(fullSwipeChoice, (n, o) => {
      if (n === true) {
        widthCalc.value = viewportWidth.value
      }
    })

    watch(initIsSwiping, (n, o) => {
      if (n === true) {
        const scrollBarGap =
          window.innerWidth - document.documentElement.clientWidth
        document.body.style.overflow = 'hidden'
        document.body.style.paddingRight = `${scrollBarGap}px`
      }
    })

    // watch(widthCalc, (n, o) => {
    //   if (n !== buttonWidth && !confirmChoice.value) {
    //     initIsSwiping.value = true
    //   }
    // })

    function processTouchEnd() {
      console.log('processTouchEnd')
      initIsSwiping.value = false
      if (!fullSwipeChoice.value && widthCalc.value) {
        confirmChoice.value = true
        widthCalc.value = buttonWidth
      }
      document.body.style.overflow = null
      document.body.style.paddingRight = null
    }

    function processTouchStart() {
      // setTimeout(() => {
      //   console.log('processTouchStart', widthCalc.value)
      // }, 500)
      // const scrollBarGap =
      //   window.innerWidth - document.documentElement.clientWidth
      // document.body.style.overflow = 'hidden'
      // document.body.style.paddingRight = `${scrollBarGap}px`
    }

    function processChoice() {
      navigator.vibrate(200)
      initIsSwiping.value = false
      widthCalc.value = viewportWidth.value
      fullSwipeChoice.value = true
    }

    return {
      el,
      direction,
      lengthX,
      isSwiping,
      initDirection,
      initIsSwiping,
      fullSwipeChoice,
      confirmChoice,
      processTouchEnd,
      processTouchStart,
      widthCalc,
      processChoice,
    }
  },
})
</script>
