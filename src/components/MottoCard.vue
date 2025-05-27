<template>
    <CardWrapper title="Hitokoto" icon="fas fa-quote-left" iconColor="text-teal-400" :isDarkMode="isDarkMode"
        class="md:col-span-2 lg:col-span-1">
        <p class="italic">
            "{{ mottoContent.text }}"
        </p>
        <p class="text-right text-sm mt-2" :class="isDarkMode ? 'text-gray-400' : 'text-gray-600'">
            - {{ mottoContent.author }}
        </p>
    </CardWrapper>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import CardWrapper from './CardWrapper.vue';

defineProps({
    isDarkMode: Boolean
});

const mottoContent = ref({
    text: 'Loading...',
    author: ''
});

const fetchLocalMotto = async () => {
    try {
        const response = await fetch("https://v1.hitokoto.cn/");
        if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
        }
        const data = await response.json();
        mottoContent.value = {
            text: data.hitokoto,
            author: data.from_who || data.from || '',
        };
    } catch (error) {
        console.error("Failed to fetch motto:", error);
        mottoContent.value = { text: 'Not today.', author: 'Network' };
    }
};

onMounted(() => {
    fetchLocalMotto();
});
</script>
