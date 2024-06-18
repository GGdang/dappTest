<script setup lang="ts">
import { ref } from 'vue'
import erc20ABI from '../src/abi/erc20'
import { RouterLink, RouterView } from 'vue-router'
import { Contract, ethers } from 'ethers'
import { createWeb3Modal, defaultConfig, useWeb3ModalAccount } from '@web3modal/ethers5/vue'

const { address, chainId, isConnected } = useWeb3ModalAccount()
const signMessage = ref('')
const userBalance = ref({
  main: '0',
  usdt: '0',
  usdc: '0'
})
const nativeAmount = ref('0')
const erc20Amount = ref('0')

// 1. Get projectId at https://cloud.walletconnect.com
const projectId = '97c2e47c2dee9a82d216339f564f2dbd'
// 2. Set chains
const chains = [
  {
    chainId: 11155111,
    name: 'Ethereum',
    currency: 'ETH',
    explorerUrl: 'https://sepolia.etherscan.io/',
    rpcUrl: 'https://endpoints.omniatech.io/v1/eth/sepolia/public'
  },
  {
    chainId: 80002,
    name: 'Polygon Amoy',
    currency: 'MATIC',
    explorerUrl: 'https://amoy.polygonscan.com/',
    rpcUrl: 'https://polygon-amoy.blockpi.network/v1/rpc/public	'
  },
  {
    chainId: 97,
    name: 'BNB Smart ChainTest',
    currency: 'BNBt',
    explorerUrl: 'https://testnet.bscscan.com/',
    rpcUrl: 'https://endpoints.omniatech.io/v1/bsc/testnet/public'
  }
]
// 3. Create your application's metadata object
const metadata = {
  name: 'My Website',
  description: 'My Website description',
  url: 'https://mywebsite.com', // url must match your domain & subdomain
  icons: ['https://avatars.mywebsite.com/']
}
// 4. Create Ethers config
const ethersConfig = defaultConfig({
  /*Required*/
  metadata,

  /*Optional*/
  enableEIP6963: true, // true by default
  enableInjected: true // true by default
  // enableCoinbase: true, // true by default
  // rpcUrl: '...', // used for the Coinbase SDK
  // defaultChainId: 1 // used for the Coinbase SDK
})
// 5. Create a Web3Modal instance
const modal = createWeb3Modal({
  ethersConfig,
  chains,
  projectId,
  enableAnalytics: true, // Optional - defaults to your Cloud configuration
  enableOnramp: true // Optional - false as default
})

// ---- test erc20 contract
const supportTokens = {
  Ethereum: {
    USDT: '0xdAC17F958D2ee523a2206206994597C13D831ec7',
    USDC: '0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48'
  },
  'Polygon Amoy': {
    USDT: '0xc2132D05D31c914a87C6611C10748AEb04B58e8F',
    USDC: '0x2791Bca1f2de4661ED88A30C99A7a9449Aa84174'
  },
  'BNB Smart ChainTest': {
    USDT: '0x55d398326f99059fF775485246999027B3197955',
    USDC: '0x8AC76a51cc950d9822D68b83fE1Ad97B32Cd580d'
  }
}

// The ERC-20 Contract ABI, which is a common contract interface
// for tokens (this is the Human-Readable ABI format)
const USDTAbi = erc20ABI

async function getBalance() {
  try {
    console.log('----', address.value, chainId.value)
    // const walletProvider = modal.getWalletProvider()
    // // console.log('---', walletProvider)
    // if (!isConnected.value) throw Error('User disconnected')
    // if (walletProvider !== undefined && address.value !== undefined) {
    //   const ethersProvider = new ethers.providers.Web3Provider(walletProvider)
    //   const getEthers = await ethersProvider.getBalance(address.value)
    //   const signer = await ethersProvider.getSigner()
    //   // The Contract object
    //   const USDTContract = new Contract(USDTAddress, USDTAbi, signer)
    //   const USDTBalance = await USDTContract.balanceOf(address.value)
    // }
  } catch (error) {
    console.log('getBalance error:', error)
  }
}

