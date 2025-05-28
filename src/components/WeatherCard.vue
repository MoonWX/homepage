<template>
    <CardWrapper title="Weather Here" icon="fas fa-cloud-sun" iconColor="text-yellow-300" :isDarkMode="isDarkMode">
        <div class="transition-colors duration-500 ease-in-out" :class="isDarkMode ? 'text-gray-100' : 'text-gray-700'">
            <p class="text-lg">
                {{ weatherDetails.city }}:
                <span>{{ weatherDetails.temp }}</span>,
                <span>{{ weatherDetails.desc }}</span>
            </p>
            <button @click="fetchLocalWeather" :disabled="isFetching"
                class="mt-4 px-4 py-2 text-sm rounded-lg transition-colors duration-300 focus:outline-none focus:ring-2"
                :class="isDarkMode
                    ? 'bg-gray-700 hover:bg-gray-600 text-white focus:ring-gray-500 disabled:bg-gray-800'
                    : 'bg-blue-600 hover:bg-blue-500 text-white focus:ring-blue-300 disabled:bg-blue-300'">
                <i :class="isFetching ? 'fas fa-spinner fa-spin' : 'fas fa-sync-alt'" class="mr-1"></i>
                {{ isFetching ? 'Refreshing...' : 'Refresh Weather' }}
            </button>
        </div>
    </CardWrapper>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import CardWrapper from './CardWrapper.vue';

const props = defineProps({
    isDarkMode: Boolean,
    locationForFetch: {
        type: String,
        default: 'changchun' // Default location if not provided
    }
});

const weatherDetails = ref({
    temp: 'N/A',
    desc: 'Loading...',
    city: props.locationForFetch
});
const isFetching = ref(false);

const fetchLocalWeather = async () => {
    isFetching.value = true;
    weatherDetails.value.desc = 'Refreshing...';
    try {
        const response = await fetch(`https://api.seniverse.com/v3/weather/now.json?key=SzBy2rns8CStxZTsE&location=${props.locationForFetch}&language=en&unit=c`);
        if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
        }
        const data = await response.json();
        if (data.results && data.results.length > 0) {
            const weatherResult = data.results[0];
            weatherDetails.value = {
                temp: `${weatherResult.now.temperature}°C`,
                desc: weatherResult.now.text,
                city: weatherResult.location.name,
            };
        } else {
            throw new Error("Weather data unavailable");
        }
    } catch (error) {
        console.error("Failed to fetch weather:", error);
        weatherDetails.value = { temp: 'N/A', desc: 'Failed to retrieve', city: props.locationForFetch };
    } finally {
        isFetching.value = false;
    }
};

onMounted(() => {
    fetchLocalWeather();
});
</script>
