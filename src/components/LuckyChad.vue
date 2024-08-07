<template>
  <div class="lottery-container">
    <img src="@/assets/logo.svg" alt="Lucky Chad Logo" class="logo" />
    <h1>Lucky Chad Lottery</h1>
    <p>Entry Fee: {{ entryFee }} ETH</p>
    <p>Total Pot Balance: {{ contractBalance }} ETH</p>
    <button @click="enterLottery" :disabled="!isConnected">Enter Lottery</button>
    <p>{{ message }}</p>
    <h2>Players:</h2>
    <ul>
      <li v-for="(player, index) in players" :key="index">{{ player }}</li>
    </ul>
  </div>
</template>

<script>
import { ethers } from 'ethers';

export default {
  data() {
    return {
      players: [],
      entryFee: '',
      contractBalance: '',
      message: '',
      contractAddress: '0x8e87FE1B75bF8f1d8808323cA5A32F152491C5A1',
      contractABI: [
        {
          inputs: [],
          stateMutability: 'nonpayable',
          type: 'constructor',
        },
        {
          inputs: [],
          name: 'enter',
          outputs: [],
          stateMutability: 'payable',
          type: 'function',
        },
        {
          inputs: [],
          name: 'entryFee',
          outputs: [
            {
              internalType: 'uint256',
              name: '',
              type: 'uint256',
            },
          ],
          stateMutability: 'view',
          type: 'function',
        },
        {
          inputs: [],
          name: 'getOwner',
          outputs: [
            {
              internalType: 'address',
              name: '',
              type: 'address',
            },
          ],
          stateMutability: 'view',
          type: 'function',
        },
        {
          inputs: [],
          name: 'getPlayers',
          outputs: [
            {
              internalType: 'address[]',
              name: '',
              type: 'address[]',
            },
          ],
          stateMutability: 'view',
          type: 'function',
        },
        {
          inputs: [],
          name: 'owner',
          outputs: [
            {
              internalType: 'address',
              name: '',
              type: 'address',
            },
          ],
          stateMutability: 'view',
          type: 'function',
        },
        {
          inputs: [],
          name: 'pickWinner',
          outputs: [],
          stateMutability: 'nonpayable',
          type: 'function',
        },
        {
          inputs: [
            {
              internalType: 'uint256',
              name: '',
              type: 'uint256',
            },
          ],
          name: 'players',
          outputs: [
            {
              internalType: 'address',
              name: '',
              type: 'address',
            },
          ],
          stateMutability: 'view',
          type: 'function',
        },
      ],
      isConnected: false,
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
          const provider = new ethers.BrowserProvider(window.ethereum); 
          const network = await provider.getNetwork();

          // Abstract Testnet chain ID
          const abstractTestnetChainId = 11124n; 

          if (network.chainId !== abstractTestnetChainId) {
            await this.switchToAbstractTestnet();
          } else {
            this.isConnected = true;
            const signer = await provider.getSigner();
            const contract = new ethers.Contract(
              this.contractAddress,
              this.contractABI,
              signer
            );

            // Fetch entry fee and players
            const fee = await contract.entryFee();
            this.entryFee = ethers.formatEther(fee); 

            const playersArray = await contract.getPlayers();
            this.players = playersArray;

            // Fetch contract balance
            const balance = await provider.getBalance(this.contractAddress);
            this.contractBalance = ethers.formatEther(balance);
          }
        } catch (error) {
          console.error('Error connecting to contract:', error);
          this.message = 'Error connecting to contract';
        }
      } else {
        console.log('Ethereum provider not found');
        this.message = 'Ethereum provider not found';
      }
    },
    async switchToAbstractTestnet() {
      try {
        // Request MetaMask to switch to the Abstract Testnet
        await window.ethereum.request({
          method: 'wallet_switchEthereumChain',
          params: [{ chainId: '0x2B74' }], 
        });
        this.isConnected = true;
        this.message = 'Connected to Abstract Testnet!';
        this.init();
      } catch (switchError) {
        
        if (switchError.code === 4902) {
          try {
            await window.ethereum.request({
              method: 'wallet_addEthereumChain',
              params: [
                {
                  chainId: '0x2B74', 
                  chainName: 'Abstract Testnet',
                  rpcUrls: ['https://api.testnet.abs.xyz'], 
                  nativeCurrency: {
                    name: 'Abstract ETH',
                    symbol: 'ETH', 
                    decimals: 18,
                  },
                  blockExplorerUrls: [
                    'https://explorer.testnet.abs.xyz',
                  ], 
                },
              ],
            });
            this.isConnected = true;
            this.message = 'Abstract Testnet added and switched!';
            this.init();
          } catch (addError) {
            console.error('Failed to add Abstract Testnet:', addError);
            this.message = 'Not connected to Abstract Testnet';
          }
        } else {
          console.error('Failed to switch to Abstract Testnet:', switchError);
          this.message = 'Not connected to Abstract Testnet';
        }
      }
    },
    async enterLottery() {
      if (typeof window.ethereum !== 'undefined' && this.isConnected) {
        try {
          const provider = new ethers.BrowserProvider(window.ethereum);
          const signer = await provider.getSigner();
          const contract = new ethers.Contract(
            this.contractAddress,
            this.contractABI,
            signer
          );

          const tx = await contract.enter({
            value: ethers.parseEther(this.entryFee),
          }); 
          await tx.wait();
          this.message = 'Entered the lottery!';
          
          const balance = await provider.getBalance(this.contractAddress);
          this.contractBalance = ethers.formatEther(balance);
        } catch (error) {
          console.error('Error entering lottery:', error);
          this.message = 'Failed to enter the lottery';
        }
      }
    },
    async pickWinner() {
      if (typeof window.ethereum !== 'undefined' && this.isConnected) {
        try {
          const provider = new ethers.BrowserProvider(window.ethereum);
          const signer = await provider.getSigner();
          const contract = new ethers.Contract(
            this.contractAddress,
            this.contractABI,
            signer
          );

          const tx = await contract.pickWinner();
          await tx.wait();
          this.message = 'Winner picked successfully!';
          
          const balance = await provider.getBalance(this.contractAddress);
          this.contractBalance = ethers.formatEther(balance);
        } catch (error) {
          console.error('Error picking winner:', error);
          this.message = 'Failed to pick winner';
        }
      }
    },
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

button:disabled {
  background-color: #999;
  cursor: not-allowed;
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
