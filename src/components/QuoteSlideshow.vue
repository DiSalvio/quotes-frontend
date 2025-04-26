<template>
  <div class="slideshow-container">
    <h1>Inspirational Quotes</h1>

    <!-- Add Quote Form -->
    <form class="add-quote-form" @submit.prevent="addQuote">
      <input
        v-model="newQuote"
        type="text"
        placeholder="Enter a new quote"
        :disabled="adding"
        required
      />
      <input
        v-model="newAuthor"
        type="text"
        placeholder="Enter author (optional)"
        :disabled="adding"
      />
      <button type="submit" :disabled="adding || !newQuote.trim()">
        {{ adding ? 'Adding...' : 'Add Quote' }}
      </button>
    </form>
    <div v-if="addError" class="error">{{ addError }}</div>

    <div v-if="loading" class="loading">
      Loading quotes...
    </div>

    <div v-else-if="quotes.length === 0" class="no-quotes">
      No quotes found. Add some through the form above!
    </div>

    <div v-else class="quote-card">
      <transition name="fade" mode="out-in">
        <div :key="currentIndex">
          <p class="quote-text">"{{ currentQuote.text }}"</p>
          <p class="quote-author">- {{ currentQuote.author || 'Unknown' }}</p>
          <p class="quote-number">Quote {{ currentIndex + 1 }} of {{ quotes.length }}</p>
        </div>
      </transition>

      <div class="controls">
        <button @click="prevQuote" class="nav-button">← Previous</button>
        <button @click="nextQuote" class="nav-button">Next →</button>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  data() {
    return {
      quotes: [],
      currentIndex: 0,
      loading: true,
      error: null,
      newQuote: '',
      newAuthor: '',    // <-- new data property for author
      adding: false,
      addError: null
    }
  },
  computed: {
    currentQuote() {
      return this.quotes[this.currentIndex] || { text: '', author: 'Unknown' }
    }
  },
  async mounted() {
    await this.fetchQuotes()
  },
  methods: {
    async fetchQuotes() {
      this.loading = true
      try {
        const response = await axios.get('/api/quotes/')
        this.quotes = response.data
        if (this.currentIndex >= this.quotes.length) {
          this.currentIndex = 0
        }
      } catch (err) {
        this.error = 'Failed to load quotes. Please try again later.'
        console.error(err)
      } finally {
        this.loading = false
      }
    },
    nextQuote() {
      this.currentIndex = (this.currentIndex + 1) % this.quotes.length
    },
    prevQuote() {
      this.currentIndex = (this.currentIndex - 1 + this.quotes.length) % this.quotes.length
    },
    async addQuote() {
      if (!this.newQuote.trim()) return
      this.adding = true
      this.addError = null
      try {
        const payload = { text: this.newQuote.trim() }
        if (this.newAuthor.trim()) {
          payload.author = this.newAuthor.trim()
        }
        const response = await axios.post('/api/quotes/', payload)
        this.quotes.push(response.data)
        this.currentIndex = this.quotes.length - 1 // show the new quote
        this.newQuote = ''
        this.newAuthor = ''
      } catch (err) {
        this.addError = 'Failed to add quote. Please try again.'
        console.error(err)
      } finally {
        this.adding = false
      }
    }
  }
}
</script>

<style scoped>
.slideshow-container {
  text-align: center;
}

.add-quote-form {
  display: flex;
  justify-content: center;
  gap: 1rem;
  margin-bottom: 2rem;
}

.add-quote-form input {
  flex: 1;
  padding: 0.6rem;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-size: 1rem;
}

.add-quote-form button {
  padding: 0.6rem 1.2rem;
  background-color: #42b983;
  color: white;
  border: none;
  border-radius: 4px;
  font-size: 1rem;
  cursor: pointer;
  transition: background-color 0.2s;
}

.add-quote-form button:disabled {
  background-color: #b2dfdb;
  cursor: not-allowed;
}

.error {
  color: #d32f2f;
  margin-bottom: 1rem;
}

.loading, .no-quotes {
  font-size: 1.2rem;
  color: #666;
  margin: 2rem 0;
}

.quote-card {
  background: #ffffff;
  border-radius: 10px;
  padding: 2rem;
  margin: 2rem 0;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  min-height: 200px;
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.quote-text {
  font-size: 1.5rem;
  color: #2c3e50;
  line-height: 1.6;
  margin: 0 0 1rem 0;
}

.quote-author {
  font-size: 1.1rem;
  color: #888;
  margin: 0.5rem 0 0 0;
  font-style: italic;
}

.quote-number {
  color: #7f8c8d;
  font-size: 0.9rem;
  margin: 0;
}

.controls {
  margin-top: 2rem;
  display: flex;
  justify-content: center;
  gap: 1rem;
}

.nav-button {
  padding: 0.8rem 1.5rem;
  background-color: #42b983;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s ease;
  font-size: 1rem;
}

.nav-button:hover {
  background-color: #33a06f;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
