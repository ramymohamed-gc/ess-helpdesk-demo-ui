<script setup lang="ts">
import LexChatbot from './components/LexChatbot.vue'

import { Authenticator } from "@aws-amplify/ui-vue";
import "@aws-amplify/ui-vue/styles.css";
</script>

<template>
  <main>
    <authenticator>
      <template v-slot="{ signOut }">
        <LexChatbot />
        <button @click="signOut">Sign Out</button>
      </template>
    </authenticator>
  </main>
</template>

