<template>
    <CardWrapper title="Time Comparison" icon="fas fa-clock" iconColor="text-indigo-500 dark:text-indigo-400"
        :isDarkMode="isDarkMode">
        <div class="space-y-3">
            <div>
                <p class="text-sm font-medium" :class="isDarkMode ? 'text-gray-300' : 'text-gray-600'">Visitor Local
                    Time:</p>
                <p class="text-2xl font-semibold tabular-nums" :class="isDarkMode ? 'text-white' : 'text-gray-800'">
                    {{ visitorTime }}
                </p>
                <p class="text-xs" :class="isDarkMode ? 'text-gray-400' : 'text-gray-500'">{{ visitorTimezone }}</p>
            </div>
            <div>
                <p class="text-sm font-medium" :class="isDarkMode ? 'text-gray-300' : 'text-gray-600'">My Local Time ({{
                    locationInfo.city }}):</p>
                <p class="text-2xl font-semibold tabular-nums" :class="isDarkMode ? 'text-white' : 'text-gray-800'">
                    {{ ownerTime }}
                </p>
                <p class="text-xs" :class="isDarkMode ? 'text-gray-400' : 'text-gray-500'">{{ ownerTimezoneDisplay }}
                </p>
            </div>
        </div>
    </CardWrapper>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue';
import CardWrapper from './CardWrapper.vue';
import profileDataJson from '../data/profileData.json';

defineProps({
    locationInfo: Object,
    isDarkMode: Boolean
});

const ownerTimezone = profileDataJson.ownerTimezone || 'Asia/Shanghai';
const ownerTimezoneDisplay = ref('');

const visitorTime = ref('');
const ownerTime = ref('');
const visitorTimezone = ref('');

let timer = null;

const formatTime = (date, timeZone) => {
    try {
        return new Intl.DateTimeFormat('zh-CN', {
            hour: '2-digit',
            minute: '2-digit',
            second: '2-digit',
            hour12: false,
            timeZone: timeZone,
        }).format(date);
    } catch (e) {
        console.error("Error formatting time for timezone:", timeZone, e);
        return 'N/A';
    }
};

const updateTimes = () => {
    const now = new Date();

    // Visitor's time
    try {
        const detectedTimezone = Intl.DateTimeFormat().resolvedOptions().timeZone;
        visitorTimezone.value = detectedTimezone;
        visitorTime.value = formatTime(now, detectedTimezone);
    } catch (e) {
        visitorTimezone.value = 'Unknown Timezone';
        visitorTime.value = formatTime(now, undefined); // Use system default if detection fails
        console.error("Error detecting visitor's timezone:", e);
    }

    // Owner's time
    ownerTime.value = formatTime(now, ownerTimezone);
    ownerTimezoneDisplay.value = ownerTimezone;
};

onMounted(() => {
    updateTimes();
    timer = setInterval(updateTimes, 1000); // Update every second
});

onUnmounted(() => {
    if (timer) {
        clearInterval(timer);
    }
});
</script>

<style scoped>
/* tabular-nums ensures numbers don't jump widths as they change */
.tabular-nums {
    font-variant-numeric: tabular-nums;
}
</style>
