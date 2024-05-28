<script setup lang="ts">
import { ref } from 'vue'
import erc20ABI from '../src/abi/erc20'
import { RouterLink, RouterView } from 'vue-router'
import { Contract, ethers } from 'ethers'
import { createWeb3Modal, defaultConfig, useWeb3ModalAccount } from '@web3modal/ethers5/vue'
const signMessage = ref('')
// 1. Get projectId at https://cloud.walletconnect.com
const projectId = '97c2e47c2dee9a82d216339f564f2dbd'
// 2. Set chains
const mainnet = {
  chainId: 11155111,
  name: 'Ethereum',
  currency: 'ETH',
  // explorerUrl: 'https://etherscan.io',
  // rpcUrl: 'https://cloudflare-eth.com'
  explorerUrl: 'https://sepolia.etherscan.io/',
  rpcUrl: 'https://endpoints.omniatech.io/v1/eth/sepolia/public'
}
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
  chains: [mainnet],
  projectId,
  enableAnalytics: true, // Optional - defaults to your Cloud configuration
  enableOnramp: true // Optional - false as default
})

const { address, chainId, isConnected } = useWeb3ModalAccount()

// ---- test usdt contract
const USDTAddress = '0x428Eec6a3CcAe60527AE47AC6e8f0f871283d8EB'

// The ERC-20 Contract ABI, which is a common contract interface
// for tokens (this is the Human-Readable ABI format)
const USDTAbi = erc20ABI

async function getBalance() {
  try {
    console.log('----', address.value, chainId.value)
    const walletProvider = modal.getWalletProvider()
    // console.log('---', walletProvider)

    if (!isConnected.value) throw Error('User disconnected')
    if (walletProvider !== undefined && address.value !== undefined) {
      const ethersProvider = new ethers.providers.Web3Provider(walletProvider)
      const getEthers = await ethersProvider.getBalance(address.value)
      const signer = await ethersProvider.getSigner()
      // The Contract object
      const USDTContract = new Contract(USDTAddress, USDTAbi, signer)
      const USDTBalance = await USDTContract.balanceOf(address.value)
    }
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

async function mnemonicSendUsdt() {
  try {
    const mnemonic =
      'cross select eight science grief taste climb bind gadget number material alley'
    const rpc = `https://eth-sepolia.g.alchemy.com/v2/ZcZCvZMQ4ac0jBAlzS06FVrYffz9AW13`
    console.log(1)

    const provider = await new ethers.providers.JsonRpcProvider(rpc)
    console.log(2)
    const wallet = await ethers.Wallet.fromMnemonic(mnemonic).connect(provider)
    console.log(3)
    const nonce = await wallet.getTransactionCount()
    console.log('---nonce---', nonce)
    const erc20Contract = await new ethers.Contract(
      '0x428Eec6a3CcAe60527AE47AC6e8f0f871283d8EB',
      erc20ABI,
      wallet
    )
    console.log(5)
    const signedTx = await erc20Contract.transfer(
      '0xD2B7b2E073A1e36326ba5d40A9f042846Ef4A4A3',
      ethers.utils.parseUnits('1', 6),
      {
        gasLimit: 100000,
        nonce: nonce,
        gasPrice: ethers.utils.parseUnits('150', 'gwei')
      }
    )
    console.log('----signedTx----', signedTx)

    // 簽署交易並發送
    const txhash = await signedTx.wait()
    console.log('----txhash----', txhash)
  } catch (error) {
    console.log(error)
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
  <header>
    <img alt="Vue logo" class="logo" src="@/assets/logo.svg" width="125" height="125" />
    <div>
      <!-- <nav>
        <RouterLink to="/">Home</RouterLink>
        <RouterLink to="/about">About</RouterLink>
        <RouterLink to="/walletConnect">walletConnect</RouterLink>
      </nav> -->
      <div>
        <button @click="modal.open()">Open Connect Modal</button>
        <button @click="modal.disconnect()">Disconnect Modal</button>
        <button @click="modal.open({ view: 'Networks' })">Open Network Modal</button>
      </div>

      <div>
        <textarea v-model="signMessage" cols="30" rows="10"></textarea>
        <button @click="signMsg">signMessage</button>
      </div>

      <div>
        <button @click="getBalance">get balance</button>
        <button @click="sendBalance">send usdt balance</button>
        <button @click="sendEth">send eth balance</button>
      </div>
    </div>
  </header>

  <RouterView />
</template>

<style scoped>
header {
  line-height: 1.5;
  max-height: 100vh;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

nav {
  width: 100%;
  font-size: 12px;
  text-align: center;
  margin-top: 2rem;
}

nav a.router-link-exact-active {
  color: var(--color-text);
}

nav a.router-link-exact-active:hover {
  background-color: transparent;
}

nav a {
  display: inline-block;
  padding: 0 1rem;
  border-left: 1px solid var(--color-border);
}

nav a:first-of-type {
  border: 0;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }

  .wrapper {
    /* display: flex; */
  }

  nav {
    text-align: left;
    margin-left: -1rem;
    font-size: 1rem;

    padding: 1rem 0;
    margin-top: 1rem;
  }
}
</style>
