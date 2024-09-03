# MultiAccount Deployment Guide

This documentation provides a comprehensive guide on deploying a MultiAccount contract on a specific blockchain. It details the process, starting from deploying a `TransparentUpgradeableProxy` and implementation, to verifying and interacting with the deployed proxy.&#x20;

#### Prerequisites

* Install Node.js and npm.
* Install Hardhat in your project directory: `npm install --save-dev hardhat`
* Install Typescript and `ts-node`: `npm install --save-dev typescript ts-node`

### Step 1: Setting up Hardhat

1.  Initialize a new Hardhat project:

    ```bash
    npx hardhat init
    ```
2. Follow the prompts to create a sample project.

To run a TypeScript deployment script with Hardhat, you'll need to make sure your environment is set up correctly for TypeScript execution.

#### TypeScript Configuration:

Ensure you have a `tsconfig.json` file in your project root. This file configures options for TypeScript in your project. Hardhat automatically generates this for you if you initialize your project with TypeScript support:

```json
{
  "compilerOptions": {
    "target": "es2020",
    "module": "commonjs",
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "strict": true,
    "skipLibCheck": true,
    "resolveJsonModule": true,
    "noErrorTruncation": true
  }
}
```

### Step 2: Adding the MultiAccount Contract to our Project

Once you have your TypeScript configuration ready, the next step in setting up your deployment is to incorporate the MultiAccount contract and its essential interfaces. You'll need to place the contracts and its associated interfaces within the `/contracts` directory of your project. These components are available in the SYMMIO's protocol-core GitHub repository, within the `/contracts/multiAccount` and `/contracts/interfaces` directories. Ensure the following files are added to your project:

**Contracts: (`/contracts/multiAccount`)**

* `MultiAccount.sol`: The primary contract for managing multiple accounts.
* `SymmioPartyA.sol`: This is used for deploying user sub-accounts through the CREATE-2 opcode, offering a more efficient and flexible account creation process.

**Interfaces: (`/contracts/interfaces`)**

* `IMultiAccount.sol`: Interface for the MultiAccount contract.
* `ISymmio.sol`: SYMMIO protocol interface.
* `ISymmioPartyA.sol`: Interface specific to the SymmioPartyA contract.

### Step 3: Installing Required Packages

The MultiAccount.sol contract imports from earlier versions of Openzeppelin's libraries. We'll install `@openzeppelin/contracts@^4.9.0` and `@openzeppelin/contracts-upgradeable@^4.7.3`

Open your terminal or command prompt, navigate to your project's root directory and run the following commands:

1.  **Install OpenZeppelin Contracts**:

    ```bash
    npm install @openzeppelin/contracts@^4.9.0
    ```

    This command installs the OpenZeppelin Contracts package at version `4.9.0` or the latest minor version within the `4.9.x` range.
2.  **Install OpenZeppelin Contracts-Upgradeable**:

    ```bash
    npm install @openzeppelin/contracts-upgradeable@^4.7.3
    ```

After running these commands, your `package.json` file will be automatically updated to reflect these dependencies and their versions under the `dependencies` section. It should look something like this:

```json
"dependencies": {
  "@openzeppelin/contracts": "^4.9.0",
  "@openzeppelin/contracts-upgradeable": "^4.7.3",
}
```

### Step 3: Saving the Deployed Address

After successful deployment of the `MultiAccount` and `SymmioPartyA` contracts, it's helpful to track the deployed contract addresses for future deployments. We'll incorporate a utility module specifically designed for saving and loading these critical addresses. This approach ensures that when the MultiAccount contract is initialized, it can readily access the protocol address (`symmioAddress`) and the bytecode for the `SymmioPartyA` contract. The `/utilty/file.ts` script handles this for us, and we'll incorporate it into our main deployment script.

```typescript
import fs from "fs"

export type Addresses = {
  symmioAddress?: string,
  collateralAddress?: string,
  multiAccountAddress?: string,
  hedgerProxyAddress?: string,
  MulticallAddress?: string,
}

export function loadAddresses(): Addresses {
  let output: Addresses = {}
  if (fs.existsSync("./output/addresses.json")) {
    output = JSON.parse(fs.readFileSync("./output/addresses.json", "utf8"))
  } else {
    if (!fs.existsSync("./output"))
      fs.mkdirSync("./output")
    output = {}
    fs.writeFileSync("./output/addresses.json", JSON.stringify(output))
  }
  return output
}

export function saveAddresses(content: Addresses): void {
  if (!fs.existsSync("./output/addresses.json")) {
    if (!fs.existsSync("./output"))
      fs.mkdirSync("./output")
  }
  fs.writeFileSync("./output/addresses.json", JSON.stringify(content))
}
```

