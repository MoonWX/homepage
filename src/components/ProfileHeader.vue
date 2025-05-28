<template>
    <header class="text-center pt-12">

        <!--
            I commented this because I need to optimize the loading speed of website and there should be no JavaScript
            If you want to use this, please uncomment it and comment the next one.
        -->

        <!-- <img :src="avatarUrl" alt="avatar"
            class="w-24 h-24 sm:w-32 sm:h-32 rounded-full mx-auto mb-4 border-4 shadow-lg transition-colors duration-500"
            :class="isDarkMode ? 'border-gray-700/80' : 'border-white/80'">
        -->

        <img src="https://q.qlogo.cn/headimg_dl?dst_uin=2546595550&spec=100&img_type=webp" alt="avatar"
            fetchpriority="high"
            class="w-24 h-24 sm:w-32 sm:h-32 rounded-full mx-auto mb-4 border-4 shadow-lg transition-colors duration-500"
            :class="isDarkMode ? 'border-gray-700/80' : 'border-white/80'">

        <h1 class="text-3xl sm:text-4xl font-bold transition-colors duration-500 ease-in-out text-readable-on-bg"
            :class="isDarkMode ? 'text-white' : 'text-gray-900'">
            {{ profile.name }}
        </h1>
        <p class="text-lg mt-2 min-h-[1.5em] transition-colors duration-500 ease-in-out text-readable-on-bg"
            :class="isDarkMode ? 'text-white' : 'text-gray-900'">
            {{ displayedTagline }}<span class="typing-cursor" v-if="showCursor">|</span>
        </p>
    </header>
</template>

<script setup>
import { ref, onMounted, onUnmounted, watch, computed } from 'vue';

const props = defineProps({
    profile: Object,
    avatarUrl: String,
    isDarkMode: Boolean,
    contentVisible: Boolean
});

// 计算文本阴影，与文字颜色形成反差以提高可读性
const dynamicTextShadow = computed(() => {
    if (props.isDarkMode) {
        // 深色模式下，文字是浅色，添加一个细微的深色阴影/辉光
        return '0px 0px 6px rgba(0, 0, 0, 0.7), 0px 0px 3px rgba(0,0,0,0.9)';
    } else {
        // 浅色模式下，文字是深色，添加一个细微的浅色阴影/辉光
        return '0px 0px 4px rgba(255, 255, 255, 0.6), 0px 0px 2px rgba(255,255,255,0.8)';
    }
});

const displayedTagline = ref('');
const fullTagline = computed(() => props.profile.tagline || '');
const showCursor = ref(false);

let typingTimeout = null;
let charIndex = 0;
let isDeleting = false;
const typingSpeed = 100;
const deletingSpeed = 60;
const pauseBeforeDelete = 2000;
const pauseBeforeTyping = 500;
const initialDelayBeforeTypingStart = 300;

function typeEffect() {
    clearTimeout(typingTimeout);
    const currentTagline = fullTagline.value;

    if (!currentTagline) {
        displayedTagline.value = '';
        showCursor.value = false;
        return;
    }

    showCursor.value = true;

    if (isDeleting) {
        if (charIndex > 0) {
            displayedTagline.value = currentTagline.substring(0, charIndex - 1);
            charIndex--;
            typingTimeout = setTimeout(typeEffect, deletingSpeed);
        } else {
            isDeleting = false;
            showCursor.value = true;
            typingTimeout = setTimeout(typeEffect, pauseBeforeTyping);
        }
    } else {
        if (charIndex < currentTagline.length) {
            displayedTagline.value = currentTagline.substring(0, charIndex + 1);
            charIndex++;
            typingTimeout = setTimeout(typeEffect, typingSpeed);
        } else {
            isDeleting = true;
            showCursor.value = true;
            typingTimeout = setTimeout(typeEffect, pauseBeforeDelete);
        }
    }
}

function startOrResetTypingAnimation() {
    clearTimeout(typingTimeout);
    displayedTagline.value = '';
    charIndex = 0;
    isDeleting = false;
    if (fullTagline.value) {
        setTimeout(() => {
            typeEffect();
        }, initialDelayBeforeTypingStart);
    } else {
        showCursor.value = false;
    }
}

onMounted(() => {
    if (props.contentVisible && fullTagline.value) {
        startOrResetTypingAnimation();
    }
});

watch(() => props.contentVisible, (isVisible) => {
    if (isVisible && fullTagline.value) {
        startOrResetTypingAnimation();
    } else {
        clearTimeout(typingTimeout);
        displayedTagline.value = '';
        showCursor.value = false;
    }
});

watch(fullTagline, (newTagline, oldTagline) => {
    if (newTagline !== oldTagline && props.contentVisible) {
        startOrResetTypingAnimation();
    } else if (!newTagline) {
        clearTimeout(typingTimeout);
        displayedTagline.value = '';
        showCursor.value = false;
    }
});

onUnmounted(() => {
    clearTimeout(typingTimeout);
});
</script>

<style scoped>
.typing-cursor {
    animation: blink 0.7s infinite;
    font-weight: normal;
    margin-left: 1px;
}

@keyframes blink {

    0%,
    100% {
        opacity: 1;
    }

    50% {
        opacity: 0;
    }
}

.min-h-\[1\.5em\] {
    min-height: 1.5em;
}

.text-readable-on-bg {
    text-shadow: v-bind('dynamicTextShadow');
}
</style>
