<template>
  <div class="app-container">
    <QuoteSlideshow />
  </div>
</template>

<script>
import QuoteSlideshow from './components/QuoteSlideshow.vue'

export default {
  components: {
    QuoteSlideshow
  }
}
</script>

<style>
.app-container {
  max-width: 800px;
  margin: 0 auto;
  padding: 2rem;
  font-family: Arial, sans-serif;
}
</style>

