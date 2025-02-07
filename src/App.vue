<template>
  <div id="app">
    <div>Ethereum Account: {{ account }}</div>
    <div>Chain Id: {{ chainId }}</div>
    <div>Ripple Account: {{ rippleAccount }}</div>
    <div>Tron Account: {{ tronAccount }}</div>
    <div>Eos Account: {{ eosAccount }}</div>
    <div>
      <button @click="connect">Connect</button>
    </div>
    <div style="margin-top: 10px;">
      <button @click="switchPOLYGON">Switch to POLYGON</button>
    </div>
    <div style="margin-top: 10px;">
      <input type="text" v-model="nonce" placeholder="Nonce" />
    </div>
    <div style="margin-top: 10px;">
      <input type="text" v-model="maxFeePerGas" placeholder="MaxFeePerGas"/>
      <input type="text" v-model="maxPriorityFeePerGas" placeholder="MaxPriorityFeePerGas" style="margin-left: 4px;"/>
      <button @click="sendEIP1559Transaction" style="margin-left: 4px;">Send EIP1559 Transaction</button>
    </div>
    <div style="margin-top: 10px;">
      <input type="text" v-model="gasPrice" placeholder="GasPrice" />
      <button @click="sendLegacyTransaction" style="margin-left: 4px;">Send Legacy Transaction</button>
    </div>

    <!-- <button @click="connectRipple">Connect Ripple</button>
    <button @click="sendXRP">Send XRP</button>

    <button @click="connectTron">Connect Tron</button>
    <button @click="sendTrx">Send Trx</button>
    <button @click="sendTronUsdt">Send Tron USDT</button>


    <button @click="connectEos">Connect EOS</button>
    <button @click="sendEos">Send EOS</button> -->
  </div>
</template>

<script>
const { stringToBytes, bytesToHex, remove0x } = require("@metamask/utils");
// import axios from "axios";

const ethereum = window.ethereum;
const ChainWebToken = window.ChainWebToken;
console.log(ChainWebToken);


// const fetch = async (cwt) => {
//   const config = {
//     method: "post",
//     url: "http://192.168.66.254:50500/",
//     headers: {
//       cwt_auth: cwt
//     },
//     data: {
//       method: "server_info",
//       params: [{}]
//     }
//   };
//   const res = await axios(config);
//   return res;
// };

