# Cureoreum – CureCoin 2.0 Blockchain Explorer

> **Cureoreum** is a lightweight, single-page blockchain explorer for the **CureCoin 2.0** network.  
> Built as a static **HTML5/JavaScript** application using **jQuery** and **Bootstrap 3**, it requires no build step and can be hosted anywhere. It provides real-time data visualization, search capabilities, and address analysis.

---

## 📑 Table of Contents
- [🚀 Features](#-features)
- [🛠 Tech Stack](#-tech-stack)
- [🏁 Quick Start](#-quick-start)
- [⚙️ Configuration](#️-configuration)
- [📡 API Specification](#️-api-specification)
- [🎨 Styling & Theming](#-styling--theming)
- [🧩 Search Logic](#-search-logic)
- [🔍 Code Architecture](#-code-architecture)
- [🚀 Deployment](#-deployment)
- [📄 License & Credits](#-license--credits)

---

## 🚀 Features

*   **Block Explorer**
    *   Displays latest blocks with metadata (Height, Hash, Time, Difficulty, Size, Minted CURE, Mint Method).
    *   Clickable links to view block details and contained transactions.
    *   Auto-refresh capability (30s interval) with a toggle switch.
*   **Transaction Viewer**
    *   Detailed view of `vin` (inputs) and `vout` (outputs).
    *   Parses Coinbase scripts and normal transaction inputs.
    *   Links to referenced transactions and addresses within the transaction details.
*   **Address Analysis**
    *   **Transaction History:** Paginated list of transactions involving the address.
    *   **UTXO Set:** View Unspent Transaction Outputs directly in the browser.
    *   **Balance Calculation:** Computes total balance client-side by summing UTXO values.
    *   **Balance Toggle:** Privacy feature to hide/show balance on the address page.
    *   **QR Code:** Generates a QR code for the address and provides a `cure:` payment link.
*   **Intelligent Search**
    *   Automatically detects query type using Regex validation:
        *   **Block Height:** Numeric only (`^\d+$`).
        *   **Tx/Block Hash:** 64-character Hex string.
        *   **Address:** 20-45 character length.
*   **Responsive UI**
    *   Custom media queries for tablets (`768px`) and mobile (`480px`).
    *   Horizontal scrolling tables for mobile devices.
    *   Sticky header and navigation tabs optimized for touch.
*   **Dark Mode**
    *   System-agnostic toggle that persists user preference via `localStorage`.
    *   Fully themed tables, backgrounds, and borders.

---

## 🛠 Tech Stack

| Component | Technology | Version |
|------|------|---|
| **Core** | HTML5 | 5.0 |
| **CSS Framework** | Bootstrap | 3.3.7 |
| **JavaScript Library** | jQuery | 3.6.0 |
| **Build System** | None | Static |
| **Storage** | Browser `localStorage` | N/A |
| **Networking** | `$.get` (AJAX) | jQuery |

---

## 🏁 Quick Start

This project is a **static single-file application**. No Node.js, NPM, or build tools are required.

1.  **Get the Code**
    ```bash
    git clone https://github.com/Cureoreum/cureoreum-explorer-frontend.git
    cd cureoreum-explorer-frontend
    ```

2.  **Run Locally**
    Because the app fetches data from a remote API, **CORS** may prevent it from running via `file://`. Use a simple local server:

    ```bash
    # Python 3
    python3 -m http.server 8000

    # Node (if installed)
    npx serve .

    # PHP
    php -S localhost:8000
    ```

3.  **Access**
    Open `http://localhost:8000` in your browser.

---

## ⚙️ Configuration

### 1. API Endpoint
The explorer defaults to the public Cureoreum API (This will cause cors issues, think of it more of a placeholder), the public cureoreum explorer is available at https://cureoreum.com

To use your own node or proxy:

1.  Open `index.html`.
2.  Scroll to the `<script>` tag near the bottom.
3.  Locate the `state` object and modify `API_BASE`:

```javascript
const state = {
    // ...
    API_BASE: 'https://cureoreum.com/api', // <-- CHANGE THIS
    // ...
};
```

### 2. Version Number
The version displayed in the footer is controlled by the `VERSION` constant.

```javascript
const VERSION = "0.9.9-prerelease";
```

### 3. Pagination Limits
You can adjust how many blocks or transactions are loaded per page by modifying `state.blocksPerPage` in the `state` object (default: 10 for blocks, 50 for address history).

---

## 📡 API Specification

The frontend expects the backend to provide JSON data conforming to the following structure.

| Endpoint | Method | Parameters | Description |
|------|------|------|------|
| `/blocks` | `GET` | `page`, `limit` | Returns array of recent blocks. Includes `total` count. |
| `/blocks/height` | `GET` | None | Returns current chain height (`{ "height": 12345 }`). |
| `/block/{height}` | `GET` | None | Returns block details. Must include `hash`, `time`, `txCount`. |
| `/block/{height}/txs` | `GET` | None | Returns array of transactions within a block. |
| `/tx/{txid}` | `GET` | None | Returns transaction object. Must include `vin`, `vout`, `block_height`. |
| `/address/{address}` | `GET` | `page` | Returns transaction history for address. Supports `txs` array. |
| `/address/{address}/txs/count` | `GET` | None | Returns total transaction count (`{ "count": 100 }`). |
| `/utxo/{address}` | `GET` | None | Returns array of Unspent Outputs (`{ "value": ..., "txid": ... }`). |
| `/qr/{address}` | `GET` | None | Returns a QR code image (URL or Base64 data). |

> **Note on JSON Resilience:** The code includes logic to handle slight variations in JSON keys (e.g., `txs` vs `transactions`, `txid` vs `hash`) to maximize compatibility with backend implementations.

---

## 🎨 Styling & Theming

The explorer uses custom CSS classes to manage themes.

### Dark Mode
*   **Trigger:** Clicking the header button toggles `state.darkMode`.
*   **Persistence:** Saved in `localStorage` under key `cureoreumExplorerDark`.
*   **Implementation:** Adds/removes class `dark-mode` to `<body>`.
*   **Custom CSS:**
    *   Background changes to `#121212`.
    *   Text color becomes `#e0e0e0`.
    *   Tables and blocks use `#1e1e1e` and `#2c2c2c` backgrounds.
    *   Borders adjust to `#333` or `#444`.

### Responsive Breakpoints
*   **Desktop:** Default Bootstrap 3 grid.
*   **Tablet (`≤ 768px`):**
    *   Font sizes reduced (13px -> 12px).
    *   Navigation tabs shrink padding.
    *   Search input/button padding adjusted.
*   **Mobile (`≤ 480px`):**
    *   Font sizes further reduced (12px -> 11px).
    *   Tables utilize `table-responsive` for horizontal scrolling.
    *   Headers collapse to single column.

---

## 🧩 Search Logic

The `search()` function validates input before querying the API to ensure efficiency:

1.  **Block Height Check:**
    *   Regex: `/^\d+$/`
    *   If match: Queries `/block/{q}`.
2.  **Hash Check (Tx or Block):**
    *   Regex: `/^[0-9a-fA-F]{64}$/`
    *   If match: Queries `/tx/{q}`. If fails, retries as `/block/{q}`.
3.  **Address Check:**
    *   Length Check: `20 <= length <= 45`
    *   If match: Queries `/address/{q}`.
4.  **Fallback:**
    *   Treats as Transaction ID if all else fails.

---

## 🔍 Code Architecture

The application logic is contained within a global `state` object and several functional modules.

### State Object
```javascript
const state = {
    activeTab: 'blocks',       // Current view
    blocks: [],                // Cached block data
    currentBlock: null,        // Selected block data
    currentTx: null,           // Selected tx data
    currentAddressHistory: {}, // Address history data
    currentUtxos: [],          // UTXO data
    API_BASE: '...',           // Endpoint URL
    autoRefreshEnabled: true,  // Auto-update flag
    darkMode: false,           // Theme preference
    // ... pagination data
};
```

### Key Functions
*   `init()`: Initializes UI, loads first page of blocks, starts refresh timer.
*   `setActiveTab()`: Manages Bootstrap tab switching and content visibility.
*   `loadLatestBlocks()`: Fetches block data and renders the list with pagination.
*   `search()`: Orchestrates the query validation and routing.
*   `computeBalance()`: Sums `utxo.value` for address balance display.
*   `applyTheme()`: Toggles CSS classes based on `state.darkMode`.

### Event Handling
*   **Tabs:** Click events delegate to `setActiveTab`.
*   **Search:** Click on `#searchButton` or `Enter` key in `#searchQuery`.
*   **Navigation:** Links use `e.preventDefault()` to intercept clicks and update state without page reloads.

---

## 🚀 Deployment

Since this is a static file, it can be hosted on virtually any platform.

### GitHub Pages
1.  Create a new repository.
2.  Upload `index.html`.
3.  Go to **Settings > Pages**.
4.  Select the branch and save.
5.  *Note:* You may need to configure the `API_BASE` to a public endpoint to avoid CORS issues if not using a proxy.

### Vercel / Netlify
1.  Drag and drop the `index.html` file into the dashboard.
2.  Deploy.
3.  These platforms automatically handle HTTPS, which is recommended for production.

### Self-Hosted
If running behind a proxy (e.g., Nginx), ensure you do not rewrite the paths for the API calls, as they are hardcoded relative to `API_BASE`.

---

## 📄 License & Credits

*   **License:** MIT (Check `LICENSE` file in repo).
*   **Credits:**
    *   **Cureoreum** - Data Aggregator and service provider
    *   **CureCoin** – Network & Protocol.
    *   **Bootstrap & jQuery** – Core UI Components.

---

## 🐛 Known Issues & Limitations

*   **CORS:** The browser may block API requests if running locally without a proxy and the API does not support `Access-Control-Allow-Origin: *`.
*   **Balance Calculation:** Balance is computed client-side from the UTXO set provided by the API. If the API UTXO endpoint is slow or limited, balance updates may lag.
*   **Single Page:** All JavaScript is inline. For easier maintenance, consider splitting `styles.css` and `script.js` into separate files, then referencing them in `index.html`.
*   **Browser Support:** Requires a modern browser supporting ES6 syntax and `localStorage` (IE11+ compatible due to jQuery 3.6 usage).