To initiate our first deployment, create an `addresses.json` file that contains the contract addresses needed to initialize the `MultiAccount` contract. Use the provided [spreadsheet](https://docs.google.com/spreadsheets/d/1XkbnBeuScKQMIR7NTY1r9DkuxCV1Nudal5r4xzzzuQw/edit?pli=1) to populate the fields according to the specific chain you'd like to deploy on.

```json
{
    "symmioAddress":"",
    "collateralAddress":"",
    "multiAccountAddress":"",
    "partyBAddress":"",
    "MulticallAddress":""
}
```

### Step 4: Installing Dependencies for the Deployment Script

Before moving on to the actual deployment, we'll be installing and using `hardhat-ethers` and `hardhat-upgrades`. The `ethers.js` library is designed to provide a comprehensive yet compact toolkit for interacting with the Ethereum Blockchain and its vast ecosystem, serving as the foundation of our deployment scripts. `hardhat-upgrades`, is a plugin aimed at streamlining the deployment and upgrade process of smart contracts through proxies.&#x20;

Run this command in the command prompt:

```bash
npm install @nomiclabs/hardhat-ethers@^2.2.3 @openzeppelin/hardhat-upgrades@^1.21.0 ethers@^5.7.2
```

## Step 5: Writing the Deployment Script

First import the dependencies we installed in Step 4:

```typescript
import { ethers, run, upgrades } from "hardhat"
```

Then import the file utility for loading and saving addresses:

```typescript
import { Addresses, loadAddresses, saveAddresses } from "../utils/file"
```

The deployment script should perform the following:

1. **Getting Signers**: The script starts by retrieving the list of signers (accounts) available from the node it's connected to, using Hardhat's `ethers` plugin. It assumes the first account (`deployer`) is the one to be used for deploying the contracts. The private key for the deployer account is declared in `hardhat.config.ts`
2. **Loading Contract Deployments**: It attempts to load previously deployed addresses from a function `loadAddresses()`
3. **Contract Factory for `SymmioPartyA`**: It creates a contract factory for `SymmioPartyA`. This factory is used to deploy new instances of the contract.
4. **Deploying `MultiAccount` as Upgradeable**: It proceeds to deploy `MultiAccount` in an upgradeable manner using OpenZeppelin's `upgrades` plugin. The deployment uses a proxy pattern where the actual contract logic (implementation) and state are separated. The contract initializes with:

* **`admin`** : the `admin` address.
* **`symmioAddress`**: the SYMMIO contract address.
* **`accountImplementation_`**: the bytecode of `SymmioPartyA`

1. **Logging Deployment Addresses**: After deployment, it logs the addresses related to the deployed upgradeable contract, including the proxy address, the admin address and the implementation address.
2. **Saving Addresses**: The script then updates the `addresses.json` with the new `multiAccountAddress` for reference in future deployments.
3. **Verification Process**: It attempts to verify the deployed contract on a blockchain explorer. The verification process is delayed by 15 seconds before using Hardhat's `verify:verify` task to submit verification information.
4. **Error Handling and Script Completion**: The script ends with a basic structure to handle promise resolution, logging any errors encountered and exiting the process accordingly.

{% code title="deployMultiAccount.ts" %}
```typescript
async function main() {
	const [deployer] = await ethers.getSigners()

	console.log("Deploying contracts with the account:", deployer.address)
	let deployedAddresses: Addresses = loadAddresses()
	console.log("Deployed Addresses:", deployedAddresses);

	const SymmioPartyA = await ethers.getContractFactory("SymmioPartyA")

	const Factory = await ethers.getContractFactory("MultiAccount")
	console.log("Factory Deployed. ");

	const admin = process.env.ADMIN_PUBLIC_KEY
	const contract = await upgrades.deployProxy(Factory, [
		admin, deployedAddresses.symmioAddress,
		SymmioPartyA.bytecode,
	], { 
		initializer: "initialize",
	  })
	await contract.deployed()
	console.log("Contract Deployed. ");


	const addresses = {
		proxy: contract.address,
		admin: await upgrades.erc1967.getAdminAddress(contract.address),
		implementation: await upgrades.erc1967.getImplementationAddress(
		  contract.address,
		),
	}
	console.log(addresses)

	deployedAddresses.multiAccountAddress = contract.address
	saveAddresses(deployedAddresses)

	try {
		console.log("Verifying contract...")
		await new Promise((r) => setTimeout(r, 15000))
		await run("verify:verify", { address: addresses.implementation })
		console.log("Contract verified!")
	} catch (e) {
		console.log(e)
	}
}

main()
  .then(() => process.exit(0))
  .catch((error) => {
	  console.error(error)
	  process.exit(1)
  })
```
{% endcode %}

## Step 5: Configuring the Deployment Script for Specified Chains

Deploying the MultiAccount contract requires the deployment account's private and public keys. Due to their sensitive nature, its important to integrate `dotenv` with our script. `dotenv` facilitates the safe storage and access of private variables, like the deployment account's keys, through an environment file (`*.env`).

```bash
npm install dotenv
```

#### Create a `.env` File

In the root of your project, create a file named `.env`. Inside this file, you'll define environment variables. For example:

{% code title=".env.example" %}
```
ADMIN_PUBLIC_KEY=
POLYGON_API_KEY=
PRIVATE_KEY=""
PRIVATE_KEYS_STR=""
ADMIN_PUBLIC_KEY=""
```
{% endcode %}

In our config file, we'll declare these as:

```typescript
const privateKey: string | undefined = process.env.PRIVATE_KEY
if (!privateKey)
	throw new Error("Please set your PRIVATE_KEY in a .env file")

const privateKeysStr: string | undefined = process.env.PRIVATE_KEYS_STR
const privateKeyList: string[] = privateKeysStr?.split(",") || []

const mantleAPIKey: string = process.env.MANTLE_API_KEY || ""
const ftmAPIKey: string = process.env.FTM_API_KEY || ""
const bnbApiKey: string = process.env.BNB_API_KEY || ""
const baseApiKey: string = process.env.BASE_API_KEY || ""
const polygonApiKey: string = process.env.POLYGON_API_KEY || ""
const zkEvmApiKey: string = process.env.ZKEVM_API_KEY || ""
const opBnbApiKey: string = process.env.OPBNB_API_KEY || ""

const hardhatDockerUrl: string | undefined = process.env.HARDHAT_DOCKER_URL || ""

```

### **Networks Configuration**

Each network has specific settings that tell Hardhat where to deploy the contracts, which accounts to use, and any network-specific options like gas prices. Here are examples for various chains:

```javascript
networks: {
	mantle: {
		url: "https://1rpc.io/mantle",
		accounts: [ privateKey ],
	},
	
}, //other networks...
```

* **URL**: Each network has a `url` that specifies the JSON-RPC endpoint to connect to. This is essential for deploying to that specific network.
* **Accounts**: Specifies which accounts to use for transactions on the network. This can be a list of private keys or a single key.

### **Explorer Configuration**

This part of the configuration is essential for verifying contract deployments on different networks, allowing you to interact with your contracts through blockchain explorers.

```javascript
etherscan: {
	apiKey: {
		mantle: mantleAPIKey,
		fantom: ftmAPIKey,
		bnb: bnbApiKey,
		base: baseApiKey,
		polygon: polygonApiKey,
		zkEvm: zkEvmApiKey,
		opbnb: opBnbApiKey,
	},
	customChains: [
		{
			network: "mantle",
			chainId: 5000,
			urls: {
				apiURL: "https://explorer.mantle.xyz/api",
				browserURL: "https://explorer.mantle.xyz"
			}
		},
		// other networks...
	],
},
```

* **API Key**: Each entry under `apiKey` is crucial for authentication when using the Etherscan-like services to verify contracts. The API key can be obtained from creating an account with an explorer.
* **Custom Chains**: Configuration for custom networks (not standard like Ethereum Mainnet or Ropsten) to ensure that the Hardhat Etherscan plugin knows how to interact with these networks' explorers.

An example `hardhat-config.ts` file can be found at [this Github repository](https://github.com/academy17/multiaccount-deploy). Ensure you install the necessary dependency versions if you're using this method.

## Step 6:  Deploying the MultiAccount contract

To run the deployment script, enter the following command in the terminal, where `{chain}` is the desired deployment chain:

```
npx hardhat run scripts/deployMultiAccount.ts --network {chain}
```

### Troubleshooting

It might be necessary to set a `gasPrice` to ensure timely deployments. This can be customized in the hardhat config file:

<pre class="language-typescript"><code class="lang-typescript">{chain}: {
      url: "",
<strong>      accounts: [ privateKey ],
</strong>      gasPrice: ,
</code></pre>
