<template>
    <div class="w-full max-w-4xl mx-auto relative py-8 px-4 sm:px-6 lg:px-8 content-wrapper"
        :class="contentVisible ? 'opacity-100' : 'opacity-0'"
        :style="{ transition: 'opacity 0.5s ease-out', transitionDelay: contentTransitionDelay }">

        <ThemeSwitcher :current-preference="currentThemePreference" @set-preference="onSetThemePreference" />

        <ProfileHeader :profile="profileData.profile" :avatarUrl="profileData.avatarUrl" :isDarkMode="isEffectivelyDark"
            :contentVisible="contentVisible" />

        <main class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6 mt-8 cards-grid-container">
            <HobbiesCard :hobbies="profileData.hobbies" :isDarkMode="isEffectivelyDark"
                class="md:col-span-1 lg:col-span-1" />
            <ContactCard :contacts="profileData.contacts" :isDarkMode="isEffectivelyDark"
                class="md:col-span-1 lg:col-span-1" />
            <MottoCard :isDarkMode="isEffectivelyDark" class="md:col-span-2 lg:col-span-1" />
            <WeatherCard :locationForFetch="staticLocationData.city" :isDarkMode="isEffectivelyDark"
                class="md:col-span-1 lg:col-span-1" />
            <WebsitesCard :websites="profileData.websites" :isDarkMode="isEffectivelyDark"
                class="md:col-span-1 lg:col-span-2" />
            <TimeComparisonCard :locationInfo="staticLocationData" :isDarkMode="isEffectivelyDark"
                class="md:col-span-1 lg:col-span-1" />
            <GpgKeyCard v-if="profileData.gpgKey && profileData.gpgKey.publicKey" :gpgKeyData="profileData.gpgKey"
                :isDarkMode="isEffectivelyDark" class="md:col-span-1 lg:col-span-2" />
            <LocationCard :locationInfo="staticLocationData" :isDarkMode="isEffectivelyDark"
                class="md:col-span-2 lg:col-span-3" />

        </main>

        <footer class="text-center mt-12 pb-4">
            <p class="text-sm" :class="isEffectivelyDark ? 'text-gray-300' : 'text-gray-900'">
                &copy; 2022 - {{ currentYear }} {{ profileData.profile.name }}. <a
                    href="https://github.com/MoonWX/homepage" class="underline">Source</a>. <br />
                Images sourced online. Background Images Artists: JLT4n on Pixiv (103933948) & evrmore on Pixiv
                (91202097). Please contact us for removal if any infringement occurs. <br />
                "Hitokoto" (one-sentence service) is provided by <a href="https://hitokoto.cn/">Hitokoto</a> API. The
                content displayed does not represent the
                opinions of this website or its owner. <br />
                <a href="https://beian.miit.gov.cn/#/Integrated/index">蒙ICP备2023000487号-2</a>
            </p>
        </footer>
    </div>
</template>

<script setup>
import { ref, computed } from 'vue';
import profileDataJson from '../data/profileData.json';

import ThemeSwitcher from '../components/ThemeSwitcher.vue';
import ProfileHeader from '../components/ProfileHeader.vue';
import HobbiesCard from '../components/HobbiesCard.vue';
import ContactCard from '../components/ContactCard.vue';
import WebsitesCard from '../components/WebsitesCard.vue';
import MottoCard from '../components/MottoCard.vue';
import LocationCard from '../components/LocationCard.vue';
import WeatherCard from '../components/WeatherCard.vue';
import TimeComparisonCard from '../components/TimeComparisonCard.vue';
import GpgKeyCard from '../components/GpgKeyCard.vue';

// eslint-disable-next-line no-unused-vars
const props = defineProps({
    isEffectivelyDark: Boolean,
    contentVisible: Boolean,
    contentTransitionDelay: String,
    currentThemePreference: String,
});

const emit = defineEmits(['update-theme-preference']);

const profileData = ref(profileDataJson);

const staticLocationData = ref({
    city: profileDataJson.initialLocationInfo?.city || 'Changchun',
    country: profileDataJson.initialLocationInfo?.country || 'China',
    latitude: profileDataJson.initialLocationInfo?.latitude || 43.817003,
    longitude: profileDataJson.initialLocationInfo?.longitude || 125.323629,
});

const onSetThemePreference = (preference) => {
    emit('update-theme-preference', preference);
};

const currentYear = computed(() => new Date().getFullYear());
</script>

<style scoped></style>
