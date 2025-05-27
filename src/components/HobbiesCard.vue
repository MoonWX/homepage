<template>
    <CardWrapper title="Hobbies" icon="fas fa-heart" iconColor="text-red-400" :isDarkMode="isDarkMode">
        <ul class="space-y-2">
            <li v-for="hobby in hobbies" :key="hobby.name" class="flex items-center">
                <i :class="[hobby.icon, hobby.color]" class="mr-2 w-5 text-center"></i>{{ hobby.name }}
            </li>
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
