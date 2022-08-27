<template>
  <div class="hello">
    <div v-if="isInstallMetaMask">메타 마스크 설치 됨
      <div v-if="!currentConnectedAddress">
        <button v-on:click="connectWithMetaMask">메타마스크 연결</button>
      </div>
      <section v-if="currentConnectedAddress">
        <div> 현재계정 주소: {{ currentConnectedAddress }}
          <!--button with contract connection-->
          <div v-if="!connectedContract">
            <button v-on:click="connectContract">컨트랙트 연결</button>
          </div>
          <div v-else>
            <div style="margin-top: 64px">
              <div>
                <button v-on:click="ownerOf">ownerOf</button>
                <input type="text" placeholder="tokenId" v-model="ownerOfTokenId" />
                <div v-if="ownerOfResult.length > 0">
                  ownerOf: {{ ownerOfResult }}
                </div>
              </div>
              <div style="margin-top: 24px">
                <button v-on:click="balanceOf">balanceOf</button>
                <input type="text" placeholder="owner's address" v-model="ownerAddress" />
                <div v-if="balance > 0">
                  balanceOf: {{ balance }}
                </div>
              </div>
              <div style="margin-top: 24px">
                <button v-on:click="tokenURI">tokenURI</button>
                <input type="text" placeholder="tokenId" v-model="tokenId" />
                <div v-if="currentTokenId.length > 0">
                  tokenURI: {{ currentTokenId }}
                </div>
              </div>
            </div>
          </div>
        </div>
      </section>
    </div>
    <div v-else>메타 마스크 설치 안됨, 설치 해주세요</div>
  </div>
</template>

<script>
import { ethers } from "ethers";
import abi from "../assets/abi.json";

export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  data() {
    return {
      isInstallMetaMask : window.ethereum !== null && window.ethereum.isMetaMask,
      currentConnectedAddress: document.cookie ? document.cookie.split("; ")[0].split("=")[1] : null,
      connectedContract: null,

      //ownerof에 필요한 변수
      ownerOfTokenId: null,
      ownerOfResult: "",
      //balanceof 변수
      ownerAddress: "",
      balance: -1,
      //tokenURI 변수
      tokenId: null,
      currentTokenId: "",
    }
  },
  methods: {
    async connectWithMetaMask() {
      try {
        const account = await window.ethereum.request({ method: "eth_requestAccounts" });
        this.currentConnectedAddress = account[0];
        document.cookie=`currentConnectedAddress=${account[0]}`;
        console.log(this.currentConnectedAddress);
      } catch (e) {
        console.log(e.response?.data);
      }
    },

    async connectContract() {
      const contractAddress = "0xF649adbe0eA31d11E17214Ad6Ad721B72F5d4822";
      const provider = new ethers.providers.Web3Provider(window.ethereum);
      const tempSigner = provider.getSigner();
      const connectedContract = new ethers.Contract(contractAddress, abi, tempSigner);
      console.log(connectedContract);
      this.connectedContract = connectedContract;
    },

    async ownerOf() {
      const ownerAddress = await this.connectedContract.ownerOf(Number(this.ownerOfTokenId));
      console.log(ownerAddress);
      this.ownerOfResult = ownerAddress;
    },

    async balanceOf() {
      const balanceVal = await this.connectedContract.balanceOf(this.ownerAddress);
      this.balance = balanceVal;
    },

    async tokenURI() {
      const ipfsTokenURI = await this.connectedContract.tokenURI(this.tokenId);
      this.currentTokenId = ipfsTokenURI;
    }
  }
}
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
</style>
