<template>
    <div class="min-h-screen w-full transition-colors duration-500 ease-in-out" :style="appBackgroundStyle"
        :class="[isEffectivelyDark ? 'dark' : '']">

        <LoadingView v-if="showLoadingScreen" :isLoading="isAppLoading"
            @loading-sequence-finished="handleLoadingSequenceFinished" />

        <HomePageView v-if="!isAppLoading" :isEffectivelyDark="isEffectivelyDark" :contentVisible="contentVisible"
            :contentTransitionDelay="contentTransitionDelay" :current-theme-preference="themePreference"
            @update-theme-preference="handleThemePreferenceChange" />
    </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, computed, watch } from 'vue';
import profileData from './data/profileData.json';

import LoadingView from './views/LoadingView.vue';
import HomePageView from './views/HomePageView.vue';

const isAppLoading = ref(true);
const showLoadingScreen = ref(true);

const contentVisible = ref(false);
const contentTransitionDelay = ref('1.3s');
const initialLoadingTime = 1500;

// --- Theme Management Logic ---
const themePreference = ref('system');
const systemIsDark = ref(false);

const isEffectivelyDark = computed(() => {
    if (themePreference.value === 'light') return false;
    if (themePreference.value === 'dark') return true;
    return systemIsDark.value;
});

const appBackgroundStyle = computed(() => ({
    backgroundImage: `url(${isEffectivelyDark.value ? profileData.backgroundImages.dark : profileData.backgroundImages.light})`,
    backgroundSize: 'cover',
    backgroundPosition: 'center',
    backgroundAttachment: 'fixed'
}));

let mediaQueryListener = null;

onMounted(() => {
    console.log("   _____                        _____  _                ")
    console.log("  / ____|                      |  __ \\(_)               ")
    console.log(" | |     __ _ _ __ _ __   ___  | |  | |_  ___ _ __ ___  ")
    console.log(" | |    / _` | '__| '_ \\ / _ \\ | |  | | |/ _ \\ '_ ` _ \\ ")
    console.log(" | |___| (_| | |  | |_) |  __/ | |__| | |  __/ | | | | |")
    console.log("  \\_____\\__,_|_|  | .__/ \\___| |_____/|_|\\___|_| |_| |_|")
    console.log("                  | |                                   ")
    console.log("                  |_|                                   ")
    console.log("  Welcome to MoonWX's Homepage! Enjoy your stay! \n");
    setTimeout(() => {
        isAppLoading.value = false;
    }, initialLoadingTime);

    const savedPreference = localStorage.getItem('themePreference');
    if (['light', 'dark', 'system'].includes(savedPreference)) {
        themePreference.value = savedPreference;
    } else {
        themePreference.value = 'system';
    }

    const mediaQuery = window.matchMedia('(prefers-color-scheme: dark)');
    systemIsDark.value = mediaQuery.matches;

    mediaQueryListener = (e) => {
        systemIsDark.value = e.matches;
    };
    mediaQuery.addEventListener('change', mediaQueryListener);
});

onUnmounted(() => {
    if (mediaQueryListener) {
        const mediaQuery = window.matchMedia('(prefers-color-scheme: dark)');
        mediaQuery.removeEventListener('change', mediaQueryListener);
    }
});

watch(isEffectivelyDark, (newValue) => {
    document.documentElement.classList.toggle('dark', newValue);
}, { immediate: true });

const handleThemePreferenceChange = (preference) => {
    themePreference.value = preference;
    try {
        localStorage.setItem('themePreference', preference);
    } catch (e) {
        console.error('[App.vue] Error saving themePreference to localStorage:', e);
    }
};
// --- End Theme Management Logic ---

watch(isAppLoading, (newIsAppLoading) => {
    if (!newIsAppLoading) {
        contentVisible.value = true;
    } else {
        contentVisible.value = false;
    }
});

const handleLoadingSequenceFinished = () => {
    showLoadingScreen.value = false;
};
</script>

<style scoped>
/* App.vue specific styles if any */
</style>
