<template>
    <CardWrapper title="My Geographical Location" icon="fas fa-map-marker-alt" iconColor="text-orange-400"
        :isDarkMode="isDarkMode" class="lg:col-span-2">
        <p class="mb-3">
            I'm living in {{ locationInfo.city }}, {{ locationInfo.country }} now.
        </p>
        <div class="aspect-video rounded-lg overflow-hidden border"
            :class="isDarkMode ? 'border-gray-700/50' : 'border-gray-300/50'">
            <iframe sandbox="allow-scripts allow-same-origin"
                src="https://apis.map.qq.com/tools/poimarker?type=0&marker=coord:43.817003,125.323629;title:%E9%95%BF%E6%98%A5;addr:%E9%95%BF%E6%98%A5%E5%B8%82%E4%BA%BA%E6%B0%91%E6%94%BF%E5%BA%9C&key=H6DBZ-4KYYZ-MPEXE-TZXOE-AAEEQ-ZHFPG&referer=homepage"
                width="100%" height="450" style="border:0;" allowfullscreen="" loading="lazy" title="Tencent Map"
                referrerpolicy="no-referrer-when-downgrade"></iframe>
        </div>
    </CardWrapper>
</template>

<script setup>
import CardWrapper from './CardWrapper.vue';
defineProps({
    locationInfo: Object,
    isDarkMode: Boolean
});
</script>
