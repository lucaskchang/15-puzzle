<template :class="$colorMode.value">
  <div
    class="min-h-screen text-gray-950 transition duration-300 ease-in-out dark:bg-gray-900 dark:text-white"
  >
    <slot />
    <Footer />
  </div>
</template>
