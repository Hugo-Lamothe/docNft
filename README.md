# NFT

## Créer un wallet

### Bitcoin:

```node
const bitcoin = require('bitcoinjs-lib');

// Créez une clé privée Bitcoin aléatoire
const keyPair = bitcoin.ECPair.makeRandom();

// Obtenir l'adresse Bitcoin associée à la clé privée
const address = keyPair.getAddress();

// Obtenir la clé privée WIF (Wallet Import Format)
const privateKeyWIF = keyPair.toWIF();

console.log('Adresse Bitcoin:', address);
console.log('Clé privée WIF:', privateKeyWIF);
```
ou

```node
var CoinKey = require('coinkey'); 

var wallet = new CoinKey.createRandom();

console.log("SAVE BUT DO NOT SHARE THIS:", wallet.privateKey.toString('hex'));
console.log("Address:", wallet.publicAddress);
```


### Ethereum:

```node
const { ethers } = require('ethers');

// Créez un portefeuille Ethereum
const wallet = ethers.Wallet.createRandom();

// Obtenir l'adresse Ethereum associée au portefeuille
const address = wallet.address;

// Obtenir la clé privée du portefeuille
const privateKey = wallet.privateKey;

// Obtenir la clé publique du portefeuille
const publicKey = wallet.publicKey;

console.log('Adresse Ethereum:', address);
console.log('Clé privée:', privateKey);
console.log('Clé publique:', publicKey);
```

## Créer des nfts

### Immutable X:

https://docs.immutable.com/docs/x/zero-to-hero-nft-minting/

### NodeJS:

```node
const { ethers } = require('ethers');
const crypto = require('crypto');

// Configurez votre fournisseur Ethereum (infura.io est utilisé ici)
const provider = new ethers.providers.JsonRpcProvider('https://rinkeby.infura.io/v3/INFURA_PROJECT_ID');

// Votre clé privée pour le portefeuille Ethereum
const privateKey = 'YOUR_PRIVATE_KEY';

// Adresse de votre contrat ERC-721
const contractAddress = 'CONTRACT_ADDRESS';

// ABI (Application Binary Interface) du contrat ERC-721
const contractABI = [
    // ... Ajoutez ici l'ABI de votre contrat ERC-721 ...
];

async function createAndStoreNFT() {
    // Connecter un portefeuille en utilisant la clé privée
    const wallet = new ethers.Wallet(privateKey, provider);

    // Connectez-vous au contrat ERC-721 à l'aide de l'adresse et de l'ABI
    const contract = new ethers.Contract(contractAddress, contractABI, wallet);

    // Générez un nouvel identifiant unique pour le certificat NFT
    const tokenId = crypto.randomBytes(32).toString('hex');

    // Déployez le certificat NFT en associant le tokenId à l'adresse du portefeuille
    const tx = await contract.mintNFT(wallet.address, tokenId);

    // Attendez la confirmation de la transaction
    await tx.wait();

    console.log('Le certificat NFT a été créé avec succès !');
    console.log('Token ID:', tokenId);
}

createAndStoreNFT().catch((error) => {
    console.error('Erreur:', error);
});
```

ligne 72 `YOUR_PRIVATE_KEY` est la clé privée de votre portefeuille\
ligne 75 `CONTRACT_ADDRESS` est l'adresse de votre contrat ERC-721

étape du code ci-dessus:

    1-Connectez-vous à la blockchain Ethereum en utilisant le fournisseur fourni.
    2-Déployez un contrat ERC-721 sur la blockchain (vous devez remplacer contractABI avec l'ABI réel de votre contrat ERC-721).
    3-Générez un nouvel ID unique pour le NFT.
    4-Émettez un NFT en associant le nouvel ID à l'adresse du portefeuille créé.
    5-Affichez l'ID du NFT créé avec succès.

#### Contrat ERC_721

Dans le code ci-dessus nous avons besoin de l'adresse du contrat ERC-721 et on en aura besoin 
dans du code plus bas aussi. Dans un premier temps il faut écrire un contrat qui respecte la norme
ERC-721, il est conseillé de l'écrire en solidity (langage de programmation de contrat intelligent d'Ethereum).\
Voici un exemple simple de contrat ERC_721:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract MyNFT is ERC721, Ownable {
    constructor(string memory name, string memory symbol) ERC721(name, symbol) {}

    function mint(address to, uint256 tokenId) public onlyOwner {
        _mint(to, tokenId);
    }
}
```

il faut ensuite déployer le contrat et récupérer son ABI voici le code NodeJS qui permet cela:

```node
const { ethers } = require('ethers');
const fs = require('fs');

