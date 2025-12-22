<template>
  <div class="d-flex justify-content-center align-items-center vh-100">
    <div class="card p-4 shadow" style="max-width:400px; width:100%">
      <h3 class="mb-3">Enter PIN</h3>
      <div v-if="error" class="text-danger mb-2">{{ error }}</div>
      <div class="mb-3">
        <label class="form-label">PIN Code</label>
        <input v-model="pin" type="password" class="form-control" @input="error = ''" @keyup.enter="submit" autofocus />
      </div>
      <div class="d-flex justify-content-between align-items-center">
        <small class="text-muted">Enter 4‑digit PIN</small>
        <button class="btn btn-primary" @click="submit">Enter</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'
const router = useRouter()
const pin = ref('')
const error = ref('')
const CORRECT = '3131'

function submit() {
  if (pin.value === CORRECT) {
    sessionStorage.setItem('marryme-authenticated', '1')
    router.push('/dashboard')
  } else {
    error.value = 'wrong pin code'
    pin.value = ''
  }
}
</script>
