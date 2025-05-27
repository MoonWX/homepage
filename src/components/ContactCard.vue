<template>
    <CardWrapper title="Contact" icon="fas fa-link" iconColor="text-indigo-400" :isDarkMode="isDarkMode">
        <div class="space-y-3">
            <a v-for="contact in contacts" :key="contact.name" :href="contact.url" target="_blank"
                rel="noopener noreferrer" class="flex items-center group"
                :class="isDarkMode ? 'text-gray-200 hover:text-white' : 'text-gray-700 hover:text-gray-900'">
                <i :class="[contact.icon, contact.color]" class="fa-lg mr-3 w-5 text-center" :title="contact.name"></i>
                <span>{{ contact.handle }}</span>
            </a>
        </div>
    </CardWrapper>
</template>

<script setup>
import CardWrapper from './CardWrapper.vue';
defineProps({
    contacts: Array,
    isDarkMode: Boolean
});
</script>