// Chargez le code source du contrat
const contractSource = fs.readFileSync('MyNFT.sol', 'utf8');

async function deployContract() {
    const provider = new ethers.providers.JsonRpcProvider('YOUR_RPC_URL'); // Remplacez par votre fournisseur RPC
    const wallet = new ethers.Wallet('YOUR_PRIVATE_KEY', provider); // Remplacez par votre clé privée

    // Compilez le contrat
    const contractFactory = new ethers.ContractFactory(contractSource, ['MyNFT'], wallet);
    const contract = await contractFactory.deploy('MyNFT', 'NFT');

    await contract.deployed();

    console.log('Contract deployed at:', contract.address);

    // Enregistrez l'ABI dans un fichier
    fs.writeFileSync('MyNFT.json', JSON.stringify(contract.interface.abi, null, 2));
}

deployContract().catch(error => {
    console.error('Error:', error);
});
```

ligne 148 `MyNFT.sol` est le nom du fichier de création du contrat\
ligne 155 et 156 `MyNFT` est le nom du contrat dans le fichier .sol\
ligne 151 `YOUR_RPC_URL` est l'url de votre fournisseur Etherum RPC\
ligne 152 `YOUR_PRIVATE_KEY` est la clé privée de votre portefeuille\

## Transférer des nfts

### NodeJS:

```node
const { ethers } = require('ethers');

// Configurez votre fournisseur Ethereum (infura.io est utilisé ici)
const provider = new ethers.providers.JsonRpcProvider('https://rinkeby.infura.io/v3/INFURA_PROJECT_ID');

// Clé privée du portefeuille expéditeur
const senderPrivateKey = 'SENDER_PRIVATE_KEY';

// Adresse du portefeuille expéditeur
const senderAddress = 'SENDER_ADDRESS';

// Adresse du portefeuille destinataire
const recipientAddress = 'RECIPIENT_ADDRESS';

// Adresse du contrat ERC-721
const contractAddress = 'CONTRACT_ADDRESS';

// ABI (Application Binary Interface) du contrat ERC-721
const contractABI = [
    // ... Ajoutez ici l'ABI de votre contrat ERC-721 ...
];

async function transferNFT() {
    // Connectez-vous au portefeuille expéditeur en utilisant la clé privée
    const senderWallet = new ethers.Wallet(senderPrivateKey, provider);

    // Connectez-vous au contrat ERC-721 à l'aide de l'adresse et de l'ABI
    const contract = new ethers.Contract(contractAddress, contractABI, senderWallet);

    // ID du NFT à transférer
    const tokenId = 'TOKEN_ID';

    // Transférez le NFT au portefeuille destinataire
    const tx = await contract.transferFrom(senderAddress, recipientAddress, tokenId);

    // Attendez la confirmation de la transaction
    await tx.wait();

    console.log('NFT transféré avec succès au portefeuille destinataire');
}

transferNFT().catch((error) => {
    console.error('Erreur:', error);
});
```

ligne 187 `SENDER_PRIVATE_KEY` est la clé privée du portefeuille expéditeur\
ligne 190 `SENDER_ADDRESS` est l'adresse du portefeuille expéditeur\
ligne 193 `RECIPIENT_ADDRESS` est l'adresse du portefeuille destinataire\
ligne 196 `CONTRACT_ADDRESS` est l'adresse du contrat ERC-721\
ligne 211 `TOKEN_ID` est l'ID du NFT que vous souhaitez transférer
