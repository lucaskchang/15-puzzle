<template :class="$colorMode.value">
  <div
    class="min-h-screen transition duration-300 ease-in-out dark:bg-gray-950"
  >
    <slot />
    <Footer />
  </div>
</template>
