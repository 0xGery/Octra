# OCTRA WALLET EXTENSION - COMPLETE DOCUMENTATION

## **ARCHITECTURE OVERVIEW**

The Octra Wallet Extension is a Chrome browser extension that provides cryptocurrency wallet functionality with advanced features like private balance encryption, mnemonic backup, and secure transaction handling.

### **Core Technologies:**
- **Frontend**: Vanilla JavaScript, CSS3, HTML5
- **Cryptography**: Ed25519 (via TweetNaCl), BIP39 mnemonic generation
- **Storage**: Chrome Extension Storage API (encrypted)
- **Network**: REST API communication with Octra blockchain
- **Architecture**: Component-based modular design

---

## **FILE STRUCTURE & RESPONSIBILITIES**

### **CORE FILES**

#### **`popup.html` - Main UI Structure**
```html
<!-- Extension popup interface (400x600px) -->
```
**Purpose**: Defines the complete UI structure for the wallet extension  
**Key Screens**:
- `main-screen` - Main wallet interface with balance, send/receive
- `first-time-setup` - Initial wallet creation/import  
- `wallet-list-screen` - Multi-wallet management
- `send-screen` - Transaction sending interface
- `receive-screen` - Address display with QR code
- `settings-screen` - Wallet settings and security
- `wallet-details-screen` - Recovery phrase and private key display
- `password-unlock-screen` - Password entry for locked wallets

**Special Elements**:
- `message-overlay` - Modal overlay for verification/messages
- `loading-overlay` - Loading indicator during operations
- Private balance sections for encrypted balance features

---

#### **`popup.js` - Extension Entry Point**
```javascript
// Main initialization and startup logic
```
**Purpose**: Extension bootstrap and initialization coordinator  
**Key Functions**:
- `initializePopup()` - Main initialization sequence
- `checkWalletLockStatus()` - Determines if wallet needs unlocking
- `setupGlobalErrorHandlers()` - Error boundary setup
- `setupKeyboardShortcuts()` - Keyboard navigation

**Initialization Flow**:
1. DOM readiness check
2. Library availability verification  
3. UI Manager creation and initialization
4. Error handler setup
5. Session state restoration

---

### **STYLING**

#### **`css/main.css` - Core Styling**
```css
/* Glassmorphism design system with purple-blue gradient theme */
```
**Design System**:
- **Theme**: Purple-blue gradient (`#667eea` → `#764ba2`)
- **Style**: Glassmorphism with backdrop filters
- **Layout**: Fixed 400x600px popup dimensions
- **Typography**: System fonts with responsive sizing

**CSS Variables**:
```css
:root {
    --primary-gradient: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    --glass-bg: rgba(255, 255, 255, 0.15);
    --text-primary: #2d3748;
    --success-color: #10b981;
    --error-color: #ef4444;
}
```

**Z-Index Hierarchy**:
- Base content: 1
- Navigation: 200  
- Content: 300
- Dropdowns: 1000
- Modals: 6000
- Loading: 7000

#### **`css/dialogs.css` - Modal & Dialog Styling**
```css
/* Specialized styling for overlays, modals, and dialogs */
```
**Components**:
- Message overlays with glassmorphism effects
- Loading spinners and progress indicators
- Form validation styling
- Button states and interactions

---

### **CORE LOGIC MODULES**

#### **`js/ui.js` - UI Management (Primary Controller)**
```javascript
class UIManager {
    constructor() {
        this.wallet = new OctraWallet();
        this.passwordManager = new PasswordManager();
        this.walletListManager = new WalletListManager();
        // ... initialization
    }
}
```

**Purpose**: Central UI controller coordinating all user interactions  
**Key Responsibilities**:
- Screen navigation and transitions
- Form handling and validation
- Wallet creation and import flows
- Error display and user feedback
- Session management and auto-lock

**Critical Methods**:

##### **Wallet Creation Flow**
```javascript
async createInitialWallet() {
    // 1. Generate wallet data with mnemonic
    const walletData = await window.crypto.generateWallet();
    
    // 2. Store wallet in manager
    const wallet = walletManager.walletStorage.createWalletObject(walletData, name);
    walletManager.wallets.push(wallet);
    
    // 3. Show wallet details for backup
    this.showWalletDetails(this.currentWalletData, 'created');
}
```

##### **Screen Management**
```javascript
async showScreen(screenId) {
    // Safe screen transitions with DOM utilities
    await window.domUtils.safeShowScreen(screenId, this.currentScreen);
    this.currentScreen = screenId;
    await this.updateScreenContent(screenId);
}
```

