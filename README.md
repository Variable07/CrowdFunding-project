<div align="center">

<img src="https://img.shields.io/badge/Blockchain-Crowdfunding-00d4ff?style=for-the-badge&logo=ethereum&logoColor=white" />

# ⛓️ ChainFund — Decentralized Crowdfunding Platform

**Transparent · Trustless · Permissionless**

[![Next.js](https://img.shields.io/badge/Next.js-14-black?style=flat-square&logo=next.js)](https://nextjs.org/)
[![Solidity](https://img.shields.io/badge/Solidity-^0.8.0-363636?style=flat-square&logo=solidity)](https://soliditylang.org/)
[![Hardhat](https://img.shields.io/badge/Hardhat-2.x-f7dc6f?style=flat-square&logo=hardhat)](https://hardhat.org/)
[![Thirdweb](https://img.shields.io/badge/Thirdweb-SDK-7c3aed?style=flat-square)](https://thirdweb.com/)
[![Firebase](https://img.shields.io/badge/Firebase-Auth-FF6F00?style=flat-square&logo=firebase)](https://firebase.google.com/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.x-3178C6?style=flat-square&logo=typescript)](https://typescriptlang.org/)
[![Arbitrum](https://img.shields.io/badge/Network-Arbitrum%20Sepolia-2d374b?style=flat-square&logo=arbitrum)](https://arbitrum.io/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green?style=flat-square)](LICENSE)

<br/>

> A fully decentralized crowdfunding dApp built on Ethereum (Arbitrum Sepolia), combining on-chain transparency with an admin moderation layer for campaign legitimacy — no banks, no middlemen, no trust issues.

<br/>

[🚀 Live Demo](https://crowd-funding-project-sage.vercel.app/) · [📖 Documentation](#) · [🐛 Report Bug](https://github.com/Variable07/CrowdFunding-project/issues) · [💡 Request Feature](https://github.com/Variable07/CrowdFunding-project/issues)

</div>

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Architecture](#-architecture)
- [User Flow Diagram](#-user-flow-diagram)
- [Smart Contract Design](#-smart-contract-design)
- [Class Diagram](#-class-diagram)
- [Sequence Diagrams](#-sequence-diagrams)
- [Features](#-features)
- [Tech Stack](#-tech-stack)
- [Project Structure](#-project-structure)
- [Getting Started](#-getting-started)
- [Smart Contract Reference](#-smart-contract-reference)
- [Environment Variables](#-environment-variables)
- [Difficulties & Solutions](#-difficulties--solutions)
- [Roadmap](#-roadmap)
- [Contributing](#-contributing)
- [License](#-license)
- [Acknowledgements](#-acknowledgements)

---

## 🔭 Overview

Traditional crowdfunding platforms (GoFundMe, Kickstarter) are plagued by:

| Problem | Traditional | ChainFund |
|---|---|---|
| 💸 Platform Fees | 5–10% | ~0% (gas only) |
| 🔍 Transparency | Opaque | Fully on-chain |
| 🛡️ Trust | Centralized control | Smart contract enforced |
| 🌍 Accessibility | Geographic limits | Global, permissionless |
| 🔒 Fund Safety | Platform holds funds | Contract-locked |

ChainFund solves these by deploying campaign logic as an immutable smart contract on Arbitrum Sepolia, while using Firebase to power an admin moderation layer that keeps the platform legitimate without fully centralizing it.

---

## 🏛️ Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                        CLIENT LAYER                                 │
│                                                                     │
│   ┌─────────────────────┐         ┌────────────────────────────┐    │
│   │   User Interface    │         │       Admin Panel          │    │
│   │   (Next.js + TS)    │         │   (Firebase Auth-gated)    │    │
│   └──────────┬──────────┘         └──────────────┬─────────────┘    │
│              │                                    │                 │
│   ┌──────────▼──────────┐         ┌──────────────▼─────────────┐    │
│   │  Thirdweb SDK       │         │   Firebase Firestore       │    │
│   │  (Wallet + Contract)│         │   (Metadata + Status)      │    │
│   └──────────┬──────────┘         └──────────────┬─────────────┘    │
└──────────────┼──────────────────────────────────┼──────────────────┘
               │                                   │
               │         INTEGRATION LAYER         │
               │  ┌────────────────────────────┐   │
               └──►  blockchain.ts / firebase.ts◄───┘
                  └────────────────┬───────────┘
                                   │
┌──────────────────────────────────▼──────────────────────────────────┐
│                       BLOCKCHAIN LAYER                               │
│                                                                      │
│   ┌─────────────────────────────────────────────────────────────┐    │
│   │              CrowdFunding.sol (Solidity Smart Contract)      │   │
│   │                                                              │   │
│   │   createCampaign()  ──►  Campaign Storage (on-chain)         │   │
│   │   donateToCampaign() ──► ETH Transfer (trustless)            │   │
│   │   withdrawFunds()   ──►  Owner Payout (conditional)          │   │
│   │   getCampaigns()    ──►  Public Read Access                  │   │
│   └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│                    ⟠  Arbitrum Sepolia Testnet                       │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 🔄 User Flow Diagram

### General User Flow

```
                            ┌──────────────────┐
                            │   User Visits App│
                            └────────┬─────────┘
                                     │
                            ┌────────▼─────────┐
                            │  Connect MetaMask │
                            │     Wallet        │
                            └────────┬─────────┘
                                     │
                      ┌──────────────┼──────────────┐
                      │              │              │
             ┌────────▼────┐ ┌───────▼─────┐ ┌─────▼──────────┐
             │ Browse Live  │ │  Create New │ │  View Rejected  │
             │  Campaigns   │ │  Campaign   │ │   Campaigns     │
             └─────┬───────┘ └───────┬─────┘ └────────────────┘
                   │                 │
                   │        ┌────────▼──────────────┐
                   │        │  Fill Campaign Details │
                   │        │  Title / Description   │
                   │        │  Image / Target / Date │
                   │        └────────┬──────────────┘
                   │                 │
                   │        ┌────────▼──────────────┐
                   │        │  Campaign Submitted    │
                   │        │  Status: ⏳ PENDING    │
                   │        └────────┬──────────────┘
                   │                 │
                   │        ┌────────▼──────────────┐
                   │        │    Admin Reviews       │◄─────────────────┐
                   │        └──────┬──────────┬──────┘                  │
                   │               │          │                         │
                   │      ┌────────▼──┐   ┌───▼──────────┐              │
                   │      │ ✅ APPROVED│   │ ❌ REJECTED   │            │
                   │      └────────┬──┘   └───┬──────────┘              │
                   │               │          │                         │
                   │      ┌────────▼──┐   ┌───▼──────────┐              │
                   │      │ Campaign  │   │ Reason shown │              │
                   │      │ Goes Live │   │ publicly     │              │
                   │      └────┬──────┘   └──────────────┘              │
                   │           │                                        │
             ┌─────▼────┐ ┌────▼──────────────────┐                     │
             │  Donate   │ │  Fund with ETH         │                   │
             │  History  │ │  (Any wallet)          │                   │
             └──────────┘ └────────┬──────────────┘                     │
                                   │                                    │
                          ┌────────▼──────────────┐                     │
                          │  Target Reached?       │                    │
                          └────────┬──────────────┘                     │
                                   │                                    │
                      ┌────────────┼─────────────┐                      │
                      │                           │                     │
             ┌────────▼────┐           ┌──────────▼───┐                 │
             │  YES →       │           │  NO →         │               │
             │  Owner can   │           │  Campaign     ├──────────────┘
             │  Withdraw ✅  │           │  Continues    │
             └─────────────┘           └──────────────┘
```

### Admin Flow

```
         ┌─────────────────────┐
         │  Admin visits /admin │
         └──────────┬──────────┘
                    │
         ┌──────────▼──────────┐
         │  Firebase Auth Login │
         │  (Email + Password)  │
         └──────────┬──────────┘
                    │
         ┌──────────▼──────────┐       ┌───────────────────┐
         │  View All Campaigns  ├──────►│ Filter by Status  │
         │  (Pending / All)     │       │ Pending/Approved/ │
         └──────────┬──────────┘       │ Rejected          │
                    │                  └───────────────────┘
         ┌──────────▼──────────┐
         │  Select a Campaign   │
         └────────┬──────┬─────┘
                  │      │
       ┌──────────▼──┐ ┌─▼────────────────────┐
       │  ✅ Approve  │ │  ❌ Reject             │
       │  Campaign    │ │  → Enter Reason       │
       └──────────┬──┘ └─┬────────────────────┘
                  │      │
       ┌──────────▼──┐ ┌─▼────────────────────┐
       │  Status →   │ │  Status → REJECTED    │
       │  APPROVED   │ │  Reason stored in     │
       │  Stored in  │ │  Firebase + visible   │
       │  Firebase   │ │  publicly             │
       └─────────────┘ └──────────────────────┘
```

---

## 📜 Smart Contract Design

### CrowdFunding.sol — State Machine

```
                    ┌─────────────┐
                    │   CREATED   │  ← createCampaign() called
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │   PENDING   │  ← Awaiting admin decision
                    └──────┬──────┘
                           │
           ┌───────────────┼──────────────────┐
           │                                  │
    ┌──────▼──────┐                  ┌────────▼──────┐
    │  APPROVED   │                  │   REJECTED    │
    │  (fundable) │                  │  (read-only)  │
    └──────┬──────┘                  └───────────────┘
           │
    ┌──────▼──────────────┐
    │  donateToCampaign() │ ← Any wallet can fund
    │  ETH accumulates    │
    └──────┬──────────────┘
           │
    ┌──────▼──────────────┐
    │  Target reached OR  │
    │  Deadline passed?   │
    └──────┬──────────────┘
           │
    ┌──────▼──────────────┐
    │  withdrawFunds()    │ ← Only campaign owner
    └─────────────────────┘
```

### Contract Data Structures

```solidity
struct Campaign {
    address owner;          // Creator's wallet address
    string title;           // Campaign name
    string description;     // Full description
    uint256 target;         // Funding goal (in wei)
    uint256 deadline;       // UNIX timestamp
    uint256 amountCollected; // Current balance
    string image;           // IPFS / URL
    address[] donators;     // List of donors
    uint256[] donations;    // Parallel amounts array
    CampaignStatus status;  // Enum: Pending | Approved | Rejected
    string rejectionReason; // Populated if Rejected
}

enum CampaignStatus {
    Pending,    // 0 - Default on creation
    Approved,   // 1 - Admin approved, fundable
    Rejected    // 2 - Admin rejected with reason
}
```

---

## 🧩 Class Diagram

```
┌──────────────────────────────────────────────────────────────────────┐
│                          CrowdFunding.sol                            │
├──────────────────────────────────────────────────────────────────────┤
│  - campaigns: Campaign[]                                             │
│  - owner: address                                                    │
│  - numberOfCampaigns: uint256                                        │
├──────────────────────────────────────────────────────────────────────┤
│  + createCampaign(owner, title, desc, target, deadline, image)       │
│    → uint256 (campaign ID)                                           │
│  + donateToCampaign(id: uint256) payable                             │
│    → void                                                            │
│  + withdrawFunds(id: uint256)                                        │
│    → bool                                                            │
│  + getCampaigns()                                                    │
│    → Campaign[]                                                      │
│  + getDonators(id: uint256)                                          │
│    → address[], uint256[]                                            │
│  + updateStatus(id, status, reason) [admin only]                     │
│    → void                                                            │
└──────────────────────────┬───────────────────────────────────────────┘
                           │ uses
          ┌────────────────┴──────────────────┐
          │                                   │
┌─────────▼────────────┐          ┌───────────▼───────────────┐
│    Campaign Struct    │          │      CampaignStatus Enum   │
├──────────────────────┤          ├───────────────────────────  ┤
│ owner: address        │          │  Pending = 0               │
│ title: string         │          │  Approved = 1              │
│ description: string   │          │  Rejected = 2              │
│ target: uint256       │          └───────────────────────────┘
│ deadline: uint256     │
│ amountCollected: uint │
│ image: string         │
│ donators: address[]   │
│ donations: uint256[]  │
│ status: CampaignStatus│
│ rejectionReason: str  │
└──────────────────────┘

┌────────────────────────────┐      ┌──────────────────────────┐
│       blockchain.ts         │      │       firebase.ts          │
├────────────────────────────┤      ├──────────────────────────┤
│ + getContract()             │      │ + adminLogin(email, pass) │
│ + createCampaign(form)      │      │ + getCampaignMeta(id)     │
│ + getCampaigns()            │      │ + updateCampaignStatus(id)│
│ + donateToCampaign(id, amt) │      │ + setRejectionReason(id)  │
│ + withdrawFunds(id)         │      │ + listenToUpdates(id)     │
│ + getDonators(id)           │      └──────────────────────────┘
└────────────────────────────┘

┌────────────────────────────┐      ┌──────────────────────────┐
│       CampaignCard.tsx      │      │         Navbar.tsx         │
├────────────────────────────┤      ├──────────────────────────┤
│ Props:                      │      │ Props:                     │
│   campaign: Campaign        │      │   address: string          │
│   onClick: () => void       │      │   connect: () => void      │
├────────────────────────────┤      ├──────────────────────────┤
│ Renders:                    │      │ Renders:                   │
│   - Title, Image            │      │   - Logo                   │
│   - Progress Bar            │      │   - Nav Links              │
│   - Status Badge            │      │   - Wallet Button          │
│   - Deadline                │      │   - Address display        │
│   - Raised / Target         │      └──────────────────────────┘
└────────────────────────────┘
```

---

## 🔁 Sequence Diagrams

### Campaign Creation & Approval

```
User          Frontend         Blockchain.ts      Contract       Firebase       Admin
 │                │                  │               │               │             │
 ├──fill form────►│                  │               │               │             │
 │                ├──createCampaign()►│               │               │             │
 │                │                  ├──tx.send()────►│               │             │
 │                │                  │               ├──emit event───►│             │
 │                │                  │               │               ├─store meta──►│
 │                │◄─────────────────┤               │               │             │
 │◄──success──────┤                  │               │               │             │
 │                │                  │               │               │             │
 │                │                  │               │               │      ┌──────┤
 │                │                  │               │               │      │ Admin│
 │                │                  │               │               │      │ logs │
 │                │                  │               │               │      │  in  │
 │                │                  │               │               │◄─────┤      │
 │                │                  │               │               ├─campaigns──► │
 │                │                  │               │               │             ├─review─┐
 │                │                  │               │               │             │        │
 │                │                  │               │               │◄──approve───┤        │
 │                │                  │               │               ├─update status         │
 │                │                  │               │               │             │        │
 │◄──status:      │                  │               │               │             │
 │   APPROVED─────┤                  │               │               │             │
```

### Donation Flow

```
Donor         Frontend        blockchain.ts      Contract        Owner
 │                │                 │                │              │
 ├──select amount►│                 │                │              │
 │                ├─donateToCampaign►│               │              │
 │                │                 ├──tx({value})──►│              │
 │                │                 │                ├─store donor──┤
 │                │                 │                ├─add amount   │
 │                │                 │                │              │
 │                │◄────────────────┤                │              │
 │◄──confirmed────┤                 │                │              │
 │                │                 │                │              │
 │                │                 │        [Target Reached]       │
 │                │                 │                │              │
 │                │                 │                │◄─withdraw()──┤
 │                │                 │                ├──transfer────►│
 │                │                 │                │   ETH        │
```

---

## 🚀 Features

### 🧑‍💼 General User Features

| Feature | Description |
|---|---|
| 🔐 Wallet Auth | Connect via MetaMask — no sign-up required |
| 🖼 Create Campaign | Title, description, image URL, target ETH, deadline |
| 📊 Live Status | Real-time campaign status: Pending / Approved / Rejected |
| 💸 Fund Campaigns | Donate ETH to any approved campaign |
| 🧾 Donor History | See all donors and amounts for each campaign |
| 🛑 Rejection Reason | Transparent public reason for every rejected campaign |
| 💰 Withdraw | Campaign owner can withdraw collected ETH |

### 🛠 Admin Panel Features

| Feature | Description |
|---|---|
| 🔑 Secure Login | Firebase email/password authentication |
| 📋 Campaign Queue | View all pending campaigns for review |
| ✅ Approve | Mark campaign as live and fundable |
| ❌ Reject | Reject with mandatory reason (shown publicly) |
| 🏷️ Status Tags | Visual badges: Pending / Approved / Rejected |

---

## 🏗️ Tech Stack

```
┌─────────────────────────────────────────────────────────────┐
│                        TECH STACK                            │
│                                                              │
│  Frontend                    Backend / Blockchain            │
│  ─────────                   ──────────────────              │
│  ▸ Next.js 14                ▸ Solidity ^0.8.0               │
│  ▸ TypeScript 5              ▸ Hardhat 2.x                   │
│  ▸ Tailwind CSS              ▸ Thirdweb CLI                  │
│  ▸ Thirdweb SDK              ▸ Firebase Firestore            │
│                              ▸ Firebase Auth                 │
│                                                              │
│  Network                     Dev Tooling                     │
│  ──────                      ──────────                      │
│  ▸ Arbitrum Sepolia          ▸ ESLint                        │
│    (Ethereum L2)             ▸ Prettier                      │
│  ▸ MetaMask                  ▸ dotenv                        │
│  ▸ Ethers.js                                                 │
└─────────────────────────────────────────────────────────────┘
```

---

## 📂 Project Structure

```
crowdfunding-blockchain/
│
├── 📁 smart_contracts/              # Blockchain layer
│   ├── CrowdFunding.sol             # Main smart contract
│   ├── hardhat.config.ts            # Hardhat configuration
│   └── scripts/
│       └── deploy.ts                # Deployment script
│
├── 📁 pages/                        # Next.js routing
│   ├── index.tsx                    # Homepage — campaign listing
│   ├── create-campaign.tsx          # Campaign creation form
│   ├── campaign/[id].tsx            # Individual campaign page
│   └── admin.tsx                    # Admin moderation panel
│
├── 📁 components/                   # Reusable UI components
│   ├── CampaignCard.tsx             # Campaign display card
│   ├── Navbar.tsx                   # Navigation + wallet connector
│   ├── StatusBadge.tsx              # Pending / Approved / Rejected
│   ├── DonationList.tsx             # Donor history table
│   └── ProgressBar.tsx              # Funding progress indicator
│
├── 📁 utils/                        # Service layer
│   ├── blockchain.ts                # Contract interaction helpers
│   └── firebase.ts                  # Firebase CRUD operations
│
├── 📁 context/                      # Global state
│   └── index.tsx                    # ThirdwebProvider + state
│
├── 📁 types/                        # TypeScript definitions
│   └── campaign.ts                  # Campaign, Donation types
│
├── .env.local                       # Environment variables
├── next.config.js                   # Next.js config
├── tailwind.config.js               # Tailwind CSS config
└── package.json
```

---

## ⚙️ Getting Started

### Prerequisites

- Node.js `>= 18.x`
- npm or yarn
- MetaMask browser extension
- A Firebase project (free tier works)
- Thirdweb account (free)

### 1. Clone the Repository

```bash
git clone https://github.com/your-username/crowdfunding-blockchain
cd crowdfunding-blockchain
```

### 2. Install Dependencies

```bash
npm install
```

### 3. Configure Environment Variables

Create a `.env.local` file in the root:

```env
# Blockchain
NEXT_PUBLIC_CONTRACT_ADDRESS=0xYourDeployedContractAddress
NEXT_PUBLIC_THIRDWEB_CLIENT_ID=your_thirdweb_client_id

# Firebase
NEXT_PUBLIC_FIREBASE_API_KEY=your_firebase_api_key
NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN=your_project.firebaseapp.com
NEXT_PUBLIC_FIREBASE_PROJECT_ID=your_project_id
NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET=your_project.appspot.com
NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID=your_sender_id
NEXT_PUBLIC_FIREBASE_APP_ID=your_app_id
```

### 4. Deploy Smart Contract (Optional — use existing)

```bash
cd smart_contracts
npm install
npx hardhat compile
npx thirdweb deploy
```

> Follow the Thirdweb CLI prompts to deploy on **Arbitrum Sepolia**.  
> Copy the deployed contract address into your `.env.local`.

### 5. Run the Development Server

```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000) in your browser.

### 6. Configure Admin Access

In Firebase Console:
1. Enable **Email/Password** Authentication
2. Create an admin user manually
3. In Firestore, create an `admins` collection with the admin's UID

---

## 📘 Smart Contract Reference

### `createCampaign`

```solidity
function createCampaign(
    address _owner,
    string memory _title,
    string memory _description,
    uint256 _target,
    uint256 _deadline,
    string memory _image
) public returns (uint256)
```

Creates a new campaign with `Pending` status. Returns the campaign ID.

### `donateToCampaign`

```solidity
function donateToCampaign(uint256 _id) public payable
```

Sends ETH to the campaign. Only callable on `Approved` campaigns. Records donor address and amount.

### `withdrawFunds`

```solidity
function withdrawFunds(uint256 _id) public returns (bool)
```

Transfers collected ETH to the campaign owner. Requires `msg.sender == campaign.owner`.

### `getCampaigns`

```solidity
function getCampaigns() public view returns (Campaign[] memory)
```

Returns the full array of all campaigns (all statuses).

### `getDonators`

```solidity
function getDonators(uint256 _id) public view 
    returns (address[] memory, uint256[] memory)
```

Returns parallel arrays of donor addresses and their donation amounts.

---

## 🔐 Environment Variables

| Variable | Description | Required |
|---|---|---|
| `NEXT_PUBLIC_CONTRACT_ADDRESS` | Deployed CrowdFunding contract address | ✅ |
| `NEXT_PUBLIC_THIRDWEB_CLIENT_ID` | Thirdweb project client ID | ✅ |
| `NEXT_PUBLIC_FIREBASE_API_KEY` | Firebase project API key | ✅ |
| `NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN` | Firebase Auth domain | ✅ |
| `NEXT_PUBLIC_FIREBASE_PROJECT_ID` | Firebase project ID | ✅ |
| `NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET` | Firebase storage bucket | ✅ |
| `NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID` | Firebase messaging ID | ✅ |
| `NEXT_PUBLIC_FIREBASE_APP_ID` | Firebase app ID | ✅ |

---

## 🧗 Difficulties & Solutions

| Challenge | Solution Applied |
|---|---|
| **Thirdweb SDK integration** | Deeply studied SDK docs; used `useContract` and `useContractWrite` hooks correctly with proper ABI typing |
| **Async blockchain transactions** | Used `await` chains with loading states; React state managed confirmation stages (idle → pending → confirmed) |
| **Admin moderation vs decentralization** | Hybrid model: on-chain stores truth (funds, donors), Firebase stores metadata (status, reason) — keeps funds trustless |
| **Security & usability balance** | Admin status check via Firebase rules; on-chain withdraw only for campaign owner; frontend guards for UX |
| **State sync (Firebase ↔ Blockchain)** | Campaign ID used as foreign key; Firestore listener + contract polling combined for real-time UI |

---

## 🗺️ Roadmap

```
Phase 1 — Current ✅
├── Campaign creation, approval, rejection
├── ETH donations + withdrawal
├── Admin panel with Firebase auth
└── Arbitrum Sepolia deployment

Phase 2 — In Progress 🔄
├── Campaign search & filtering
├── Responsive mobile UI
└── Improved error handling + toasts

Phase 3 — Planned 📋
├── 🔁 Multi-chain: Polygon, Base, Optimism
├── 🪙 Token rewards for early backers
├── 📊 Campaign analytics dashboard
└── 📤 Email notifications (SendGrid)

Phase 4 — Future 🔮
├── 💬 Community comments per campaign
├── 🏷️ Campaign categories & tags
├── 🤝 DAO-based admin governance
└── 🧠 ML-based spam detection
```

---

## 🤝 Contributing

Contributions are welcome and appreciated!

```bash
# Fork the repository
# Create your feature branch
git checkout -b feature/AmazingFeature

# Commit your changes
git commit -m 'Add AmazingFeature'

# Push to the branch
git push origin feature/AmazingFeature

# Open a Pull Request
```

Please read [CONTRIBUTING.md](CONTRIBUTING.md) for details on our code of conduct and pull request guidelines.

---

## 📜 License

This project is licensed under the **MIT License** — see the [LICENSE](LICENSE) file for full details.

```
MIT License

Copyright (c) 2024 ChainFund Contributors

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software...
```

---

## 🙌 Acknowledgements

| Tool | Purpose |
|---|---|
| [Thirdweb](https://thirdweb.com/) | Wallet connection, SDK, contract deployment |
| [Hardhat](https://hardhat.org/) | Smart contract testing and compilation |
| [Firebase](https://firebase.google.com/) | Admin auth and campaign metadata |
| [Arbitrum](https://arbitrum.io/) | Low-cost Ethereum L2 network |
| [Next.js](https://nextjs.org/) | React framework for the frontend |
| [Tailwind CSS](https://tailwindcss.com/) | Utility-first styling |
| [OpenZeppelin](https://openzeppelin.com/) | Solidity security best practices |

---

<div align="center">

**Built with ❤️ on Ethereum**

⭐ Star this repo if you found it useful!

[![GitHub stars](https://img.shields.io/github/stars/your-username/crowdfunding-blockchain?style=social)](https://github.com/your-username/crowdfunding-blockchain)

</div>
