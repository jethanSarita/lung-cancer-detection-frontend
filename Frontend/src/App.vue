<template>
  <div class="flex h-screen">
    <div class="w-[30%] bg-rose-200">
      <PoweredByJethan />
    </div>
    <div class="h-screen w-[70%] bg-red-100 overflow-auto">
      <router-view v-slot="{ Component }">
        <transition name="fade">
          <component :is="Component" />
        </transition>
      </router-view>
    </div>
  </div>
</template>

<style scoped>
.fade-leave-to,
.fade-enter-from{
  opacity: 0;
}

.fade-leave-from,
.fade-enter-to {
  opacity: 1;
}

.fade-leave-active,
.fade-enter-active {
  transition: opacity 0.5s;
}
</style>