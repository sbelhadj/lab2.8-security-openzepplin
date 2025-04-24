### **Lab 8 : Utilisation d’OpenZeppelin pour Déployer des Contracts Sécurisés**

#### **Description du Lab**

Dans ce lab, vous apprendrez à sécuriser vos contrats solidity en utilisant les bibliothèques **OpenZeppelin**. Vous allez intégrer des composants comme **Ownable**, **Pausable**, **ReentrancyGuard**, et **SafeMath** pour sécuriser votre contrat **Instant Payment Hub** contre les vulnérabilités courantes et améliorer sa robustesse.

Les objectifs incluent :

*   La gestion des permissions avec **Ownable**.
*   L'ajout de la possibilité de **mettre en pause** le contrat avec **Pausable**.
*   La sécurisation des transactions avec **ReentrancyGuard** pour éviter les attaques par réentrance.
*   L'utilisation de **SafeMath** pour éviter les dépassements et sous-déversements lors des opérations mathématiques.  
     

* * *

### **Prérequis**

Avant de commencer ce lab, assurez-vous d'avoir configuré votre environnement comme suit :

*   **Node.js >= 16.x**
*   **Metamask** ou un autre wallet Ethereum installé dans votre navigateur
*   **Compte Ethereum sur Sepolia Testnet**
*   **GitHub Codespaces** ou un IDE local avec **Hardhat** et **solidity** installés
*   **Infura/Alchemy** pour interagir avec Sepolia Testnet  
     

* * *

### **Objectifs du Lab**

1.  Apprendre à sécuriser un smart contract en utilisant les bibliothèques **OpenZeppelin**.
2.  Ajouter des fonctionnalités de sécurité à un contrat **Instant Payment Hub** :  
     
    *   **Ownable** : Assurer que seules des adresses spécifiques peuvent effectuer certaines actions.
    *   **Pausable** : Mettre en pause le contrat en cas de problème.
    *   **ReentrancyGuard** : Sécuriser les fonctions sensibles contre les attaques par réentrance.
    *   **SafeMath** : Assurer que les opérations mathématiques sont effectuées de manière sûre.  
         
3.  Déployer et interagir avec le contrat sécurisé sur le testnet **Sepolia**.  
     

* * *

### **Étapes du Lab**

#### **1. Préparer l'environnement**  
 

**Installer les dépendances** en utilisant npm :  
  
```bash  
cd lab-oz-secure-contracts

npm install

```

**Vérifier la configuration de Hardhat** pour vous assurer que vous avez un contrat déployé sur Sepolia, comme mentionné dans le **Lab 3**. Si vous avez déjà déployé le contrat, récupérez son adresse.  
 

* * *

#### **2. Créer le contrat solidity avec OpenZeppelin**

