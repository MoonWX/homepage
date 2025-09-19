<template>
    <CardWrapper title="Hobbies" icon="fas fa-heart" iconColor="text-red-400" :isDarkMode="isDarkMode">
        <ul class="space-y-3">
            <a v-for="hobby in hobbies" :key="hobby.name" :href="hobby.url" target="_blank"
                rel="noopener noreferrer" class="flex items-center group"
                :class="isDarkMode ? 'text-gray-200 hover:text-white' : 'text-gray-700 hover:text-gray-900'">
                <i :class="[hobby.icon, hobby.color]" class="fa-lg mr-3 w-5 text-center" :title="hobby.name"></i>
                <span>{{ hobby.name }}</span>
            </a>
        </ul>
    </CardWrapper>
</template>

<script setup>
import CardWrapper from './CardWrapper.vue';
defineProps({
    hobbies: Array,
    isDarkMode: Boolean
});
</script>
