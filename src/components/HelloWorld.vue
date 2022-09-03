<template>
  <div class="background" />
  <h1 id="cssTest" class="topTitle">
    DEPLOY & RUN TEST
  </h1>
  <div v-if="isInstallMetaMask" style="font-size: 16px; text-align: end; padding: 1rem; background: #606060; border-radius: 1rem">메타 마스크 설치 됨
    <div v-if="!currentConnectedAddress" style="font-size: 16px; text-align: end; padding-right: 1rem">
      <button v-on:click="connectWithMetaMask">메타마스크 연결</button>
    </div>
    <section v-if="currentConnectedAddress" style="font-size: 16px; text-align: end; margin-top: 1rem">
      <div> 현재계정 주소: <strong>{{ currentConnectedAddress }}</strong>
        <!--button with contract connection-->
        <div v-if="!connectedContract">
          <button class="contractBtn" v-on:click="connectContract">컨트랙트 연결</button>
        </div>
        <div v-else>
          <div style="margin-top: 64px">
            <div class="function">
              <div class="title">ownerOf</div>
              <div class="parameters">
                <span>tokenId</span>
                <input class="input" type="text" placeholder="tokenId" v-model="ownerOfTokenId" />
                <button class="button" v-on:click="ownerOf">Call</button>
              </div>
              <div class="result" v-if="ownerOfResult.length > 0">
                ownerOf: {{ ownerOfResult }}
              </div>
            </div>
            <div class="function" style="margin-top: 24px">
              <div class="title">balanceOf</div>
              <div class="parameters">
                <span>ownerAddress</span>
                <input class="input" type="text" placeholder="tokenId" v-model="ownerAddress" />
                <button class="button" v-on:click="balanceOf">Call</button>
              </div>
              <div class="result" v-if="ownerOfResult.length > 0">
                balanceOf: {{ balance }}
              </div>
            </div>
            <div class="function" style="margin-top: 24px">
              <div class="title">tokenURI</div>
              <div class="parameters">
                <span>tokenId</span>
                <input class="input" type="text" placeholder="tokenId" v-model="tokenId" />
                <button class="button" v-on:click="tokenURI">Call</button>
              </div>
              <div class="result" v-if="ownerOfResult.length > 0">
                tokenURI: {{ currentTokenId }}
              </div>
            </div>
            <div class="function" style="margin: 24px 0;">
              <div class="title">Mint</div>
              <div style="display: flex; flex-direction: column; gap: 16px">
                <div class="parameters">
                  <span>Target Address</span>
                  <input class="input" type="text" placeholder="target address" v-model="targetAddress" />
                </div>
                <div class="parameters">
                  <span>TokenID</span>
                  <input class="input" type="text" placeholder="target address" v-model="mintTokenId" />
                </div>
                <div class="parameters">
                  <span>IPFS URI</span>
                  <input class="input" type="text" placeholder="target address" v-model="mintURI" />
                  <button class="button" v-on:click="mint">Call</button>
                </div>
              </div>
              <div class="result" v-if="mintResult">
                민팅결과: {{ mintResult }}
              </div>
            </div>
          </div>
        </div>
      </div>
    </section>
  </div>
  <div v-else>메타 마스크 설치 안됨, 설치 해주세요</div>
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

      //mint 변수
      targetAddress: "",
      mintTokenId: null,
      mintURI: "",
      mintResult: null,

      // event 감지
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
    },

    async mint() {
      try {
        const transaction = await this.connectedContract.mint(this.targetAddress, this.mintTokenId, this.mintURI);
        this.mintResult = "생성 대기중";
        await transaction.wait();
        this.mintResult = transaction;
        console.log(transaction);
      } catch(e) {
        console.log(e.response?.data);
      }
    },

    async eventTransfer () {
      try {
        const eventResult = await this.connectedContract.on("Transfer");
        console.log(eventResult);
      } catch (e) {
        console.log(e.response.data);
      }
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Fira+Code:wght@300&display=swap');

.function {
  background-color: #2c3e50;
  min-height: 200px;
  height: fit-content;
  border-radius: 1rem;
  border-bottom: 3px;
}

.contractBtn {
  border: none;
  border-radius: 1rem;
  width: 258px;
  height: 64px;
  margin-top: 64px;
  color: white;
  background: coral;
  opacity: 0.6;
  transition: all ease-out 0.14s;
  cursor: pointer;
}

.contractBtn:hover {
  opacity: 1;
  box-shadow: 4px 4px blanchedalmond;
  transform: translate(-0.5rem, -0.5rem);
}

.title {
  text-align: left;
  color: #ffffff;
  position: relative;
  margin: 16px;
  padding-top: 16px;
  font-weight: bold;
}

.parameters {
  margin: 20px 0;
  color: white;
  display: flex;
  justify-content: flex-start;
  gap: 16px;
  align-items: center;
  padding: 0 16px;
}

.input {
  flex: 1;
  background: #303030;
  border: 1px solid white;
  border-radius: 1rem;
  height: 24px;
  padding: 1rem;
  transition: all ease-out 0.14s;
  color: white;
}

.input:focus {
  box-shadow: 2px 2px 2px white;
  outline: none;
}

.button {
  height: 100%;
  border: none;
  background: #ff0000;
  opacity: 0.7;
  color: white;
  font-size: 24px;
  padding: 16px 24px;
  border-radius: 1rem;
}

.button:hover {
  opacity: 1;
}

.result {
  color: white;
  text-align: right;
  padding: 0 1rem;
  font-size: 2rem;
}
</style>
