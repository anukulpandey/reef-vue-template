<template>
  <div v-if="isWalletConnected && selectedAccount">
    <div>Selected Address {{ selectedAccount.address }}</div>
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
import { WsProvider } from "@polkadot/api";
import { Provider, Signer } from "@reef-defi/evm-provider";
import { network } from "../dapp.config";

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
    };
  },
  async created() {
    await this.initializeDapp();
  },
  methods: {
    async initializeDapp() {
      await this.getReefAccounts();
      if (this.accounts.length > 0) {
        this.isWalletConnected = true;
        this.selectedAccount = this.accounts[0];
        this.provider = await this.getProvider();
        if (this.provider && this.selectedAccount) {
          await this.getSigner(this.provider);
        }
      }
    },
    async getReefAccounts() {
      await web3Enable("reef");
      this.accounts = await web3Accounts();
    },
    async getProvider() {
      const allInjected = await web3Enable("Reef");
      if (allInjected.length == 0) {
        this.error = "Extension not installed";
        return;
      }
      const ReefProvider = new Provider({
        provider: new WsProvider(network.wsUrlMainnet),
      });
      return ReefProvider;
    },
    async getSigner(provider) {
      provider.api.on("ready", () => {
        const injector = web3FromAddress(this.selectedAccount.address);
        this.signer = new Signer(
          provider,
          this.selectedAccount.address,
          injector.signer
        );
      });
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