async function sendBalance() {
  const provider = new ethers.providers.JsonRpcProvider(
    'https://ethereum-sepolia-rpc.publicnode.com'
  )
  const mnemonic = 'false orange legal duty lock ability bag sunny fame winner where best'
  const wallet = ethers.Wallet.fromMnemonic(mnemonic).connect(provider)
  const contract = new Contract(USDTAddress, USDTAbi, wallet)
  console.log('decimals', await contract.decimals())

  // const walletProvider = modal.getWalletProvider()
  // // console.log('---', walletProvider)

  // if (!isConnected.value) throw Error('User disconnected')
  // const ethersProvider = new ethers.providers.Web3Provider(walletProvider)
  // console.log()

  // const signer = await ethersProvider.getSigner()
  // // The Contract object
  // const USDTContract = new Contract(USDTAddress, USDTAbi, signer)
  // console.log('decimals', await USDTContract.decimals())

  // const txhash = await USDTContract.transfer(
  //   '0xC607a64C78DF03A9E567288e4aC6b7869369201A',
  //   ethers.utils.parseUnits('10', 6)
  // )
  // console.log('---txhash', txhash)
}

async function sendEth() {
  const walletProvider = modal.getWalletProvider()

  if (!isConnected.value) throw Error('User disconnected')
  if (walletProvider !== undefined && address.value !== undefined) {
    const ethersProvider = new ethers.providers.Web3Provider(walletProvider)
    const signer = await ethersProvider.getSigner()
    const txhash = await signer.sendTransaction({
      to: '0xC607a64C78DF03A9E567288e4aC6b7869369201A',
      value: ethers.utils.parseEther('0.0000001')
      // maxPriorityFeePerGas: ethers.utils.parseUnits('10', 'gwei'),
      // maxFeePerGas: ethers.utils.parseUnits('2', 'gwei')
    })

    console.log('---txhash', txhash)
  }
}

async function signMsg() {
  const walletProvider = modal.getWalletProvider()
  if (walletProvider !== undefined && address.value !== undefined) {
    const ethersProvider = new ethers.providers.Web3Provider(walletProvider)
    const signer = await ethersProvider.getSigner()
    const signature = await signer.signMessage(signMessage.value)
    console.log('Signature:', signature)
  }
}
</script>

<template>
  <v-row>
    <v-col cols="4">
      <v-btn color="blue" block elevation="4" @click="modal.open()">Open Connect Modal</v-btn>
    </v-col>
    <v-col cols="4">
      <v-btn color="black" block elevation="4" @click="modal.disconnect()">Disconnect Modal</v-btn>
    </v-col>
    <v-col cols="4">
      <v-btn color="purple" block elevation="4" @click="modal.open({ view: 'Networks' })"
        >Open Network Modal</v-btn
      >
    </v-col>
  </v-row>
  <div class="mb-5"></div>
  <v-row>
    <v-col cols="4">
      <v-card variant="outlined">
        <v-card-title> 使用者的餘額 </v-card-title>
        <v-card-text> 原生幣:{{ userBalance.main }} </v-card-text>
        <v-card-text> USDT:{{ userBalance.usdt }} </v-card-text>
        <v-card-text> USDC:{{ userBalance.usdc }} </v-card-text>
      </v-card>
      <div class="mb-2"></div>
      <v-btn color="black" block elevation="4" @click="getBalance">get balance</v-btn>
    </v-col>
  </v-row>
  <div class="mb-5"></div>
  <v-row>
    <v-col cols="4">
      <v-textarea
        v-model="signMessage"
        no-resize
        rows="6"
        bg-color="grey-lighten-2"
        label="輸入要簽名的內容"
        variant="solo"
      ></v-textarea>
      <v-btn color="purple" block elevation="4" @click="signMsg">signMessage</v-btn>
    </v-col>
    <v-col cols="4">
      <v-text-field
        v-model="nativeAmount"
        label="輸入數量"
        bg-color="grey-lighten-2"
        variant="outlined"
      ></v-text-field>
      <v-btn color="purple" block elevation="4" @click="sendEth">send native balance</v-btn>
    </v-col>
    <v-col cols="4">
      <v-text-field
        v-model="erc20Amount"
        label="輸入數量"
        bg-color="grey-lighten-2"
        variant="outlined"
      ></v-text-field>
      <v-btn color="purple" block elevation="4" @click="sendBalance">send usdt balance</v-btn>
    </v-col>
  </v-row>
  <div class="mb-5"></div>
  <RouterView />
</template>

<style lang="scss" scoped></style>
