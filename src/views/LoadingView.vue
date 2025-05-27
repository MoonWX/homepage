<template>
    <div class="loading-view-wrapper fixed inset-0 z-50">
        <LoadingSpinner :isLoading="isLoading" @animation-complete="onSpinnerAnimationComplete" />
    </div>
</template>

<script setup>
import LoadingSpinner from '../components/LoadingSpinner.vue';

defineProps({
    isLoading: {
        type: Boolean,
        default: true
    }
});

const emit = defineEmits(['loadingSequenceFinished']);

const onSpinnerAnimationComplete = () => {
    emit('loadingSequenceFinished');
};
</script>

<style scoped></style>
