<template>
  <div class="min-h-screen bg-[#F7F9FB]">
    <div class="container mx-auto py-20">
      <h1 class="text-3xl font-semibold text-center text-[#04060b] !mb-6">TON Wallet Batch Generator</h1>

      <div class="mb-4 text-center">
        <label for="walletCount" class="block text-lg text-[#191f2f] mb-2">Number of wallets:</label>
        <input
            id="walletCount"
            type="number"
            min="1"
            v-model.number="walletCount"
            class="w-full border border-gray-300 rounded-lg px-4 py-2 text-center text-lg focus:outline-none focus:ring-2 focus:ring-blue-500"
        />
      </div>

      <div class="mt-6">
        <button
            @click="generateKeys"
            class="text-white shadow-brand cursor-pointer border rounded-lg py-2 px-4 w-full bg-gradient-to-r from-[#2D83EC]/90 to-[#1AC9FF]/90 hover:from-[#2D83EC] hover:to-[#1AC9FF] transition duration-300"
        >
          Generate Wallets
        </button>
      </div>
    </div>

    <div class="container mx-auto pb-20">
      <div v-for="(wallet, index) in wallets" :key="index" class="mb-6 p-4 bg-white shadow rounded-lg">
        <h2 class="text-xl font-semibold text-[#191f2f]">Wallet #{{ index + 1 }}</h2>

        <div class="mt-2">
          <p class="text-lg text-[#191f2f]">Mnemonic Phrase:</p>
          <p class="text-sm text-[#728a96] break-all">{{ wallet.mnemonics.join(" ") }}</p>
        </div>

        <div class="mt-2">
          <p class="text-lg text-[#191f2f]">Private Key (Hex):</p>
          <p class="text-sm text-[#728a96] break-all">{{ wallet.privateKey }}</p>
        </div>

        <div class="mt-2">
          <p class="text-lg text-[#191f2f]">Public Key (Hex):</p>
          <p class="text-sm text-[#728a96] break-all">{{ wallet.publicKey }}</p>
        </div>

        <div class="mt-2">
          <p class="text-lg text-[#191f2f]">Wallet Address v4:</p>
          <p class="text-sm text-[#728a96] break-all">{{ wallet.walletAddressV4 }}</p>
        </div>
        <div class="mt-2">
          <p class="text-lg text-[#191f2f]">Wallet Address v5:</p>
          <p class="text-sm text-[#728a96] break-all">{{ wallet.walletAddressV5R1 }}</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { WalletContractV4, WalletContractV5R1 } from '@ton/ton'
import { mnemonicNew, mnemonicToPrivateKey } from '@ton/crypto'

const walletCount = ref<number>(1)
const wallets = ref<{ mnemonics: string[]; privateKey: string; publicKey: string; walletAddressV4: string; walletAddressV5R1: string }[]>([])

// convert Uint8Array to hex string
const toHex = (bytes: Uint8Array): string =>
    Array.from(bytes)
        .map((b) => b.toString(16).padStart(2, '0'))
        .join('')

const generateKeys = async () => {
  wallets.value = [] // clear previous wallets

  for (let i = 0; i < walletCount.value; i++) {
    try {
      // generate mnemonic phrase
      const mnemonicsData = await mnemonicNew()

      // generate key pair
      const keyPair = await mnemonicToPrivateKey(mnemonicsData)
      const privateKeyHex = toHex(keyPair.secretKey)
      const publicKeyHex = toHex(keyPair.publicKey)

      // create wallet contract
      const workchain = 0
      const walletV4 = WalletContractV4.create({ workchain, publicKey: keyPair.publicKey })
      const walletV5R1 = WalletContractV5R1.create({ workchain, publicKey: keyPair.publicKey })
      const walletAddrV4 = walletV4.address.toString()
      const walletAddrV5R1 = walletV5R1.address.toString()

      // store wallet details in the array
      wallets.value.push({
        mnemonics: mnemonicsData,
        privateKey: privateKeyHex,
        publicKey: publicKeyHex,
        walletAddressV4: walletAddrV4,
        walletAddressV5R1: walletAddrV5R1,
      })
    } catch (error) {
      console.error('Error generating wallet:', error)
    }
  }
}
</script>
