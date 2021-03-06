<script setup lang="ts">
import { ethers } from "ethers"
import {onMounted, ref} from "vue"
import myEpicNft from '../utils/MyEpicNFT.json';

defineProps<{
  msg: string
}>()
const CONTRACT_ADDRESS = "0x311970bA54385ae9bD68287843B974c6525d7c64";

const currentAccount = ref()
const isMining = ref(false)
const isMintDone = ref(false)

const info = ref('')
const error = ref('')


const setupEventListener = async () => {
  error.value = ""
  try {
    const { ethereum } = window;

    if (ethereum) {
      const provider = new ethers.providers.Web3Provider(ethereum);
      const signer = provider.getSigner();
      const connectedContract = new ethers.Contract(CONTRACT_ADDRESS, myEpicNft.abi, signer);


      // This will essentially "capture" our event when our contract throws it.
      // If you're familiar with webhooks, it's very similar to that!
      connectedContract.on("NewEpicNFTMinted", (from, tokenId) => {
        console.log(from, tokenId.toNumber())
        alert(`Hey there! We've minted your NFT and sent it to your wallet. It may be blank right now. It can take a max of 10 min to show up on OpenSea. Here's the link: https://testnets.opensea.io/assets/${CONTRACT_ADDRESS}/${tokenId.toNumber()}`)
      });
    } else {
      error.value = "Ethereum object doesn't exist!"
      console.log("Ethereum object doesn't exist!");
    }
  } catch (err) {
    console.log(err)
    error.value = err.message
    info.value = ""
  }
}

const checkIfWalletIsConnected = async () => {
  error.value = ""
  const { ethereum } = window;

  if (!ethereum) {
    error.value = "Make sure you have metamask!"
    console.log("Make sure you have metamask!");
    return;
  } else {
    console.log("We have the ethereum object", ethereum);
  }

  const accounts = await ethereum.request({ method: 'eth_accounts' })

  if (accounts.length !== 0) {
    const account = accounts[0];
    console.log("Found an authorized account:", account);
    // Setup listener! This is for the case where a user comes to our site
    // and ALREADY had their wallet connected + authorized.
    setupEventListener()
    currentAccount.value = account
  } else {
    console.log("No authorized account found");
  }
}

const connectWallet = async () => {
  error.value = ""
  try {
    const { ethereum } = window;

    if (!ethereum) {
      alert("Get MetaMask!");
      return;
    }

    const accounts = await ethereum.request({ method: "eth_requestAccounts" });

    let chainId = await ethereum.request({ method: 'eth_chainId' });
    console.log("Connected to chain " + chainId);
    const rinkebyChainId = "0x4";
    if (chainId !== rinkebyChainId) {
      alert("You are not connected to the Rinkeby Test Network!");
      error.value = "You are not connected to the Rinkeby Test Network!"
    } else {
      info.value = "Connected!"
      console.log("Connected", accounts[0]);
      currentAccount.value = accounts[0]
      setupEventListener()
    }
  } catch (err) {
    console.log(err);
    error.value = err.message
    info.value = ""
  }
}

const askContractToMintNft = async () => {
  error.value = ""
  info.value = ""
  try {
    const { ethereum } = window;

    if (ethereum) {
      const provider = new ethers.providers.Web3Provider(ethereum);
      const signer = provider.getSigner();
      const connectedContract = new ethers.Contract(CONTRACT_ADDRESS, myEpicNft.abi, signer);

      info.value = "Going to pop wallet now to pay gas... \n"
      console.log("Going to pop wallet now to pay gas...")
      let nftTxn = await connectedContract.makeAnEpicNFT();

      isMining.value = true
      info.value = "Mining...please wait. \n"
      console.log("Mining...please wait.")
      await nftTxn.wait();
      isMining.value = false

      // link
      info.value = `Mined, see transaction: https://rinkeby.etherscan.io/tx/${nftTxn.hash}`
      isMintDone.value = true
      console.log(`Mined, see transaction: https://rinkeby.etherscan.io/tx/${nftTxn.hash}`);

    } else {
      error.value = "Ethereum object doesn't exist!"
      console.log("Ethereum object doesn't exist!");
    }
  } catch (err) {
    console.log(err)
    error.value = err.message
    info.value = ""
  }
}

