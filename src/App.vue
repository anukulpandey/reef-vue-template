<template>
  <div v-if="isWalletConnected && selectedAccount">
    <div>Selected Address {{ selectedAccount }}</div>
    <button @click="getFlipperState">Get flipper value</button>
    <button @click="flip">Flip</button>
  </div>
  <button v-else @click="initializeDapp">Connect Wallet</button>
</template>

<script>
// eslint-disable-next-line no-unused-vars
import {
  web3Accounts,
  web3Enable,
  web3FromAddress,
} from "@polkadot/extension-dapp";
// import { network } from "../dapp.config";
import { Provider, Signer } from "@reef-defi/evm-provider";
import { WsProvider } from "@polkadot/api";
import { ethers } from "ethers";
import { abi } from "./utils/contract/flipper.json";
import { contractAddress } from "./utils/contract/address";

export default {
  name: "App",
  components: {},
  data() {
    return {
      selectedAccount: null,
      isWalletConnected: false,
      error: null,
      provider: null,
      signer: null,
      contract: null,
      arguments: null,
    };
  },
  async created() {
    try {
      const provider = new Provider({
        provider: new WsProvider("ws://rpc-testnet.reefscan.info/ws"),
      });
      if (provider) {
        await provider.api.isReady;
        this.provider = provider;
      }
    } catch (error) {
      console.log(error);
    }
  },
  methods: {
    async initializeDapp() {
      await this.getReefAccounts();
      if (this.accounts.length > 0) {
        this.isWalletConnected = true;
        this.selectedAccount = this.accounts[0];
      }
    },
    async getReefAccounts() {
      await web3Enable("reef");
      this.accounts = await web3Accounts();
    },
    async getFlipperState() {
      this.contract = new ethers.Contract(contractAddress, abi, this.provider);
      const val = await this.contract.getFlipperState();
      console.log(val);
    },
    async getSigner() {
      await web3FromAddress(this.selectedAccount.address).then(
        async (injector) => {
          this.signer = new Signer(
            this.provider,
            this.selectedAccount.address,
            injector.signer
          );
          this.contract = new ethers.Contract(
            contractAddress,
            abi,
            this.signer
          );
        }
      );
    },
    async flip() {
      this.getSigner();
      if (this.signer) {
        await this.contract.flip();
      }
    },
  },
};
</script>

<style scoped>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
