<template>
    <div class="fixed top-5 right-5 z-50 flex space-x-2">
        <button @click="setPreference('light')" title="Light Mode"
            class="w-12 h-12 rounded-full focus:outline-none focus:ring-2 transition-all duration-300 flex items-center justify-center"
            :class="currentPreference === 'light'
                ? 'bg-sky-400 text-white ring-sky-300 scale-110 shadow-lg'
                : 'bg-gray-200 dark:bg-gray-700 hover:bg-gray-300 dark:hover:bg-gray-600 text-gray-700 dark:text-gray-200 focus:ring-gray-400'">
            <i class="fas fa-sun text-base"></i>
        </button>
        <button @click="setPreference('dark')" title="Dark Mode"
            class="w-12 h-12 rounded-full focus:outline-none focus:ring-2 transition-all duration-300 flex items-center justify-center"
            :class="currentPreference === 'dark'
                ? 'bg-indigo-500 text-white ring-indigo-400 scale-110 shadow-lg'
                : 'bg-gray-200 dark:bg-gray-700 hover:bg-gray-300 dark:hover:bg-gray-600 text-gray-700 dark:text-gray-200 focus:ring-gray-400'">
            <i class="fas fa-moon text-base"></i>
        </button>
        <button @click="setPreference('system')" title="System Preference"
            class="w-12 h-12 rounded-full focus:outline-none focus:ring-2 transition-all duration-300 flex items-center justify-center"
            :class="currentPreference === 'system'
                ? 'bg-gray-500 text-white ring-gray-400 scale-110 shadow-lg'
                : 'bg-gray-200 dark:bg-gray-700 hover:bg-gray-300 dark:hover:bg-gray-600 text-gray-700 dark:text-gray-200 focus:ring-gray-400'">
            <i class="fas fa-desktop text-base"></i>
        </button>
    </div>
</template>

<script setup>
defineProps({
    currentPreference: String // 'light', 'dark', 'system'
});

const emit = defineEmits(['set-preference']);

const setPreference = (preference) => {
    emit('set-preference', preference);
};
</script>

<style scoped></style>