onMounted(() => {
  error.value = ""
  info.value = ""
  checkIfWalletIsConnected()
})
</script>

<template>
    <div class="container">
      <div class="header-container">
        <p class="header gradient-text">My NFT Collection</p>
        <p class="sub-text">
          Each unique. Each beautiful. Discover your NFT today.
        </p>
        <button v-if="!currentAccount" class="cta-button connect-wallet-button" @click="connectWallet">Connect to Wallet</button>
        <button v-else @click="askContractToMintNft" class="cta-button mint-button">
          Mint NFT
        </button>

      </div>

      <div class="loader" v-if="isMining">Minting...</div>
      <div class="footer-container">
<!--        <div v-if="isMintDone">-->
<!--          <p>Minted</p>-->
<!--          INSERT_TOKEN_ID_HERE -->
<!--          <a :href=`https://testnets.opensea.io/assets/${CONTRACT_ADDRESS}` class="cta-button collection-button">-->
<!--            🌊 View Collection on OpenSea-->
<!--          </a>-->
<!--        </div>-->
        <!--        <p class="error" v-if="error">Error: {{ error }}</p>-->
<!--        <p class="info" v-if="info">Info: <br /> {{ info }}</p>-->
      </div>
    </div>

</template>

<style scoped>

.container {
  height: 100%;
  background-color: #0d1116;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}

.header-container {
  padding-top: 30px;
}

.header {
  margin: 0;
  font-size: 50px;
  font-weight: bold;
}

.sub-text {
  font-size: 25px;
  color: white;
}

.gradient-text {
  background: -webkit-linear-gradient(left, #60c657 30%, #35aee2 60%);
  background-clip: text;
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

.cta-button {
  height: 45px;
  border: 0;
  width: auto;
  padding-left: 40px;
  padding-right: 40px;
  border-radius: 5px;
  cursor: pointer;
  font-size: 16px;
  font-weight: bold;
  color: white;
}

.connect-wallet-button {
  background: -webkit-linear-gradient(left, #60c657, #35aee2);
  background-size: 200% 200%;
  animation: gradient-animation 4s ease infinite;
}

.mint-button {
  background: -webkit-linear-gradient(left, #a200d6, #ff6fdf);
  background-size: 200% 200%;
  animation: gradient-animation 4s ease infinite;
  margin-right: 15px;
}

.opensea-button {
  background-color: rgb(32, 129, 226);
}

.mint-count {
  color: white;
  font-size: 18px;
  font-weight: bold;
}

.footer-container {
  display: flex;
  justify-content: center;
  align-items: center;
  padding-bottom: 30px;

  color: white;
  font-size: 25px;
}


/* KeyFrames */
@-webkit-keyframes gradient-animation {
  0% {
    background-position: 0% 50%;
  }
  50% {
    background-position: 100% 50%;
  }
  100% {
    background-position: 0% 50%;
  }
}
@-moz-keyframes gradient-animation {
  0% {
    background-position: 0% 50%;
  }
  50% {
    background-position: 100% 50%;
  }
  100% {
    background-position: 0% 50%;
  }
}
@keyframes gradient-animation {
  0% {
    background-position: 0% 50%;
  }
  50% {
    background-position: 100% 50%;
  }
  100% {
    background-position: 0% 50%;
  }
}




.loader {
  animation: spinningColor 1.5s ease-in-out infinite;
  margin: 50px auto;
  border: 5px double #f0eff5;
  border-radius: 50%;
  width: 80px;
  height: 80px;

}



@keyframes spinningColor {
  0% {
    transform: rotate(360deg);
    border-top:5px dashed #ff6fdf;
    border-bottom:5px dashed #35aee2;
  }
  25% {
    border-top:5px dashed #ff6fdf;
    border-bottom:5px dashed #35aee2;
  }
  50% {
    border-top:5px dashed #ff6fdf;
    border-bottom:5px dashed #35aee2;
  }
  75% {
    border-top:5px dashed #ff6fdf;
    border-bottom:5px dashed #35aee2;
  }
  100% {
    border-top:5px dashed #ff6fdf;
    border-bottom:5px dashed #35aee2;
  }
}

</style>
