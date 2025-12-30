<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>2-Player Tic-Tac-Toe</title>
    <style>
        :root {
            --bg-color: #f8f9fa;
            --card-bg: #ffffff;
            --primary-color: #4a90e2;
            --accent-a: #e74c3c; /* Player A - Circle */
            --accent-b: #2ecc71; /* Player B - Cross */
            --text-color: #333;
            --border-color: #ddd;
            --shadow: 0 4px 6px rgba(0,0,0,0.1);
        }

        * {
            box-sizing: border-box;
            -webkit-tap-highlight-color: transparent;
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
        }

        body {
            margin: 0;
            padding: 0;
            background-color: var(--bg-color);
            color: var(--text-color);
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
        }

        /* Responsive Layout */
        .container {
            display: flex;
            flex-direction: column;
            width: 100%;
            max-width: 1100px;
            padding: 20px;
            gap: 20px;
        }

        @media (min-width: 768px) {
            .container {
                flex-direction: row;
                align-items: stretch;
                height: 90vh;
            }
        }

        /* Left Side: Game Board */
        .board-section {
            flex: 2;
            display: flex;
            justify-content: center;
            align-items: center;
            background: var(--card-bg);
            border-radius: 16px;
            box-shadow: var(--shadow);
            padding: 20px;
        }

        .board {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            grid-template-rows: repeat(3, 1fr);
            gap: 12px;
            width: 100%;
            max-width: 500px;
            aspect-ratio: 1 / 1;
        }

        .cell {
            background: #eee;
            border: none;
            border-radius: 12px;
            font-size: 3rem;
            font-weight: bold;
            display: flex;
            justify-content: center;
            align-items: center;
            cursor: pointer;
            transition: transform 0.1s, background-color 0.2s;
            min-width: 44px;
            min-height: 44px;
        }

        .cell:active {
            transform: scale(0.95);
            background-color: #ddd;
        }

        .cell.occupied {
            cursor: default;
        }

        .cell[data-player="A"] {
            color: var(--accent-a);
        }

        .cell[data-player="B"] {
            color: var(--accent-b);
        }

        /* Right Side: Info & Controls */
        .info-section {
            flex: 1;
            display: flex;
            flex-direction: column;
            gap: 20px;
        }

        .status-card {
            background: var(--card-bg);
            padding: 24px;
            border-radius: 16px;
            box-shadow: var(--shadow);
            text-align: center;
            border-bottom: 8px solid transparent;
            transition: border-color 0.3s;
        }

        .status-card.turn-a { border-color: var(--accent-a); }
        .status-card.turn-b { border-color: var(--accent-b); }

        .current-turn-label {
            font-size: 0.9rem;
            color: #666;
            margin-bottom: 8px;
            text-transform: uppercase;
            letter-spacing: 1px;
        }

        .status-text {
            font-size: 1.8rem;
            font-weight: bold;
        }

        .btn {
            width: 100%;
            padding: 16px;
            font-size: 1.1rem;
            font-weight: bold;
            border: none;
            border-radius: 12px;
            cursor: pointer;
            transition: background 0.2s;
            display: flex;
            justify-content: center;
            align-items: center;
            gap: 8px;
        }

        .btn-reset {
            background: #333;
            color: white;
        }

        .btn-reset:hover { background: #555; }

        .log-container {
            flex-grow: 1;
            background: var(--card-bg);
            border-radius: 16px;
            padding: 16px;
            box-shadow: var(--shadow);
            overflow: hidden;
            display: flex;
            flex-direction: column;
        }

        .log-title {
            font-weight: bold;
            margin-bottom: 10px;
            border-bottom: 1px solid var(--border-color);
            padding-bottom: 5px;
        }

        .log-list {
            list-style: none;
            padding: 0;
            margin: 0;
            overflow-y: auto;
            font-size: 0.9rem;
            flex-grow: 1;
        }

        .log-item {
            padding: 6px 0;
            border-bottom: 1px solid #f0f0f0;
        }

        details {
            background: #fff;
            padding: 12px;
            border-radius: 12px;
            box-shadow: var(--shadow);
        }

        summary {
            cursor: pointer;
            font-weight: bold;
            outline: none;
        }

        .rules-content {
            margin-top: 10px;
            font-size: 0.85rem;
            line-height: 1.5;
            color: #555;
        }

        /* Winner Highlight */
        .cell.winner {
            background-color: #fff9c4;
            animation: pulse 1s infinite alternate;
        }

        @keyframes pulse {
            from { transform: scale(1); }
            to { transform: scale(1.05); }
        }

        /* Modal Simple Implementation */
        #custom-modal {
            display: none;
            position: fixed;
            top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.5);
            justify-content: center;
            align-items: center;
            z-index: 1000;
        }
        .modal-box {
            background: white;
            padding: 24px;
            border-radius: 16px;
            max-width: 300px;
            width: 80%;
            text-align: center;
        }
        .modal-btns {
            display: flex;
            gap: 10px;
            margin-top: 20px;
        }
    </style>