##### **Authentication Integration**
```javascript
async init() {
    // Use centralized authentication checking
    const authStatus = await WalletService.getAuthStatus(this.passwordManager);
    
    if (!authStatus.isPasswordSet) {
        this.showScreen('first-time-setup');
    } else if (!authStatus.isUnlocked) {
        this.showScreen('password-unlock-screen');
    }
}
```

##### **Mnemonic Verification System**
```javascript
async showSimpleMnemonicVerification() {
    // Select 3 random word positions
    const positions = [];
    while (positions.length < 3) {
        const pos = Math.floor(Math.random() * mnemonic.length) + 1;
        if (!positions.includes(pos)) positions.push(pos);
    }
    
    // Create verification overlay UI
    // User must enter correct words to proceed
}
```

**Form Handling Patterns**:
- Async form submission with loading states
- Input validation with real-time feedback
- Error display with user-friendly messages
- Progress tracking for multi-step processes

---

#### **`js/wallet.js` - Core Wallet Functionality**
```javascript
class OctraWallet {
    constructor() {
        this.crypto = new CryptoManager();
        this.network = new NetworkClient();
        this.cache = new Map();
    }
}
```

**Purpose**: Core wallet operations and blockchain interaction  
**Key Features**:
- Balance checking with caching (30s TTL)
- Transaction creation and signing
- Address validation and formatting
- Private balance encryption/decryption
- Network communication handling

**Critical Methods**:

##### **Transaction Creation**
```javascript
async createTransaction(toAddress, amount, message = '') {
    // 1. Get current balance and nonce
    const balanceData = await this.getBalance();
    
    // 2. Validate sufficient funds (including fees)
    const totalRequired = amount + this.calculateFee(amount);
    if (balanceData.balance < totalRequired) {
        throw new Error('Insufficient funds');
    }
    
    // 3. Create transaction object
    const transaction = {
        from: this.address,
        to_: toAddress,
        amount: amount.toString(),
        nonce: (balanceData.nonce + 1).toString(),
        ou: this.getFeeLevel(amount),
        timestamp: this.generateTimestamp(),
        message: message,
        public_key: this.crypto.getPublicKey()
    };
    
    // 4. Sign transaction (excluding message field)
    const signData = { ...transaction };
    delete signData.message;
    transaction.signature = this.crypto.signTransaction(signData);
    
    return transaction;
}
```

##### **Private Balance Operations**
```javascript
async encryptBalance(amount) {
    // Convert public balance to encrypted using AES-GCM
    const encryptionKey = this.deriveEncryptionKey();
    const encryptedData = await this.crypto.encryptData(amount, encryptionKey);
    
    return await this.network.encryptBalance(this.address, encryptedData);
}

async createPrivateTransfer(toAddress, amount) {
    // Create encrypted transfer using ephemeral keys
    const ephemeralKeyPair = this.crypto.generateEphemeralKeys();
    const sharedSecret = this.crypto.deriveSharedSecret(ephemeralKeyPair, toAddress);
    const encryptedAmount = this.crypto.encryptWithSharedSecret(amount, sharedSecret);
    
    return await this.network.createPrivateTransfer(toAddress, encryptedAmount);
}
```

**Caching Strategy**:
- Balance/nonce: 30 seconds
- Transaction history: 60 seconds  
- Private balance data: 45 seconds
- Cache invalidation on transaction sending

---

#### **`js/crypto.js` - Cryptographic Operations**
```javascript
class CryptoManager {
    constructor(privateKey = null) {
        this.privateKey = privateKey;
        this.publicKey = null;
        this.signingKey = null;
    }
}
```

**Purpose**: All cryptographic operations using TweetNaCl and BIP39  
**Key Features**:
- Ed25519 key generation and signing
- BIP39 mnemonic generation and validation
- AES-GCM encryption for private balances
- Address derivation and validation

**Critical Methods**:

##### **Wallet Generation (BIP39 Standard)**
```javascript
async generateKeyPair() {
    // 1. Generate entropy (128 bits)
    const entropy = crypto.getRandomValues(new Uint8Array(16));
    
    // 2. Create BIP39 mnemonic
    const mnemonic = await this.entropyToMnemonic(entropyHex);
    
    // 3. Derive seed from mnemonic
    const seed = await this.mnemonicToSeed(mnemonic);
    
    // 4. Derive master key using "Octra seed" HMAC-SHA512
    const masterKey = await this.deriveMasterKey(seed);
    
    // 5. Create Ed25519 key pair
    this.signingKey = nacl.sign.keyPair.fromSeed(masterKey.privateKey);
    
    // 6. Generate Octra address
    const address = this.generateOctraAddress(this.signingKey.publicKey);
    
    return {
        privateKeyBase64: this.bytesToBase64(masterKey.privateKey),
        publicKeyBase64: this.bytesToBase64(this.signingKey.publicKey),
        address: address,
        mnemonic: mnemonic,
        mnemonicWords: mnemonic.split(' ')
    };
}
```

