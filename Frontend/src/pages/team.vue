<script setup>
import TeamImage from '@/components/TeamImage.vue';


</script>

<template>
    <div class="h-screen flex justify-center">
        <div class="w-[75%] flex flex-col justify-center">
            <!-- 2:3 -->
            <div>
                <p class="text-5xl font-bold text-surface-700 pb-5 text-center">
                    The Team
                </p>
            </div>
            <!-- Images -->
            <div class="flex flex-row justify-center gap-5">
                <TeamImage image="Jethan.jpg" name="Jethañ Sarita"/>
                <TeamImage image="Franco.jpg" name="Franco Pleños"/>
                <TeamImage image="Jeno.jpg" name="Jeño Flores"/>
            </div>
        </div>
    </div>
</template>

<style scoped>
</style>