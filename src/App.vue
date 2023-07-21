<template>
  <div v-if="isWalletConnected && selectedAccount">
    <div>connected as {{ selectedAccount.address }}</div>
    <br />
    <div>
      {{
        flipperState === null
          ? "wait few seconds and click on refresh"
          : flipperState
      }}
    </div>
    <button @click="getFlipperState">refresh</button>

    <button @click="flip">flip state of flipper</button>
  </div>
  <button v-else @click="init">Connect Wallet</button>
  <p>{{ error }}</p>
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
      flipperState: null,
    };
  },
  async created() {
    await this.init();
  },
  methods: {
    async init() {
      await web3Enable("reef");
      this.accounts = await web3Accounts();
      if (this.accounts.length > 0) {
        this.isWalletConnected = true;
        this.selectedAccount = this.accounts[0];
        try {
          const provider = new Provider({
            provider: new WsProvider("wss://rpc-testnet.reefscan.com/ws"),
          });
          await provider.api.isReadyOrError;
          provider.api.on("disconnected", () => {
            console.log("disconnected");
          });
          if (provider) {
            await provider.api.isReady;
            this.provider = provider;
          }
        } catch (error) {
          console.log(error);
        }
        if (this.provider) {
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
        }
      }
    },
    async getFlipperState() {
      if (this.provider) {
        this.contract = new ethers.Contract(
          contractAddress,
          abi,
          this.provider
        );
        const val = await this.contract.getFlipperState();
        this.flipperState = val;
        await this.getSigner();
      } else {
        this.error = "unable to connect to web socket";
      }
    },
    async getSigner() {
      if (!this.provider) {
        this.error = "unable to connect to web socket";
        return;
      }
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
      try {
        this.getSigner();
        if (this.signer) {
          await this.contract.flip();
          this.error = 'tx success : kindly refresh';
        } else {
          this.error = "unable to connect to web socket, please refresh page";
        }
        await this.flipperState();
      } catch ( error ) {
        if (error.message == "_canceled") {
          this.error = "You cancelled the transaction";
        }
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
