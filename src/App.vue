<script setup>
import { ref } from "vue"
import VueMarkdown from 'vue3-markdown-it'

const messages = ref([
  { sender: "bot", text: "Halo! Ada yang bisa saya bantu?" }
])
const userInput = ref("")
const loading = ref(false)

const sendMessage = async () => {
  if (!userInput.value.trim()) return

  messages.value.push({ sender: "user", text: userInput.value })

  loading.value = true
  try {
    const res = await fetch("http://localhost:5678/webhook/chatbot", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ message: userInput.value, sessionId: "user-123" })
    })
    const data = await res.json()

    messages.value.push({ sender: "bot", text: data.output || "Maaf, ada error." })
  } catch (err) {
    messages.value.push({ sender: "bot", text: "Gagal terhubung ke server." })
  } finally {
    userInput.value = ""
    loading.value = false
  }
}
</script>

<template>
  <div class="chat-container">
    <div class="messages">
      <div v-for="(msg, i) in messages" :key="i" :class="['msg', msg.sender]">
        <VueMarkdown :source="msg.text" />
      </div>
    </div>

    <div class="input-area">
      <input
          v-model="userInput"
          type="text"
          placeholder="Ketik pesan..."
          @keyup.enter="sendMessage"
      />
      <button @click="sendMessage" :disabled="loading">Kirim</button>
    </div>
  </div>
</template>

<style scoped>
.chat-container {
  font-family: system-ui;
  max-width: 1000px;
  margin: 0 auto;
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 12px;
  display: flex;
  flex-direction: column;
  height: 94vh;
}

.messages {
  flex: 1;
  overflow-y: auto;
  margin-bottom: 10px;
}

.msg {
  padding: 8px;
  margin: 5px 0;
  border-radius: 6px;
  max-width: 70%;
}

.msg.user {
  align-self: flex-end;
  background: #007bff;
  color: white;
}

.msg.bot {
  align-self: flex-start;
  background: #f1f1f1;
}

.input-area {
  display: flex;
  gap: 5px;
}

input {
  flex: 1;
  padding: 8px;
}

button {
  padding: 8px 12px;
}
</style>
