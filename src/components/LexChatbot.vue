<script setup lang="ts">
import { ref, onMounted, nextTick } from 'vue';
import { Interactions } from '@aws-amplify/interactions';
import { Amplify } from 'aws-amplify';

Amplify.configure({
  ...Amplify.getConfig(),
  Interactions: {
    LexV2: {
      'ESS_CIEA_Test': {
        aliasId: 'TSTALIASID',
        botId: '96FQRORKYN',
        localeId: 'en_US',
        region: 'ca-central-1'
      }
    }
  }
});

const messages = ref<{ from: string; content: string }[]>([]);
const userInput = ref('');
const chatWindow = ref<HTMLElement | null>(null);

// Function to scroll to the bottom of the chat window
const scrollToBottom = () => {
  nextTick(() => {
    if (chatWindow.value) {
      chatWindow.value.scrollTop = chatWindow.value.scrollHeight;
    }
  });
};

const sendMessage = async () => {
  if (!userInput.value.trim()) return;

  // Add user message to chat
  messages.value.push({ from: 'user', content: userInput.value });
  const input = userInput.value;
  userInput.value = '';

  try {
    // Send message to Amazon Lex
    const response = await Interactions.send({ botName: "ESS_CIEA_Test", message: input });

    console.log('Lex response:', response);  // Log the full response for future debugging
    console.log("Response Messages:", response.messages);
    // Add all bot responses to the chat
    if (response.messages && response.messages.length > 0) {
      response.messages.forEach((msg: { content: string }) => {
        messages.value.push({ from: 'bot', content: msg.content });
      });
    } else {
      // Default fallback in case no messages are returned
      messages.value.push({ from: 'bot', content: 'Sorry, I did not understand that.' });
    }
  } catch (error) {
    console.error('Error interacting with Lex bot:', error);
    messages.value.push({ from: 'bot', content: 'An error occurred. Please try again later.' });
  }

  // Scroll to the bottom of the chat window after the message is added
  scrollToBottom();
};

// Ensure chat scrolls to the bottom on page load
onMounted(() => {
  scrollToBottom();
});
</script>



<template>
  <div class="chatbot">
    <h1>ESS Help Desk Chatbot</h1>
    <div class="chat-window" ref="chatWindow">
      <div v-for="(message, index) in messages" :key="index" :class="message.from">
        <p>{{ message.content }}</p>
      </div>
    </div>
    <input
      v-model="userInput"
      @keyup.enter="sendMessage"
      placeholder="Type your message..."
      class="chat-input"
    />
  </div>
</template>

<style scoped>
.chatbot {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
  background: linear-gradient(135deg, #7b42f6, #b76df346);
  border-radius: 12px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.2);
  font-family: 'Arial', sans-serif;
  color: white;
  font-size: 18px; /* Increase the base font size */
  
}

.chat-window {
  background: rgb(255, 255, 255);
  border: 1px solid rgba(255, 255, 255, 0.3);
  padding: 15px;
  height: 400px;
  overflow-y: auto;
  margin-bottom: 15px;
  border-radius: 8px;
  text-align: left; /* Align all chat content to the left */
}

.user {
  text-align: left;
  font-weight: bold;
  color: #000000;
  font-size: 18px; /* Increase font size for user messages */
}

.bot {
  text-align: center;
  color: #2321ad;
  font-size: 20px; /* Increase font size for bot messages */
}

.chat-input {
  width: 100%;
  padding: 12px;
  border-radius: 8px;
  border: none;
  outline: none;
  background: rgba(255, 255, 255, 0.8);
  font-size: 18px;
  font-weight: bold;
  color: #333;
}

.chat-input::placeholder {
  color: #666;
}

button {
  margin-top: 10px;
  background-color: #000;
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 8px;
  font-size: 16px;
  cursor: pointer;
  width: 100%;
  transition: background-color 0.3s ease;
}

button:hover {
  background-color: #444;
}
</style>

