<template>
    <div ref="cardRef" class="rounded-xl p-6 content-card" :class="[
        isDarkMode ? 'glassmorphism-dark' : 'glassmorphism-light',
        // Apply is-hovering class only if effect is not disabled and mouse is over
        (!hoverEffectDisabled && isHoveringLocal) ? (isDarkMode ? 'dark-theme-bg' : 'light-theme-bg') + ' is-hovering' : ''
    ]" :style="cardStyle" @mousemove="handleMouseMove" @mouseenter="handleMouseEnter" @mouseleave="handleMouseLeave">
        <h2 v-if="title" class="text-xl font-semibold mb-4 transition-colors duration-500"
            :class="isDarkMode ? 'text-white' : 'text-gray-800'">
            <i v-if="icon" :class="[icon, iconColor ? iconColor : (isDarkMode ? 'text-gray-300' : 'text-gray-600')]"
                class="mr-2 transition-colors duration-500"></i>
            {{ title }}
        </h2>
        <div class="card-slot-content transition-colors duration-500 ease-in-out"
            :class="isDarkMode ? 'text-gray-100' : 'text-gray-700'">
            <slot></slot>
        </div>
    </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue';

const props = defineProps({
    title: String,
    icon: String,
    iconColor: String,
    isDarkMode: Boolean,
    hoverEffectDisabled: {
        type: Boolean,
        default: false
    }
});

const cardRef = ref(null);
const rotateX = ref(0);
const rotateY = ref(0);
const scale = ref(1);
const translateZ = ref(0);
const isHoveringLocal = ref(false); // Local state for mouse being over this specific card

const maxRotate = 4;
const hoverScale = 1.015;
const hoverTranslateZ = 0;

// Watcher to reset transform when hoverEffectDisabled becomes true
watch(() => props.hoverEffectDisabled, (isDisabled) => {
    if (isDisabled) {
        // Force reset transform values when effect becomes disabled
        rotateX.value = 0;
        rotateY.value = 0;
        scale.value = 1;
        translateZ.value = 0;
    }
    // When it becomes enabled (false), the normal hover logic will take over if mouse is present.
});

const cardStyle = computed(() => {
    if (props.hoverEffectDisabled) {
        // If the effect is disabled (e.g., map interaction), force no transform.
        // Add a very quick transition to 'none' to avoid jumps if it was mid-transform.
        return {
            transform: 'none',
            transition: 'transform 0.05s ease-out' // Quick transition to identity
        };
    }
    // If effect is enabled, apply transform based on interaction values.
    // The main transition for transform is defined in main.css on .content-card
    return {
        transform: `rotateX(${rotateX.value}deg) rotateY(${rotateY.value}deg) scale(${scale.value}) translateZ(${translateZ.value}px)`,
    };
});

const handleMouseEnter = () => {
    isHoveringLocal.value = true;
    if (props.hoverEffectDisabled) {
        // Ensure transform values are at identity if effect is disabled
        rotateX.value = 0;
        rotateY.value = 0;
        scale.value = 1;
        translateZ.value = 0;
        return;
    }
    // Apply hover scale and Z (which is 0)
    scale.value = hoverScale;
    translateZ.value = hoverTranslateZ;
    // rotateX/Y will be set by mousemove
};

const handleMouseMove = (event) => {
    if (props.hoverEffectDisabled || !isHoveringLocal.value) {
        // If effect is disabled, or mouse isn't truly over, ensure no rotation is calculated from mouse.
        // If disabled, values should already be at identity due to watcher or mouseEnter logic.
        return;
    }

    if (!cardRef.value) return;
    const rect = cardRef.value.getBoundingClientRect();
    const mouseX = event.clientX - rect.left;
    const mouseY = event.clientY - rect.top;
    const centerX = rect.width / 2;
    const centerY = rect.height / 2;
    const normalizedX = (mouseX - centerX) / centerX;
    const normalizedY = (mouseY - centerY) / centerY;

    rotateY.value = normalizedX * maxRotate;
    rotateX.value = -normalizedY * maxRotate;
};

const handleMouseLeave = () => {
    isHoveringLocal.value = false;
    // Always reset transform values when mouse leaves the card.
    // The computed style will then correctly apply 'none' if hoverEffectDisabled is true,
    // or the identity transform if hoverEffectDisabled is false.
    rotateX.value = 0;
    rotateY.value = 0;
    scale.value = 1;
    translateZ.value = 0;
};
</script>

<style scoped></style>
