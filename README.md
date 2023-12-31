## AxelART: Unleashing the Future of Social Media, Digital Art NFTs with Seamless Interchain Minting, Web3Auth Login, AI Art Generation, and Real-time pAInt Token Streaming.

Welcome to AxelART, the revolutionary platform that transforms the world of AI art. Imagine Instagram, but with a unique twist – instead of conventional photo uploads, users provide text prompts that drive AI to craft captivating images. AxelART seamlessly combines "web 2.0" social features, such as commenting, liking, and reposting, with the power of "web3" functionalities, allowing users to mint images as NFTs on various blockchain networks, thanks to the integration of Axelar, Safe, and The Graph.

### Features

#### Web 2.0 Features
- **Creating and Sharing AI Images:** Users can input text prompts, and our AI model, powered by OpenAI's DALL-E, generates unique and mesmerizing art.
- **Social Interactions:** Like, comment, repost, and follow other users, fostering a vibrant community.

#### Web 3.0 Features
- **NFT Minting:** Users can mint their AI-generated art as NFTs on the blockchain.
- **Account Abstraction (AA):** Seamless NFT minting without the need for a seed phrase, making it user-friendly.

#### Signup and Login
- **Web3Auth Integration:** Users can log in using social, email, or wallet options without the need for complex wallet setups.

#### ENS Name Support
- **User-Friendly Display:** Utilizing ENS names for wallet logins, enhancing user identification.

### The First Mint - Behind the Scenes

#### Safe Integration 
- **Implementation:** AxelART implemented the Safe{Core} Protocol and Safe{Core} Account Abstraction SDK.
- **Features:** The app showcases the features enabled by Safe, emphasizing secure and simplified digital ownership.
- **Universal Ownership:** A "1 of 3" multi-signature Safe is deployed for users, ensuring digital ownership without compromising security.
- **Gasless Minting:** ERC20 Approval transactions are managed within the Safe, simplifying the first mint process.

#### Axelar Integration
- **Interchain Functionality:** Leveraging Axelar's General Message Passing (GMP) for seamless contract interactions across multiple chains.
- **DApp Development:** Building DApps that showcase the power of Axelar GMP, enhancing interchain capabilities.

### The pAInt Super Token
- **Real-time Streaming:** Powered by Superfluid, pAInt is a native Super Token supporting real-time streaming, acting as a utility token for NFT minting.

### AIrtist PRO
- **Freemium Model:** Supporting FREE users with sponsored cloud-computing and gas costs while offering premium features through AIrtist PRO.
- **Stripe Billing Integration:** Monthly subscription powered by Stripe Billing, enabling PRO users to access enhanced features and benefits.

### The Graph Integration
- **Subgraph Development:** A custom subgraph to index data from smart contracts, providing a robust querying mechanism.

### Acknowledgments
AxelART is made possible with the support of the following:

1. Safe: A universal ownership protocol, making digital assets and identities accessible to all.

- Safe{Core} Protocol
- Safe{Core} Account Abstraction SDK

2. Axelar: The full-stack interoperability layer for Web3, enabling seamless interaction with contracts across multiple chains.

- Axelar GMP Documentation

3. The Graph: An indexing protocol for querying networks like Ethereum, Polygon, and IPFS.

- The Graph Documentation

---

## In-depth tech details
### Choosing a Chain

Users can choose a preferred chain in their settings, which will be used when posting new AI-generated art. The live demo supports testnets for Ethereum, Optimism, and Arbitrum, but many more are planned for a production launch. When minting artwork posted by others, a dropdown is displayed for single-click interchain minting:

### How to mint an Interchain NFT using AxelART

- To mint your own AI-generated art: _check a box when creating your art_.
- To mint others' art: _click on the desired chain from the dropdown_.

_It's that easy_. No messages to sign, no transactions to approve, no gas required.

### How AxelART Interchain NFTs Work

As described above there are very few steps for the user, and it is very easy to mint an AxelART NFT on any supported chain. The complexity has been abstracted away, with all the magic happening behind the scenes. 