##### **Transaction Signing**
```javascript
signTransaction(transactionData) {
    // 1. Create canonical JSON (deterministic ordering)
    const canonicalJson = this.createCanonicalJson(transactionData);
    
    // 2. Convert to bytes
    const messageBytes = new TextEncoder().encode(canonicalJson);
    
    // 3. Sign with Ed25519
    const signature = nacl.sign.detached(messageBytes, this.signingKey.secretKey);
    
    return this.bytesToBase64(signature);
}
```

**Address Generation**:
```javascript
generateOctraAddress(publicKey) {
    // 1. SHA256 hash of public key
    const hash = crypto.subtle.digest('SHA-256', publicKey);
    
    // 2. Take first 25 bytes
    const addressBytes = new Uint8Array(hash).slice(0, 25);
    
    // 3. Base58 encode with 'oct' prefix
    return 'oct' + this.base58Encode(addressBytes);
}
```

---

#### **`js/network.js` - Blockchain Communication**
```javascript
class NetworkClient {
    constructor() {
        this.baseUrl = 'https://octra.network';
        this.timeout = 30000;
    }
}
```

**Purpose**: All network communication with Octra blockchain  
**Key Features**:
- RESTful API communication
- Request/response handling with retries
- Error parsing and user-friendly messages
- Private endpoints with authentication headers

**API Endpoints**:

##### **Public Endpoints**
```javascript
// Get balance and nonce
GET /balance/{address}
Response: { balance: number, nonce: number }

// Send transaction  
POST /send-tx
Body: { from, to_, amount, nonce, ou, timestamp, message, signature, public_key }

// Get transaction history
GET /address/{address}  
Response: Array of transaction objects

// Get pending transactions
GET /staging
Response: Array of pending transactions
```

##### **Private Endpoints (Require X-Private-Key Header)**
```javascript
// View encrypted balance
GET /view_encrypted_balance/{address}
Header: X-Private-Key: {privateKey}

// Encrypt public balance
POST /encrypt_balance
Body: { address, amount }
Header: X-Private-Key: {privateKey}

// Create private transfer
POST /private_transfer  
Body: { to_address, encrypted_amount, ephemeral_public_key }
Header: X-Private-Key: {privateKey}

// Claim private transfer
POST /claim_private_transfer
Body: { recipient_address, private_key, transfer_id }
Header: X-Private-Key: {privateKey}
```

**Error Handling**:
```javascript
async makeRequest(endpoint, options = {}) {
    try {
        const response = await fetch(this.baseUrl + endpoint, {
            timeout: this.timeout,
            ...options
        });
        
        if (!response.ok) {
            throw new Error(`HTTP ${response.status}: ${response.statusText}`);
        }
        
        return await response.json();
    } catch (error) {
        // Convert technical errors to user-friendly messages
        throw this.parseNetworkError(error);
    }
}
```

---

#### **`js/password-manager.js` - Security & Session Management**
```javascript
class PasswordManager {
    constructor() {
        this.isSetup = false;
        this.sessionKey = null;
        this.walletManager = null;
        this.autoLockTimer = null;
    }
}
```

**Purpose**: Password security, session management, and auto-lock functionality  
**Key Features**:
- PBKDF2 password hashing with salt
- Session key generation for encryption
- Auto-lock timer with configurable duration
- Secure storage of unlock status

**Critical Methods**:

##### **Password Setup**
```javascript
async setPassword(password) {
    // 1. Generate random salt
    const salt = crypto.getRandomValues(new Uint8Array(16));
    
    // 2. Derive key using PBKDF2
    const passwordKey = await crypto.subtle.deriveKey(
        {
            name: 'PBKDF2',
            salt: salt,
            iterations: 100000,
            hash: 'SHA-256'
        },
        await crypto.subtle.importKey('raw', new TextEncoder().encode(password), 'PBKDF2', false, ['deriveKey']),
        { name: 'AES-GCM', length: 256 },
        false,
        ['encrypt', 'decrypt']
    );
    
    // 3. Generate session key for wallet encryption
    this.sessionKey = crypto.getRandomValues(new Uint8Array(32));
    
    // 4. Store encrypted session key
    await this.storeEncryptedSessionKey(passwordKey);
}
```

