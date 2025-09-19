<template>
    <CardWrapper title="My Geographical Location" icon="fas fa-map-marker-alt" iconColor="text-orange-400"
        :isDarkMode="isDarkMode" :hoverEffectDisabled="isMapInteracting">
        <p class="mb-3 text-sm">
            I'm currently doing an internship in {{ locationInfo.city }}, {{ locationInfo.country }} now.
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
import { ref, onMounted, onUnmounted, watch, computed, nextTick } from 'vue';
import CardWrapper from './CardWrapper.vue';

const props = defineProps({
    locationInfo: {
        type: Object,
        required: true
    },
    isDarkMode: Boolean
});

const mapContainerRef = ref(null);
const mapInstance = ref(null);
const mapError = ref('');

const isMouseOverMap = ref(false);
const isMapDragging = ref(false);

const isMapInteracting = computed(() => isMouseOverMap.value || isMapDragging.value);

const LIGHT_MAP_STYLE_ID = 'style1';
const DARK_MAP_STYLE_ID = 'style2';

const mapEventListeners = ref([]);

function ensureQQMapsIsReady() {
    return new Promise((resolve, reject) => {
        if (window.qq && window.qq.maps && window.qq.maps.LatLng) {
            resolve(window.qq.maps);
        } else {
            let attempts = 0;
            const interval = setInterval(() => {
                if (window.qq && window.qq.maps && window.qq.maps.LatLng) {
                    clearInterval(interval);
                    resolve(window.qq.maps);
                } else if (attempts++ > 100) {
                    clearInterval(interval);
                    console.error('Tencent Map JS API V2 (qq.maps) not found after 10 seconds.');
                    reject(new Error('Tencent Map JS API V2 (qq.maps) not found. Ensure it is loaded in index.html with the correct SDK URL.'));
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
        const qqMaps = await ensureQQMapsIsReady();

        if (mapInstance.value) {
            mapEventListeners.value.forEach(listener => qqMaps.event.removeListener(listener));
            mapEventListeners.value = [];
        }
        mapContainerRef.value.innerHTML = '';

        const center = new qqMaps.LatLng(props.locationInfo.latitude, props.locationInfo.longitude);

        mapInstance.value = new qqMaps.Map(mapContainerRef.value, {
            center: center,
            zoom: 5,
            mapStyleId: props.isDarkMode ? DARK_MAP_STYLE_ID : LIGHT_MAP_STYLE_ID,
            mapTypeId: qqMaps.MapTypeId.ROADMAP,
            draggable: true,
            scrollwheel: true,
            disableDoubleClickZoom: false
        });

        const redMarkerIconUrl = 'https://mapapi.qq.com/web/lbs/javascriptGL/demo/img/markerNew.png';
        const markerIcon = new qqMaps.MarkerImage(
            redMarkerIconUrl,
            new qqMaps.Size(64, 64), // 图标的原始尺寸 (根据实际图片调整)
            new qqMaps.Point(0, 0),  // 图标的起点 (相对于图片左上角)
            new qqMaps.Point(20.5, 41.5), // 图标的锚点 (标记的哪个点对准地图坐标，通常是底部中心)
            new qqMaps.Size(42, 42)  // 图标在地图上显示的缩放尺寸 (根据实际图片调整)
        );
        new qqMaps.Marker({
            map: mapInstance.value,
            position: center,
            icon: markerIcon,
        });

        mapEventListeners.value.push(
            qqMaps.event.addListener(mapInstance.value, 'dragstart', handleMapDragStart)
        );
        mapEventListeners.value.push(
            qqMaps.event.addListener(mapInstance.value, 'dragend', handleMapDragEnd)
        );

    } catch (error) {
        console.error("Error initializing Tencent Map (qq.maps):", error);
        mapError.value = `Map load failed: ${error.message || 'Unknown error'}`;
    }
}

onMounted(async () => {
    await nextTick();
    if (mapContainerRef.value) {
        initMap();
    } else {
        const unwatch = watch(mapContainerRef, (newEl) => {
            if (newEl && !mapInstance.value) {
                initMap();
                unwatch();
            }
        });
    }
});

watch(() => props.isDarkMode, (newDarkModeValue) => {
    if (mapInstance.value && window.qq && window.qq.maps) {
        try {
            const newStyleId = newDarkModeValue ? DARK_MAP_STYLE_ID : LIGHT_MAP_STYLE_ID;
            mapInstance.value.setMapStyleId(newStyleId);
        } catch (e) {
            console.error("Error setting map style (qq.maps):", e);
            mapError.value = `Switch map style failed: ${e.message}`;
        }
    }
});

onUnmounted(() => {
    if (window.qq && window.qq.maps && mapInstance.value) {
        mapEventListeners.value.forEach(listener => window.qq.maps.event.removeListener(listener));
        mapEventListeners.value = [];
    }
    mapInstance.value = null;
});
</script>

<style scoped>
.map-container-wrapper {
    background-color: #f0f0f0;
}

.dark .map-container-wrapper {
    background-color: #333333;
}
</style>
