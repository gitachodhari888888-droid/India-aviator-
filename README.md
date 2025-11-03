<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>India Aviator - Owner Economy</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/tone/14.8.49/Tone.min.js"></script> 
    <link rel="manifest" href="manifest.json">
    <script>
        // PWA setup (for installing app)
        if ('serviceWorker' in navigator) {
            window.addEventListener('load', () => {
                // Registering Service Worker for PWA (using the root path)
                navigator.serviceWorker.register('/india-aviator/service-worker.js')
                    .then(registration => console.log('SW registered'))
                    .catch(err => console.log('SW registration failed:', err));
            });
        }
    </script>
    <style>
        /* Mobile-First Styling */
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;700;900&display=swap');
        body { font-family: 'Inter', sans-serif; background-color: #0d1117; color: #c9d1d9; padding: 0; margin: 0; }
        .container { min-height: 100vh; display: flex; flex-direction: column; align-items: center; justify-content: flex-start; }
        .card { background-color: #161b22; border: 1px solid #30363d; border-radius: 12px; padding: 16px; width: 100%; }
        .game-area { background-color: #000000; border: 2px solid #30363d; height: 300px; position: relative; overflow: hidden; border-radius: 12px; }
        .plane { position: absolute; bottom: 10px; left: 10px; font-size: 30px; transition: transform 0.1s; }
        .multiplier { position: absolute; top: 20px; left: 50%; transform: translateX(-50%); font-size: 3rem; font-weight: 900; color: #4CAF50; text-shadow: 0 0 10px rgba(76, 175, 80, 0.8); }
        .btn { padding: 12px 15px; border-radius: 8px; font-weight: 700; width: 100%; transition: background-color 0.2s; }
        .btn-bet { background-color: #f7931a; color: white; }
        .btn-cashout { background-color: #4CAF50; color: white; }
        .btn-primary { background-color: #238636; color: white; }
        .btn-secondary { background-color: #584a30; color: #f7f0e6; }
        .btn-management { background-color: #7b003a; color: white; }
        .input-field { background-color: #0d1117; border: 1px solid #30363d; color: #c9d1d9; border-radius: 6px; padding: 8px; width: 100%; }
        .modal-backdrop { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background-color: rgba(0, 0, 0, 0.8); display: flex; justify-content: center; align-items: center; z-index: 1000; }
        .chat-box { height: 200px; overflow-y: auto; border: 1px solid #30363d; background-color: #0d1117; padding: 8px; border-radius: 6px; margin-bottom: 10px; }
        .chat-message { margin-bottom: 5px; word-wrap: break-word; }
        .qr-container { background-color: white; padding: 10px; border-radius: 8px; text-align: center; display: flex; justify-content: center; }
        .qr-container img { width: 100%; max-width: 200px; height: auto; }
        .settings-panel {
            position: fixed; top: 0; right: 0; width: 90%; max-width: 350px; height: 100%; 
            background-color: #0d1117; z-index: 1000; transform: translateX(100%); transition: transform 0.3s ease-in-out;
            padding: 16px; border-left: 2px solid #30363d; overflow-y: auto;
        }
        .settings-panel.open { transform: translateX(0); }

        /* New: Click handler area for closing the panel */
        .settings-overlay {const MASTER_OWNER_ID = '#nnophdbdbcdvb6gvs';
        
            position: fixed; top: 0; left: 0; width: 100%; height: 100%; z-index: 999;
            background-color: transparent; display: none;
        }
        .settings-panel.open + .settings-overlay { display: block; }
        
        .bank-details-card { 
            background-color: #1e242c; 
            padding: 10px; 
            border-radius: 8px;
            margin-top: 10px;
        }
        .deposit-option { border: 1px solid #30363d; border-radius: 8px; padding: 10px; text-align: center; cursor: pointer; transition: background-color 0.2s; }
        .deposit-option:hover { background-color: #1a202a; }
    </style>
</head>
<body>
    <div class="container p-2 sm:p-4 w-full max-w-xl">
        <h1 class="text-2xl font-bold mb-4 text-center text-red-400">✈️ India Aviator (Owner Edition)</h1>

        <!-- Top Bar: Info and Settings Button -->
        <div class="card mb-4 p-3 flex justify-between items-center text-sm">
            <div>
                <p class="text-gray-400">👤 User:</p>
                <p id="user-name-display" class="font-bold text-yellow-400">Loading...</p>
            </div>
            <div class="text-center">
                <button class="text-xl text-gray-400 hover:text-white" onclick="openSettingsPanel()">
                    ⚙️ Community / Settings
                </button>
            </div>
            <div class="text-right">
                <p class="text-gray-400">Balance:</p>
                <p class="text-xl font-bold text-yellow-400"><span id="user-balance">...</span> Tokens</p>
            </div>
        </div>

        <!-- Game Area -->
        <div class="card mb-4 p-0">
            <div class="game-area">
                <div id="plane" class="plane">
                    <span id="avatar-icon">✈️</span>
                </div>
                <!-- Canvas for drawing the flight path -->
                <canvas id="gameCanvas" width="500" height="300" class="w-full h-full"></canvas>

                <div id="multiplier" class="multiplier">1.00x</div>
                <div id="status-message" class="absolute inset-0 flex items-center justify-center text-white text-2xl font-bold bg-black bg-opacity-70">
                    Waiting for next round...
                </div>
            </div>
        </div>
        
        <!-- Controls and Bet -->
        <div class="card mb-4 grid grid-cols-2 gap-3">
            <input type="number" id="bet-amount" class="input-field col-span-2" placeholder="Bet Amount (Tokens, Min 10)" value="100" min="10" max="1000">
            
            <button id="bet-button" class="btn btn-bet" onclick="placeBet()">Place Bet 100</button>
            <button id="cashout-button" class="btn btn-cashout opacity-50" disabled onclick="cashOut()">Cash Out</button>

            <!-- Auto Bet Controls -->
             <div class="col-span-2 grid grid-cols-2 gap-2 mt-2">
                <input type="number" id="auto-bet-amount" class="input-field" placeholder="Auto Bet Amt (e.g. 100)" value="100" min="10" max="1000">
                <input type="number" id="auto-cashout-multiplier" class="input-field" placeholder="Auto Cashout (e.g. 2.5)" value="2.0" step="0.01" min="1.01">
                <button id="auto-toggle-btn" class="btn btn-primary col-span-2" onclick="toggleAutoMode()">Auto: OFF</button>
            </div>

        </div>

        <!-- Transaction/Info Message -->
        <p id="transaction-message" class="mt-3 text-center text-sm font-semibold text-white h-5"></p>

        <!-- Purchase & Withdraw Buttons (Minimized) -->
        <div class="card mb-4 grid grid-cols-2 gap-3">
            <button class="btn btn-primary" onclick="showDepositModal()">💰 Buy Tokens</button>
            <button class="btn btn-secondary" onclick="showWithdrawalModal()">Withdraw Winnings</button>
        </div>

    </div>

    <!-- SETTINGS / COMMUNITY PANEL (Hidden by default) -->
    <div id="settings-panel" class="settings-panel">
        <div class="flex justify-between items-center mb-4">
            <h2 class="text-xl font-bold text-blue-400">Settings & Community</h2>
            <button class="text-2xl text-red-400" onclick="closeSettingsPanel()">×</button>
        </div>

        <!-- Sound Control -->
        <div class="card mb-4">
            <h3 class="font-bold mb-2">🔊 Game Sounds</h3>
            <button id="sound-toggle-btn" class="btn btn-primary" onclick="toggleSound()">Sound: ON</button>
        </div>
        
        <!-- Owner's Vault Display -->
        <div class="card mb-4">
            <h3 class="font-bold mb-2 text-yellow-500">👑 Owner Vault (Your Assets)</h3>
            <p id="vault-balance" class="text-xl font-mono text-center text-yellow-300">Loading...</p>
            <p class="text-xs text-gray-500 text-center mt-1">(Reloads 1 Crore Tokens every 7 days)</p>
        </div>

        <!-- Live Chat Area -->
        <div class="card mb-4">
            <h2 class="text-xl font-bold mb-2 text-teal-400">💬 Live Chat</h2>
            <div id="chat-messages" class="chat-box">
                <!-- Chat messages will be added here -->
            </div>
            <div class="flex space-x-2">
                <input type="text" id="chat-input" class="input-field flex-grow" placeholder="Type your message..." onkeypress="if(event.key === 'Enter') sendMessage()">
                <button class="btn btn-primary w-1/4" onclick="sendMessage()">Send</button>
            </div>
        </div>

        <!-- Leaderboard -->
        <div class="card mb-4">
            <h2 class="text-xl font-bold mb-3 text-center text-teal-400">🌎 Live Leaderboard</h2>
            <div id="leaderboard-list" class="space-y-2 max-h-60 overflow-y-auto">
                <div class="text-center text-gray-400">Loading live user data...</div>
            </div>
        </div>
        
        <!-- Owner/Manager Actions -->
        <div class="card p-3">
            <h2 class="text-xl font-bold mb-3 text-center text-red-300">👑 Owner/Manager Control</h2>
            <p class="text-sm text-gray-400 mb-2">Owner ID: <span id="owner-id-display" class="font-mono text-xs"></span></p>
            <button class="btn btn-management mb-2" onclick="showIssueTokensModal()">💰 Issue Tokens (Approve Deposit)</button>
            <button class="btn btn-management" onclick="reloadOwnerVault()">🔋 Reload 1 Crore Vault</button>
             <a href="https://wa.me/917977481210" target="_blank" class="btn btn-primary mt-2 block text-center">💬 WhatsApp Support</a>
        </div>
    </div>
    <div class="settings-overlay" onclick="closeSettingsPanel()"></div>


    <!-- DEPOSIT MODAL (UPI/QR) -->
    <div id="deposit-modal" class="modal-backdrop" style="display: none;">
        <div class="modal-content card max-w-sm">
            <h2 class="text-xl font-bold text-green-400 mb-4">Scan & Deposit Tokens</h2>
            
            <p class="text-sm text-gray-400 mb-4">Choose amount to deposit (1 Token = ₹1):</p>
            
            <!-- Deposit Amount Options -->
            <div class="grid grid-cols-2 gap-3 mb-4">
                <div class="deposit-option" onclick="showPaymentQr(100)">₹100 (100 Tokens)</div>
                <div class="deposit-option" onclick="showPaymentQr(200)">₹200 (200 Tokens)</div>
                <div class="deposit-option" onclick="showPaymentQr(300)">₹300 (300 Tokens)</div>
                <div class="deposit-option" onclick="showPaymentQr(400)">₹400 (400 Tokens)</div>
                <div class="deposit-option" onclick="showPaymentQr(500)">₹500 (500 Tokens)</div>
            </div>
            
            <p id="deposit-amount-display" class="text-xl font-bold mb-4 text-yellow-400 hidden">Amount: ₹0</p>
            
            <div id="qr-display-area" class="hidden">
                <div class="qr-container mb-4">
                    <!-- Base64 QR Code (Your actual QR Code Image) -->
                    <img id="qr-code-image" src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAQAAAAEACAYAAABccqhmAAAABmFsdGVkTW9udXMTK0pXAAAgAElEQVR4nOy9b3B...[LONG STRING REMOVED FOR BREVITY]...H+bwdkMAAAAASUVORK5CYII=" alt="UPI QR Code">
                </div>

                <p class="text-sm text-gray-400 mb-4">Scan this QR Code to pay ₹<span id="qr-amount-text">0</span>. Then enter the Transaction ID below.</p>

                <input type="text" id="tx-id-input" class="input-field mb-2" placeholder="Enter Transaction ID (UTI)">
                <button class="btn btn-primary w-full mb-3" onclick="confirmPayment()">Confirm Payment</button>
            </div>
            
            <button class="btn btn-secondary w-full" onclick="hideDepositModal()">Close</button>
        </div>
    </div>

    <!-- WITHDRAWAL MODAL (Bank Details) -->
    <div id="withdrawal-modal" class="modal-backdrop" style="display: none;">
        <div class="modal-content card max-w-sm">
            <h2 class="text-xl font-bold text-red-400 mb-4">Withdraw Winnings (Min ₹200)</h2>
            <p class="text-sm text-gray-400 mb-4">Enter your bank details to receive funds. Total deduction: <span id="withdraw-cost-display">220</span> Tokens (Includes 10% Fee).</p>

            <input type="text" id="w-account-name" class="input-field mb-2" placeholder="Account Holder Name" required>
            <input type="text" id="w-account-number" class="input-field mb-2" placeholder="Bank Account Number" required>
            <input type="text" id="w-ifsc" class="input-field mb-2" placeholder="IFSC Code" required>
            <input type="tel" id="w-phone" class="input-field mb-2" placeholder="Phone Number (Mandatory)" required>

            <button class="btn btn-secondary w-full mb-3" onclick="processWithdrawal()">Submit Withdrawal Request</button>
            <button class="btn btn-primary w-full" onclick="document.getElementById('withdrawal-modal').style.display='none'">Cancel</button>
        </div>
    </div>

    <!-- ISSUE TOKENS MODAL (Owner Only) -->
    <div id="issue-tokens-modal" class="modal-backdrop" style="display: none;">
        <div class="modal-content card max-w-sm">
            <h2 class="text-xl font-bold text-red-400 mb-4">👑 Issue Tokens to Player</h2>
            <p class="text-sm text-gray-400 mb-4">Use this after verifying payment/share alert in chat.</p>

            <input type="text" id="issue-target-id" class="input-field mb-2" placeholder="Target User ID (e.g., from chat)" required>
            <input type="number" id="issue-amount" class="input-field mb-2" placeholder="Tokens to Issue (e.g., 30 or 200)" required>

            <button class="btn btn-management w-full mb-3" onclick="issueTokensToPlayer()">Issue Tokens Now</button>
            <button class="btn btn-secondary w-full" onclick="document.getElementById('issue-tokens-modal').style.display='none'">Cancel</button>
        </div>
    </div>

    <!-- AGE/PROFILE MODAL -->
    <div id="age-modal" class="modal-backdrop" style="display: none;">
        <div class="modal-content card max-w-sm">
            <h2 class="text-xl font-bold text-red-400 mb-4">✈️ Welcome to India Aviator</h2>
            <p class="text-sm text-gray-400 mb-3">Please set up your profile to start playing.</p>

            <input type="text" id="username-input" class="input-field mb-2" placeholder="Enter Your Name (Profile)" required>
            <input type="tel" id="phone-input" class="input-field mb-4" placeholder="Enter Phone Number (Mandatory)" required>
            <button class="btn btn-primary w-full" onclick="verifyAge()">Start Game</button>
        </div>
    </div>

    <script type="module">
        // Firebase Imports
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, setDoc, onSnapshot, collection, query, limit, orderBy, getDoc, addDoc } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
        
        // --- Configuration ---
        const MASTER_OWNER_ID = 'nnophdbdbcdvb6gvs'; // <--- **YOUR FIXED OWNER ID**
        const GAME_CYCLE_TIME = 10; 
        const MAX_MULTIPLIER = 1000; 
        const MIN_BET = 10;
        const MAX_BET = 1000;

        // Owner's Wallet Setup
        const VAULT_RELOAD_AMOUNT = 10000000; // 1 Crore Tokens
        const VAULT_RELOAD_DAYS = 7; 
        
        // Base64 QR Code (PhonePe Icon) - Your actual QR Code Image
        const QR_CODE_BASE64 = 'data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAQAAAAEACAYAAABccqhmAAAABmFsdGVkTW9udXMTK0pXAAAgAElEQVR4nOy9b3B...[LONG STRING REMOVED FOR BREVITY]...H+bwdkMAAAAASUVORK5CYII=';

        // --- Global State ---
        let db, auth, userId = 'loading...';
        let userBalance = 500;
        let userName = 'New Player';
        let userPhone = '';
        let isAuthReady = false;
        let isProfileSet = false;
        let soundOn = localStorage.getItem('soundOn') === 'true';

        // Game State
        let gameState = 'WAITING'; // WAITING, BETTING, STARTED, CRASHED
        let currentMultiplier = 1.00;
        let betPlaced = 0;
        let hasCashedOut = false;
        let roundTimer = GAME_CYCLE_TIME;
        let crashPoint = 0; 
        let currentDepositAmount = 0; 
        let manualCrashPoint = null; 

        // Sound Setup (Tone.js)
        let synth, crashSynth, tickSynth, cashoutSynth;
        
        const initializeAudio = () => {
             // Ensure Tone is initialized only once
             if (typeof Tone !== 'undefined' && Tone.context.state !== 'running') {
                 Tone.start().then(() => {
                    synth = new Tone.PolySynth(Tone.Synth).toDestination();
                    crashSynth = new Tone.Synth().toDestination();
                    tickSynth = new Tone.Synth({ oscillator: { type: "square" }, envelope: { attack: 0.005, decay: 0.1, sustain: 0.05, release: 0.1 } }).toDestination();
                    cashoutSynth = new Tone.Synth({ oscillator: { type: "triangle" }, envelope: { attack: 0.01, decay: 0.1, sustain: 0.2, release: 0.5 } }).toDestination();
                 });
             }
        };

        const playSound = (note, duration = "8n", time = "+0.05") => {
            if (soundOn && Tone.context.state === 'running' && synth) {
                synth.triggerAttackRelease(note, duration, time);
            }
        };

        const playCrashSound = () => {
             if (soundOn && Tone.context.state === 'running' && crashSynth) {
                crashSynth.triggerAttackRelease("A3", "2n");
             }
        };

        const playTick = () => {
             if (soundOn && Tone.context.state === 'running' && tickSynth) {
                tickSynth.triggerAttackRelease("C4", "16n");
             }
        };

        const playCashout = () => {
             if (soundOn && Tone.context.state === 'running' && cashoutSynth) {
                cashoutSynth.triggerAttackRelease("G4", "8n");
             }
        };

        // --- Utility Functions ---

        const updateUI = () => {
            document.getElementById('user-id-display').innerText = userId === MASTER_OWNER_ID ? '👑 OWNER' : userId.substring(0, 8) + '...';
            document.getElementById('user-balance').innerText = userBalance.toLocaleString();
            document.getElementById('user-name-display').innerText = userName;
            document.getElementById('owner-id-display').innerText = MASTER_OWNER_ID;

            const betBtn = document.getElementById('bet-button');
            const cashoutBtn = document.getElementById('cashout-button');
            const betInput = document.getElementById('bet-amount');

            const betValue = parseInt(betInput.value);

            // AUTO MODE BUTTON TEXT
            const isAutoMode = document.getElementById('auto-toggle-btn').innerText.includes('ON');
            document.getElementById('auto-toggle-btn').classList.toggle('btn-management', isAutoMode);
            document.getElementById('auto-toggle-btn').classList.toggle('btn-primary', !isAutoMode);


            if (gameState === 'BETTING') {
                betBtn.innerText = betPlaced > 0 ? `BET LOCKED ${betPlaced}` : `Place Bet ${betValue}`;
                betBtn.disabled = betPlaced > 0;
            } else if (gameState === 'STARTED') {
                betBtn.innerTex
