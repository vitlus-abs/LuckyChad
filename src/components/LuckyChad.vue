<template>
  <div class="lottery-container">
    <img src="@/assets/logo.svg" alt="Lucky Chad Logo" class="logo" />
    <h1>Lucky Chad Lottery</h1>
    <p>Entry Fee: {{ entryFee }} ETH</p>
    <button @click="enterLottery">Enter Lottery</button>
    <h2>Players:</h2>
    <ul>
      <li v-for="(player, index) in players" :key="index">{{ player }}</li>
    </ul>
    <p>{{ message }}</p>
  </div>
</template>

<script>
import { ethers } from 'ethers';

export default {
  data() {
    return {
      players: [],
      entryFee: '',
      message: '',
      contractAddress: '0x8e87FE1B75bF8f1d8808323cA5A32F152491C5A1', // Replace with your contract address
      contractABI: [
        {
          inputs: [],
          stateMutability: "nonpayable",
          type: "constructor",
        },
        {
          inputs: [],
          name: "enter",
          outputs: [],
          stateMutability: "payable",
          type: "function",
        },
        {
          inputs: [],
          name: "entryFee",
          outputs: [
            {
              internalType: "uint256",
              name: "",
              type: "uint256",
            },
          ],
          stateMutability: "view",
          type: "function",
        },
        {
          inputs: [],
          name: "getOwner",
          outputs: [
            {
              internalType: "address",
              name: "",
              type: "address",
            },
          ],
          stateMutability: "view",
          type: "function",
        },
        {
          inputs: [],
          name: "getPlayers",
          outputs: [
            {
              internalType: "address[]",
              name: "",
              type: "address[]",
            },
          ],
          stateMutability: "view",
          type: "function",
        },
        {
          inputs: [],
          name: "owner",
          outputs: [
            {
              internalType: "address",
              name: "",
              type: "address",
            },
          ],
          stateMutability: "view",
          type: "function",
        },
        {
          inputs: [],
          name: "pickWinner",
          outputs: [],
          stateMutability: "nonpayable",
          type: "function",
        },
        {
          inputs: [
            {
              internalType: "uint256",
              name: "",
              type: "uint256",
            },
          ],
          name: "players",
          outputs: [
            {
              internalType: "address",
              name: "",
              type: "address",
            },
          ],
          stateMutability: "view",
          type: "function",
        },
      ],
    };
  },
  mounted() {
    this.init();
  },
  methods: {
    async init() {
      if (typeof window.ethereum !== 'undefined') {
        try {
          await window.ethereum.request({ method: 'eth_requestAccounts' });
          const provider = new ethers.BrowserProvider(window.ethereum); // Use BrowserProvider for ethers v6
          const signer = await provider.getSigner();
          const contract = new ethers.Contract(this.contractAddress, this.contractABI, signer);

          // Fetch entry fee and players
          const fee = await contract.entryFee();
          this.entryFee = ethers.formatEther(fee); // Updated for ethers v6

          const playersArray = await contract.getPlayers();
          this.players = playersArray;
        } catch (error) {
          console.error('Error connecting to contract:', error);
          this.message = 'Error connecting to contract';
        }
      } else {
        console.log('Ethereum provider not found');
        this.message = 'Ethereum provider not found';
      }
    },
    async enterLottery() {
      if (typeof window.ethereum !== 'undefined') {
        try {
          const provider = new ethers.BrowserProvider(window.ethereum);
          const signer = await provider.getSigner();
          const contract = new ethers.Contract(this.contractAddress, this.contractABI, signer);

          const tx = await contract.enter({ value: ethers.parseEther(this.entryFee) }); // Updated for ethers v6
          await tx.wait();
          this.message = 'Entered the lottery!';
        } catch (error) {
          console.error('Error entering lottery:', error);
          this.message = 'Failed to enter the lottery';
        }
      }
    },
    async pickWinner() {
      if (typeof window.ethereum !== 'undefined') {
        try {
          const provider = new ethers.BrowserProvider(window.ethereum);
          const signer = await provider.getSigner();
          const contract = new ethers.Contract(this.contractAddress, this.contractABI, signer);

          const tx = await contract.pickWinner();
          await tx.wait();
          this.message = 'Winner picked successfully!';
        } catch (error) {
          console.error('Error picking winner:', error);
          this.message = 'Failed to pick winner';
        }
      }
    }
  },
};
</script>

<style scoped>
@font-face {
  font-family: 'Avenue Mono';
  src: url('@/assets/fonts/Avenue Mono.ttf') format('truetype');
  font-weight: normal;
  font-style: normal;
}

.lottery-container {
  text-align: center;
  padding: 20px;
  background-image: url('@/assets/background-tile.png'); /* Ensure this path is correct */
  background-repeat: repeat; /* Ensures the background is tiled */
  background-size: contain; /* Adjust size of tile if needed */
  min-height: 100vh; /* Ensures the background covers the full viewport */
  font-family: 'Avenue Mono', monospace;
}

.logo {
  width: 150px;
  height: auto;
  margin-bottom: 20px;
}

h1 {
  font-size: 28px;
  color: #333;
}

button {
  background-color: #4CAF50;
  color: white;
  border: none;
  padding: 10px 20px;
  font-size: 16px;
  cursor: pointer;
  border-radius: 5px;
}

button:hover {
  background-color: #45a049;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  font-size: 18px;
  margin: 5px 0;
}

p {
  font-size: 16px;
  color: #555;
}
</style>