</head>
<body>

    <div class="container">
        <!-- Game Board -->
        <section class="board-section">
            <div class="board" id="board" role="grid">
                <!-- Generated by JS -->
            </div>
        </section>

        <!-- Sidebar -->
        <section class="info-section">
            <div id="status-card" class="status-card">
                <div class="current-turn-label" id="turn-label">Current Turn</div>
                <div class="status-text" id="status-text">Player A</div>
            </div>

            <button class="btn btn-reset" id="reset-btn" aria-label="Reset Game">
                🔄 Reset Game
            </button>

            <div class="log-container">
                <div class="log-title">History Log</div>
                <ul class="log-list" id="log-list"></ul>
            </div>

            <details>
                <summary>How to Play</summary>
                <div class="rules-content">
                    1. プレイヤーAは「〇」、プレイヤーBは「×」です。<br>
                    2. 空いているマスをタップして記号を配置します。<br>
                    3. 縦・横・斜めのいずれかに3つ揃うと勝利です。<br>
                    4. 全てのマスが埋まり、揃わなかった場合は引き分けとなります。
                </div>
            </details>
        </section>
    </div>

    <!-- Confirmation Modal -->
    <div id="custom-modal">
        <div class="modal-box">
            <p>ゲームをリセットしてもよろしいですか？</p>
            <div class="modal-btns">
                <button class="btn" style="background:#eee; color:#333;" id="modal-cancel">Cancel</button>
                <button class="btn" style="background:var(--accent-a); color:white;" id="modal-confirm">Yes</button>
            </div>
        </div>
    </div>

    <script>
        const STORAGE_KEY = 'tictactoe_ipad_state';
        const boardEl = document.getElementById('board');
        const statusCard = document.getElementById('status-card');
        const statusText = document.getElementById('status-text');
        const turnLabel = document.getElementById('turn-label');
        const logList = document.getElementById('log-list');
        const resetBtn = document.getElementById('reset-btn');
        const modal = document.getElementById('custom-modal');

        // Game State
        let state = {
            board: Array(9).fill(null),
            currentPlayer: 'A', // A or B
            isGameOver: false,
            winner: null,
            logs: []
        };

        // Initialize
        function init() {
            loadState();
            renderBoard();
            updateUI();
            
            resetBtn.addEventListener('click', () => {
                modal.style.display = 'flex';
            });

            document.getElementById('modal-cancel').addEventListener('click', () => {
                modal.style.display = 'none';
            });

            document.getElementById('modal-confirm').addEventListener('click', () => {
                resetGame();
                modal.style.display = 'none';
            });
        }

        function renderBoard() {
            boardEl.innerHTML = '';
            state.board.forEach((cell, index) => {
                const btn = document.createElement('button');
                btn.className = 'cell';
                if (cell) {
                    btn.textContent = cell === 'A' ? '〇' : '×';
                    btn.classList.add('occupied');
                    btn.dataset.player = cell;
                }
                btn.setAttribute('aria-label', `Cell ${index + 1}, ${cell || 'Empty'}`);
                btn.addEventListener('click', () => handleMove(index));
                boardEl.appendChild(btn);
            });

            if (state.winner) {
                highlightWinner();
            }
        }

        function handleMove(index) {
            if (state.isGameOver || state.board[index]) return;

            const playerSymbol = state.currentPlayer === 'A' ? '〇' : '×';
            state.board[index] = state.currentPlayer;
            
            addLog(`Player ${state.currentPlayer} placed at ${index + 1}`);

            checkWin();

            if (!state.isGameOver) {
                state.currentPlayer = state.currentPlayer === 'A' ? 'B' : 'A';
            }

            saveState();
            renderBoard();
            updateUI();
        }

        function checkWin() {
            const lines = [
                [0, 1, 2], [3, 4, 5], [6, 7, 8], // Rows
                [0, 3, 6], [1, 4, 7], [2, 5, 8], // Cols
                [0, 4, 8], [2, 4, 6]             // Diagonals
            ];

            for (let line of lines) {
                const [a, b, c] = line;
                if (state.board[a] && state.board[a] === state.board[b] && state.board[a] === state.board[c]) {
                    state.isGameOver = true;
                    state.winner = state.board[a];
                    state.winningLine = line;
                    addLog(`★ Player ${state.winner} WON!`);
                    return;
                }
            }

            if (!state.board.includes(null)) {
                state.isGameOver = true;
                state.winner = 'DRAW';
                addLog(`It's a DRAW!`);
            }
        }

        function highlightWinner() {
            if (!state.winningLine) return;
            const cells = boardEl.querySelectorAll('.cell');
            state.winningLine.forEach(idx => {
                cells[idx].classList.add('winner');
            });
        }

        function updateUI() {
            // Update Card appearance based on turn
            statusCard.className = `status-card turn-${state.currentPlayer.toLowerCase()}`;
            
            if (state.isGameOver) {
                turnLabel.textContent = "Game Result";
                if (state.winner === 'DRAW') {
                    statusText.textContent = "引き分け";
                    statusCard.className = `status-card`;
                } else {
                    statusText.textContent = `${state.winner === 'A' ? 'Aの勝ち' : 'Bの勝ち'}`;
                    statusCard.className = `status-card turn-${state.winner.toLowerCase()}`;
                }
            } else {
                turnLabel.textContent = "Current Turn";
                statusText.textContent = `Player ${state.currentPlayer} (${state.currentPlayer === 'A' ? '〇' : '×'})`;
            }

            // Update Logs
            logList.innerHTML = '';
            state.logs.slice().reverse().forEach(msg => {
                const li = document.createElement('li');
                li.className = 'log-item';
                li.textContent = msg;
                logList.appendChild(li);
            });
        }

        function addLog(msg) {
            const now = new Date();
            const time = `${now.getHours()}:${String(now.getMinutes()).padStart(2, '0')}`;
            state.logs.push(`[${time}] ${msg}`);
        }

        function resetGame() {
            state = {
                board: Array(9).fill(null),
                currentPlayer: 'A',
                isGameOver: false,
                winner: null,
                logs: []
            };
            addLog("Game Reset");
            saveState();
            renderBoard();
            updateUI();
        }

        function saveState() {
            localStorage.setItem(STORAGE_KEY, JSON.stringify(state));
        }

        function loadState() {
            const saved = localStorage.getItem(STORAGE_KEY);
            if (saved) {
                try {
                    const parsed = JSON.parse(saved);
                    state = parsed;
                } catch (e) {
                    console.error("Failed to load state", e);
                }
            }
        }

        window.onload = init;
    </script>
</body>
</html>
