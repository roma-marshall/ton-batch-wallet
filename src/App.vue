<template>
  <div class="min-h-screen bg-[#F7F9FB]">
    <div class="container mx-auto py-20">
      <h1 class="text-3xl font-semibold text-center text-[#04060b] !mb-6">Wallet Generator</h1>

      <div class="mt-6">
        <button
            @click="generateKeys"
            class="text-white shadow-brand cursor-pointer border rounded-lg py-2 px-4 w-full bg-linear-to-r from-[#2D83EC]/90 to-[#1AC9FF]/90 hover:from-[#2D83EC] hover:to-[#1AC9FF] transition duration-300"
        >
          Generate New Wallet
        </button>
      </div>
    </div>

    <div class="container mx-auto">
      <div v-if="mnemonics.length > 0" class="mb-4">
        <p class="text-xl text-[#191f2f]">Mnemonic Phrase</p>
        <p class="text-base text-[#728a96]">{{ mnemonics.join(" ") }}</p>
      </div>

      <div v-if="privateKey" class="mb-4">
        <p class="text-xl text-[#191f2f]">Private Key (Hex):</p>
        <p class="text-base text-[#728a96] break-all">{{ privateKey }}</p>
      </div>

      <div v-if="publicKey" class="mb-4">
        <p class="text-xl text-[#191f2f] mt-2">Public Key (Hex):</p>
        <p class="text-base text-[#728a96] break-all">{{ publicKey }}</p>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { WalletContractV4 } from '@ton/ton'
import { mnemonicNew, mnemonicToPrivateKey } from '@ton/crypto'

// Reactive variables
const mnemonics = ref<string[]>([])
const privateKey = ref<string>('')
const publicKey = ref<string>('')
const walletAddress = ref<string>('')

// Convert Uint8Array to hex string
const toHex = (bytes: Uint8Array): string =>
    Array.from(bytes)
        .map((b) => b.toString(16).padStart(2, '0'))
        .join('')

// Function to generate wallet keys
const generateKeys = async () => {
  try {
    // Generate mnemonic
    const mnemonicsData = await mnemonicNew()
    mnemonics.value = mnemonicsData

    // Generate key pair
    const keyPair = await mnemonicToPrivateKey(mnemonicsData)
    privateKey.value = toHex(keyPair.secretKey)
    publicKey.value = toHex(keyPair.publicKey)
  } catch (error) {
    console.error('Error generating wallet:', error)
  }
}
</script>
