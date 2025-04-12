<script setup>
import { ref } from 'vue';
import { computed } from 'vue';
import { useRoute } from 'vue-router';

const selected = ref(false);
const route = useRoute();
const path = computed(() => {
    return route.path;
});

</script>

<template>
    <nav class="h-screen flex justify-center items-center">
        <ul class="items-start space-y-1 text-surface-700">
            <li>
                <img src="../../public/favicon.ico" alt="LOGO" class="w-5 h-auto" />
            </li>
            <li>    
                <RouterLink :class="['text-xl', path==='/' && 'text-surface-500']" to="/">LungVision</RouterLink>
            </li>   
            <li>
                <RouterLink :class="['text-xl', path==='/demo/' && 'text-surface-500']" to="/demo/">Demo</RouterLink>
            </li>
            <li>
                <RouterLink :class="['text-xl', path==='/research/' && 'text-surface-500']" to="/research/">Research</RouterLink>
            </li>
            <li>
                <RouterLink :class="['text-xl', path==='/team/' && 'text-surface-500']" to="/team/">Team</RouterLink>
            </li>
        </ul>
    </nav>
</template>

<style lang="scss" scoped>

</style>