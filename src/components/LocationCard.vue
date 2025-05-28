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
            <div v-if="mapError" class="absolute inset-0 flex items-center justify-center p-4 text-center">
                <p :class="isDarkMode ? 'text-red-400' : 'text-red-500'">{{ mapError }}</p>
            </div>
        </div>
    </CardWrapper>
</template>

<script setup>
import { ref, onMounted, onUnmounted, watch, computed } from 'vue';
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

const TENCENT_MAP_KEY = 'H6DBZ-4KYYZ-MPEXE-TZXOE-AAEEQ-ZHFPG';

let sdkLoadPromise = null;

function loadTencentMapSDK() {
    if (!sdkLoadPromise) {
        sdkLoadPromise = new Promise((resolve, reject) => {
            if (window.TMap) {
                resolve(window.TMap);
                return;
            }
            if (document.getElementById('tencent-map-sdk-gljs')) {
                let attempts = 0;
                const interval = setInterval(() => {
                    if (window.TMap) {
                        clearInterval(interval);
                        resolve(window.TMap);
                    } else if (attempts++ > 50) {
                        clearInterval(interval);
                        reject(new Error('TMap SDK script present but TMap global not found after timeout.'));
                    }
                }, 100);
                return;
            }

            const script = document.createElement('script');
            script.id = 'tencent-map-sdk-gljs';
            script.src = `https://map.qq.com/api/gljs?v=1.exp&key=${TENCENT_MAP_KEY}`;
            script.async = true;
            script.defer = true;

            script.onload = () => {
                if (window.TMap) {
                    resolve(window.TMap);
                } else {
                    reject(new Error('TMap SDK loaded but TMap global not found.'));
                }
            };
            script.onerror = (event) => {
                console.error('Tencent Map SDK script loading error:', event);
                reject(new Error('Failed to load Tencent Map SDK script.'));
            };
            document.head.appendChild(script);
        });
    }
    return sdkLoadPromise;
}

const handleMapDragStart = () => {
    isMapDragging.value = true;
};
const handleMapDragEnd = () => {
    isMapDragging.value = false;
};

async function initMap() {
    if (!mapContainerRef.value) {
        mapError.value = "Map container not ready.";
        return;
    }
    mapError.value = '';

    try {
        const TMap = await loadTencentMapSDK();

        if (mapInstance.value && typeof mapInstance.value.destroy === 'function') {
            mapInstance.value.destroy();
            mapInstance.value = null;
        }
        mapContainerRef.value.innerHTML = '';

        const center = new TMap.LatLng(props.locationInfo.latitude, props.locationInfo.longitude);

        mapInstance.value = new TMap.Map(mapContainerRef.value, {
            center: center,
            zoom: 13,
            mapStyleId: props.isDarkMode ? 'style2' : 'style1',
            viewMode: '2D',
            draggable: true,
            scrollWheel: true,
            doubleClickZoom: true
        });

        mapInstance.value.on('dragstart', handleMapDragStart);
        mapInstance.value.on('dragend', handleMapDragEnd);

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
    } catch (error) {
        console.error("Error initializing Tencent Map:", error);
        mapError.value = `Map Load Failed: ${error.message}`;
        if (mapContainerRef.value) {
            mapContainerRef.value.innerHTML = `<p class="text-red-500 p-4 text-center">${mapError.value}</p>`;
        }
    }
}

onMounted(() => {
    watch(mapContainerRef, (newEl) => {
        if (newEl && !mapInstance.value) {
            initMap();
        }
    }, { immediate: true });
});

watch(() => props.isDarkMode, (newDarkModeValue) => {
    if (mapInstance.value && window.TMap) {
        try {
            mapInstance.value.setMapStyleId(newDarkModeValue ? 'style2' : 'style1');
        } catch (e) {
            console.error("Error setting map style:", e);
        }
    }
});

onUnmounted(() => {
    if (mapInstance.value) {
        if (typeof mapInstance.value.destroy === 'function') {
            mapInstance.value.destroy();
        }
    }
});
</script>

<style scoped></style>
