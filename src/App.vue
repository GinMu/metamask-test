<template>
  <div id="app">
    <div>Account: {{ account }}</div>
    <div>Chain Id: {{ chainId }}</div>
    <button @click="connect">Connect</button>
    <button @click="switchChain">Switch chain</button>
  </div>
</template>

<script>

const ethereum = window.ethereum;
export default {
  name: 'App',
  data() {
    return {
      account: "",
      chainId: "0x1"
    }
  },
  async mounted() {
    // this.chainId = ethereum.chainId;
    await this.connect();
  },
  methods: {
    async connect() {
      try {
        const accounts = await ethereum.request({ method: 'eth_requestAccounts' });
        console.log('accounts: ', accounts)
        this.account = accounts[0]
      } catch (error) {
        console.log('error: ', error)
      }
    },
    async switchChain() {
      try {
        const connected = ethereum.isConnected();
        console.log("connected: ", connected);
        const chainId = this.chainId === "0x1" ? "0x38" : "0x1";
        await ethereum.request({
          method: 'wallet_switchEthereumChain',
          params: [{ chainId }],
        })
        const blockNumber = await ethereum.request({
          method: "eth_blockNumber",
          params: []
        });
        console.log("block number: ", blockNumber);
        this.chainId = chainId;
      } catch (error) {
        console.log("switch error: ", error);
      }
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