export default {
  name: 'App',
  data() {
    return {
      account: "",
      maxFeePerGas: "",
      maxPriorityFeePerGas: "",
      gasPrice: "",
      nonce: "",
      rippleAccount: "",
      tronAccount: "",
      eosAccount: "",
      chainId: "0x1"
    }
  },
  async mounted() {
    await this.connect();
    ethereum.on('chainChanged', (chainId) => {
      console.log('chainId: ', chainId)
      this.chainId = chainId;
    });

    // const [usr, webToken] = ["jingtum_secp256k1", new ChainWebToken.JingtumWebToken("sajoigynKobrB8U59g1puxu8GM7Hg")];
    // const payload = {
    //   usr
    // };
    // const cwt = webToken.sign(payload);
    // try {
    //   await fetch(cwt);
    //   console.log((`auth success on ${usr}`));
    // } catch (error) {
    //   console.log(error);
    // }
  },
  methods: {
    async connectRipple() {
      try {
        const accounts = await ethereum.request({ method: 'ripple_requestAccounts' });
        console.log('accounts: ', accounts)
        this.rippleAccount = accounts[0]
      } catch (error) {
        console.log('error: ', error)
      }
    },
    async connectTron() {
      try {
        const accounts = await ethereum.request({ method: 'tron_requestAccounts' });
        console.log('accounts: ', accounts)
        this.tronAccount = accounts[0]
      } catch (error) {
        console.log('error: ', error)
      }
    },
    async sendEIP1559Transaction() {
      const data = {
        from: this.account,
        to: "0xdf4fb256872d416d7949598d3043c58f747dbd7c",
        value: "0x0"
      }
      if (this.maxFeePerGas) {
        data.maxFeePerGas = Number(this.maxFeePerGas * 1e9).toString(16);
      }
      if (this.maxPriorityFeePerGas) {
        data.maxPriorityFeePerGas = Number(this.maxPriorityFeePerGas * 1e9).toString(16);
      }
      if (this.nonce) {
        data.nonce = Number(this.nonce).toString(16);
      }
      const hash = await ethereum.request({
        method: 'eth_sendTransaction',
        params: [data]
      });
      console.log('eip1559 hash: ', hash)
    },
    async sendLegacyTransaction() {
      const data = {
        from: this.account,
        to: "0xdf4fb256872d416d7949598d3043c58f747dbd7c",
        value: "0x0",
        type: "0x0"
      }
      if (this.gasPrice) {
        data.gasPrice = Number(this.gasPrice * 1e9).toString(16);
      }
      if (this.nonce) {
        data.nonce = Number(this.nonce).toString(16);
      }
      const hash = await ethereum.request({
        method: 'eth_sendTransaction',
        params: [data]
      });
      console.log('legacy hash: ', hash)
    },
    async connectEos() {
      try {
        const accounts = await ethereum.request({ method: 'eos_requestAccounts' });
        console.log('accounts: ', accounts)
        this.eosAccount = accounts[0]
      } catch (error) {
        console.log('error: ', error)
      }
    },
    async sendXRP() {
      try {
        const tx = await ethereum.request({
          method: 'ripple_sendTransaction',
          params: [{
            "TransactionType": "Payment",
            "Account": this.rippleAccount,
            "Destination": "raXkvLAoxuqQPTPcrfR9ecB92gM5HE9bH9",
            "Amount": "10",
            Memos: [{
              Memo: {
                MemoData: remove0x(bytesToHex(stringToBytes(JSON.stringify({
                  jtaddress: "j4rmEZiaTdXBkgzXPdsu1JRBf5onngqfUi"
                }))))
              }
            }]
          }]
        });
        console.log('tx: ', tx)
      } catch (error) {
        console.log('error: ', error)
      }
    },
    async sendEos() {
      try {
        const tx = await ethereum.request({
          method: 'eos_sendTransaction',
          params: [{
            transactionType: "Transfer",
            from: this.eosAccount,
            to: "luodaliluoli",
            code: "eosio.token",
            quantity: "0.0001 EOS",
            memo: "test"
          }]
        });
        console.log('tx: ', tx)
      } catch (error) {
        console.log('error: ', error)
      }
    },
    async sendTrx() {
      try {
        const tx = await ethereum.request({
          method: 'tron_sendTransaction',
          params: [{
            transactionType: "SendTrx",
            from: this.tronAccount,
            to: "TGB1LwXzNPQhHPacGMjseZbqcwjktZtp5N",
            value: "0.0001",
            data: "test"
          }]
        });
        console.log('tx: ', tx)
      } catch (error) {
        console.log('error: ', error)
      }
    },
    async sendTronUsdt() {
      try {
        const tx = await ethereum.request({
          method: 'tron_sendTransaction',
          params: [{
            transactionType: "SendToken",
            from: this.tronAccount,
            to: "TGB1LwXzNPQhHPacGMjseZbqcwjktZtp5N",
            value: "0.0001",
            contract: "TR7NHqjeKQxGTCi8q8ZY4pL8otSzgjLj6t",
            data: JSON.stringify({
              jtaddress: "j4rmEZiaTdXBkgzXPdsu1JRBf5onngqfUi"
            })
          }]
        });
        console.log('tx: ', tx)
      } catch (error) {
        console.log('error: ', error)
      }
    },
    async connect() {
      console.log('connect')
      try {
        const accounts = await ethereum.request({ method: 'eth_requestAccounts' });
        console.log('accounts: ', accounts)
        this.account = accounts[0]
        const chainId = await ethereum.request({ method: 'eth_chainId' });
        console.log('chainId: ', chainId)
        this.chainId = chainId;
      } catch (error) {
        console.log('error: ', error)
      }
    },
    async switchPOLYGON() {
      try {
        const connected = ethereum.isConnected();
        console.log("connected: ", connected);
        const chainId = "0x89";
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
