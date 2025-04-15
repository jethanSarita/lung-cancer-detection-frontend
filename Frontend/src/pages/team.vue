<script setup>
import TeamImage from '@/components/TeamImage.vue';


</script>

<template>
    <div class="flex flex-row justify-center">
        <div class="w-[90%] flex flex-col justify-center">
            <!-- 2:3 -->
            <div class="pt-10 pb-10">
                <p class="text-3xl text-surface-700 text-center pb-5">
                    Meet
                </p>
                <p class="text-5xl font-bold text-surface-700 text-center">
                    The Team
                </p>
            </div>
            <!-- Images -->
            <div class="flex flex-row justify-center gap-5 pb-10">
                <TeamImage image="Jethan.jpg" name="Jethañ Sarita" subtext="Software Engineer"/>
                <TeamImage image="Franco.jpg" name="Franco Pleños" subtext="Lead Researcher"/>
                <TeamImage image="Jeno.jpg" name="Jeño Flores" subtext="AI Engineer"/>
            </div>
        </div>
    </div>
</template>

<style scoped>
</style>