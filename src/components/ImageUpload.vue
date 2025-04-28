<template>
  <div class="image-upload">
    <input
      type="file"
      ref="fileInput"
      accept="image/*"
      @change="handleFileSelect"
      hidden
    />
    <button @click="triggerFileSelect" :disabled="uploading">
      {{ uploading ? 'Processing...' : 'Upload Screenshot' }}
    </button>
    <div v-if="error" class="error">{{ error }}</div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      uploading: false,
      error: null
    }
  },
  methods: {
    triggerFileSelect() {
      this.$refs.fileInput.click();
    },
    async handleFileSelect(event) {
      const file = event.target.files[0];
      if (!file) return;
      this.uploading = true;
      this.error = null;
      try {
        const formData = new FormData();
        formData.append('image', file);
        const response = await axios.post('/api/quotes/from-image/', formData, {
          headers: { 'Content-Type': 'multipart/form-data' }
        });
        this.$emit('text-extracted', {
          extracted_text: response.data.extracted_text,
          extracted_author: response.data.extracted_author
        });
      } catch (err) {
        this.error = err.response?.data?.detail || 'Failed to process image';
      } finally {
        this.uploading = false;
        event.target.value = '';
      }
    }
  }
}
</script>

<style scoped>
.image-upload {
  margin: 1rem 0;
}
button {
  padding: 0.8rem 1.5rem;
  background-color: #42b983;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.2s;
}
button:disabled {
  background-color: #b2dfdb;
  cursor: not-allowed;
}
.error {
  color: #d32f2f;
  margin-top: 0.5rem;
}
</style>
