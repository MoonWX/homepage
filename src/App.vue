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
    console.log('[App.vue] Computing isEffectivelyDark: themePreference =', themePreference.value, ', systemIsDark =', systemIsDark.value);
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
    console.log('[App.vue] Mounted.');
    setTimeout(() => {
        console.log('[App.vue] Initial loading time finished, setting isAppLoading to false.');
        isAppLoading.value = false;
    }, initialLoadingTime);

    const savedPreference = localStorage.getItem('themePreference');
    console.log('[App.vue] Loaded themePreference from localStorage:', savedPreference);
    if (['light', 'dark', 'system'].includes(savedPreference)) {
        themePreference.value = savedPreference;
    } else {
        themePreference.value = 'system';
    }
    console.log('[App.vue] Current themePreference set to:', themePreference.value);

    const mediaQuery = window.matchMedia('(prefers-color-scheme: dark)');
    systemIsDark.value = mediaQuery.matches;
    console.log('[App.vue] Initial systemIsDark:', systemIsDark.value);

    mediaQueryListener = (e) => {
        console.log('[App.vue] System theme changed, new systemIsDark:', e.matches);
        systemIsDark.value = e.matches;
    };
    mediaQuery.addEventListener('change', mediaQueryListener);
});

onUnmounted(() => {
    console.log('[App.vue] Unmounted, removing media query listener.');
    if (mediaQueryListener) {
        const mediaQuery = window.matchMedia('(prefers-color-scheme: dark)');
        mediaQuery.removeEventListener('change', mediaQueryListener);
    }
});

watch(isEffectivelyDark, (newValue, oldValue) => {
    console.log(`[App.vue] isEffectivelyDark changed from ${oldValue} to ${newValue}. Toggling 'dark' class on html.`);
    document.documentElement.classList.toggle('dark', newValue);
}, { immediate: true });

const handleThemePreferenceChange = (preference) => {
    console.log('[App.vue] handleThemePreferenceChange called with:', preference);
    themePreference.value = preference;
    try {
        localStorage.setItem('themePreference', preference);
        console.log('[App.vue] themePreference saved to localStorage.');
    } catch (e) {
        console.error('[App.vue] Error saving themePreference to localStorage:', e);
    }
};
// --- End Theme Management Logic ---

watch(isAppLoading, (newIsAppLoading) => {
    console.log('[App.vue] isAppLoading changed to:', newIsAppLoading);
    if (!newIsAppLoading) {
        contentVisible.value = true;
    } else {
        contentVisible.value = false;
    }
});

const handleLoadingSequenceFinished = () => {
    console.log('[App.vue] handleLoadingSequenceFinished called, setting showLoadingScreen to false.');
    showLoadingScreen.value = false;
};
</script>

<style scoped>
/* App.vue specific styles if any */
</style>
