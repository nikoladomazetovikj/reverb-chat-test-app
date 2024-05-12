<script setup>
import { ref, reactive, watchEffect } from 'vue';
import Echo from 'laravel-echo';
import Pusher from 'pusher-js';

const messages = ref([]);
const newMessage = ref('');
const conversationId = import.meta.env.VITE_CHAT_ID;

window.Pusher = Pusher;
const echo = new Echo({
  broadcaster: 'reverb',
  key: import.meta.env.VITE_REVERB_APP_KEY,
  wsHost: import.meta.env.VITE_REVERB_HOST,
  wsPort: import.meta.env.VITE_REVERB_PORT,
  wssPort: import.meta.env.VITE_REVERB_PORT,
  forceTLS: (import.meta.env.VITE_REVERB_SCHEME ?? 'https') === 'https',
  enabledTransports: ['ws', 'wss'],
  authEndpoint: `${import.meta.env.VITE_BASE_URL}/api/broadcasting/auth`,
  auth: {
    headers: {
      Authorization: `Bearer ${import.meta.env.VITE_BEARER}`,
      Accept: 'application/json',
    },
  },
});

const messagesChannel = echo.private(conversationId);

watchEffect(() => {
  const handleNewMessage = (data) => {
    messages.value.push(data.data);
    console.log(messages)
  };

  messagesChannel.listen('SendMessage', (e) => {
    handleNewMessage(e);
  });

  return () => {
    messagesChannel.stopListening('SendMessage', handleNewMessage);
  };
});

const sendMessage = () => {
  if (newMessage.value.trim() !== '') {
    const messageData = {
      sender_id: 1,
      users: {
        user1: {
          name: "Test",
          id: 1,
          avatar: "img",
          role: "Admin"
        },
        user2: {
          name: "Test",
          id: 2,
          avatar: "img",
          role: "Employee"
        },
      },
      message: newMessage.value,
      channel_name: conversationId,
    };

    echo.connector.pusher.send_event('SendMessage', JSON.stringify({ messageData }),  conversationId);
    newMessage.value = '';
  }
};
</script>



<template>
  <div>
    <ul>
      <li v-for="(message, index) in messages" :key="index">
        <strong>{{ message.senderName }} {{ message.senderSurname }}:</strong> {{ message.message }}
      </li>
    </ul>
    <input type="text" v-model="newMessage" />
    <button @click="sendMessage">Send</button>
  </div>
</template>

