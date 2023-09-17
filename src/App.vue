<script setup>
import XButton from './components/XButton.vue';
import XTextField from './components/XTextField.vue';
import XPost from './components/XPost.vue';

import { ethers } from "ethers";
import abi from "@/contracts/XPost.json";

import { ref, onMounted, watchEffect } from "vue";

const connectedWallet = ref(null);
const loading = ref(false);
const loadingPosts = ref(false);
const message = ref(null);

const contractAddress = "0xb3B14fBB32186d97B7a5795411AA0d728970Ba3c";
const contractABI = abi.abi;

const getEthereumObject = () => window.ethereum;

const findMetamaskAccount = async () => {
  loading.value = true;
  try {
    const ethereum = getEthereumObject();

    if (!ethereum) {
      console.error("Instale a extensão do MetaMask!");
      return null;
    }

    console.log("A extensão do MetaMask está instalada:", ethereum);

    const accounts = await ethereum.request({ method: "eth_accounts" });
    if (accounts.length !== 0) {
      const account = accounts[0];
      console.log("Conta MetaMask conectada:", account);
      connectedWallet.value = account;
      return account;
    } else {
      console.error("Nenhuma conta MetaMask conectada D:");
      return null;
    }
  } catch (error) {
    console.error(error);
    return null;
  } finally {
    loading.value = false;
  }
};

const connectWallet = async () => {
  loading.value = true;
  try {
    const ethereum = getEthereumObject();
    if (!ethereum) {
      alert("Instale a extensão do MetaMask!");
      return;
    }

    const accounts = await ethereum.request({
      method: "eth_requestAccounts",
    });

    console.log("Connected", accounts[0]);
    connectedWallet.value = accounts[0];
    getAllPosts();
  } catch (error) {
    console.error(error);
  }
  loading.value = false;
};

const createPost = async () => {
  loadingPosts.value = true;

  const ethereum = getEthereumObject();
  if (!ethereum) {
    alert("Instale a extensão do MetaMask!");
    return;
  }

  try {
    const provider = new ethers.BrowserProvider(window.ethereum);
    const signer = await provider.getSigner();

    const xPostContract = new ethers.Contract(
      contractAddress,
      contractABI,
      signer
    );

    let count = await xPostContract.getTotalPosts();
    console.log("Número total de posts antes de ser criado um novo:", Number(count));

    const postTxn = await xPostContract.createPost(message.value, {
      gasLimit: 300000,
    });
    console.log("Criando um post novo...", postTxn.hash);

    await postTxn.wait();
    console.log("Criado e salvo na blockchain!", postTxn.hash);

    count = await xPostContract.getTotalPosts();
    console.log("Número total de posts depois de ser criado um novo:", Number(count));
  } catch (error) {
    console.log(error);
  } finally {
    loadingPosts.value = false;
  }
};

const allPosts = ref([]);

const getAllPosts = async () => {
  loading.value = true;
  
  const ethereum = getEthereumObject();
  if (!ethereum) {
    alert("Instale a extensão do MetaMask!");
    return;
  }

  try {
    const provider = new ethers.BrowserProvider(ethereum);
    const signer = await provider.getSigner();
    const xPostContract = new ethers.Contract(
      contractAddress,
      contractABI,
      signer
    );

    const posts = await xPostContract.getAllPosts();

    const normalizedPosts = posts
      .map((post) => {
        return {
          address: post[0],
          timestamp: new Date(Number(post[2]) * 1000),
          message: post[1],
        };
      })
      .sort((a, b) => new Date(b.timestamp) - new Date(a.timestamp));

    allPosts.value = normalizedPosts;
  } catch (error) {
    console.log(error);
  }
  loading.value = false;
};

watchEffect(async (onCleanup) => {
  let xPostContract;

  const onNewPost = (from, timestamp, message) => {
    try {
      console.log("Evento de NewPost recebido:", {from, timestamp, message});
      allPosts.value.unshift({
        address: from,
        timestamp: new Date(Number(timestamp) * 1000),
        message: message,
      })
    } catch(e) {
      console.error(e)
    }
  };

  if (window.ethereum) {
    const provider = new ethers.BrowserProvider(window.ethereum);
    const signer = await provider.getSigner();

    xPostContract = new ethers.Contract(
      contractAddress,
      contractABI,
      signer
    );
    xPostContract.on("NewPost", onNewPost);
  }

  onCleanup(() => {
    if (xPostContract) {
      xPostContract.off("NewPost", onNewPost);
    }
  });
});

onMounted(async() => {
  await findMetamaskAccount();
  await getAllPosts();
})

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
      <XButton v-if="!connectedWallet" text="Conectar carteira" @click="connectWallet" :loading="loading" />
      <template v-else>
        <XTextField v-model="message" label="Post" name="post" class="mb-2" type="text" id="post" placeholder="John" required />
        <XButton text="Enviar post" @click="createPost" />
        <div class="flex items-center">
          <span
            class="bg-green-100 text-green-800 text-xs font-semibold mr-2 px-2.5 py-0.5 rounded dark:bg-green-200 dark:text-green-900 h-fit"
          >
            Carteira conectada!
          </span>
          <span class="truncate">
            {{ connectedWallet }}
          </span>
        </div>
      </template>
    </div>
    <div
      class="mt-8 rounded-md border border-gray-700 text-white bg-gray-800 p-6 mx-auto w-full max-w-[600px]"
      v-if="connectedWallet && allPosts?.length > 0"
    >
      <h1 class="text-white text-lg mb-4">Todos os posts</h1>
      <div v-if="loadingPosts" class="text-center mb-4">Carregando...</div>
      <div v-else>
        <XPost v-for="post in allPosts" :key="post" :address="post.address" :timestamp="post.timestamp.toString()" :message="post.message" />
      </div>
    </div>
  </main>
</template>
