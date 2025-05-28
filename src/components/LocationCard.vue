<template>
    <CardWrapper title="My Geographical Location" icon="fas fa-map-marker-alt" iconColor="text-orange-400"
        :isDarkMode="isDarkMode" :hoverEffectDisabled="isMapInteracting">
        <p class="mb-3 text-sm">
            I'm living in {{ locationInfo.city }}, {{ locationInfo.country }} now.
        </p>
        <div class="aspect-video rounded-lg overflow-hidden border relative"
            :class="isDarkMode ? 'border-gray-700/50' : 'border-gray-300/50'">
            <div ref="mapContainerRef" class="w-full h-full absolute top-0 left-0" @mouseenter="isMouseOverMap = true"
                @mouseleave="isMouseOverMap = false">
            </div>
            <div v-if="mapError"
                class="absolute inset-0 flex items-center justify-center p-4 text-center bg-white/50 dark:bg-black/50">
                <p :class="isDarkMode ? 'text-red-300' : 'text-red-600'">{{ mapError }}</p>
            </div>
        </div>
    </CardWrapper>
</template>

<script setup>
import { ref, onMounted, onUnmounted, watch, computed, nextTick } from 'vue'; // Added nextTick
import CardWrapper from './CardWrapper.vue';

const props = defineProps({
    locationInfo: {
        type: Object,
        required: true,
        default: () => ({ city: 'Changchun', country: 'China', latitude: 43.817003, longitude: 125.323629 })
    },
    isDarkMode: Boolean
});

const mapContainerRef = ref(null);
const mapInstance = ref(null);
const mapError = ref('');

const isMouseOverMap = ref(false);
const isMapDragging = ref(false);

const isMapInteracting = computed(() => isMouseOverMap.value || isMapDragging.value);

function ensureTMapIsReady() {
    return new Promise((resolve, reject) => {
        if (window.TMap && window.TMap.LatLng) { // Check for a specific TMap constructor
            resolve(window.TMap);
        } else {
            let attempts = 0;
            const interval = setInterval(() => {
                if (window.TMap && window.TMap.LatLng) {
                    clearInterval(interval);
                    resolve(window.TMap);
                } else if (attempts++ > 100) { // Wait up to 10 seconds
                    clearInterval(interval);
                    console.error('Tencent Map SDK (window.TMap) not found after 10 seconds.');
                    reject(new Error('Tencent Map SDK (TMap) not found. Ensure it is loaded in index.html.'));
                }
            }, 100);
        }
    });
}

const handleMapDragStart = () => {
    isMapDragging.value = true;
};
const handleMapDragEnd = () => {
    isMapDragging.value = false;
};

async function initMap() {
    if (!mapContainerRef.value) {
        mapError.value = "Map container DOM element not ready.";
        console.warn(mapError.value);
        return;
    }
    mapError.value = '';

    try {
        console.log("Attempting to initialize map...");
        const TMap = await ensureTMapIsReady();
        console.log("TMap SDK ready:", TMap);

        if (mapInstance.value && typeof mapInstance.value.destroy === 'function') {
            console.log("Destroying existing map instance.");
            mapInstance.value.destroy();
            mapInstance.value = null;
        }
        // Ensure container is empty before initializing
        mapContainerRef.value.innerHTML = '';

        const center = new TMap.LatLng(props.locationInfo.latitude, props.locationInfo.longitude);
        console.log("Map center:", center);

        mapInstance.value = new TMap.Map(mapContainerRef.value, {
            center: center,
            zoom: 13,
            mapStyleId: props.isDarkMode ? 'style2' : 'style1',
            viewMode: '2D',
            draggable: true,
            scrollWheel: true,
            doubleClickZoom: true
        });
        console.log("Map instance created:", mapInstance.value);

        mapInstance.value.on('dragstart', handleMapDragStart);
        mapInstance.value.on('dragend', handleMapDragEnd);
        console.log("Map event listeners added.");

        new TMap.MultiMarker({
            map: mapInstance.value,
            styles: {
                "myLocationStyle": new TMap.MarkerStyle({
                    "width": 42,
                    "height": 42,
                    "anchor": { x: 15, y: 42 },
                    "src": 'https://mapapi.qq.com/web/lbs/javascriptGL/demo/img/markerNew.png',
                }),
            },
            geometries: [{
                "id": "current-location",
                "styleId": 'myLocationStyle',
                "position": center,
                "properties": { "title": props.locationInfo.city }
            }]
        });
        console.log("Marker added to map.");

    } catch (error) {
        console.error("Error initializing Tencent Map:", error);
        mapError.value = `Map Load Failed: ${error.message || 'Unknown error'}`;    
        if (mapContainerRef.value) {
            // mapContainerRef.value.innerHTML = `<p class="text-red-500 p-4 text-center">${mapError.value}</p>`;
        }
    }
}

onMounted(async () => {
    console.log("LocationCard.vue onMounted: mapContainerRef.value is", mapContainerRef.value);
    // Wait for next tick to ensure DOM is fully ready, especially if there are v-if directives higher up.
    await nextTick();
    if (mapContainerRef.value) {
        console.log("mapContainerRef is available, calling initMap.");
        initMap();
    } else {
        // This case should be less likely with nextTick, but as a fallback:
        console.warn("mapContainerRef not available on mount, watching...");
        const unwatch = watch(mapContainerRef, (newEl) => {
            if (newEl && !mapInstance.value) {
                console.log("mapContainerRef became available via watch, calling initMap.");
                initMap();
                unwatch(); // Stop watching once initialized
            }
        });
    }
});

watch(() => props.isDarkMode, (newDarkModeValue) => {
    if (mapInstance.value && window.TMap) {
        try {
            console.log("Setting map style for dark mode:", newDarkModeValue);
            mapInstance.value.setMapStyleId(newDarkModeValue ? 'style2' : 'style1');
        } catch (e) {
            console.error("Error setting map style:", e);
        }
    }
});

onUnmounted(() => {
    console.log("LocationCard.vue onUnmounted: Destroying map instance.");
    if (mapInstance.value) {
        if (typeof mapInstance.value.destroy === 'function') {
            mapInstance.value.destroy();
        }
        mapInstance.value = null;
    }
});
</script>

<style scoped>
/* Ensure map container has a background for better visual during loading or if map fails */
div[ref="mapContainerRef"] {
    background-color: #f0f0f0;
    /* Light gray fallback */
}

.dark div[ref="mapContainerRef"] {
    /* If you have .dark on a parent for dark mode */
    background-color: #333333;
    /* Dark gray fallback */
}
</style>
