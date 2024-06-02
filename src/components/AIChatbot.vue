<template>
  <div>
    <h1>AI Chatbot</h1>
    <div id="chat-container">
      <div v-for="message in messages" :key="message.id" class="message">
        <strong>{{ message.role }}:</strong> {{ message.content }}
      </div>
      <input v-model="userMessage" @keyup.enter="sendMessage" placeholder="Type a message...">
      <button @click="sendMessage">Send</button>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import axios from 'axios';

const messages = ref([{ id: 1, role: 'system', content: 'You are a helpful assistant.' }]);
const userMessage = ref('');

const openaiApiKey = process.env.VUE_APP_OPENAI_API_KEY;
console.log('OpenAI API key:', openaiApiKey);

async function sendMessage() {

  // Add user's message to the chat
  messages.value.push({ id: Date.now(), role: 'user', content: userMessage.value });

  try {
    const response = await axios.post('https://api.openai.com/v1/chat/completions', {
      model: 'gpt-4',
      messages: messages.value.map(msg => ({ role: msg.role, content: msg.content })),
    }, {
      headers: {
        'Authorization': `Bearer ${openaiApiKey}`,
        'Content-Type': 'application/json'
      }
    });

    const assistantMessage = response.data.choices[0].message.content;

    // Add assistant's message to the chat
    messages.value.push({ id: Date.now(), role: 'assistant', content: assistantMessage });
    userMessage.value = ''; // Clear the input field

  } catch (error) {
    console.error('Error communicating with OpenAI API:', error);
  }
}
</script>

<style>
#chat-container {
  max-width: 600px;
  margin: auto;
  padding: 1em;
  border: 1px solid #ccc;
  border-radius: 8px;
}
.message {
  margin: 0.5em 0;
}
input {
  width: calc(100% - 50px);
  padding: 0.5em;
}
button {
  padding: 0.5em;
}
</style>
