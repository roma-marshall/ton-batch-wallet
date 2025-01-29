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
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { mnemonicNew } from '@ton/crypto'

// Reactive variables
const mnemonics = ref<string[]>([])

// Function to generate wallet keys
const generateKeys = async () => {
  try {
    // Generate mnemonic
    mnemonics.value = await mnemonicNew()
  } catch (error) {
    console.error('Error generating wallet:', error)
  }
}
</script>