##### **Auto-Lock System**
```javascript
startAutoLockTimer(durationSeconds = 300) {
    this.clearAutoLockTimer();
    
    this.autoLockTimer = setTimeout(() => {
        this.lockWallet();
    }, durationSeconds * 1000);
    
    // Update activity on user interaction
    this.updateLastActivity();
}

updateLastActivity() {
    this.lastActivity = Date.now();
    // Store in chrome.storage for persistence
    chrome.storage.local.set({ lastActivity: this.lastActivity });
}
```

---

#### **`js/wallet-manager.js` - Multi-Wallet Management**
```javascript
class WalletManager {
    constructor() {
        this.wallets = [];
        this.activeWallet = null;
        this.activeWalletId = null;
        this.walletStorage = new WalletStorage();
    }
}
```

**Purpose**: Managing multiple wallets within the extension  
**Key Features**:
- Multi-wallet storage and retrieval
- Active wallet tracking
- Wallet metadata management
- Import/export functionality

**Critical Methods**:

##### **Wallet Creation**
```javascript
async createWallet(name, password, setAsActive = true) {
    // 1. Generate wallet data
    const walletData = await window.crypto.generateWallet();
    
    // 2. Validate uniqueness
    const isAddressUnique = await this.walletStorage.isWalletAddressUnique(walletData.address);
    if (!isAddressUnique) {
        throw new Error('Generated wallet address already exists');
    }
    
    // 3. Create wallet object
    const wallet = this.walletStorage.createWalletObject(walletData, name);
    
    // 4. Add to collection
    this.wallets.push(wallet);
    
    // 5. Set as active if requested
    if (setAsActive) {
        this.setActiveWallet(wallet.id);
    }
    
    // 6. Save to storage
    await this.walletStorage.storeWallets(this.wallets, password, this.activeWalletId);
    
    return { success: true, wallet };
}
```

##### **Wallet Import**
```javascript
async importWallet(name, privateKey, address, password, setAsActive = true) {
    // 1. Validate import data
    if (!this.crypto.setPrivateKey(privateKey)) {
        throw new Error('Invalid private key format');
    }
    
    // 2. Create wallet object
    const walletData = {
        name: name,
        address: address,
        privateKey: privateKey,
        publicKey: this.crypto.getPublicKey(),
        createdAt: Date.now(),
        isImported: true
    };
    
    const wallet = this.walletStorage.createWalletObject(walletData, name);
    wallet.metadata.source = 'imported_key';
    
    // 3. Add and store
    this.wallets.push(wallet);
    await this.walletStorage.storeWallets(this.wallets, password, this.activeWalletId);
    
    return { success: true, wallet };
}
```

**Wallet Object Structure**:
```javascript
{
    id: 'unique-id',
    name: 'User-provided name',
    address: 'oct...',
    privateKey: 'base64-encoded-key',
    publicKey: 'base64-encoded-key', 
    createdAt: timestamp,
    updatedAt: timestamp,
    isActive: boolean,
    metadata: {
        source: 'generated' | 'imported_key' | 'imported_mnemonic',
        version: '1.0',
        backupVerified: boolean
    }
}
```

---

#### **`js/wallet-storage.js` - Secure Storage Layer**
```javascript
class WalletStorage {
    constructor() {
        this.MAX_WALLETS = 5;
        this.STORAGE_VERSION = '1.0';
    }
}
```

**Purpose**: Encrypted storage and retrieval of wallet data  
**Key Features**:
- AES-GCM encryption of sensitive data
- Corruption detection and recovery
- Version management and migration
- Secure key derivation

**Critical Methods**:

##### **Secure Storage**
```javascript
async storeWallets(wallets, sessionKey, activeWalletId) {
    // 1. Prepare storage data
    const storageData = {
        wallets: wallets,
        activeWalletId: activeWalletId,
        version: this.STORAGE_VERSION,
        timestamp: Date.now()
    };
    
    // 2. Encrypt sensitive data
    const encryptedData = await this.encryptStorageData(storageData, sessionKey);
    
    // 3. Store in chrome.storage.local
    await chrome.storage.local.set({
        'octra_wallets': encryptedData,
        'wallet_count': wallets.length
    });
}
```

