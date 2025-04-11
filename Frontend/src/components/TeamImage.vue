<script setup>

const props = defineProps({
    image: {
        type: String,
        required: true
    },
    name: {
        type: String,
        required: true
    },
    subtext: {
        type: String,
        required: false
    },
    bio: {
        type: String,
        required: false
    }
});

const get_image_url = (image) => {
    return new URL(`../assets/team/${image}`, import.meta.url).href;
};

</script>

<template>
    <div class="flex flex-col">
        <img :src="get_image_url(props.image)" :alt="props.name" class="w-96 h-auto object-cover object-top aspect-square rounded-md" />
        <p class="text-lg font-bold text-surface-700">{{ props.name }}</p>
        <p class="text-lg text-surface-700">{{ props.subtext }}</p>
    </div>
</template>

<style lang="scss" scoped>

</style>