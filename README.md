# lucky-wheel
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.2, user-scalable=yes">
    <title>🌌 命运之轮 · 概率抽奖</title>
    <style>
        :root {
            --bg-deep: #0f0c1f;
            --bg-card: #1a1630;
            --bg-card2: #221e3a;
            --gold: #d4a843;
            --gold-light: #f0d078;
            --gold-dark: #b8861e;
            --text: #e8e4f0;
            --text-soft: #a9a4c0;
            --accent: #c9a24b;
            --danger: #c05550;
            --border: #2e2950;
            --border-gold: #5a4a2a;
            --shadow-gold: 0 0 30px rgba(200, 150, 40, 0.25);
            --shadow-card: 0 8px 32px rgba(0, 0, 0, 0.4);
            --radius: 18px;
            --radius-sm: 12px;
            --font: 'Segoe UI', 'PingFang SC', 'Microsoft YaHei', 'Helvetica Neue', sans-serif;
        }

        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: var(--font);
            background: var(--bg-deep);
            background-image:
                radial-gradient(ellipse at 50% 0%, rgba(120, 100, 180, 0.18) 0%, transparent 60%),
                radial-gradient(ellipse at 85% 20%, rgba(200, 160, 60, 0.10) 0%, transparent 55%),
                radial-gradient(ellipse at 15% 80%, rgba(140, 120, 200, 0.10) 0%, transparent 50%);
            min-height: 100vh;
            display: flex;
            align-items: center;
            justify-content: center;
            padding: 20px;
            color: var(--text);
            -webkit-tap-highlight-color: transparent;
            position: relative;
            overflow-x: hidden;
        }

        .stars-canvas {
            position: fixed;
            inset: 0;
            pointer-events: none;
            z-index: 0;
        }

        .app-container {
            position: relative;
            z-index: 1;
            display: flex;
            flex-wrap: wrap;
            gap: 32px;
            align-items: flex-start;
            justify-content: center;
            max-width: 1100px;
            width: 100%;
        }

        .wheel-section {
            flex: 1 1 480px;
            max-width: 540px;
            min-width: 340px;
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 20px;
        }

        .wheel-stage {
            position: relative;
            width: 100%;
            aspect-ratio: 1/1;
            max-width: 500px;
            max-height: 500px;
            filter: drop-shadow(0 0 45px rgba(180, 130, 40, 0.35));
        }

        .wheel-outer-ring {
            position: absolute;
            inset: -16px;
            border-radius: 50%;
            border: 4px solid transparent;
            background: conic-gradient(from 0deg,
                    var(--gold-dark), var(--gold-light), var(--gold-dark),
                    var(--gold-light), var(--gold-dark), var(--gold-light),
                    var(--gold-dark), var(--gold-light), var(--gold-dark)) border-box;
            -webkit-mask: radial-gradient(farthest-side, transparent calc(100% - 4px), #000 calc(100% - 3px));
            mask: radial-gradient(farthest-side, transparent calc(100% - 4px), #000 calc(100% - 3px));
            pointer-events: none;
            z-index: 0;
            animation: ringShimmer 8s linear infinite;
        }
        @keyframes ringShimmer {
            0% { filter: brightness(1); }
            50% { filter: brightness(1.3); }
            100% { filter: brightness(1); }
        }
        .wheel-stage.spinning .wheel-outer-ring {
            animation: ringShimmer 1.5s linear infinite;
        }

        .pointer {
            position: absolute;
            top: -8px;
            left: 50%;
            transform: translateX(-50%);
            z-index: 10;
            width: 0;
            height: 0;
            border-left: 17px solid transparent;
            border-right: 17px solid transparent;
            border-top: 34px solid #e8c547;
            filter: drop-shadow(0 0 12px rgba(220, 170, 30, 0.7)) drop-shadow(0 4px 8px rgba(0, 0, 0, 0.5));
            pointer-events: none;
        }
        .pointer::after {
            content: '';
            position: absolute;
            top: -42px;
            left: -8px;
            width: 16px;
            height: 16px;
            background: radial-gradient(circle, #fde68a, #d4a017);
            border-radius: 50%;
            box-shadow: 0 0 18px 6px rgba(240, 200, 60, 0.6);
        }

        .wheel-spinner {
            position: relative;
            z-index: 2;
            width: 85%;
            height: 85%;
            margin: 7.5% auto;
            border-radius: 50%;
            transition: transform 5.2s cubic-bezier(0.06, 0.7, 0.12, 0.98);
            will-change: transform;
        }
        .wheel-spinner canvas {
            display: block;
            width: 100%;
            height: 100%;
            border-radius: 50%;
            box-shadow: 0 0 0 6px #1a1630, 0 0 0 10px #3d3450, 0 0 0 13px #2a2440;
        }

        .btn-spin {
            padding: 15px 50px;
            font-size: 1.2rem;
            font-weight: 700;
            letter-spacing: 0.06em;
            color: #1a1630;
            background: linear-gradient(135deg, #f0d078 0%, #d4a843 40%, #b8861e 100%);
            border: 2px solid #f0d078;
            border-radius: 50px;
            cursor: pointer;
            box-shadow: 0 6px 28px rgba(200, 140, 30, 0.45), 0 0 60px rgba(200, 150, 30, 0.2);
            transition: all 0.25s;
            outline: none;
            font-family: var(--font);
            text-transform: uppercase;
            position: relative;
            overflow: hidden;
        }
        .btn-spin::after {
            content: '';
            position: absolute;
            top: -50%;
            left: -50%;
            width: 200%;
            height: 200%;
            background: radial-gradient(circle, rgba(255, 255, 255, 0.3) 0%, transparent 60%);
            opacity: 0;
            transition: opacity 0.3s;
        }
        .btn-spin:hover::after { opacity: 1; }
        .btn-spin:hover {
            background: linear-gradient(135deg, #f8e8a0 0%, #e8c050 40%, #c99828 100%);
            box-shadow: 0 8px 36px rgba(220, 160, 40, 0.55), 0 0 80px rgba(220, 160, 40, 0.3);
            transform: translateY(-3px);
        }
        .btn-spin:active { transform: translateY(1px) scale(0.96); }
        .btn-spin:disabled {
            background: #4a4458;
            border-color: #5a5468;
            color: #8a8498;
            cursor: not-allowed;
            box-shadow: 0 3px 10px rgba(0, 0, 0, 0.3);
        }

        .panel-section {
            flex: 0 0 370px;
            max-width: 430px;
            min-width: 300px;
            display: flex;
            flex-direction: column;
            gap: 14px;
        }

        .panel-card {
            background: var(--bg-card);
            border-radius: var(--radius);
            padding: 20px 22px;
            box-shadow: var(--shadow-card);
            border: 1px solid var(--border);
            transition: box-shadow 0.2s;
        }
        .panel-card:hover { box-shadow: var(--shadow-card), var(--shadow-gold); }

        .panel-card h3 {
            font-size: 1.05rem;
            font-weight: 700;
            margin-bottom: 14px;
            display: flex;
            align-items: center;
            gap: 8px;
            letter-spacing: 0.03em;
            color: var(--gold-light);
        }

        .prize-list {
            list-style: none;
            display: flex;
            flex-direction: column;
            gap: 6px;
            max-height: 320px;
            overflow-y: auto;
            padding-right: 4px;
        }
        .prize-list::-webkit-scrollbar { width: 4px; }
        .prize-list::-webkit-scrollbar-thumb { background: #3d3558; border-radius: 8px; }

        .prize-item {
            display: flex;
            align-items: center;
            gap: 8px;
            background: var(--bg-card2);
            border-radius: var(--radius-sm);
            padding: 9px 12px;
            border: 1px solid var(--border);
            transition: 0.15s;
        }
        .prize-item:hover { background: #292450; border-color: #4a3d6a; }

        .color-dot {
            width: 12px; height: 12px; border-radius: 50%; flex-shrink: 0;
            box-shadow: 0 0 6px currentColor;
        }

        .prize-name {
            flex: 2; font-weight: 500; font-size: 0.88rem;
            overflow: hidden; text-overflow: ellipsis; white-space: nowrap; color: #d5d0e8;
        }

        .weight-input {
            width: 54px; padding: 4px 4px; border-radius: 18px;
            border: 1.5px solid var(--border); text-align: center;
            font-size: 0.78rem; font-weight: 600; font-family: var(--font);
            background: #1a1630; color: #e0d8f0; outline: none; transition: 0.15s;
        }
        .weight-input:focus { border-color: #c9a84b; box-shadow: 0 0 0 2px rgba(200,150,40,0.2); }
        .weight-input:disabled { opacity: 0.5; }

        .percent-badge {
            font-size: 0.7rem; font-weight: 600; color: #a095c0;
            width: 40px; text-align: right; flex-shrink: 0;
        }

        .btn-delete {
            width: 26px; height: 26px; border-radius: 50%;
            border: 1px solid transparent; background: #2a2540; color: #9a90b8;
            font-size: 0.9rem; cursor: pointer; display: flex; align-items: center;
            justify-content: center; transition: 0.15s; flex-shrink: 0;
        }
        .btn-delete:hover { background: #4a2030; color: #e87060; border-color: #6a3040; }
        .btn-delete:disabled { opacity: 0.3; pointer-events: none; }

        .add-row { display: flex; gap: 10px; margin-top: 6px; }
        .input-new-prize {
            flex: 1; padding: 10px 14px; border-radius: 50px;
            border: 1.5px solid var(--border); background: var(--bg-card2);
            color: #e0d8f0; font-size: 0.88rem; font-family: var(--font); outline: none;
        }
        .input-new-prize::placeholder { color: #6a6490; }
        .input-new-prize:focus { border-color: #c9a84b; box-shadow: 0 0 0 2px rgba(200,150,40,0.15); }
        .btn-add {
            padding: 10px 18px; border-radius: 50px;
            background: linear-gradient(135deg, #5a8a6e, #3d6a50);
            color: #e0f0e4; font-weight: 600; border: 1px solid #6aaa7e;
            cursor: pointer; white-space: nowrap; letter-spacing: 0.02em;
            transition: 0.15s; font-family: var(--font);
        }
        .btn-add:hover { background: linear-gradient(135deg, #6aaa7e, #4d7a5e); box-shadow: 0 0 16px rgba(80,160,100,0.25); }
        .btn-add:disabled { background: #3a3550; border-color: #4a4560; color: #7a7590; cursor: not-allowed; box-shadow: none; }

        .panel-actions { display: flex; gap: 8px; flex-wrap: wrap; }
        .btn-secondary {
            flex: 1; min-width: 70px; padding: 9px 12px; border-radius: 50px;
            border: 1.5px solid var(--border); background: var(--bg-card2);
            color: var(--text-soft); font-weight: 500; font-size: 0.78rem;
            cursor: pointer; text-align: center; transition: 0.15s; white-space: nowrap; font-family: var(--font);
        }
        .btn-secondary:hover { background: #2d2850; border-color: #5a4d7a; color: #d5d0e8; }
        .btn-secondary.danger { border-color: #4a2830; color: #c87060; }
        .btn-secondary.danger:hover { background: #3d2030; border-color: #6a3840; }

        .info-row {
            display: flex; justify-content: space-between; align-items: center;
            font-size: 0.78rem; color: var(--text-soft); margin-top: 4px;
        }
        .info-row strong { color: var(--gold-light); font-weight: 700; }

        .history-card {
            background: var(--bg-card); border-radius: var(--radius);
            padding: 16px 18px; box-shadow: var(--shadow-card); border: 1px solid var(--border);
        }
        .history-card h3 {
            font-size: 0.95rem; font-weight: 700; margin-bottom: 10px;
            color: var(--gold-light); display: flex; align-items: center; gap: 6px;
        }
        .history-list { list-style: none; max-height: 160px; overflow-y: auto; display: flex; flex-direction: column; gap: 5px; }
        .history-list::-webkit-scrollbar { width: 3px; }
        .history-list::-webkit-scrollbar-thumb { background: #3d3558; border-radius: 8px; }
        .history-item {
            display: flex; justify-content: space-between; align-items: center;
            padding: 7px 10px; border-radius: 8px; background: var(--bg-card2);
            font-size: 0.8rem; color: #c5bfe0; border: 1px solid transparent; transition: 0.15s;
        }
        .history-item:hover { border-color: #3d3558; }
        .history-prize { font-weight: 600; color: #f0d078; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; flex: 1; margin-right: 8px; }
        .history-time { font-size: 0.7rem; color: #7a7298; flex-shrink: 0; }
        .history-empty { text-align: center; color: #5a5580; font-size: 0.8rem; padding: 12px 0; }
        .btn-clear-history { font-size: 0.7rem; color: #a090c0; background: none; border: none; cursor: pointer; text-decoration: underline; padding: 2px 6px; transition: 0.15s; }
        .btn-clear-history:hover { color: #e87060; }

        .toast {
            position: fixed; top: 20px; left: 50%; transform: translateX(-50%) translateY(-130px);
            z-index: 200; background: #1a1630; padding: 12px 26px; border-radius: 50px;
            box-shadow: 0 8px 30px rgba(0,0,0,0.6), 0 0 0 2px #5a4d2a;
            font-weight: 600; display: flex; align-items: center; gap: 8px;
            transition: transform 0.4s cubic-bezier(0.34,1.56,0.64,1);
            pointer-events: none; max-width: 90vw; color: #f0e8d0;
        }
        .toast.show { transform: translateX(-50%) translateY(0); }

        .modal-overlay {
            position: fixed; inset: 0; z-index: 300; background: rgba(5,3,15,0.7);
            display: flex; align-items: center; justify-content: center; animation: fadeIn 0.2s;
        }
        @keyframes fadeIn { from { opacity:0; } to { opacity:1; } }
        .modal-dialog {
            background: var(--bg-card); border-radius: 24px; padding: 30px 34px; text-align: center;
            max-width: 380px; width: 90%; box-shadow: 0 20px 50px rgba(0,0,0,0.6), 0 0 60px rgba(200,150,30,0.2);
            border: 1px solid var(--border-gold); animation: pop 0.4s cubic-bezier(0.34,1.56,0.64,1);
        }
        @keyframes pop { 0%{transform:scale(0.7);opacity:0} 70%{transform:scale(1.04)} 100%{transform:scale(1);opacity:1} }
        .modal-emoji { font-size: 3.5rem; margin-bottom: 6px; }
        .modal-prize-name { font-size: 1.5rem; font-weight: 800; color: #f0d078; margin: 8px 0 18px; text-shadow: 0 0 20px rgba(240,200,80,0.4); }
        .btn-close-modal { padding: 10px 32px; border-radius: 50px; background: linear-gradient(135deg, #f0d078, #b8861e); color: #1a1630; font-weight: 700; border: none; cursor: pointer; font-family: var(--font); letter-spacing: 0.03em; transition: 0.15s; }
        .btn-close-modal:hover { box-shadow: 0 0 20px rgba(220,170,40,0.5); }

        @media (max-width: 800px) {
            .app-container { flex-direction: column; align-items: center; }
            .panel-section { max-width: 460px; width: 100%; }
            .wheel-stage { max-width: 420px; max-height: 420px; }
            .pointer { border-left-width: 13px; border-right-width: 13px; border-top-width: 26px; top: -4px; }
            .pointer::after { top: -33px; left: -6px; width: 12px; height: 12px; }
            .wheel-outer-ring { inset: -12px; }
        }
        @media (max-width: 420px) {
            .wheel-stage { max-width: 320px; max-height: 320px; }
            .btn-spin { padding: 12px 32px; font-size: 1rem; }
        }
    </style>
</head>
<body>
    <canvas class="stars-canvas" id="starsCanvas"></canvas>

    <div class="app-container">
        <div class="wheel-section">
            <div class="wheel-stage" id="wheelStage">
                <div class="wheel-outer-ring"></div>
                <div class="pointer"></div>
                <div class="wheel-spinner" id="wheelSpinner">
                    <canvas id="wheelCanvas" width="500" height="500"></canvas>
                </div>
            </div>
            <button class="btn-spin" id="btnSpin">✦ 扭 转 命 运 ✦</button>
        </div>

        <div class="panel-section">
            <div class="panel-card">
                <h3>✨ 奖品与概率配置</h3>
                <ul class="prize-list" id="prizeList"></ul>
                <div class="info-row">
                    <span>奖品数量 <strong id="prizeCount">0</strong></span>
                    <span><button class="btn-secondary" id="btnEqualWeight" style="font-size:0.7rem; padding:4px 10px;">⚖️ 等权重</button></span>
                </div>
                <div class="add-row">
                    <input type="text" class="input-new-prize" id="inputPrizeName" placeholder="输入新奖品名称..." maxlength="24" autocomplete="off">
                    <button class="btn-add" id="btnAdd">＋ 添加</button>
                </div>
            </div>

            <div class="panel-actions">
                <button class="btn-secondary" id="btnReset">🔄 恢复默认</button>
                <button class="btn-secondary danger" id="btnClearAll">🗑 清空全部</button>
            </div>

            <div class="history-card">
                <h3>📜 抽奖历史 <button class="btn-clear-history" id="btnClearHistory">清空记录</button></h3>
                <ul class="history-list" id="historyList">
                    <li class="history-empty">暂无抽奖记录</li>
                </ul>
            </div>
        </div>
    </div>

    <div class="toast" id="toast"><span id="toastIcon">🎯</span><span id="toastText"></span></div>
    <div class="modal-overlay" id="modalOverlay" style="display:none;">
        <div class="modal-dialog">
            <div class="modal-emoji" id="modalEmoji">🌟</div>
            <h2 id="modalTitle">命运之轮停转</h2>
            <div class="modal-prize-name" id="modalPrizeName"></div>
            <button class="btn-close-modal" id="btnCloseModal">确 定</button>
        </div>
    </div>

    <script>
        (function() {
            const starsCanvas = document.getElementById('starsCanvas');
            const starsCtx = starsCanvas.getContext('2d');
            const wheelStage = document.getElementById('wheelStage');
            const wheelSpinner = document.getElementById('wheelSpinner');
            const canvas = document.getElementById('wheelCanvas');
            const ctx = canvas.getContext('2d');
            const btnSpin = document.getElementById('btnSpin');
            const prizeListEl = document.getElementById('prizeList');
            const prizeCountEl = document.getElementById('prizeCount');
            const inputPrizeName = document.getElementById('inputPrizeName');
            const btnAdd = document.getElementById('btnAdd');
            const btnEqualWeight = document.getElementById('btnEqualWeight');
            const btnReset = document.getElementById('btnReset');
            const btnClearAll = document.getElementById('btnClearAll');
            const historyListEl = document.getElementById('historyList');
            const btnClearHistory = document.getElementById('btnClearHistory');
            const toast = document.getElementById('toast');
            const toastIcon = document.getElementById('toastIcon');
            const toastText = document.getElementById('toastText');
            const modalOverlay = document.getElementById('modalOverlay');
            const modalEmoji = document.getElementById('modalEmoji');
            const modalTitle = document.getElementById('modalTitle');
            const modalPrizeName = document.getElementById('modalPrizeName');
            const btnCloseModal = document.getElementById('btnCloseModal');

            const STORAGE_KEY = 'destiny_wheel_prizes_v2';
            const HISTORY_KEY = 'destiny_wheel_history_v2';
            const MAX_HISTORY = 50;

            // ★★★ 修改后的默认奖品（三个，精确权重） ★★★
            const DEFAULT_PRIZES = [
                { name: '🥈 二等奖', weight: 47.1 },
                { name: '🥇 一等奖', weight: 47.1 },
                { name: '👑 神秘大奖', weight: 5.9 }
            ];

            const SECTOR_COLORS = [
                '#6B3A5B', '#8B3A62', '#A8444A', '#C0553A',
                '#3A5B6B', '#3A6B5B', '#4A6B8A', '#5B4A7A',
                '#7A5A4A', '#5A6A4A', '#8A5A5A', '#4A5A7A',
            ];

            let prizes = [];
            let history = [];
            let currentRotation = 0;
            let isSpinning = false;
            let toastTimer = null;

            // 星空背景 (同前)
            let stars = [];
            function initStars() {
                starsCanvas.width = window.innerWidth;
                starsCanvas.height = window.innerHeight;
                stars = [];
                const count = Math.floor((starsCanvas.width * starsCanvas.height) / 2800);
                for (let i = 0; i < count; i++) {
                    stars.push({
                        x: Math.random() * starsCanvas.width,
                        y: Math.random() * starsCanvas.height,
                        r: Math.random() * 1.6 + 0.3,
                        twinkleSpeed: Math.random() * 0.015 + 0.005,
                        twinkleOffset: Math.random() * Math.PI * 2,
                        baseAlpha: Math.random() * 0.5 + 0.3,
                    });
                }
            }
            function drawStars() {
                starsCtx.clearRect(0, 0, starsCanvas.width, starsCanvas.height);
                const time = Date.now() * 0.001;
                stars.forEach(star => {
                    const alpha = star.baseAlpha + Math.sin(time * star.twinkleSpeed * 60 + star.twinkleOffset) * 0.25;
                    starsCtx.beginPath();
                    starsCtx.arc(star.x, star.y, star.r, 0, Math.PI * 2);
                    starsCtx.fillStyle = `rgba(220,210,240,${Math.max(0.08, Math.min(0.85, alpha))})`;
                    starsCtx.fill();
                    if (star.r > 1.2 && alpha > 0.55) {
                        starsCtx.beginPath();
                        starsCtx.arc(star.x, star.y, star.r * 2.5, 0, Math.PI * 2);
                        starsCtx.fillStyle = `rgba(200,180,220,${alpha * 0.12})`;
                        starsCtx.fill();
                    }
                });
                requestAnimationFrame(drawStars);
            }
            window.addEventListener('resize', initStars);
            initStars();
            drawStars();

            function loadPrizes() {
                try {
                    const raw = localStorage.getItem(STORAGE_KEY);
                    if (raw) {
                        const parsed = JSON.parse(raw);
                        if (Array.isArray(parsed) && parsed.length >= 2 && parsed.every(p => p && typeof p.name === 'string' && typeof p.weight === 'number' && p.weight > 0)) {
                            prizes = parsed.map(p => ({ name: p.name.trim(), weight: p.weight }));
                            return;
                        }
                    }
                } catch (e) {}
                prizes = DEFAULT_PRIZES.map(p => ({ ...p }));
                savePrizes();
            }

            function savePrizes() {
                localStorage.setItem(STORAGE_KEY, JSON.stringify(prizes));
            }

            function loadHistory() {
                try {
                    const raw = localStorage.getItem(HISTORY_KEY);
                    if (raw) {
                        const parsed = JSON.parse(raw);
                        if (Array.isArray(parsed)) {
                            history = parsed.filter(h => h && typeof h.name === 'string' && typeof h.time === 'number');
                            return;
                        }
                    }
                } catch (e) {}
                history = [];
            }

            function saveHistory() {
                if (history.length > MAX_HISTORY) history = history.slice(0, MAX_HISTORY);
                localStorage.setItem(HISTORY_KEY, JSON.stringify(history));
            }

            function addHistory(prizeName) {
                history.unshift({ name: prizeName, time: Date.now() });
                saveHistory();
                renderHistory();
            }

            function totalWeight() {
                return prizes.reduce((sum, p) => sum + p.weight, 0);
            }

            function drawWheel() {
                const w = canvas.width, h = canvas.height;
                const cx = w/2, cy = h/2;
                const outerR = 226, innerR = 30;
                ctx.clearRect(0, 0, w, h);
                if (prizes.length === 0) {
                    const grad = ctx.createRadialGradient(cx, cy, innerR, cx, cy, outerR);
                    grad.addColorStop(0, '#2a2440');
                    grad.addColorStop(1, '#1a1630');
                    ctx.beginPath(); ctx.arc(cx, cy, outerR, 0, 2*Math.PI);
                    ctx.fillStyle = grad; ctx.fill();
                    ctx.strokeStyle = '#3d3558'; ctx.lineWidth = 2; ctx.stroke();
                    return;
                }

                const totalW = totalWeight();
                let startAngle = -Math.PI/2;

                prizes.forEach((prize, i) => {
                    const sliceAngle = (prize.weight / totalW) * 2 * Math.PI;
                    const endAngle = startAngle + sliceAngle;
                    const midAngle = startAngle + sliceAngle/2;
                    const color = SECTOR_COLORS[i % SECTOR_COLORS.length];

                    ctx.beginPath();
                    ctx.moveTo(cx, cy);
                    ctx.arc(cx, cy, outerR, startAngle, endAngle);
                    ctx.closePath();
                    ctx.fillStyle = color;
                    ctx.fill();
                    ctx.strokeStyle = 'rgba(240,210,130,0.3)';
                    ctx.lineWidth = 1.2;
                    ctx.stroke();

                    const grad = ctx.createRadialGradient(cx, cy, outerR*0.4, cx, cy, outerR);
                    grad.addColorStop(0, 'rgba(255,255,255,0.05)');
                    grad.addColorStop(0.7, 'rgba(0,0,0,0.1)');
                    grad.addColorStop(1, 'rgba(0,0,0,0.3)');
                    ctx.fillStyle = grad;
                    ctx.fill();

                    ctx.save();
                    ctx.translate(cx, cy);
                    ctx.rotate(midAngle);
                    ctx.textAlign = 'center';
                    ctx.textBaseline = 'middle';
                    let fontSize = prizes.length <= 4 ? 16 : (prizes.length <= 6 ? 13 : 11);
                    const textR = outerR * 0.6;
                    ctx.fillStyle = 'rgba(0,0,0,0.35)';
                    ctx.font = `bold ${fontSize}px "${getFont()}"`;
                    ctx.fillText(prize.name, textR+0.6, 0.6);
                    ctx.fillStyle = '#f8f0e0';
                    ctx.shadowColor = 'rgba(0,0,0,0.5)';
                    ctx.shadowBlur = 3;
                    ctx.fillText(prize.name, textR, 0);
                    ctx.shadowBlur = 0;
                    ctx.restore();

                    startAngle = endAngle;
                });

                const centerGrad = ctx.createRadialGradient(cx, cy, 0, cx, cy, innerR);
                centerGrad.addColorStop(0, '#f8f0e0');
                centerGrad.addColorStop(0.4, '#e8d5a0');
                centerGrad.addColorStop(0.8, '#b8963a');
                centerGrad.addColorStop(1, '#6b4a1a');
                ctx.beginPath(); ctx.arc(cx, cy, innerR, 0, 2*Math.PI);
                ctx.fillStyle = centerGrad; ctx.fill();
                ctx.strokeStyle = 'rgba(240,210,130,0.6)'; ctx.lineWidth = 2.5; ctx.stroke();
                ctx.beginPath(); ctx.arc(cx, cy, 8, 0, 2*Math.PI);
                ctx.fillStyle = '#fef8e8'; ctx.fill();
                ctx.strokeStyle = '#c9a030'; ctx.lineWidth = 1; ctx.stroke();
            }

            function getFont() { return "'PingFang SC','Microsoft YaHei','Helvetica Neue',sans-serif"; }

            function spinByWeight() {
                const totalW = totalWeight();
                let rand = Math.random() * totalW;
                for (let i = 0; i < prizes.length; i++) {
                    rand -= prizes[i].weight;
                    if (rand <= 0) return i;
                }
                return prizes.length - 1;
            }

            function startSpin() {
                if (isSpinning || prizes.length < 2) {
                    showToast('⚠️', '至少需要2个奖品才能抽奖');
                    shakeElement(btnSpin);
                    return;
                }
                isSpinning = true;
                updateUIState();
                btnSpin.disabled = true;
                btnSpin.textContent = '🌌 命运转动中...';
                wheelStage.classList.add('spinning');

                const winIndex = spinByWeight();
                const totalW = totalWeight();
                let accumulatedAngle = 0;
                for (let i = 0; i < winIndex; i++) {
                    accumulatedAngle += (prizes[i].weight / totalW) * 360;
                }
                const sectorMidDeg = accumulatedAngle + (prizes[winIndex].weight / totalW) * 180;
                let targetOffset = (-sectorMidDeg) % 360;
                if (targetOffset < 0) targetOffset += 360;
                const extraSpins = Math.floor(Math.random() * 4) + 5;
                const totalSpinDeg = extraSpins * 360 + targetOffset;
                const targetRotation = currentRotation + totalSpinDeg;

                wheelSpinner.style.transition = `transform ${4.6 + extraSpins*0.25}s cubic-bezier(0.05,0.68,0.08,0.98)`;
                wheelSpinner.style.transform = `rotate(${targetRotation}deg)`;
                currentRotation = targetRotation;

                const onEnd = () => {
                    wheelSpinner.removeEventListener('transitionend', onEnd);
                    finishSpin(winIndex);
                };
                wheelSpinner.addEventListener('transitionend', onEnd, { once: true });
                setTimeout(() => { if (isSpinning) onEnd(); }, 7500);
            }

            function finishSpin(winIndex) {
                wheelSpinner.style.transition = '';
                wheelStage.classList.remove('spinning');
                isSpinning = false;
                btnSpin.disabled = false;
                btnSpin.textContent = '✦ 扭 转 命 运 ✦';
                updateUIState();
                const prize = prizes[winIndex];
                addHistory(prize.name);
                showResultModal(prize.name);
                showToast('🌟', `命运之选：${prize.name}`);
            }

            function renderPrizeList() {
                prizeListEl.innerHTML = '';
                const totalW = totalWeight();
                prizes.forEach((prize, idx) => {
                    const percent = totalW > 0 ? ((prize.weight / totalW) * 100).toFixed(1) : '0.0';
                    const li = document.createElement('li');
                    li.className = 'prize-item';
                    li.innerHTML = `
                        <span class="color-dot" style="background:${SECTOR_COLORS[idx % SECTOR_COLORS.length]}; box-shadow: 0 0 6px ${SECTOR_COLORS[idx % SECTOR_COLORS.length]};"></span>
                        <span class="prize-name" title="${escapeHtml(prize.name)}">${escapeHtml(prize.name)}</span>
                        <input type="number" class="weight-input" value="${prize.weight}" min="0.1" step="0.1" data-index="${idx}" ${isSpinning ? 'disabled' : ''}>
                        <span class="percent-badge">${percent}%</span>
                        <button class="btn-delete" data-index="${idx}" ${isSpinning ? 'disabled' : ''}>×</button>
                    `;
                    prizeListEl.appendChild(li);
                });

                prizeListEl.querySelectorAll('.weight-input').forEach(inp => {
                    inp.addEventListener('input', function() {
                        const idx = parseInt(this.dataset.index);
                        let val = parseFloat(this.value);
                        if (isNaN(val) || val < 0.1) val = 0.1;
                        if (val > 999) val = 999;
                        prizes[idx].weight = val;
                        this.value = val;
                        savePrizes();
                        drawWheel();
                        renderPrizeList();
                    });
                });

                prizeListEl.querySelectorAll('.btn-delete').forEach(btn => {
                    btn.addEventListener('click', function() {
                        if (isSpinning) return;
                        const idx = parseInt(this.dataset.index);
                        if (prizes.length <= 2) { showToast('⚠️', '至少保留2个奖品'); return; }
                        prizes.splice(idx, 1);
                        savePrizes();
                        resetRotation();
                        drawWheel();
                        renderPrizeList();
                        updateCount();
                    });
                });
                updateCount();
            }

            function renderHistory() {
                historyListEl.innerHTML = '';
                if (history.length === 0) {
                    historyListEl.innerHTML = '<li class="history-empty">🌙 暂无抽奖记录</li>';
                    return;
                }
                history.forEach(h => {
                    const li = document.createElement('li');
                    li.className = 'history-item';
                    const timeStr = formatTime(h.time);
                    li.innerHTML = `<span class="history-prize" title="${escapeHtml(h.name)}">${escapeHtml(h.name)}</span><span class="history-time">${timeStr}</span>`;
                    historyListEl.appendChild(li);
                });
            }

            function formatTime(timestamp) {
                const d = new Date(timestamp);
                const now = new Date();
                const diffMs = now - d;
                const diffMin = Math.floor(diffMs / 60000);
                if (diffMin < 1) return '刚刚';
                if (diffMin < 60) return `${diffMin}分钟前`;
                const diffHour = Math.floor(diffMin / 60);
                if (diffHour < 24) return `${diffHour}小时前`;
                const month = d.getMonth()+1;
                const day = d.getDate();
                return `${month}/${day}`;
            }

            function updateCount() { prizeCountEl.textContent = prizes.length; }

            function updateUIState() {
                btnAdd.disabled = isSpinning;
                btnReset.disabled = isSpinning;
                btnClearAll.disabled = isSpinning || prizes.length === 0;
                btnEqualWeight.disabled = isSpinning || prizes.length === 0;
                inputPrizeName.disabled = isSpinning;
                document.querySelectorAll('.weight-input').forEach(i => i.disabled = isSpinning);
                document.querySelectorAll('.btn-delete').forEach(b => b.disabled = isSpinning);
            }

            function resetRotation() {
                wheelSpinner.style.transition = 'none';
                wheelSpinner.style.transform = 'rotate(0deg)';
                currentRotation = 0;
                wheelSpinner.offsetHeight;
                wheelSpinner.style.transition = '';
            }

            function addPrize() {
                if (isSpinning) return;
                const name = inputPrizeName.value.trim();
                if (!name) return;
                if (prizes.length >= 20) { showToast('⚠️', '最多支持20个奖品'); return; }
                prizes.push({ name, weight: 1 });
                savePrizes();
                resetRotation();
                drawWheel();
                renderPrizeList();
                inputPrizeName.value = '';
                inputPrizeName.focus();
            }

            function setEqualWeights() {
                if (isSpinning || prizes.length === 0) return;
                prizes.forEach(p => p.weight = 1);
                savePrizes();
                drawWheel();
                renderPrizeList();
                showToast('⚖️', '所有奖品概率已均等');
            }

            function resetToDefault() {
                prizes = DEFAULT_PRIZES.map(p => ({ ...p }));
                savePrizes();
                resetRotation();
                drawWheel();
                renderPrizeList();
                showToast('🔄', '已恢复默认奖品');
            }

            function clearAll() {
                if (isSpinning || prizes.length === 0) return;
                if (confirm('确定要清空所有奖品吗？')) {
                    prizes = [];
                    savePrizes();
                    resetRotation();
                    drawWheel();
                    renderPrizeList();
                    showToast('🗑', '已清空全部奖品');
                }
            }

            function clearHistory() {
                history = [];
                saveHistory();
                renderHistory();
                showToast('🧹', '历史记录已清空');
            }

            function showResultModal(name) {
                const isConsolation = /谢谢|遗憾|未中|下次|再来|没有|sorry|empty|no\s+prize/i.test(name);
                modalEmoji.textContent = isConsolation ? '😊' : '🌟';
                modalTitle.textContent = isConsolation ? '命运微笑，下次好运' : '命运之轮为你停驻';
                modalPrizeName.textContent = name;
                modalOverlay.style.display = 'flex';
            }
            function closeModal() { modalOverlay.style.display = 'none'; }

            function showToast(icon, msg) {
                if (toastTimer) clearTimeout(toastTimer);
                toastIcon.textContent = icon;
                toastText.textContent = msg;
                toast.classList.add('show');
                toastTimer = setTimeout(() => toast.classList.remove('show'), 2600);
            }

            function shakeElement(el) {
                el.style.animation = 'none';
                el.offsetHeight;
                el.style.animation = 'shake 0.5s ease';
                setTimeout(() => { el.style.animation = ''; }, 500);
            }

            function escapeHtml(s) {
                const d = document.createElement('div');
                d.textContent = s;
                return d.innerHTML;
            }

            const shakeStyle = document.createElement('style');
            shakeStyle.textContent = `
                @keyframes shake {
                    0%,100% { transform: translateX(0); }
                    20% { transform: translateX(-5px); }
                    40% { transform: translateX(5px); }
                    60% { transform: translateX(-4px); }
                    80% { transform: translateX(4px); }
                }
            `;
            document.head.appendChild(shakeStyle);

            btnSpin.addEventListener('click', startSpin);
            btnAdd.addEventListener('click', addPrize);
            inputPrizeName.addEventListener('keydown', e => { if (e.key === 'Enter') addPrize(); });
            btnEqualWeight.addEventListener('click', setEqualWeights);
            btnReset.addEventListener('click', resetToDefault);
            btnClearAll.addEventListener('click', clearAll);
            btnClearHistory.addEventListener('click', clearHistory);
            btnCloseModal.addEventListener('click', closeModal);
            modalOverlay.addEventListener('click', e => { if (e.target === modalOverlay) closeModal(); });
            document.addEventListener('keydown', e => { if (e.key === 'Escape') closeModal(); });

            function init() {
                loadPrizes();
                loadHistory();
                if (prizes.length < 2) {
                    prizes = DEFAULT_PRIZES.map(p => ({ ...p }));
                    savePrizes();
                }
                drawWheel();
                renderPrizeList();
                renderHistory();
                updateUIState();
                resetRotation();
            }
            init();
        })();
    </script>
</body>
</html>