##### **Data Encryption**
```javascript
async encryptStorageData(data, sessionKey) {
    // 1. Generate random IV
    const iv = crypto.getRandomValues(new Uint8Array(12));
    
    // 2. Convert session key to CryptoKey
    const key = await crypto.subtle.importKey(
        'raw', sessionKey, 'AES-GCM', false, ['encrypt']
    );
    
    // 3. Encrypt data
    const dataString = JSON.stringify(data);
    const dataBuffer = new TextEncoder().encode(dataString);
    const encryptedBuffer = await crypto.subtle.encrypt(
        { name: 'AES-GCM', iv: iv }, key, dataBuffer
    );
    
    // 4. Combine IV and encrypted data
    return {
        iv: Array.from(iv),
        data: Array.from(new Uint8Array(encryptedBuffer))
    };
}
```

---

#### **`js/wallet-list.js` - Wallet List UI Management**
```javascript
class WalletListManager {
    constructor() {
        this.passwordManager = null;
        this.crypto = new CryptoManager();
        this.currentWalletMenus = new Map();
    }
}
```

**Purpose**: UI management for wallet list screen and operations  
**Key Features**:
- Wallet card rendering and interactions
- Wallet switching and activation
- Wallet deletion and renaming
- Export functionality

**Critical Methods**:

##### **Wallet List Display**
```javascript
async showWalletList() {
    const walletManager = this.getWalletManager();
    const wallets = walletManager.getAllWallets();
    
    const container = await window.domUtils.getWalletListContainer();
    container.innerHTML = '';
    
    wallets.forEach(wallet => {
        const walletCard = this.createWalletCard(wallet);
        container.appendChild(walletCard);
    });
}

createWalletCard(wallet) {
    const card = document.createElement('div');
    card.className = 'wallet-card';
    card.innerHTML = `
        <div class="wallet-header">
            <h3>${wallet.name}</h3>
            ${wallet.isActive ? '<span class="active-badge">Active</span>' : ''}
        </div>
        <div class="wallet-address">${this.formatAddress(wallet.address)}</div>
        <div class="wallet-actions">
            <button onclick="setActiveWallet('${wallet.id}')">Activate</button>
            <button onclick="showWalletMenu('${wallet.id}')">⋮</button>
        </div>
    `;
    return card;
}
```

---

#### **`js/wallet-service.js` - Centralized Service Layer**
```javascript
class WalletService {
    constructor() {
        this.crypto = new CryptoManager();
        this.storage = new WalletStorage();
    }
}
```

**Purpose**: Centralized service layer eliminating redundancy across components  
**Key Features**:
- Unified wallet creation and import
- Centralized authentication checking
- Consolidated form validation
- Standardized error handling

**Critical Methods**:

##### **Unified Wallet Creation**
```javascript
async createWallet(params) {
    const { walletName, isFirstWallet, importData, walletManager, sessionKey } = params;
    
    // 1. Validation
    this.validateWalletCreationInputs({ walletName, importData });
    
    // 2. Check limits and duplicates
    const walletCount = walletManager?.getWalletCount() || 0;
    if (walletCount >= 5) {
        throw new Error('Maximum of 5 wallets allowed');
    }
    
    const existingWallets = walletManager.getAllWallets();
    const nameExists = existingWallets.some(wallet => 
        wallet.name.toLowerCase() === walletName.toLowerCase()
    );
    if (nameExists) {
        throw new Error('A wallet with this name already exists');
    }
    
    // 3. Generate or import wallet
    let walletData;
    if (importData) {
        walletData = await this.importWallet(importData, walletName);
    } else {
        walletData = await this.generateNewWallet(walletName);
    }
    
    // 4. Store wallet
    const storedWallet = await this.storeWallet(walletData, walletManager, isFirstWallet, importData, sessionKey);
    
    return {
        success: true,
        walletData: storedWallet || walletData,
        isFirstWallet,
        message: `${importData ? 'Imported' : 'Created'} wallet "${walletName}" successfully`
    };
}
```

##### **Centralized Authentication**
```javascript
static async getAuthStatus(passwordManager) {
    if (!passwordManager) {
        return {
            isAuthenticated: false,
            isPasswordSet: false,
            isUnlocked: false,
            needsSetup: true
        };
    }

    try {
        const isPasswordSet = await passwordManager.isPasswordSet();
        const isUnlocked = passwordManager.isWalletUnlocked();
        
        return {
            isAuthenticated: isPasswordSet && isUnlocked,
            isPasswordSet,
            isUnlocked,
            needsSetup: !isPasswordSet
        };
    } catch (error) {
        console.error('Failed to check auth status:', error);
        return {
            isAuthenticated: false,
            isPasswordSet: false,
            isUnlocked: false,
            needsSetup: true,
            error: error.message
        };
    }
}
```

