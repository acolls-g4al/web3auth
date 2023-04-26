<template>
  <div id="app">
    <div>
      <a target="_blank" href="http://gamesforaliving.com/" rel="noreferrer" style="margin: 4rem">
        <img src="../assets/logo.png" alt="G4al logo" height="30" />
      </a>
    </div>
    <button
      v-if="!loggedin"
      class="card"
      @click="login"
      style="cursor: pointer; width: 100px; margin: 30px"
    >
      Login
    </button>

    <div v-if="loggedin">
      <div class="flex-container">
        <div>
          <button class="card" @click="getUserInfo" style="cursor: pointer">
            Get User Info
          </button>
        </div>
        <div>
          <button
            class="card"
            @click="authenticateUser"
            style="cursor: pointer"
          >
            Get ID Token
          </button>
        </div>
        <div>
          <button class="card" @click="getChainId" style="cursor: pointer">
            Get Chain ID
          </button>
        </div>
        <div>
          <button class="card" @click="addChain" style="cursor: pointer">
            Add Chain
          </button>
        </div>
        <div>
          <button class="card" @click="switchChain" style="cursor: pointer">
            Switch Chain
          </button>
        </div>
        <div>
          <button class="card" @click="getAccounts" style="cursor: pointer">
            Get Accounts
          </button>
        </div>
        <div>
          <button class="card" @click="getBalance" style="cursor: pointer">
            Get Balance
          </button>
        </div>
        <div>
          <button class="card" @click="getPrivateKey" style="cursor: pointer">
            Get Private Key
          </button>
        </div>
        <div>
          <button class="card" @click="signMessage" style="cursor: pointer">
            Sign Message
          </button>
        </div>
        <div>
          <button class="card" @click="sendTransaction" style="cursor: pointer">
            Sign Transaction
          </button>
        </div>
        <div>
          <button class="card" @click="logout" style="cursor: pointer">
            Logout
          </button>
        </div>
      </div>
      <div id="console" style="white-space: pre-line">
        <p style="white-space: pre-line"></p>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { ref, onMounted } from "vue";
import { Web3Auth } from "@web3auth/modal";
import { CHAIN_NAMESPACES, SafeEventEmitterProvider } from "@web3auth/base";
import RPC from "../web3RPC";

// Plugins
import { TorusWalletConnectorPlugin } from "@web3auth/torus-wallet-connector-plugin";

// Adaptersd="app" class="main">
import { WalletConnectV1Adapter } from "@web3auth/wallet-connect-v1-adapter";
import { MetamaskAdapter } from "@web3auth/metamask-adapter";
import { TorusWalletAdapter } from "@web3auth/torus-evm-adapter";

