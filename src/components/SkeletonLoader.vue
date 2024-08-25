<template>
  <div class="py-10">
    <div class="p-5  md:px-10 py-20 bg-[#17191b] rounded-lg">
    <div class="flex  justify-between items-center">
      <div class="w-1/2">
        <div class="skeleton-animation h-10 w-3/5 mb-2"></div>
        <div class="skeleton-animation h-5 w-2/5"></div>
      </div>
      <div class="w-1/2 flex justify-center items-center">
        <div class="skeleton-animation w-24 h-24 rounded-full mr-5"></div>
      </div>
    </div>
    <div class="flex flex-col justify-end items-end mt-5 space-y-2">
      <div class="skeleton-animation h-8 w-[45%]"></div>
      <div class="skeleton-animation h-8 w-[45%]"></div>
    </div>
  </div>
  </div>
</template>

<style scoped>
@keyframes loading {
  0% { background-position: 200% 0; }
  100% { background-position: -200% 0; }
}

.skeleton-animation {
  background: linear-gradient(90deg, #121315 25%, #17191b 50%, #121315 75%);
  background-size: 200% 100%;
  animation: loading 1.5s infinite;
}
</style>