##### **Consolidated Form Validation**
```javascript
static validateForm(formData) {
    const errors = [];

    // Password validation
    if (formData.password !== undefined) {
        if (formData.password && formData.password.length < 4) {
            errors.push('Password must be at least 4 characters long');
        }
        
        if (formData.confirmPassword !== undefined) {
            if (formData.password !== formData.confirmPassword) {
                errors.push('Passwords do not match');
            }
        }
    }

    // Wallet name validation
    if (formData.walletName !== undefined) {
        if (!formData.walletName || !formData.walletName.trim()) {
            errors.push('Please enter a wallet name');
        } else if (formData.walletName.length > 30) {
            errors.push('Wallet name must be 30 characters or less');
        }
    }

    // Address validation
    if (formData.address !== undefined) {
        if (!formData.address || !formData.address.trim()) {
            errors.push('Please enter an address');
        } else if (!formData.address.startsWith('oct')) {
            errors.push('Invalid address format (must start with "oct")');
        }
    }

    return {
        isValid: errors.length === 0,
        errors
    };
}
```

---

#### **`js/dom-utils.js` - Safe DOM Access Layer**
```javascript
class DOMUtils {
    constructor() {
        this.elementCache = new Map();
        this.observerRegistry = new Map();
    }
}
```

**Purpose**: Safe DOM access with fallback mechanisms and error handling  
**Key Features**:
- Element caching for performance
- Retry mechanisms for element access
- Screen transition safety
- Event listener management

**Critical Methods**:

##### **Safe Element Access**
```javascript
async safeGetElement(selectors, options = {}) {
    const {
        timeout = 5000,
        retryInterval = 100,
        useCache = true,
        required = false,
        parent = document
    } = options;

    const selectorArray = Array.isArray(selectors) ? selectors : [selectors];
    const cacheKey = selectorArray.join('|');

    // Check cache first
    if (useCache && this.elementCache.has(cacheKey)) {
        const cached = this.elementCache.get(cacheKey);
        if (this.isElementVisible(cached)) {
            return cached;
        } else {
            this.elementCache.delete(cacheKey);
        }
    }

    return new Promise((resolve, reject) => {
        let attempts = 0;
        const maxAttempts = Math.ceil(timeout / retryInterval);

        const tryFind = () => {
            attempts++;

            for (const selector of selectorArray) {
                try {
                    const element = parent.querySelector(selector);
                    if (element && this.isElementAccessible(element)) {
                        if (useCache) {
                            this.elementCache.set(cacheKey, element);
                        }
                        resolve(element);
                        return;
                    }
                } catch (error) {
                    console.warn(`Error querying selector ${selector}:`, error);
                }
            }

            if (attempts >= maxAttempts) {
                if (required) {
                    reject(new Error(`Element not found: ${selectorArray.join(', ')}`));
                } else {
                    resolve(null);
                }
            } else {
                setTimeout(tryFind, retryInterval);
            }
        };

        tryFind();
    });
}
```

##### **Safe Screen Transitions**
```javascript
async safeShowScreen(targetScreenId, currentScreenId = null) {
    try {
        await this.waitForDOMReady();

        // Hide current screen
        if (currentScreenId) {
            const currentScreen = await this.safeGetElement(`#${currentScreenId}`, { required: false });
            if (currentScreen) {
                currentScreen.classList.add('hidden');
            }
        } else {
            // Hide all screens
            const allScreens = document.querySelectorAll('.screen');
            allScreens.forEach(screen => screen.classList.add('hidden'));
        }

        // Show target screen
        const targetScreen = await this.getScreenElement(targetScreenId);
        targetScreen.classList.remove('hidden');

        console.log(`Screen transition: ${currentScreenId || 'any'} → ${targetScreenId}`);
        return true;

    } catch (error) {
        console.error(`Failed to show screen ${targetScreenId}:`, error);
        throw error;
    }
}
```

---

### **UTILITY MODULES**

#### **`js/config.js` - Configuration Management**
```javascript
const OCTRA_CONFIG = {
    NETWORK: {
        PRIMARY_RPC: 'https://octra.network',
        BACKUP_RPC: 'https://backup.octra.network',
        TIMEOUT: 30000
    },
    WALLET: {
        MAX_WALLETS: 5,
        CACHE_TTL: 30000,
        AUTO_LOCK_DEFAULT: 300
    },
    TRANSACTION: {
        FEE_LEVELS: {
            1: 0.0001, // Low fee
            3: 0.001   // High fee
        },
        TIMESTAMP_OFFSET_RANGE: 1000
    }
};
```

#### **`js/error-handler.js` - Global Error Management**
```javascript
class ErrorHandler {
    constructor() {
        this.listeners = [];
        this.errorQueue = [];
        this.setupGlobalHandlers();
    }
    
