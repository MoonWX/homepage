<template>
    <div class="curtain-loader fixed inset-0 z-50 flex items-center justify-center overflow-hidden"
        :class="{ 'pointer-events-none': !showCurtains }">
        <div class="curtain curtain-left fixed top-0 left-0 h-full w-1/2 bg-gray-800 dark:bg-gray-900 transform transition-transform duration-1000 ease-in-out"
            :class="{ 'translate-x-0': showCurtains, '-translate-x-full': !showCurtains }"></div>
        <div class="curtain curtain-right fixed top-0 right-0 h-full w-1/2 bg-gray-800 dark:bg-gray-900 transform transition-transform duration-1000 ease-in-out"
            :class="{ 'translate-x-0': showCurtains, 'translate-x-full': !showCurtains }"></div>

        <div v-if="showCurtains" class="absolute z-10 transition-opacity duration-300"
            :class="showSpinnerIcon ? 'opacity-100' : 'opacity-0'">
            <div class="loader ease-linear rounded-full border-4 border-t-4 border-gray-200/50 h-16 w-16"
                :style="{ borderTopColor: 'rgb(59 130 246)' }"></div>
        </div>
    </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, watch } from 'vue';

const props = defineProps({
    isLoading: { // This prop comes from App.vue's isAppLoading
        type: Boolean,
        default: true
    }
});

const emit = defineEmits(['animationComplete']); // Define the event

const showCurtains = ref(true); // Internal state for curtain animation
const showSpinnerIcon = ref(true); // Internal state for spinner icon visibility

let curtainOpenTimer = null;
let animationCompleteTimer = null;

watch(() => props.isLoading, (newVal) => {
    if (!newVal) { // When App.vue's isAppLoading becomes false (trigger to start exit animation)
        showSpinnerIcon.value = false; // Start fading out spinner icon

        clearTimeout(curtainOpenTimer);
        clearTimeout(animationCompleteTimer);

        curtainOpenTimer = setTimeout(() => {
            showCurtains.value = false; // Start opening curtains

            animationCompleteTimer = setTimeout(() => {
                emit('animationComplete');
            }, 1000); // Duration of curtain transform CSS animation
        }, 300); // Delay after spinner starts fading before curtains open
    } else {
        showCurtains.value = true;
        showSpinnerIcon.value = true;
        clearTimeout(curtainOpenTimer);
        clearTimeout(animationCompleteTimer);
    }
});

onMounted(() => {
    if (!props.isLoading) {
        showSpinnerIcon.value = false;
        showCurtains.value = false;
        // emit('animationComplete'); // Consider if needed for this edge case
    }
});

onUnmounted(() => {
    clearTimeout(curtainOpenTimer);
    clearTimeout(animationCompleteTimer);
});

</script>

<style scoped>
.loader {
    animation: spinner 1.2s linear infinite;
}

@keyframes spinner {
    0% {
        transform: rotate(0deg);
    }

    100% {
        transform: rotate(360deg);
    }
}

/* .curtain {
  z-index: 50; /* BUG FIX: This was causing the curtains to cover the spinner. Removed or commented out. */
/* }
*/
</style>
