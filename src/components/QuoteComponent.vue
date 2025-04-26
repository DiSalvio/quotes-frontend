<template>
  <div class="quote-container">
    <div class="quote-card">
      <p class="quote-text">"{{ currentQuote.text }}"</p>
    </div>
    
    <div class="navigation-buttons">
      <button @click="$emit('prev-quote')" class="nav-button">← Previous</button>
      <button @click="$emit('next-quote')" class="nav-button">Next →</button>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    currentQuote: {
      type: Object,
      required: true
    }
  }
}
</script>

<style scoped>
.quote-container {
  margin: 2rem 0;
}

.quote-card {
  background: #f8f9fa;
  border-radius: 8px;
  padding: 2rem;
  margin: 1rem 0;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
}

.quote-text {
  font-size: 1.2rem;
  color: #2c3e50;
  line-height: 1.6;
}

.navigation-buttons {
  display: flex;
  gap: 1rem;
  justify-content: center;
  margin-top: 1.5rem;
}

.nav-button {
  padding: 0.8rem 1.5rem;
  background-color: #42b983;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.2s;
}

.nav-button:hover {
  background-color: #33a06f;
}
</style>