    setupGlobalHandlers() {
        // Unhandled promise rejections
        window.addEventListener('unhandledrejection', (event) => {
            this.handleError('promise', event.reason, event);
        });
        
        // JavaScript errors
        window.addEventListener('error', (event) => {
            this.handleError('javascript', event.error, event);
        });
    }
}
```

---

## **CRITICAL WORKFLOWS**

### 1. **Wallet Creation Flow**
```
User clicks Create Wallet → Show wallet type selection → Generate wallet data + mnemonic → 
Show wallet details screen → User clicks Continue to Wallet → Show mnemonic verification → 
User enters 3 words → Words correct? → Complete setup, go to main screen
```

### 2. **Authentication Flow**
```
Extension opens → Password set? → No: Show first-time setup
                              → Yes: Wallet unlocked? → No: Show unlock screen
                                                      → Yes: Wallets exist? → No: Show first-time setup
                                                                            → Yes: Show main screen
```

### 3. **Transaction Flow**
```
User enters recipient + amount → Validate inputs → Check balance + fees → 
Create transaction object → Sign transaction → Submit to network → 
Show confirmation → Update balance cache
```

### 4. **Private Balance Flow**
```
User has public balance → Click Encrypt Balance → Enter amount to encrypt → 
Generate encryption key → Submit encrypt request → Balance moved to private → Update display
```

---

## **SECURITY ARCHITECTURE**

### **Cryptographic Standards**
- **Key Generation**: Ed25519 with BIP39 mnemonic
- **Transaction Signing**: Ed25519 detached signatures
- **Storage Encryption**: AES-GCM with PBKDF2 key derivation
- **Private Balances**: AES-GCM with derived keys + ephemeral key pairs

### **Key Management**
```javascript
// Password → Session Key → Wallet Encryption
PBKDF2(password + salt, 100000 iterations) → passwordKey
generateRandom(32 bytes) → sessionKey
AES-GCM(sessionKey, passwordKey) → encryptedSessionKey
AES-GCM(walletData, sessionKey) → encryptedWallets
```

### **Auto-Lock System**
- Configurable timeout (5 minutes default)
- Activity tracking on user interactions
- Secure session invalidation
- Persistent lock state across extension restarts

### **Content Security Policy**
```json
{
    "content_security_policy": {
        "extension_pages": "script-src 'self'; object-src 'self'; style-src 'self' 'unsafe-inline'"
    }
}
```

---

## **DATA STRUCTURES**

### **Transaction Object**
```javascript
{
    from: "oct...",                    // Sender address
    to_: "oct...",                     // Recipient address  
    amount: "1000000",                 // Amount in micro-OCT
    nonce: "123",                      // Transaction nonce
    ou: "1",                           // Fee level (1 or 3)
    timestamp: "1640995200000",        // Unix timestamp
    message: "Optional message",        // Transaction message
    signature: "base64...",            // Ed25519 signature
    public_key: "base64..."            // Sender public key
}
```

### **Wallet Object**
```javascript
{
    id: "uuid-v4",                     // Unique identifier
    name: "Main Wallet",               // User-provided name
    address: "oct...",                 // Wallet address
    privateKey: "base64...",           // Ed25519 private key
    publicKey: "base64...",            // Ed25519 public key
    createdAt: 1640995200000,          // Creation timestamp
    updatedAt: 1640995200000,          // Last update timestamp
    isActive: true,                    // Currently active wallet
    mnemonic: "word1 word2...",        // BIP39 mnemonic (if generated)
    mnemonicWords: ["word1", "word2"], // Mnemonic as array
    metadata: {
        source: "generated",           // "generated" | "imported_key" | "imported_mnemonic"
        version: "1.0",               // Data format version
        backupVerified: false         // Mnemonic verification status
    }
}
```

### **Balance Data**
```javascript
{
    balance: 1000000,                  // Public balance in micro-OCT
    nonce: 123,                       // Current nonce
    privateBalance: 500000,           // Private/encrypted balance
    claimableTransfers: 3,            // Number of claimable private transfers
    lastUpdated: 1640995200000        // Cache timestamp
}
```

---

## **API REFERENCE**

### **Public Endpoints**
```http
GET /balance/{address}
Response: { balance: number, nonce: number }

POST /send-tx
Body: TransactionObject
Response: { success: boolean, txHash?: string, error?: string }

GET /address/{address}
Response: TransactionObject[]

