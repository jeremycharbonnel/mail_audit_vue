<script setup lang="ts">
import { ref, nextTick, onMounted, onUnmounted } from 'vue'

import RecipientsBadge from '@/components/RecipientsBadge.vue'

const props = defineProps<{
  recipients: string[]
}>()

// holds the timeout for the resize event
let resizeTimeout: number = 0

// counts the number of recipients that are hidden
let numTruncated = ref<number>(0)

// stores the element's index from which recipients are hidden
let truncatedIndex = ref<number>(0)

// defines when the tooltip with all recipients shows
let showTooltip = ref<boolean>(false)

const recipientsRef = ref<HTMLInputElement | null>(null)

const updateHiddenRecipients = async () => {
  let recipientsContainer = recipientsRef.value

  // resets values and elements in case of resize
  numTruncated.value = 0
  truncatedIndex.value = props.recipients.length - 1

  recipientsContainer?.querySelectorAll<HTMLInputElement>('.recipient').forEach((recipient) => {
    recipient.classList.remove('overflow-ellipsis', 'hidden')
  })

  // awaits that the DOM is updated
  await nextTick()

  recipientsContainer
    ?.querySelectorAll<HTMLInputElement>('.recipient')
    .forEach((recipient, index) => {
      // if previous element was hidden, automatically hide current one
      // offset can't be calculated as "display: none" removes previous elements from the DOM
      if (index > truncatedIndex.value) {
        numTruncated.value++
        recipient.classList.add('hidden')
      } else if (recipient.offsetLeft + recipient.offsetWidth >= recipientsContainer.offsetWidth) {
        // first element does not get hidden but shows an ellipsis instead
        if (index === 0) {
          recipient.classList.add('overflow-ellipsis')
        } else {
          numTruncated.value++
          recipient.classList.add('hidden')
        }

        // sets the position for the ellipsis
        // and indicate that future recipients get hidden as well
        truncatedIndex.value = index - 1
      }
    })
}

// triggers the update on load and window resize
const resizeListener = window.addEventListener('resize', function () {
  clearTimeout(resizeTimeout)
  resizeTimeout = setTimeout(updateHiddenRecipients, 200)
})

onMounted(() => {
  updateHiddenRecipients()
  resizeListener
})

onUnmounted(() => resizeListener)
</script>

<template>
  <div class="wrapper">
    <div class="recipients" ref="recipientsRef">
      <span v-for="(recipient, index) in props.recipients" :key="index" class="recipient">
        {{ recipient }}<span v-if="index !== props.recipients.length - 1">,&nbsp;</span>
        <span v-if="index === truncatedIndex && props.recipients.length > 1">...</span>
      </span>
    </div>

    <RecipientsBadge
      @mouseover="showTooltip = true"
      @mouseleave="showTooltip = false"
      v-if="numTruncated > 0"
      :numTruncated="numTruncated"
    />
  </div>

  <!-- tooltip with all recipients -->
  <div v-show="showTooltip" class="tooltip">{{ props.recipients.join(', ') }}</div>
</template>

<style scoped>
.wrapper {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 8px;
}

/* interactive cursor on the badge */
.wrapper > span:last-child {
  cursor: pointer;
}

.recipients {
  display: flex;
  min-width: 0;
}

.tooltip {
  position: fixed;
  right: 8px;
  top: 8px;
  padding: 8px 16px;

  /* unecessary with ".join" method
  display: flex;
  align-items: center; */

  background-color: #666;
  color: #f0f0f0;
  border-radius: 24px;
}

.hidden {
  display: none;
}

.overflow-ellipsis {
  text-overflow: ellipsis;
  white-space: nowrap;
  overflow: hidden;
}
</style>
