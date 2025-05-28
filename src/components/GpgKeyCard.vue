<template>
    <CardWrapper title="GPG Public Keys" icon="fas fa-key" iconColor="text-green-600 dark:text-green-400"
        :isDarkMode="isDarkMode">
        <div class="space-y-3">
            <div v-if="gpgKeyData.id" class="text-sm">
                <span class="font-medium" :class="isDarkMode ? 'text-gray-300' : 'text-gray-600'">Key ID: </span>
                <span :class="isDarkMode ? 'text-gray-100' : 'text-gray-700'">{{ gpgKeyData.id }}</span>
            </div>
            <div v-if="gpgKeyData.fingerprint" class="text-sm break-all">
                <span class="font-medium" :class="isDarkMode ? 'text-gray-300' : 'text-gray-600'">Fingerprint: </span>
                <span :class="isDarkMode ? 'text-gray-100' : 'text-gray-700'">{{ gpgKeyData.fingerprint }}</span>
            </div>

            <div class="mt-2">
                <pre class="p-3 rounded-md text-xs leading-relaxed whitespace-pre-wrap break-all overflow-auto max-h-48"
                    :class="isDarkMode ? 'bg-gray-700/50 text-gray-200' : 'bg-gray-100/70 text-gray-700'">{{ gpgKeyData.publicKey }}</pre>
            </div>

            <button @click="copyKeyToClipboard"
                class="mt-3 w-full px-4 py-2 text-sm rounded-lg transition-colors duration-300 focus:outline-none focus:ring-2 flex items-center justify-center"
                :class="isDarkMode
                    ? 'bg-green-600/80 hover:bg-green-500/80 text-white focus:ring-green-400'
                    : 'bg-green-600 hover:bg-green-500 text-white focus:ring-green-300'">
                <i :class="copySuccess ? 'fas fa-check' : 'far fa-copy'" class="mr-2"></i>
                {{ copyButtonText }}
            </button>
            <p v-if="copyError" class="text-xs text-red-500 dark:text-red-400 mt-1 text-center">{{ copyError }}</p>
        </div>
    </CardWrapper>
</template>

<script setup>
import { ref, computed } from 'vue';
import CardWrapper from './CardWrapper.vue';

const props = defineProps({
    gpgKeyData: {
        type: Object,
        required: true,
        default: () => ({ id: '-', fingerprint: '-', publicKey: 'No GPG key provided.' })
    },
    isDarkMode: Boolean
});

const copySuccess = ref(false);
const copyError = ref('');

const copyButtonText = computed(() => {
    if (copySuccess.value) return 'Copied!';
    return 'Copy Public Key';
});

const copyKeyToClipboard = () => {
    copySuccess.value = false;
    copyError.value = '';

    if (!props.gpgKeyData.publicKey) {
        copyError.value = 'No key available for copying.';
        return;
    }

    const textarea = document.createElement('textarea');
    textarea.value = props.gpgKeyData.publicKey;
    textarea.style.position = 'fixed'; // Prevent scrolling to bottom of page in MS Edge.
    textarea.style.left = '-9999px'; // Move out of screen
    document.body.appendChild(textarea);
    textarea.select();
    try {
        const successful = document.execCommand('copy');
        if (successful) {
            copySuccess.value = true;
            setTimeout(() => { copySuccess.value = false; }, 2000); // Reset after 2 seconds
        } else {
            copyError.value = 'Copy failed. Please try manually.';
            console.error('Fallback: Oops, unable to copy');
        }
    } catch (err) {
        copyError.value = 'Copy failed. Please try manually.';
        console.error('Fallback: Oops, unable to copy', err);
    }
    document.body.removeChild(textarea);
};
</script>

<style scoped>
pre {
    scrollbar-width: thin;
    /* For Firefox */
    scrollbar-color: rgba(156, 163, 175, 0.5) rgba(0, 0, 0, 0.1);
    /* For Firefox scrollbar color */
}

pre::-webkit-scrollbar {
    width: 8px;
    height: 8px;
}

pre::-webkit-scrollbar-track {
    background: rgba(0, 0, 0, 0.1);
    border-radius: 4px;
}

pre::-webkit-scrollbar-thumb {
    background-color: rgba(156, 163, 175, 0.5);
    /* gray-400 with opacity */
    border-radius: 4px;
    border: 2px solid transparent;
    background-clip: content-box;
}

pre::-webkit-scrollbar-thumb:hover {
    background-color: rgba(156, 163, 175, 0.7);
}
</style>