GET /staging  
Response: TransactionObject[]
```

### **Private Endpoints**
```http
GET /view_encrypted_balance/{address}
Headers: X-Private-Key: {base64PrivateKey}
Response: { encryptedBalance: number }

POST /encrypt_balance
Headers: X-Private-Key: {base64PrivateKey}
Body: { address: string, amount: number }
Response: { success: boolean }

POST /private_transfer
Headers: X-Private-Key: {base64PrivateKey}  
Body: { to_address: string, encrypted_amount: string, ephemeral_public_key: string }
Response: { success: boolean, transfer_id: string }

POST /claim_private_transfer
Headers: X-Private-Key: {base64PrivateKey}
Body: { recipient_address: string, private_key: string, transfer_id: string }
Response: { success: boolean, amount: number }

GET /pending_private_transfers
Headers: X-Private-Key: {base64PrivateKey}
Response: PrivateTransfer[]

GET /public_key/{address}
Response: { public_key: string }
```

---

## **PERFORMANCE OPTIMIZATIONS**

### **Caching Strategy**
- **Balance/Nonce**: 30 seconds TTL
- **Transaction History**: 60 seconds TTL  
- **Private Balance**: 45 seconds TTL
- **DOM Elements**: Cached with validity checking
- **Network Requests**: Automatic retry with exponential backoff

### **Memory Management**
- Event listener cleanup on screen transitions
- DOM element cache with LRU eviction
- Sensitive data clearing on logout
- Automatic garbage collection triggers

### **UI Optimizations**
- Debounced input validation
- Progressive loading for transaction history
- Virtual scrolling for large wallet lists
- CSS animations with hardware acceleration

---

## **TESTING APPROACH**

### **Manual Testing**
```javascript
// Test wallet creation
python -c "from src.wallet import OctraWallet; print('✅ Wallet module OK')"

// Test crypto operations  
python -c "from src.crypto import CryptoManager; print('✅ Crypto module OK')"

// Test network connectivity
python -c "from src.network import NetworkClient; print('✅ Network module OK')"
```

### **Browser Testing**
```bash
# Load extension in development mode
# Chrome: chrome://extensions -> Load unpacked -> select extension/ folder
# Firefox: about:debugging -> Load Temporary Add-on
```

### **Integration Testing**
- Multi-wallet scenarios
- Network failure recovery
- Auto-lock functionality
- Cross-session persistence
- Mnemonic verification accuracy

---

## **DEPLOYMENT**

### **Development Setup**
```bash
# 1. Clone repository
git clone <repository-url>

# 2. Load extension
# Open Chrome -> Extensions -> Developer Mode -> Load Unpacked

# 3. Test functionality
# Create wallet, send transaction, verify backup
```

### **Production Build**
```bash
# 1. Minify and optimize assets
# 2. Update manifest version
# 3. Package as .crx or .zip for distribution
# 4. Submit to Chrome Web Store
```

### **Configuration Files**
- `manifest.json` - Extension metadata and permissions
- `config.js` - Network endpoints and constants  
- `package.json` - Development dependencies (if using build tools)

---

## **TROUBLESHOOTING**

### **Common Issues**

#### **Wallet Creation Fails**
- Check crypto module initialization
- Verify network connectivity  
- Ensure sufficient entropy for key generation
- Check storage quota limits

#### **Transaction Sending Fails**
- Validate address format (must start with 'oct')
- Check balance includes fees
- Verify network endpoint availability
- Confirm private key signature

#### **UI Not Responding**  
- Check console for JavaScript errors
- Verify DOM element availability
- Test screen transition logic
- Clear extension storage if corrupted

#### **Auto-Lock Issues**
- Check timer configuration
- Verify activity tracking
- Test across browser restarts
- Validate storage permissions

### **Debug Tools**
- Chrome DevTools for extension debugging
- Network tab for API call monitoring  
- Storage tab for encrypted data inspection
- Console for error tracking and logging

---

## **EXTERNAL DEPENDENCIES**

### **Core Libraries**
- **TweetNaCl** (`libs/nacl.min.js`) - Ed25519 cryptography
- **Chrome Extensions API** - Storage, runtime, permissions
- **Web Crypto API** - PBKDF2, AES-GCM, random generation

### **Standards Compliance**
- **BIP39** - Mnemonic phrase generation and validation
- **Ed25519** - Digital signature algorithm
- **Base58** - Address encoding
- **JSON-RPC** - Network communication protocol

---

This documentation covers the complete architecture, implementation details, and usage patterns of the Octra Wallet Extension. Each component is designed for security, maintainability, and user experience while following modern web extension development practices.