Most of the AxelART transactions are sent via Gelato Relay, an account abstraction service. For more about how AxelART uses Gelato Relay,(#gelato-relay-sponsored-erc2771-calls).

The other key part of the magic is General Messaging Passing (GMP) powered by Axelar, which is used to transport NFTs from one chain to another.

### The Transporter Hub and Spoke Model

AxelART's approach to interchain NFTs is to separate the NFT logic and the (interchain) transport logic into different contracts.  The `Transporter` contract is departure and arrival hub for multiple NFT contracts/collections. The `Transporter` deployed for AxelART is powered by Axelar, but other transporters could use other GMP solutions. 

On each chain, AxelART has a single "shared" NFT contract that is the default for all FREE users. Users who pay to upgrade to (#axelart-pro) get their own dedicated NFT contract. On each chain, there is a _single_ `Transporter` contract that handles interchain transportation for _all_ of the NFT contracts. This hub-and-spoke model helps simplify things and helps minimise total _bytecode_ and thus deployment costs.

All NFTs get sent from Transporter-to-Transporter, with the departure and arrival Transporters calling the NFT contracts on each end, according to the `IERC721Transportable` interface. For the demo, the `Transporter` contracts have been deployed to `0xa64904acF704926e8032900627a5486Ee191aFe3`.

Calling the `send()` function on the `Transporter` contract starts the process of transporting an NFT from one chain to another. The `IAxelarExecutable` implemented `execute()` function handles to arrival of NFTs from other remote chains. `send()` transactions are sent via Gelato Relay using `ERC2771Context` with Open Zeppelin's `AccessControl` for secure permission. More on the latter (#gelato-relay-sponsored-erc2771-calls).

#### Introducing the IERC721Transportable Interface

For an NFT contract to be compatible with a `Transporter`, it must implement the `IERC721Transportable` interface:

```
interface IERC721Transportable {
    function depart(uint256 tokenId) external;
    function arrive(address to, uint256 tokenId) external;
}
```

- the Transporter will call the `depart()` function before sending the NFT to another chain. Some NFT contracts will choose to burn the token before departure, others may transfer the token to a holding address. Some may emit an event.
- upon arrival at the destination chain, the receiving transported will call the `arrive()` function on the contract on the destination chain. Implementing the `arrive()` function would be primarily to mint the token on the destination chain.
- access control permissions to these are important and could be implemented in various ways to suit the needs of the deployers

The goal here is for the `Transporter` contract to handle all the [interchain GMP messaging](https://testnet.axelarscan.io/address/0xa64904acF704926e8032900627a5486Ee191aFe3) aspects while the `ERC721` NFT contracts handle only the burning/minting/transferring actions needed on each end of the transport.

#### Minting on a Remote Chain

As described above, from a user perspective, they simply choose a chain to receive the NFT. Behind the scenes, AxelART has a "home chain" which acts as the default chain. (For the live demo, this is Goerli). When a "remote" (non-home) chain is chosen, the NFT is first minted on the home chain and then transported to the remote chain. Both of these happen behind the scenes -- the user does not have to sign any messages or approve any messages or pay any gas. (_Side note:_ in production, likely, some "premium" or high-gas chains might only be available to pay (#axelart-pro) users).

Note that in addition to minting and transporting transactions, the "first mint" of the user also triggers the deployment of a Safe smart contract wallet, relevant `approve()` transactions, and starting a stream of `pAInt` tokens to the user. See below for more information on (#the-paint-super-token) and (#the-first-mint---behind-the-scenes). Since AxelART can be used to create AI art _without_ minting any NFTs, on-chain transactions are only triggered once a user first chooses to mint.

##### Minting on a Remote PRO Contract

Paying PRO users get their own dedicated NFT contracts, which are deployed to the _home chain_ from the `AIrtNFTFactory` contract on the home chain (via Gelato Relay). When another user first chooses a _remote_ chain for a PRO user, this triggers -- completely behind the scenes -- a deployment of the PRO user's contract on the chosen chain -- at the same address as on the home chain, plus giving the remote `Transporter` permissions to transport for the newly deployed contract. Once these have happened behind the scenes, the same mint + transport flow is triggered to send the NFT to the PRO contract on the remote chain. _Again, the complexity is hidden from the user, who just made a single click to mint on the chosen chain_.

#### Transporting NFTs After the Initial Mint

AxelART `Transporter` contracts support the transportation of NFTs between any two deployed chains at any time. For example, if a user decides in future to move their NFT from Optimism to Ethereum Mainnet. This feature has not yet been exposed in the user interface for the live demo.

## Signup and Login

Using web3Auth, AxelART login options include social, email, or wallet. Social/email logins will generate an app-specific private key … behind the scenes, no seed phrase is needed – only the user has access to the https://web3auth.io/docs/infrastructure/key-management (self-custody), enabled by Multi-Party Computation (MPC).

Social login users:
- do not need a wallet
- do not see an EVM address
- are not asked to “sign” any messages
- do not need ETH or any other token for “gas”

### ENS Name Support

Note that when choosing the `wallet` login option, AxelART checks for an ENS name for the connecting address, and if found, will use that as the "name" of the user on AxelART.

## Web 2.0 Features

Several "web 2.0" style features include:
- *Creating and sharing AI images.* User enters text prompts to share AI-generated art. (powered by DALL-E from OpenAI)
- *Liking.* Users can "like" images shared by others
- *Commenting.* User can post comments in reply to shared images
- *Reposting.* This is a bit like _retweeting_, but with a twist. A `repost` does *not* re-share the same image, but rather uses the same text prompt to generate an entirely *new* image.
- *Following.* Users can follow other users if they like their artwork.

## Web3 Features

Some users may optionally decide to mint images as NFTs on the blockchain, as described in detail above). When users post new artwork, they can:

1. Choose to immediately mint the image on the blockchain.
2. Enable others to mint the image for a price.
3. Neither (image will not be minted)

Creators can always decide later -- after posting -- to mint their images.

Minting during art creation is done by toggling a checkbox. For images already posted, minting involves a single click/tap. In both cases, the user does not have to send transactions or sign messages.

### The pAInt Super Token

When minting your own artwork, it costs `1 pAInt` token. When minting images posted by others, you pay the price set by the creator, denominated in either `pAInt` or `WETH`.

`pAInt` is a native Super Token that supports real-time streaming, powered by Superfluid. It acts as a utility token on the AxelART platform for minting NFTs. Users start with `5 pAInt` and after their first mint, active users receive `3 pAInt` per month, streamed in real-time.

See the streams on [Superfluid Console](https://console.superfluid.finance/goerli/accounts/0x83D4A49b80Af1CE060361A457a02d057560A9aD9?tab=streams). It's like watching paint stream.😐

### The First Mint - Behind the Scenes

When the user decides to mint their first NFT -- and not before -- on-chain transactions are triggered, behind the scenes:

1. *A Safe smart wallet is deployed.* This is a “1 of 3” multi-signature Safe. The user’s web3auth-generated address has full access as 1 of the 3 owners. The second owner is an AxelART Hot wallet enabling behind-the-scenes Safe transactions on behalf of the user. The third owner is an AxelART cold wallet for emergency/recovery purposes.
2. ERC20 Approval transactions are sent from the Safe to facilitate the first and future mints.
3. Sent via Gelato Relay, a transaction is sent to start streaming `3 pAInt` per month to the Safe.
4. Also via Gelato Relay, the minting transaction is sent to mint the NFT to the shared AxelART NFT contract.
5. If a remote chain is chosen, a transport transaction is also sent via Gelato Relay.

Remember, the above 4-5 transactions happen *behind the scenes*. From the user's perspective _all they did was check a box or tap a link_.

#### Gelato Relay Sponsored ERC2771 Calls

AxelART uses [Gelato Relay](https://docs.gelato.network/developer-services/relay) to send tokens and NFTs to users' Safes (and one more action discussed below). These requests are signed and submitted on-chain by Gelato relayers, with gas paid from AxelART's [Gelato 1 Balance](https://docs.gelato.network/developer-services/relay/payment-and-fees#1balance) account. Each of the contracts deployed by AxelART support [ERC2771 Context](https://docs.gelato.network/developer-services/relay/quick-start/erc-2771) which enables secure transactions to be signed by AxelART but relayed on-chain by Gelato Relayers. This works seamlessly with OpenZeppelin's `AccessControl` permissions to restrict functions to authorized signers.

### Subsequent NFT Minting

Subsequent NFT minting triggers a single transaction -- via Gelato Relay -- to mint the NFT to their Safe, while withdrawing `1 pAInt` from the Safe for each NFT minted of their own art (or the required amount of `pAInt` or `WETH` if minting others’ art). If a user has less than `1 pAInt` they cannot mint and must wait until they accumulate enough via the incoming stream (or by other means 🦄). Again, a second transport transaction is triggered if a remote chain is chosen.

#### Selling NFTs without a Deployed Safe to Receive Payment?

*Scenario:* a creator has joined AxelART and posted several images, but has not minted any yet. But the creator has enabled minting of the posts, and set prices and currencies (`pAInt` or `WETH`) in each case. What happens when another user decides to mint these images? How does the creator get paid, when no Safe has (yet) be deployed for the creator? As mentioned above, the Safe is only deployed when the user does _their_ first mint, and not before. Even though the Safe has not been deployed, the creator still gets paid! When the creator joined AxelART, the Safe SDK was used to accurately _predict_ the user's Safe address, _even though it has not yet been deployed_. This predicted Safe address is assigned to the creator's user account, and when `pAInt` or `WETH` gets sent to that address, it just works, and the funds are [owned by the predicted Safe address](https://blog.openzeppelin.com/getting-the-most-out-of-create2/). When/if the creator triggers their first mint, the Safe will then be deployed to the predicted address and the creator will have full access to the tokens sent previously.

## AxelART PRO

Since we are sponsoring -- paying the gas for -- users' transactions, using `pAInt` as a utility token helps limit the gas costs associated with users on the FREE plan. These users receive `3 pAInt` monthly, streamed in real-time.

For serious AxelARTists, minting 3 NFTs per month may not be enough.

AxelART uses a _freemium_ business model, where cloud-computing and gas costs are subsidized (sponsored) for FREE users, while revenue from PRO users more than offsets those costs.

AxelART PRO is a monthly subscription powered by Stripe Billing. Consistent with the goal of account abstraction, PRO users pay via credit card recurring billing (with plans to add support for crypto-native billing options).

### AxelART PRO Features

- a dedicated NFT contract for each PRO user (deployed from the AxelART Factory contract via Gelato Relay
- cross-chain NFT collections (PRO contract is deployed on each chain -- behind the scenes -- if/when someone wants to mint on those chains)
- increased stream, now `30 pAInt` streamed monthly in real-time
- preview before posting: view multiple images based on your text prompt, then choose the best to post (coming soon)
- option to keep prompts private from other users (coming soon)
- more to come

### Stripe Webhooks

When a user buys a PRO subscription, an event is sent from Stripe to a webhook endpoint on the AxelART API. This triggers the 2 transactions (via Gelato Relay) to deploy an NFT contract for the user and to increase the stream to `30 pAInt` per month. Additionally, as an "upgrade bonus", `10 pAInt` is airdropped to the PRO user's Safe at the time of the upgrade. From this point forward, the user's images are minted to their own contract/collection, which contains only their own artwork.

When a PRO subscription is cancelled, another event webhook automatically triggers a transaction to reduce the stream back to `3 pAInt` per month and flip the user back to minting on the shared NFT contract.

## How it was Built

There are three main categories of code for AxelART:

- Frontend Client App
- Server APIs and Datastore
- Ethereum Smart Contracts

### Frontend Client App

The live demo of the front-end AxelART app is located at https://axelart.xyz. The app includes javascript code that primarily interfaces with AxelART server API endpoints.

The javascript web3auth SDK is used in the front-end to power all forms of authentication. Using web3 auth, the user can choose to log in with social apps (ie. Twitter/Facebook), email, or wallet (ie. Metamask/WalletConnect). The web3auth SDK interfaces with the nodes in the [Auth Network](https://medium.com/toruslabs/introducing-the-auth-network-b8fab1b5e1f6) to produce an app-specific private key for the user. The private key exists only in the browser and it is never sent to AxelART servers nor saved by any third party. The web3auth SDK also produced a JWT auth token, that is used by AxelART as an API key when calling authenticated AxelART API endpoints (see more below).

The front-end uses the Firebase Firestore SDK to fetch and render AxelART data stored in a Firestore data store: data about users, posts, followers, social reactions, etc.

The HTML and CSS of the AxelART frontend were built using the Instello Ultimate Photo Sharing HTML Template set, used under license.

The front-end is hosted using Firebase Hosting. 

### Server APIs and Datastore

AxelART uses three core services from Google Firebase for server-side functions.

- *Firestore*. Data about users and posts are stored in a noSQL Firestore datastore.
- *Storage*. Once images have been generated via OpenAI SDK, they are stored using Firebase Storage (Google Cloud Storage)
- *Firebase Functions* There are three types of serverless functions.
  - The first is an HTTPS function that handles requests to the AxelART API endpoints, used to access and modify Firestore data and interact with Ethereum smart contracts.
  - Next, several functions are triggered by adding or updating data in the Firestore database, which in turn may trigger interactions with the Safe and Gelato SDKs to execute functions onchain.
  - Finally, several "cron" functions run periodically. These poll the Gelato Relay API or Axelar SDK for the status of transactions that have been relayed to Gelato or transported via Axelar GMP. Once Gelato reports that a transaction has been executed, the transaction is fetched using EthersJS and relevant data is extracted from the event logs, such as the `tokenId` of a newly minted NFT, or the `nftContract` address of a newly deployed `AIrtNFT` contract for a PRO user.

### Ethereum Smart Contracts

Three smart contracts were written in Solidity:

- `AIrtNFT.sol` - This is an `ERC721` NFT smart contract, leveraging OpenZeppelin contracts. A notable inclusion is the support for `ERC2771Context` which enables secure permission-based function calls via Gelato Relay. Also added are two minting functions which power minting of new images, and collecting and transferring `pAInt` or `WETH` tokens when necessary.
- `Transporter.sol` - This Transport contract sends and receives NFTs from multiple `AIrtNFT.sol` contracts between chains, using GMP on the Alexar Network.
- `AIrtNFTFactory.sol` - This a factory contract used to deploy minimal [Clones](https://docs.openzeppelin.com/contracts/4.x/api/proxy#Clones) of the `AIrtNFT.sol` and `Transporter.sol` contracts. This contract was used to deploy the main NFT contract that is shared by FREE users and also used to deploy NFT contracts for each PRO user when they upgrade. *Fun fact:* _Using the minimal clone approach, it actually costs *less gas* to deploy an NFT contract for a PRO user, compared to minting an AxelART NFT!_. The factory contract is also used to deploy a single instance of `Transporter.sol` on each chain.
- `Streamer.sol` - This contract uses the Superfluid protocol to enable streaming of the `pAInt` utility token, which was also deployed by this contract at deployment time. The core function of the streamer contract enables the starting, updating or stopping of a `pAInt` stream to a recipient, while optionally dropping some `pAInt` immediately (not streamed). This contract is only deployed to the home chain.

#### Deployed Contracts

Note all equivalent contracts are _deployed to the same address on each chain_. The implementation contracts for `AIrtNFT.sol` and `Transporter.sol` as well as the `AIrtNFTFactory.sol` contracts were actually deployed on all chains via the Multichain Deployer contract on the home chain (Goerli for the live demo).

- (Shared) `AIrtNFT` contract: `0x6a531B4447fB07b10A39E99Fc25b9c2cA63eAA42`: [Goerli](https://goerli.etherscan.io/address/0x6a531B4447fB07b10A39E99Fc25b9c2cA63eAA42) - [Optimism](https://goerli-optimism.etherscan.io/address/0x6a531b4447fb07b10a39e99fc25b9c2ca63eaa42) - [Arbitrum](https://goerli.arbiscan.io/address/0x6a531b4447fb07b10a39e99fc25b9c2ca63eaa42)
- `Transporter` contract: `0xa64904acF704926e8032900627a5486Ee191aFe3`: [Goerli](https://goerli.etherscan.io/address/0xa64904acf704926e8032900627a5486ee191afe3) - [Optimism](https://goerli-optimism.etherscan.io/address/0xa64904acf704926e8032900627a5486ee191afe3) - [Arbitrum](https://goerli.arbiscan.io/address/0xa64904acf704926e8032900627a5486ee191afe3)
- `AIrtNFTFactory` contract: `0xB67E29f515106311703c793B98C7459e3198e053`: [Goerli](https://goerli.etherscan.io/address/0xB67E29f515106311703c793B98C7459e3198e053) - [Optimism](https://goerli-optimism.etherscan.io/address/0xB67E29f515106311703c793B98C7459e3198e053) - [Arbitrum](https://goerli.arbiscan.io/address/0xB67E29f515106311703c793B98C7459e3198e053)
- `Streamer` contract: `0x83D4A49b80Af1CE060361A457a02d057560A9aD9`: [Goerli](https://goerli.etherscan.io/address/0x83D4A49b80Af1CE060361A457a02d057560A9aD9)
- `pAInt` Super Token: `0xB66cf6eAf3A2f7c348e91bf1504d37a834aBEB8A` [Goerli](https://goerli.etherscan.io/address/0xB66cf6eAf3A2f7c348e91bf1504d37a834aBEB8A)
- Example PRO `AirNFT` contract deployed using the factory contract via Gelato Relay: `https://goerli.etherscan.io/address/0x5c73f6b7f56137f2c963a3534a0529e63a0b311f`: [Goerli](https://goerli.etherscan.io/address/https://goerli.etherscan.io/address/0x5c73f6b7f56137f2c963a3534a0529e63a0b311f) - [Optimism](https://goerli-optimism.etherscan.io/address/0x5C73f6B7F56137F2c963A3534A0529e63A0b311F) - [Arbitrum](https://goerli.arbiscan.io/address/0x5C73f6B7F56137F2c963A3534A0529e63A0b311F)

*Note:* the "Transactions" tab on Etherscan may show as empty for some of the above contracts. This is because almost all transactions are executed via Gelato Relay. As such, you can view the "Internal Transactions" tab to view the activity on these contracts.

### Upcoming Features
Continued Development of PRO Premium Features:
- Advancement of professional features for PRO users, including the development of sophisticated art generation tools.
- Introduction of Open Edition NFTs, offering a unique and exclusive experience for PRO subscribers.
- Implementation of NFT auctions, enabling users to participate in a dynamic marketplace for AxelART NFTs.

Development of Mobile Apps for iOS and Android:
- Creation and release of dedicated mobile applications for both iOS and Android platforms, providing users with a seamless and immersive AxelART experience on their preferred devices.

Integration with Web3 Social Platforms (Lens, Farcaster, etc.):
- Seamless integration with prominent web3 social platforms such as Lens, Farcaster, and others. This integration aims to enhance user connectivity, allowing a broader audience to engage with AxelART's creative ecosystem.

Enhanced Support for Web3-Native Users:
- Implementation of improved support for web3-native users, empowering them to use their own wallets directly instead of relying on the Safe mechanism.
- Introduction of on-chain subscriptions, providing a more decentralized and user-centric approach to AxelART's subscription model.

---

AxelART owes its magic to the collaboration of Axelar, Safe, and The Graph. Together, they empower AxelART to stand at the forefront of the convergence between creativity and decentralized technology.

Join us in this journey where imagination knows no bounds, and the blockchain becomes a canvas for your artistic expressions.

AxelART: Where Creativity Meets Blockchain - Redefining the Artistic Experience
