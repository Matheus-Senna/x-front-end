<script setup>
import XButton from './components/XButton.vue';
import XTextField from './components/XTextField.vue';
import XPost from './components/XPost.vue';
</script>

<template>
  <main
    class="bg-gray-900 min-h-screen flex items-center justify-center flex-col p-20"
  >
    <div
      class="rounded-md border border-gray-700 text-white bg-gray-800 p-6 mx-auto w-full max-w-[600px]"
    >
      <h1 class="text-2xl mb-4">𝕏 (Twitter) Descentralizado</h1>
      <p class="text-base mb-4">
        Esse é um twitter descentralizado, conecte sua sua carteira blockchain e
        use seus Ethereums para enviar uma mensagem. Cada post enviado você terá
        chance de ganhar um valor de Ethereum de volta.
      </p>
      <XButton text="Conectar carteira" />
      <XTextField label="Post" name="post" class="mb-2" type="text" id="post" placeholder="John" required />
      <XButton text="Enviar post" />
      <div class="flex items-center">
        <span
          class="bg-green-100 text-green-800 text-xs font-semibold mr-2 px-2.5 py-0.5 rounded dark:bg-green-200 dark:text-green-900 h-fit"
        >
          Conectado!
        </span>
        <span class="truncate">
          <!-- {{ currentAccount }} -->
        </span>
      </div>
    </div>
    <div
      class="mt-8 rounded-md border border-gray-700 text-white bg-gray-800 p-6 mx-auto w-full max-w-[600px]"
    >
      <h1 class="text-white text-lg mb-4">Todos os posts</h1>
      <div class="text-center mb-4">Carregando...</div>
      <XPost address="123" timestamp="16:54" message="Descrição da mensagem" />
    </div>
  </main>
</template>
