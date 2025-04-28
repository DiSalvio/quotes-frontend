<template>
  <div class="slideshow-container">
    <h1>Inspirational Quotes</h1>

    <!-- Image Upload Component -->
    <ImageUpload @text-extracted="handleExtractedText" />

    <!-- Extracted Text Review -->
    <div v-if="extractedText !== null" class="extracted-text">
      <textarea
        v-model="extractedText"
        placeholder="Edit extracted quote text..."
      ></textarea>
      <input
        v-model="extractedAuthor"
        type="text"
        placeholder="Edit extracted author (optional)"
        style="margin-top: 0.5rem; width: 100%;"
      />
      <button @click="saveExtractedText" class="save-button">
        {{ editing ? 'Saving...' : 'Save as Quote' }}
      </button>
    </div>

    <!-- Existing Add Quote Form -->
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

    <!-- Quote Display -->
    <div v-if="loading" class="loading">
      Loading quotes...
    </div>

    <div v-else-if="quotes.length === 0" class="no-quotes">
      No quotes found. Add some through the form or upload an image!
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
        <button @click="deleteQuote" class="delete-button" :disabled="deleting">
          {{ deleting ? 'Deleting...' : 'Delete' }}
        </button>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import ImageUpload from './ImageUpload.vue';

export default {
  components: {
    ImageUpload
  },
  data() {
    return {
      quotes: [],
      currentIndex: 0,
      loading: true,
      error: null,
      newQuote: '',
      newAuthor: '',
      adding: false,
      addError: null,
      deleting: false,
      deleteError: null,
      extractedText: null,
      extractedAuthor: '',
      editing: false
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
        this.currentIndex = this.quotes.length - 1
        this.newQuote = ''
        this.newAuthor = ''
      } catch (err) {
        this.addError = 'Failed to add quote. Please try again.'
        console.error(err)
      } finally {
        this.adding = false
      }
    },
    async deleteQuote() {
      if (!this.quotes.length) return
      const quoteId = this.currentQuote.id
      if (!confirm('Are you sure you want to delete this quote?')) return
      this.deleting = true
      this.deleteError = null
      try {
        await axios.delete(`/api/quotes/${quoteId}`)
        this.quotes.splice(this.currentIndex, 1)
        if (this.currentIndex >= this.quotes.length) {
          this.currentIndex = Math.max(0, this.quotes.length - 1)
        }
      } catch (err) {
        this.deleteError = 'Failed to delete quote. Please try again.'
        console.error(err)
      } finally {
        this.deleting = false
      }
    },
    handleExtractedText({extracted_text, extracted_author}) {
      this.extractedText = extracted_text
      this.extractedAuthor = extracted_author || ''
    },
    async saveExtractedText() {
      if (!this.extractedText?.trim()) return
      this.editing = true
      try {
        const payload = { text: this.extractedText.trim() }
        if (this.extractedAuthor.trim()) {
          payload.author = this.extractedAuthor.trim()
        }
        const response = await axios.post('/api/quotes/', payload)
        this.quotes.push(response.data)
        this.currentIndex = this.quotes.length - 1
        this.extractedText = null
        this.extractedAuthor = ''
      } catch (err) {
        console.error('Error saving extracted text:', err)
      } finally {
        this.editing = false
      }
    }
  }
}
</script>

<style scoped>
.slideshow-container {
  text-align: center;
  max-width: 800px;
  margin: 0 auto;
  padding: 2rem;
}

/* Image Upload Section */
.image-upload {
  margin: 2rem 0;
}

/* Extracted Text Review */
.extracted-text {
  background: #f8f9fa;
  border-radius: 8px;
  padding: 1.5rem;
  margin: 2rem 0;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.extracted-text textarea {
  width: 100%;
  height: 100px;
  padding: 1rem;
  margin: 0.5rem 0;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-family: inherit;
  font-size: 1rem;
}

.extracted-text input[type="text"] {
  padding: 0.8rem;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 1rem;
  margin-bottom: 0.5rem;
  width: 100%;
}

.save-button {
  padding: 0.8rem 1.5rem;
  background-color: #2196F3;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.2s;
}

.save-button:hover {
  background-color: #1976D2;
}

.save-button:disabled {
  background-color: #90CAF9;
  cursor: not-allowed;
}

/* Add Quote Form */
.add-quote-form {
  display: grid;
  grid-template-columns: 1fr 1fr auto;
  gap: 1rem;
  margin: 2rem 0;
}

.add-quote-form input {
  padding: 0.8rem;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 1rem;
}

.add-quote-form button {
  padding: 0.8rem 1.5rem;
  background-color: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.2s;
}

.add-quote-form button:disabled {
  background-color: #A5D6A7;
  cursor: not-allowed;
}

/* Quote Display */
.quote-card {
  background: white;
  border-radius: 12px;
  padding: 2rem;
  margin: 2rem 0;
  box-shadow: 0 4px 6px rgba(0,0,0,0.1);
}

.quote-text {
  font-size: 1.5rem;
  color: #2c3e50;
  line-height: 1.6;
  margin: 0 0 1rem 0;
}

.quote-author {
  font-size: 1.1rem;
  color: #7f8c8d;
  font-style: italic;
  margin: 0.5rem 0;
}

.quote-number {
  color: #95a5a6;
  font-size: 0.9rem;
  margin: 1rem 0 0 0;
}

/* Navigation Controls */
.controls {
  display: flex;
  justify-content: center;
  gap: 1rem;
  margin-top: 2rem;
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

.delete-button {
  background-color: #e74c3c;
}

.delete-button:hover {
  background-color: #c0392b;
}

/* Transitions */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

/* Utility Classes */
.error {
  color: #e74c3c;
  margin: 1rem 0;
  padding: 1rem;
  background: #fdeded;
  border-radius: 4px;
}

.loading {
  color: #7f8c8d;
  font-size: 1.2rem;
  margin: 2rem 0;
}

.no-quotes {
  color: #95a5a6;
  font-style: italic;
  margin: 2rem 0;
}
</style>
