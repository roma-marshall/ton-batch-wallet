<template>
  <div class="min-h-screen bg-[#F7F9FB] flex flex-col justify-between">
    <div class="container mx-auto py-20">
      <h1 class="text-3xl font-semibold text-center text-[#04060b] !mb-6">TON Wallet Batch Generator</h1>

      <div class="mb-4 text-center">
        <label for="walletCount" class="block text-lg text-[#191f2f] mb-2">Number of wallets:</label>
        <input
            id="walletCount"
            type="number"
            min="1"
            v-model.number="walletCount"
            class="w-full border border-gray-300 rounded-lg px-4 py-2 text-center text-lg focus:outline-none focus:ring-2 focus:ring-[#0098EA]"
        />
      </div>

      <div class="flex justify-center gap-4 mt-6">
        <button
            @click="generateKeys"
            class="text-white shadow-brand cursor-pointer border rounded-lg py-2 px-4 bg-gradient-to-r from-[#2D83EC]/90 to-[#1AC9FF]/90 hover:from-[#2D83EC] hover:to-[#1AC9FF] transition duration-300"
        >
          Generate Wallets
        </button>

        <button
            @click="downloadWallets"
            :disabled="wallets.length === 0"
            class="text-white shadow-brand cursor-pointer border rounded-lg py-2 px-4 bg-gradient-to-r from-[#1ABC9C]/90 to-[#2ECC71]/90 hover:from-[#1ABC9C] hover:to-[#2ECC71] transition duration-300 disabled:bg-gray-400"
        >
          Download JSON
        </button>
      </div>
    </div>

    <div class="container mx-auto">
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

    <a href="https://ton.org" target="_blank" class="flex justify-center pb-4">
      <span class="text-sm text-slate-400">Build on TON</span>
      <svg aria-hidden="true" viewBox="0 0 26 26" class="flex-none w-4 h-4 ml-1 mt-0.5 animate-pulse" fill="#0098EA">
        <path fill-rule="evenodd" clip-rule="evenodd"
              d="M4.14893 0H21.9894C22.6206 0 23.2513 0.0926993 23.9104 0.399992C24.7005 0.768174 25.1195 1.34879 25.413 1.77809C25.4359 1.81149 25.4572 1.84591 25.477 1.88113C25.8221 2.49549 26 3.15881 26 3.87234C26 4.55031 25.8387 5.28895 25.477 5.93273C25.4736 5.93886 25.47 5.945 25.4665 5.95113L14.1952 25.3128C13.9466 25.7397 13.4892 26.0017 12.9951 26C12.501 25.9982 12.0454 25.733 11.7999 25.3043L0.73554 5.98385C0.732359 5.97874 0.729178 5.9734 0.725997 5.96817C0.472779 5.55091 0.0811949 4.90566 0.0126931 4.07284C-0.0502423 3.30717 0.121863 2.5399 0.506633 1.87465C0.891402 1.20929 1.47066 0.677406 2.16738 0.351824C2.91442 0.00272648 3.67146 0 4.14893 0ZM11.617 2.76597H4.14893C3.65829 2.76597 3.46993 2.79619 3.33827 2.85776C3.15617 2.94274 3.00326 3.08247 2.90102 3.25934C2.79878 3.43622 2.75254 3.64116 2.76935 3.84621C2.77901 3.9639 2.82695 4.0984 3.1097 4.56474C3.11561 4.57451 3.1214 4.58439 3.12708 4.59428L11.617 19.4192V2.76597ZM14.383 2.76597V19.4924L23.0695 4.57076C23.1676 4.39263 23.234 4.13521 23.234 3.87234C23.234 3.65911 23.1898 3.47394 23.091 3.28297C22.9873 3.13393 22.9241 3.05498 22.8713 3.0009C22.826 2.95466 22.7911 2.9299 22.7418 2.90695C22.5364 2.81119 22.3262 2.76597 21.9894 2.76597H14.383Z"/>
      </svg>
    </a>
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

// download wallets as JSON
const downloadWallets = () => {
  if (wallets.value.length === 0) return

  const dataStr = "data:text/json;charset=utf-8," + encodeURIComponent(JSON.stringify(wallets.value, null, 2))
  const downloadAnchor = document.createElement("a")
  downloadAnchor.setAttribute("href", dataStr)
  downloadAnchor.setAttribute("download", "wallets.json")
  document.body.appendChild(downloadAnchor)
  downloadAnchor.click()
  document.body.removeChild(downloadAnchor)
}
</script>
