<template>
    <CardWrapper title="Links" icon="fas fa-globe-asia" iconColor="text-cyan-500 dark:text-cyan-400"
        :isDarkMode="isDarkMode">
        <div class="grid gap-4" style="grid-template-columns: repeat(auto-fit, minmax(min(100%, 180px), 1fr));">
            <a v-for="site in websites" :key="site.name" :href="site.url" target="_blank" rel="noopener noreferrer"
                class="flex p-3 rounded-lg transition-all duration-300 ease-in-out group" :class="isDarkMode
                    ? 'bg-gray-700/40 hover:bg-gray-600/60'
                    : 'bg-gray-100/60 hover:bg-gray-200/80'">
                <div class="flex items-center h-full w-full">
                    <i :class="[site.icon, site.iconColor ? site.iconColor : (isDarkMode ? 'text-sky-300' : 'text-sky-600')]"
                        class="fa-lg mr-3 w-6 h-6 text-center shrink-0 transition-colors duration-300
                    flex items-center justify-center leading-6">
                    </i>
                    <div class="flex-grow min-w-0 overflow-hidden">
                        <h3 class="font-semibold text-base group-hover:text-sky-500 dark:group-hover:text-sky-400"
                            :class="isDarkMode ? 'text-gray-50' : 'text-gray-800'" :title="site.name"
                            style="word-break: break-word; white-space: normal;">
                            {{ site.name }}
                        </h3>
                        <p v-if="site.description" class="text-sm mt-0.5"
                            :class="isDarkMode ? 'text-gray-400' : 'text-gray-500'" :title="site.description"
                            style="word-break: break-word; white-space: normal;">
                            {{ site.description }}
                        </p>
                    </div>
                    <i class="fas fa-external-link-alt text-xs opacity-50 group-hover:opacity-100 transition-opacity duration-300 ml-2 shrink-0 self-center"
                        :class="isDarkMode ? 'text-gray-400' : 'text-gray-500'">
                    </i>
                </div>
            </a>
        </div>
    </CardWrapper>
</template>

<script setup>
import CardWrapper from './CardWrapper.vue';

defineProps({
    websites: Array,
    isDarkMode: Boolean
});
</script>

<style scoped>
p {
    display: -webkit-box;
    -webkit-line-clamp: 2;
    line-clamp: 2;
    -webkit-box-orient: vertical;
    overflow: hidden;
    text-overflow: ellipsis;
    word-break: break-word;
}

h3 {
    display: -webkit-box;
    -webkit-line-clamp: 1;
    line-clamp: 1;
    -webkit-box-orient: vertical;
    overflow: hidden;
    text-overflow: ellipsis;
    word-break: break-word;
}

i.fa-lg {
    font-size: 1.33333333em;
    line-height: 0.75em;
    vertical-align: -0.0667em;
}
</style>
