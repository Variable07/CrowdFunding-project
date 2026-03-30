# 🚀 CrowdFunding — Blockchain-Powered Fundraising Platform

<div align="center">

![Solidity](https://img.shields.io/badge/Solidity-0.8.20-363636?style=for-the-badge&logo=solidity)
![React](https://img.shields.io/badge/React-18.2-61DAFB?style=for-the-badge&logo=react)
![TypeScript](https://img.shields.io/badge/TypeScript-4.7-3178C6?style=for-the-badge&logo=typescript)
![Arbitrum](https://img.shields.io/badge/Network-Arbitrum-28A0F0?style=for-the-badge)
![Thirdweb](https://img.shields.io/badge/SDK-Thirdweb_v4-7C3AED?style=for-the-badge)
![TailwindCSS](https://img.shields.io/badge/Tailwind_CSS-3.4-06B6D4?style=for-the-badge&logo=tailwindcss)

**A trustless, transparent, and decentralized crowdfunding platform built on blockchain technology. Every donation is verifiable, every transaction immutable.**

[Overview](#-overview) · [Architecture](#-architecture) · [Smart Contract](#-smart-contract) · [Frontend](#-frontend) · [Getting Started](#-getting-started) · [Flows](#-user-flows)

</div>

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Architecture](#-architecture)
- [Smart Contract](#-smart-contract)
- [Frontend Structure](#-frontend-structure)
- [User Flows](#-user-flows)
  - [Campaign Lifecycle](#campaign-lifecycle)
  - [Donation Flow](#donation-flow)
  - [Admin Moderation Flow](#admin-moderation-flow)
  - [Authentication & Wallet Connection](#authentication--wallet-connection)
- [Key Features](#-key-features)
- [Tech Stack](#-tech-stack)
- [Getting Started](#-getting-started)
- [Environment Variables](#-environment-variables)
- [Contract Deployment](#-contract-deployment)
- [Pages & Components](#-pages--components)
- [State Management](#-state-management)
- [Security Model](#-security-model)

---

## 🌐 Overview

CrowdFunding is a fully decentralized crowdfunding application that removes intermediaries between campaign creators and donors. Built on the **Arbitrum** network, it leverages smart contracts for:

- **Transparent fund management** — all transactions are publicly verifiable on-chain
- **Trustless donations** — funds go directly to campaign owners via smart contract
- **Admin-governed moderation** — campaigns require approval before going live
- **Immutable records** — campaign history and donation data cannot be altered

---

## 🏗 Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        CLIENT (Browser)                         │
│                                                                 │
│  ┌──────────────┐   ┌──────────────┐   ┌──────────────────┐   │
│  │  React + Vite │   │  TailwindCSS │   │  React Router v6 │   │
│  └──────────────┘   └──────────────┘   └──────────────────┘   │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                  StateContext (Global State)             │   │
│  │  address │ contract │ campaigns │ donations │ isAdmin    │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              │                                  │
│                              ▼                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │             Thirdweb SDK (@thirdweb-dev/react)           │   │
│  │        Wallet Connection │ Contract Interaction          │   │
│  └─────────────────────────────────────────────────────────┘   │
└───────────────────────────────┬─────────────────────────────────┘
                                │  JSON-RPC / WebSocket
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                    ARBITRUM NETWORK (L2)                        │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │            CrowdFunding.sol (Smart Contract)             │   │
│  │                                                          │   │
│  │  createCampaign()   │   donateToCampaign()               │   │
│  │  approveCampaign()  │   rejectCampaign()                 │   │
│  │  getCampaigns()     │   getDonators()                    │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  Contract Address: 0x642a4ebf0f280e1a107eab1ab86bc21ac1815dfc  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📜 Smart Contract

**File:** `web3/contracts/CroudFunding.sol`  
**Standard:** Solidity `^0.8.20` with OpenZeppelin `AccessControl`

### Contract Structure

```
CrowdFunding (inherits AccessControl)
│
├── Roles
│   ├── DEFAULT_ADMIN_ROLE  → Super admin (deployer)
│   └── ADMIN_ROLE          → Can approve/reject campaigns
│
├── Data
│   ├── Campaign (struct)
│   │   ├── owner           address
│   │   ├── title           string
│   │   ├── description     string
│   │   ├── target          uint256 (wei)
│   │   ├── deadline        uint256 (unix timestamp)
│   │   ├── amountCollected uint256 (wei)
│   │   ├── image           string (URL)
│   │   ├── donators        address[]
│   │   ├── donations       uint256[]
│   │   ├── status          CampaignStatus (0=Pending, 1=Approved, 2=Rejected)
│   │   └── rejectionReason string
│   │
│   └── campaigns           mapping(uint256 => Campaign)
│
├── Functions (Public)
│   ├── createCampaign()    → Creates campaign in Pending state
│   ├── donateToCampaign()  → Sends ETH to campaign owner (payable)
│   ├── getDonators()       → Returns donators + amounts for a campaign
│   └── getCampaigns()      → Returns all campaigns array
│
├── Functions (Admin Only)
│   ├── approveCampaign()   → Sets status to Approved
│   ├── rejectCampaign()    → Sets status to Rejected with reason
│   ├── grantAdminRole()    → Grants ADMIN_ROLE to an address
│   └── revokeAdminRole()   → Revokes ADMIN_ROLE from an address
│
└── Events
    ├── CampaignCreated(campaignId, owner)
    ├── CampaignApproved(campaignId)
    ├── CampaignRejected(campaignId, reason)
    └── DonationReceived(campaignId, donor, amount)
```

### Campaign Status State Machine

```
                    ┌─────────────┐
    createCampaign  │             │
   ────────────────▶│   PENDING   │
                    │   (status=0)│
                    └──────┬──────┘
                           │
               ┌───────────┴───────────┐
               │                       │
        approve│                 reject│ + reason
               ▼                       ▼
        ┌──────────┐           ┌──────────────┐
        │ APPROVED │           │   REJECTED   │
        │(status=1)│           │  (status=2)  │
        └──────────┘           └──────────────┘
               │
        Accepts donations
        (deadline must be
         in the future)
```

---

## 🖥 Frontend Structure

```
Frontend/src/
│
├── @types/
│   └── vite-env.d.ts          # Vite environment type declarations
│
├── assets/                    # Images, icons, logos
│
├── components/
│   ├── CampaignFilters.tsx    # Search + sort + category filters
│   ├── CampaignStats.tsx      # Aggregate stats (total raised, backers, etc.)
│   ├── HeroSection.tsx        # Landing page hero with CTAs
│   ├── Layout.tsx             # Page layout wrapper with Navbar
│   ├── TopNavbar.tsx          # Top navigation with wallet connect
│   ├── countBox.tsx           # Single stat display box
│   ├── customButton.tsx       # Reusable styled button
│   ├── displayCampaigns.tsx   # Campaign grid with cards
│   ├── formField.tsx          # Form input / textarea field
│   ├── loader.tsx             # Full-screen transaction loading overlay
│   ├── navbar.tsx             # Secondary navbar with search bar
│   └── sidebarIcon.tsx        # Sidebar navigation icon
│
├── constants/
│   └── index.ts               # Navigation links config
│
├── contexts/
│   └── index.tsx              # Global StateContext + blockchain functions
│
├── pages/
│   ├── admin.tsx              # Admin moderation dashboard
│   ├── campaignDetails.tsx    # Individual campaign view + donation panel
│   ├── createCampaign.tsx     # Campaign creation form
│   ├── home.tsx               # Main campaigns listing page
│   ├── landing.tsx            # Public landing page
│   └── profile.tsx            # User's own campaigns
│
└── utils/
    └── index.ts               # daysLeft, calculateBarPercentage, checkIfImage
```

---

## 🔄 User Flows

### Campaign Lifecycle

```
User                    Frontend                  Smart Contract          Admin
 │                         │                           │                    │
 │  Fill campaign form      │                           │                    │
 │─────────────────────────▶│                           │                    │
 │                         │  Validate image URL        │                    │
 │                         │  (checkIfImage util)       │                    │
 │                         │                           │                    │
 │                         │  createCampaign(args)     │                    │
 │                         │──────────────────────────▶│                    │
 │                         │                           │ Stores campaign    │
 │                         │                           │ status = PENDING   │
 │                         │  ◀── tx confirmed ────────│                    │
 │                         │                           │                    │
 │  Redirect to /admin     │                           │                    │
 │◀────────────────────────│                           │                    │
 │                         │                           │                    │
 │                         │                           │  Admin reviews     │
 │                         │                           │◀───────────────────│
 │                         │                           │                    │
 │                         │                           │  approveCampaign() │
 │                         │                           │◀───────────────────│
 │                         │                           │ status = APPROVED  │
 │                         │                           │                    │
 │  Campaign visible on /home                          │                    │
 │◀─────────────────────────────────────────────────────                    │
```

---

### Donation Flow

```
Donor                   Frontend                   Smart Contract
  │                        │                             │
  │  Browse campaigns       │                             │
  │───────────────────────▶│                             │
  │                        │  getApprovedCampaigns()    │
  │                        │───────────────────────────▶│
  │                        │◀─── campaigns array ───────│
  │                        │                             │
  │  Click campaign card   │                             │
  │───────────────────────▶│  Navigate to /campaign-details/:title
  │                        │                             │
  │  getDonations(pId)     │                             │
  │                        │───────────────────────────▶│
  │                        │◀─── [donators, amounts] ───│
  │                        │                             │
  │  Enter donation amount │                             │
  │  Click "Donate Now"    │                             │
  │───────────────────────▶│                             │
  │                        │  donateToCampaign(pId)     │
  │                        │  { value: ethers.parseEther(amount) }
  │                        │───────────────────────────▶│
  │                        │                           │
  │                        │                     ETH transferred
  │                        │                     directly to campaign.owner
  │                        │                     amountCollected += amount
  │                        │                           │
  │                        │◀─── tx confirmed ─────────│
  │  Donation list updates │                             │
  │◀───────────────────────│                             │
```

---

### Admin Moderation Flow

```
Admin                   Frontend                   Blockchain
  │                        │                           │
  │  Connect wallet         │                           │
  │───────────────────────▶│                           │
  │                        │  hasRole(ADMIN_ROLE, addr)│
  │                        │──────────────────────────▶│
  │                        │◀─── true/false ───────────│
  │                        │                           │
  │  Navigate to /admin    │  (redirects if not admin) │
  │───────────────────────▶│                           │
  │                        │                           │
  │                        │  getPendingCampaigns()    │
  │                        │──────────────────────────▶│
  │                        │◀─── pending[] ────────────│
  │                        │                           │
  │  View campaign details │                           │
  │                        │                           │
  │  ┌─────────────────────────────────────────────┐  │
  │  │            DECISION POINT                   │  │
  │  │                                             │  │
  │  │  [Approve]              [Reject + Reason]   │  │
  │  └──────────┬──────────────────────┬───────────┘  │
  │             │                      │               │
  │             ▼                      ▼               │
  │  approveCampaign(id)    rejectCampaign(id, reason) │
  │             │                      │               │
  │             └────────────┬─────────┘               │
  │                          │──────────────────────────▶│
  │                          │◀─── tx confirmed ─────────│
  │                          │                           │
  │  Data refreshes          │  refreshCampaigns()      │
  │◀─────────────────────────│──────────────────────────▶│
```

---

### Authentication & Wallet Connection

```
User                    Thirdweb SDK              MetaMask / Wallet
  │                         │                           │
  │  Click "Connect Wallet" │                           │
  │────────────────────────▶│                           │
  │                         │  connect(metamaskWallet())│
  │                         │──────────────────────────▶│
  │                         │                     Prompt user
  │                         │◀─── wallet address ───────│
  │                         │                           │
  │  address stored in      │                           │
  │  StateContext           │                           │
  │                         │                           │
  │  useEffect triggers:    │                           │
  │  ├─ checkAdminStatus()  │                           │
  │  ├─ refreshCampaigns()  │                           │
  │  └─ isAdmin flag set    │                           │
  │                         │                           │
  │  UI updates:            │                           │
  │  ├─ Show "Create Campaign" button                   │
  │  ├─ Show /admin link (if isAdmin)                   │
  │  └─ Show wallet address in nav                      │
```

---

## ✨ Key Features

| Feature | Description |
|---|---|
| 🔐 **Wallet Authentication** | MetaMask/Web3 wallet connection via Thirdweb SDK |
| 📋 **Campaign Moderation** | Admin role-based approval/rejection before campaigns go live |
| 💸 **Direct Donations** | ETH sent directly to campaign owners — no escrow, no fees |
| 🔍 **Search & Filter** | Filter by category, sort by date/funding/deadline |
| 📊 **Live Stats** | Real-time aggregate stats — total raised, backers, active campaigns |
| ⏰ **Urgency Indicators** | "Ending Soon" and "Urgent" badges for campaigns near deadline |
| 👤 **Profile Page** | View all campaigns created by the connected wallet |
| 🛡 **Admin Dashboard** | Full overview table + pending review queue for admins |
| 📱 **Responsive Design** | Mobile-first UI with collapsible drawer navigation |
| 🔄 **Campaign Caching** | 5-second cache to reduce redundant RPC calls |

---

## 🛠 Tech Stack

### Frontend
| Layer | Technology | Version |
|---|---|---|
| Framework | React | 18.2 |
| Language | TypeScript | 4.7 |
| Build Tool | Vite | 3.x |
| Styling | TailwindCSS | 3.4 |
| Routing | React Router DOM | 6.22 |
| Icons | Lucide React | 0.483 |
| Animations | Framer Motion | 12.x |
| Notifications | Sonner | 1.4 |

### Blockchain
| Layer | Technology | Version |
|---|---|---|
| Network | Arbitrum (L2) | — |
| SDK | Thirdweb React | 4.x |
| Ethereum Library | ethers.js | 5.x |
| Smart Contract | Solidity | 0.8.20 |
| Access Control | OpenZeppelin | 5.3 |
| Dev Framework | Hardhat | 2.23 |

---

## 🚀 Getting Started

### Prerequisites

- Node.js `>= 18.0.0`
- Yarn or npm
- MetaMask browser extension
- Arbitrum testnet/mainnet ETH

### 1. Clone the Repository

```bash
git clone <repository-url>
cd crowdfunding
```

### 2. Install Frontend Dependencies

```bash
cd Frontend
yarn install
```

### 3. Configure Environment

Create a `.env` file in the `Frontend/` directory:

```env
VITE_TEMPLATE_CLIENT_ID=your_thirdweb_client_id
```

### 4. Start the Development Server

```bash
yarn dev
```

The app will be available at `http://localhost:5173`

### 5. Build for Production

```bash
yarn build
```

### 6. Deploy Frontend (via Thirdweb Storage)

```bash
yarn deploy
```

---

## 🔑 Environment Variables

| Variable | Description | Required |
|---|---|---|
| `VITE_TEMPLATE_CLIENT_ID` | Thirdweb API Client ID (get from [thirdweb.com/dashboard](https://thirdweb.com/dashboard)) | ✅ Yes |

---

## 📦 Contract Deployment

### 1. Install Web3 Dependencies

```bash
cd web3
yarn install
```

### 2. Configure `.env`

```env
PRIVATE_KEY=your_deployer_wallet_private_key
```

### 3. Deploy via Thirdweb

```bash
yarn deploy
```

> The deployed contract address must be updated in `Frontend/src/contexts/index.tsx`:
> ```typescript
> const { contract } = useContract("YOUR_CONTRACT_ADDRESS")
> ```


## 📄 Pages & Components

### Pages

| Route | Component | Access | Description |
|---|---|---|---|
| `/` | `LandingPage` | Public | Hero section with CTAs and stats |
| `/home` | `Home` | Public | Browse all approved campaigns |
| `/create-campaign` | `CreateCampaign` | Wallet connected | Campaign creation form |
| `/campaign-details/:title` | `CampaignDetails` | Public | Campaign info + donation panel |
| `/profile` | `Profile` | Wallet connected | User's own campaigns |
| `/admin` | `AdminDashboard` | Admin only | Moderation queue + overview |

### Core Components

| Component | Purpose |
|---|---|
| `DisplayCampaigns` | Renders campaign grid with filtering; shows active-only or all based on `showAllCampaigns` prop |
| `CampaignFilters` | Search input + expandable sort/category dropdowns |
| `CampaignStats` | 4-stat summary bar (total raised, success rate, active campaigns, total backers) |
| `TopNavbar` | Persistent nav with wallet connect and mobile drawer |
| `Loader` | Full-screen overlay during blockchain transactions |
| `FormField` | Reusable input/textarea with consistent styling |

---

## 🗂 State Management

All global state is managed via React Context (`StateContext`):

```typescript
StateContextType {
  // Wallet
  address: string | undefined
  connect: (wallet) => void
  disconnect: () => void

  // Contract
  contract: SmartContract | undefined

  // Campaigns
  getCampaigns: () => Promise<ParsedCampaign[]>
  getApprovedCampaigns: () => Promise<ParsedCampaign[]>
  getUserCampaigns: () => Promise<ParsedCampaign[]>
  getPendingCampaigns: () => Promise<ParsedCampaign[]>
  createCampaign: (form) => Promise<void>
  refreshCampaigns: () => Promise<void>

  // Donations
  donate: (pId, amount) => Promise<any>
  getDonations: (pId) => Promise<ParsedDonation[]>

  // Admin
  isAdmin: boolean
  adminCheckCompleted: boolean
  approveCampaign: (id) => Promise<void>
  rejectCampaign: (id, reason) => Promise<void>

  // UI
  searchCampaign: string
  setSearchCampaign: (value) => void
}
```

### Data Caching Strategy

```
Request for campaigns
        │
        ▼
  Cache age < 5 seconds?
        │
   YES  │  NO
        │   │
        │   ▼
        │  contract.call('getCampaigns')
        │   │
        │   ▼
        │  Update allCampaignsCache
        │  Update lastRefresh timestamp
        │   │
        └───┘
        │
        ▼
  Return cached data
```

---

## 🔒 Security Model

### Role-Based Access Control

```
DEFAULT_ADMIN_ROLE (deployer)
        │
        └── Can grant/revoke ADMIN_ROLE
        └── Can approve/reject campaigns

ADMIN_ROLE
        │
        └── Can approve campaigns
        └── Can reject campaigns (with reason)
        └── Can grant ADMIN_ROLE to others

Any Wallet Address
        │
        └── Can create campaigns (requires connected wallet)
        └── Can donate to APPROVED campaigns
        └── Cannot approve/reject
```

### Donation Security

- Donations only accepted on campaigns with `status == APPROVED`
- Donations only accepted before campaign `deadline`
- ETH transferred directly to `campaign.owner` via `call{value}` — no platform custody
- Failed transfers do not increment `amountCollected`

### Frontend Guards

- Admin dashboard checks `isAdmin` and `adminCheckCompleted` before rendering
- Wallet connection required for campaign creation
- Campaign status badges shown on all campaign cards for full transparency

---

## 📐 Campaign Card — Data Flow

```
Campaign (from blockchain)
        │
        ├── amountCollected / target  →  Progress bar % + "X% funded"
        ├── deadline (unix timestamp) →  daysLeft() util  →  "N days left"
        ├── status (0/1/2)            →  Status badge (Pending/Active/Rejected)
        ├── donators.length           →  Backer count
        ├── owner address             →  Avatar initials + shortened address
        └── image URL                 →  Card hero image
```

---

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-feature`
3. Commit your changes: `git commit -m 'feat: add your feature'`
4. Push to the branch: `git push origin feature/your-feature`
5. Open a Pull Request

---

## 📝 License

This project is open source. See [LICENSE](LICENSE) for details.

---

<div align="center">

Built with ❤️ on **Arbitrum** · Powered by **Thirdweb** · Secured by **OpenZeppelin**

</div>
