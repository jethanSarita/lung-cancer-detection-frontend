<script setup>

const props = defineProps({
    image: {
        type: String,
        required: true
    },
    name: {
        type: String,
        required: true
    }
});

const get_image_url = (image) => {
    return new URL(`../assets/team/${image}`, import.meta.url).href;
};

</script>

<template>
    <div class="flex flex-col gap-5">
        <img :src="get_image_url(props.image)" :alt="props.name" class="w-96 h-auto object-cover object-top aspect-square rounded-md" />
        <p class="text-center text-lg font-bold text-surface-700">{{ props.name }}</p>
    </div>
</template>

<style lang="scss" scoped>

</style>