1.  Créez un fichier contracts/InstantPaymentHubSecure.sol dans le répertoire **contracts/**.
2.  Implémentez un contrat de paiement instantané en utilisant **OpenZeppelin** :  
     

```solidity

// SPDX-License-Identifier: MIT

pragma solidity ^0.8.20;

import "@openzeppelin/contracts/access/Ownable.sol";

import "@openzeppelin/contracts/security/Pausable.sol";

import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

import "@openzeppelin/contracts/utils/math/SafeMath.sol";

contract InstantPaymentHubSecure is Ownable, Pausable, ReentrancyGuard {

    using SafeMath for uint256;

    mapping(address => uint256) public balances;

    event PaymentMade(address indexed sender, address indexed receiver, uint256 amount);

    modifier whenNotPaused() {

        require(!paused(), "Contract is paused");

        _;

    }

    constructor() {

        // Le propriétaire est l'adresse qui déploie le contrat

    }

    // Fonction pour déposer de l'ether dans le contrat

    function deposit() public payable whenNotPaused {

        balances[msg.sender] = balances[msg.sender].add(msg.value);

    }

    // Fonction pour effectuer un paiement instantané entre utilisateurs

    function instantPayment(address recipient, uint256 amount) public whenNotPaused nonReentrant {

        require(balances[msg.sender] >= amount, "Insufficient balance");

        balances[msg.sender] = balances[msg.sender].sub(amount);

        balances[recipient] = balances[recipient].add(amount);

        emit PaymentMade(msg.sender, recipient, amount);

    }

    // Fonction pour retirer des fonds du contrat

    function withdraw(uint256 amount) public whenNotPaused nonReentrant {

        require(balances[msg.sender] >= amount, "Insufficient balance");

        payable(msg.sender).transfer(amount);

        balances[msg.sender] = balances[msg.sender].sub(amount);

    }

    // Fonction pour mettre en pause le contrat (uniquement accessible par le propriétaire)

    function pause() public onlyOwner {

        _pause();

    }

    // Fonction pour reprendre les opérations du contrat

    function unpause() public onlyOwner {

        _unpause();

    }

}

```

**Explication des éléments ajoutés via OpenZeppelin :**

*   **Ownable** : Le contrat est contrôlé par un propriétaire unique, qui peut mettre en pause ou reprendre le contrat.
*   **Pausable** : Le contrat permet de mettre en pause certaines fonctions en cas de problème
*   **ReentrancyGuard** : La fonction instantPayment est protégée contre les attaques de réentrance grâce au modificateur nonReentrant.
*   **SafeMath** : La bibliothèque **SafeMath** est utilisée pour éviter les dépassements d'entiers et assurer des calculs mathématiques sûrs.  
     

* * *

#### **3. Déployer le contrat sur Sepolia**

1.  **Créer un script de déploiement** dans **scripts/deploy.js** :  
     

```javascript

async function main() {

    const [deployer] = await ethers.getSigners();

    console.log("Déployé par : ", deployer.address);

    const Contract = await ethers.getContractFactory("InstantPaymentHubSecure");

    const contract = await Contract.deploy();

    console.log("Contrat déployé à l'adresse : ", contract.address);

}

main()

    .then(() => process.exit(0))

    .catch((error) => {

        console.error(error);

        process.exit(1);

    });

```

**Exécuter le script de déploiement** :  
  
```bash  
npx hardhat run scripts/deploy.js --network sepolia

```

Cela déploiera le contrat sur le testnet Sepolia et vous donnera l'adresse du contrat déployé.  
 

* * *

#### **4. Interagir avec le contrat via Hardhat**

1.  **Créer un script d'interaction** dans **scripts/interact.js** pour tester le contrat **InstantPaymentHubSecure** déployé :  
     

```javascript

async function main() {

    const [deployer] = await ethers.getSigners();

    const contractAddress = "VOTRE_ADRESSE_DE_CONTRAT"; // Remplacez par l'adresse du contrat déployé

    const contract = await ethers.getContractAt("InstantPaymentHubSecure", contractAddress);

    // Déposer des fonds dans le contrat

    const depositTx = await contract.deposit({ value: ethers.utils.parseEther("1.0") });  // Déposer 1 Ether

    await depositTx.wait();

    console.log("1 Ether déposé dans le contrat");

    // Effectuer un paiement instantané

    const recipient = "ADRESSE_D_UN_UTILISATEUR"; // Remplacez par l'adresse du destinataire

    const paymentTx = await contract.instantPayment(recipient, ethers.utils.parseEther("0.5"));

    await paymentTx.wait();

    console.log("0.5 Ether payé à", recipient);

    // Retirer des fonds du contrat

    const withdrawTx = await contract.withdraw(ethers.utils.parseEther("0.2"));

    await withdrawTx.wait();

    console.log("0.2 Ether retiré du contrat");

}

main()

    .then(() => process.exit(0))

    .catch((error) => {

        console.error(error);

        process.exit(1);

    });

```

**Exécuter le script d'interaction** :  
  
```bash  
npx hardhat run scripts/interact.js --network sepolia

```

Cela interagira avec le contrat déployé en effectuant des transactions de dépôt, de paiement et de retrait.  
  
 

* * *

#### **5. Vérification des résultats sur Sepolia via Etherscan**

*   Une fois que vous avez effectué des transactions, vous pouvez vérifier leur état sur Sepolia Etherscan.
*   Recherchez l'adresse du contrat déployé et consultez les transactions pour vérifier les paiements effectués.  
     

* * *

### **Ressources supplémentaires**

*   **solidity Documentation** : solidity Documentation
*   **OpenZeppelin Contracts** : OpenZeppelin Contracts
*   **Hardhat Documentation** : Hardhat Documentation
*   **Sepolia Testnet** : Sepolia Etherscan

* * *

### **Fichiers du Projet**

Voici un aperçu des fichiers et dossiers dans le repo pour ce lab :

```lua

lab-oz-secure-contracts/

│

├── contracts/

│   └── InstantPaymentHubSecure.sol

│

├── scripts/

│   ├── deploy.js

│   └── interact.js

│

├── test/

│   └── instant-payment-hub.test.js (optionnel pour les tests unitaires)

│

├── hardhat.config.js

├── package.json

└── README.md

```

* * *

### **Objectifs de l'étudiant après ce lab**

*   Apprendre à utiliser les **bibliothèques OpenZeppelin** pour sécuriser un smart contract.
*   Intégrer des fonctionnalités telles que **Ownable**, **Pausable**, **ReentrancyGuard**, et **SafeMath** dans un contrat solidity.  
      
     

Déployer et interagir avec un contrat sécurisé sur le testnet **Sepolia**.


