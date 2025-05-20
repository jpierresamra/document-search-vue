<script setup>
import { ref } from 'vue'

const searchTerm = ref('')
const results = ref([])

function highlight(text, filter) {
  console.log('Highlighting text:', text, 'with filter:', filter);
  if (!filter) return text
  // Escape special regex characters in filter
  const safeFilter = filter.replace(/[.*+?^${}()|[\]\\]/g, '\\$&')
  const regex = new RegExp(`(${safeFilter})`, 'gi')
  // Ensure text is a string
  const str = text == null ? '' : String(text)
  return str.replace(regex, '<span class="highlight">$1</span>')
}
const loading = ref(false)
const error = ref('')

async function search() {
  if (!searchTerm.value) return
  loading.value = true
  error.value = ''
  results.value = []
  try {
    // Replace the URL below with your actual Spring Boot API endpoint
    const response = await fetch(`http://localhost:9089/api/v1/documents?filter=${encodeURIComponent(searchTerm.value)}`)
    if (!response.ok) throw new Error('API error')
    results.value = await response.json();
    console.log('Search results:', results.value);
  } catch (e) {
    error.value = 'Failed to fetch results.'
  } finally {
    loading.value = false
  }
}
</script>

<template>
  <main style="max-width: 500px; margin: 2rem auto; padding: 2rem; border: 1px solid #eee; border-radius: 8px;">
    <h1>Document Search</h1>
    <form @submit.prevent="search">
      <input
        v-model="searchTerm"
        type="text"
        placeholder="Enter search term..."
        style="width: 70%; padding: 0.5rem;"
      />
      <button type="submit" style="padding: 0.5rem 1rem; margin-left: 0.5rem;">Search</button>
    </form>
    <div v-if="loading" style="margin-top: 1rem;">Searching...</div>
    <div v-if="error" style="color: red; margin-top: 1rem;">{{ error }}</div>
    <ul v-if="results.length" style="margin-top: 1rem;">
      <li v-for="(item, idx) in results" :key="item.id" style="margin-bottom: 1.5rem; padding-bottom: 1rem; border-bottom: 1px solid #eee;">
        <h3 v-html="highlight(item.title, searchTerm)"></h3>
        <p v-html="highlight(item.body, searchTerm)"></p>
        <div style="font-size: 0.9em; color: #666;">
          <span>By {{ item.author }}</span> |
          <span>{{ new Date(item.documentDate).toLocaleString() }}</span>
        </div>
      </li>
    </ul>
    <div v-else-if="!loading && !error" style="margin-top: 1rem; color: #888;">No results yet.</div>
  </main>
</template>

<style>

.highlight {
    background-color: yellow !important;
    color: black !important;
}

</style>