export default {
  // eslint-disable-next-line vue/multi-word-component-names
  name: "Modal",
  props: {
    msg: String,
  },
  setup() {
    const CHAIN_CONFIG = {
      chainNamespace: CHAIN_NAMESPACES.EIP155,
      chainId: "0x61",
      rpcTarget: "https://bsc-testnet.public.blastapi.io",
    };
    const loggedin = ref<boolean>(false);
    const loading = ref<boolean>(false);
    const loginButtonStatus = ref<string>("");
    const connecting = ref<boolean>(false);
    let provider = ref<SafeEventEmitterProvider | any>(false);
    const clientId =
      "BBu2lTZQutOAheR1W14jdTVBY-t255TSLvK6Nan2mnkwxZjs4Q9Ch2JKXbVUshJMNjycd1314gzkPoafmPI9PZc"; // get from https://dashboard.web3auth.io

    const web3auth = new Web3Auth({
      clientId,
      chainConfig: CHAIN_CONFIG,
      uiConfig: {
        defaultLanguage: "en",
      },
      web3AuthNetwork: "testnet",
    });

    const torusPlugin = new TorusWalletConnectorPlugin({
      torusWalletOpts: {},
      walletInitOptions: {
        whiteLabel: {
          theme: { isDark: true, colors: { primary: "#000" } },
          logoDark: "../assets/logo.png",
          logoLight: "../assets/logo.png",
        },
        useWalletConnect: true,
        enableLogging: true,
      },
    });

    const walletConnectV1Adapter = new WalletConnectV1Adapter({
      adapterSettings: {
        bridge: "https://bridge.walletconnect.org",
      },
      clientId,
    });

    web3auth.configureAdapter(walletConnectV1Adapter);

    // adding metamask adapter

    const metamaskAdapter = new MetamaskAdapter({
      clientId,
      sessionTime: 3600, // 1 hour in seconds
      web3AuthNetwork: "testnet",
      chainConfig: CHAIN_CONFIG,
    });
    // we can change the above settings using this function
    metamaskAdapter.setAdapterSettings({
      sessionTime: 86400, // 1 day in seconds
      chainConfig: CHAIN_CONFIG,
      web3AuthNetwork: "testnet",
    });

    // it will add/update  the metamask adapter in to web3auth class
    web3auth.configureAdapter(metamaskAdapter);

    const torusWalletAdapter = new TorusWalletAdapter({
      clientId,
    });

    // it will add/update  the torus-evm adapter in to web3auth class
    web3auth.configureAdapter(torusWalletAdapter);

    onMounted(async () => {
      try {
        loading.value = true;
        loggedin.value = false;
        await web3auth.initModal();
        await web3auth.addPlugin(torusPlugin);
        if (web3auth.provider) {
          provider = web3auth.provider;
          loggedin.value = true;
        }
      } catch (error) {
        uiConsole("error", error);
      } finally {
        loading.value = false;
      }
    });

    const login = async () => {
      if (!web3auth) {
        uiConsole("web3auth not initialized yet");
        return;
      }
      provider = await web3auth.connect();
      loggedin.value = true;
      uiConsole("Logged in Successfully!");
    };

    const authenticateUser = async () => {
      if (!web3auth) {
        uiConsole("web3auth not initialized yet");
        return;
      }
      const idToken = await web3auth.authenticateUser();
      uiConsole(idToken);
    };

    const getUserInfo = async () => {
      if (!web3auth) {
        uiConsole("web3auth not initialized yet");
        return;
      }
      const user = await web3auth.getUserInfo();
      uiConsole(user);
    };

    const logout = async () => {
      if (!web3auth) {
        uiConsole("web3auth not initialized yet");
        return;
      }
      await web3auth.logout();
      provider = null;
      loggedin.value = false;
    };

    const getChainId = async () => {
      if (!provider) {
        uiConsole("provider not initialized yet");
        return;
      }
      const rpc = new RPC(provider);
      const chainId = await rpc.getChainId();
      uiConsole(chainId);
    };

    const addChain = async () => {
      if (!provider) {
        uiConsole("provider not initialized yet");
        return;
      }
      const newChain = {
        chainId: "0x5",
        displayName: "Goerli",
        chainNamespace: CHAIN_NAMESPACES.EIP155,
        tickerName: "Goerli",
        ticker: "ETH",
        decimals: 18,
        rpcTarget: "https://rpc.ankr.com/eth_goerli",
        blockExplorer: "https://goerli.etherscan.io",
      };
      await web3auth?.addChain(newChain);
      uiConsole("New Chain Added");
    };

    const switchChain = async () => {
      if (!provider) {
        uiConsole("provider not initialized yet");
        return;
      }
      const newChain = {
        chainId: "0x5",
        displayName: "Goerli",
        chainNamespace: CHAIN_NAMESPACES.EIP155,
        tickerName: "Goerli",
        ticker: "ETH",
        decimals: 18,
        rpcTarget: "https://rpc.ankr.com/eth_goerli",
        blockExplorer: "https://goerli.etherscan.io",
      };
      await web3auth?.switchChain({ chainId: "0x5" });
      uiConsole("Chain Switched");
    };

    const getAccounts = async () => {
      if (!provider) {
        uiConsole("provider not initialized yet");
        return;
      }
      const rpc = new RPC(provider);
      const address = await rpc.getAccounts();
      uiConsole(address);
    };

    const getBalance = async () => {
      if (!provider) {
        uiConsole("provider not initialized yet");
        return;
      }
      const rpc = new RPC(provider);
      const balance = await rpc.getBalance();
      uiConsole(balance);
    };

    const sendTransaction = async () => {
      if (!provider) {
        uiConsole("provider not initialized yet");
        return;
      }
      const rpc = new RPC(provider);
      const receipt = await rpc.sendTransaction();
      uiConsole(receipt);
    };

    const signMessage = async () => {
      if (!provider) {
        uiConsole("provider not initialized yet");
        return;
      }
      const rpc = new RPC(provider);
      const signedMessage = await rpc.signMessage('Welcome to Game for a living!');
      uiConsole(signedMessage);
    };

    const getPrivateKey = async () => {
      if (!provider) {
        uiConsole("provider not initialized yet");
        return;
      }
      const rpc = new RPC(provider);
      const privateKey = await rpc.getPrivateKey();
      uiConsole(privateKey);
    };

    function uiConsole(...args: any[]): void {
      const el = document.querySelector("#console>p");
      if (el) {
        el.innerHTML = JSON.stringify(args || {}, null, 2);
      }
    }

    return {
      loggedin,
      loading,
      loginButtonStatus,
      connecting,
      provider,
      web3auth,
      login,
      authenticateUser,
      logout,
      getUserInfo,
      getChainId,
      addChain,
      switchChain,
      getAccounts,
      getBalance,
      sendTransaction,
      signMessage,
      getPrivateKey,
    };
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
.card {
  margin: 0.5rem;
  padding: 0.7rem;
  text-align: center;
  color: #d123c3;
  background-color: transparent;
  text-decoration: none;
  border: 1px solid #ca1ada;
  border-radius: 10px;
  transition: color 0.15s ease, border-color 0.15s ease;
  width: 100%;
}

.card:hover,
.card:focus,
.card:active {
  cursor: pointer;
  background-color: transparent;
}

.flex-container {
  display: flex;
  flex-flow: row wrap;
  justify-content: center;
}

.flex-container > div {
  width: 100px;
  margin: 10px;
  text-align: center;
  line-height: 75px;
  font-size: 30px;
}

#console {
  width: 100%;
  height: 100%;
  overflow: auto;
  word-wrap: break-word;
  font-size: 16px;
  font-family: monospace;
  text-align: left;
}
</style>
