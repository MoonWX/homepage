<template>
    <div ref="cardRef" class="rounded-xl p-6 content-card transition-all duration-500 ease-in-out" :class="[
        isDarkMode ? 'glassmorphism-dark' : 'glassmorphism-light',
        isHovering ? (isDarkMode ? 'dark-theme-bg' : 'light-theme-bg') + ' is-hovering' : ''
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
import { ref, computed } from 'vue';

defineProps({
    title: String,
    icon: String,
    iconColor: String,
    isDarkMode: Boolean
});

const cardRef = ref(null);
const rotateX = ref(0);
const rotateY = ref(0);
const scale = ref(1);
const translateZ = ref(0);
const isHovering = ref(false);

const maxRotate = 4;
const hoverScale = 1.015;
const hoverTranslateZ = 3;

const cardStyle = computed(() => ({
    transform: `rotateX(${rotateX.value}deg) rotateY(${rotateY.value}deg) scale(${scale.value}) translateZ(${translateZ.value}px)`,
}));

const handleMouseMove = (event) => {
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

const handleMouseEnter = () => {
    isHovering.value = true;
    scale.value = hoverScale;
    translateZ.value = hoverTranslateZ;
};

const handleMouseLeave = () => {
    isHovering.value = false;
    rotateX.value = 0;
    rotateY.value = 0;
    scale.value = 1;
    translateZ.value = 0;
};

</script>
