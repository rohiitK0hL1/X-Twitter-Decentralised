<template>
  <main class="bg-gradient-to-br from-purple-500 via-pink-500 to-red-500 min-h-screen flex items-center justify-center p-10">
    <div class="w-full max-w-screen-xl flex flex-row gap-6">
      <!-- Wallet and Post Section -->
      <div class="flex flex-col flex-shrink-0 w-1/3 bg-white shadow-lg rounded-lg p-6 space-y-6">
        <h1 class="text-3xl font-bold text-gray-800 text-center">
          Decentralized X
        </h1>
        <p class="text-gray-600 text-center">
          A colorful and exciting decentralized Twitter! Connect your wallet to share posts on the blockchain.
        </p>
        <div v-if="!connectedWallet" class="text-center">
          <XButton @click="connectWallet" :loading="loading">
            <MetaMaskIcon />
            Connect Wallet
          </XButton>
        </div>
        <div v-else>
          <XTextField
            v-model="message"
            label="Post a Message"
            placeholder="What's on your mind?"
            class="mb-4"
          />
          <XButton text="Send Post" @click="createPost" class="w-full" />
          <div class="mt-4 text-sm text-gray-500 text-center">
            Wallet Connected: <span class="font-bold">{{ connectedWallet }}</span>
          </div>
        </div>
      </div>

      <!-- Info and Interactive Section -->
      <div class="flex-grow bg-white shadow-lg rounded-lg p-6 space-y-4">
        <h2 class="text-2xl font-bold text-gray-800">Welcome to Decentralized X</h2>
        <p class="text-gray-600">
          This is your decentralized space to share thoughts securely on the blockchain. Every message you send is stored immutably and connected to your Ethereum address.
        </p>
        <div class="p-4 bg-gradient-to-r from-blue-500 to-green-500 text-white rounded-lg shadow-md">
          <h3 class="text-lg font-bold">Did you know?</h3>
          <p>
            Ethereum is a decentralized, open-source blockchain that supports smart contracts. It powers many dApps like this one!
          </p>
        </div>
        <div class="flex flex-row items-center space-x-4">
          <button
            @click="incrementCounter"
            class="px-4 py-2 bg-blue-500 text-white rounded-lg hover:bg-blue-600 transition"
          >
            Click Me
          </button>
          <span class="text-gray-700 font-medium">Clicked {{ counter }} times</span>
        </div>
      </div>
    </div>
  </main>
</template>

<script setup>
import XButton from './components/XButton.vue'
import XTextField from './components/XTextField.vue'
import MetaMaskIcon from './components/MetaMaskIcon.vue'
import { ref, onMounted } from 'vue'

import { ethers } from 'ethers'
import abi from '@/contracts/XPost.json'

const connectedWallet = ref(null)
const loading = ref(false)
const message = ref(null)
const counter = ref(0)

const contractAddress = '0xa182f4a8A38Aee4d08D240a9d657Dd7D554c8c67'
const contractABI = abi.abi

const getEthereumObject = () => window.ethereum

const findMetamaskAccount = async () => {
  loading.value = true
  try {
    const ethereum = getEthereumObject()

    if (!ethereum) {
      console.error('Install the MetaMask extension!')
      return null
    }

    const accounts = await ethereum.request({ method: 'eth_accounts' })
    if (accounts.length !== 0) {
      const account = accounts[0]
      connectedWallet.value = account
      return account
    } else {
      return null
    }
  } catch (error) {
    console.error(error)
    return null
  } finally {
    loading.value = false
  }
}

const connectWallet = async () => {
  loading.value = true
  try {
    const ethereum = getEthereumObject()
    if (!ethereum) {
      alert('Install the MetaMask extension!')
      return
    }

    const accounts = await ethereum.request({
      method: 'eth_requestAccounts'
    })

    connectedWallet.value = accounts[0]
  } catch (error) {
    console.error(error)
  } finally {
    loading.value = false
  }
}

const createPost = async () => {
  const ethereum = getEthereumObject()
  if (!ethereum) {
    alert('Install the MetaMask extension!')
    return
  }

  try {
    const provider = new ethers.BrowserProvider(window.ethereum)
    const signer = await provider.getSigner()
    const xPostContract = new ethers.Contract(contractAddress, contractABI, signer)

    const postTxn = await xPostContract.createPost(message.value, {
      gasLimit: 300000
    })

    await postTxn.wait()
  } catch (error) {
    console.log(error)
  }
}

const incrementCounter = () => {
  counter.value++
}

onMounted(async () => {
  await findMetamaskAccount()
})
</script>
