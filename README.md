<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>终端解密 - Terminal Decryption</title>
    <style>
        :root {
            --bg: #0a0a0f;
            --surface: #111118;
            --text: #c8d6e5;
            --accent: #00ff88;
            --accent-dim: #00cc6a;
            --warn: #ffb347;
            --error: #ff4757;
            --info: #4da6ff;
            --glow: 0 0 12px;
            --font-mono: 'Courier New', 'Consolas', 'Monaco', 'Liberation Mono', monospace;
        }

        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            background: var(--bg);
            font-family: var(--font-mono);
            color: var(--text);
            min-height: 100vh;
            display: flex;
            align-items: center;
            justify-content: center;
            overflow-x: hidden;
            user-select: none;
            -webkit-tap-highlight-color: transparent;
            cursor: default;
        }

        /* 粒子背景 */
        #particles {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            pointer-events: none;
            z-index: 0;
        }

        /* 扫描线 */
        .scanlines {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            pointer-events: none;
            z-index: 1;
            background: repeating-linear-gradient(0deg,
                    transparent,
                    transparent 2px,
                    rgba(0, 0, 0, 0.03) 2px,
                    rgba(0, 0, 0, 0.03) 4px);
        }
        .scanlines::after {
            content: '';
            position: absolute;
            top: -100%;
            left: 0;
            width: 100%;
            height: 80px;
            background: linear-gradient(180deg,
                    transparent 0%,
                    rgba(0, 255, 136, 0.015) 40%,
                    rgba(0, 255, 136, 0.03) 50%,
                    rgba(0, 255, 136, 0.015) 60%,
                    transparent 100%);
            animation: scanDown 6s linear infinite;
        }
        @keyframes scanDown {
            0% {
                top: -80px;
            }
            100% {
                top: 100%;
            }
        }

        /* 主容器 */
        .main-container {
            position: relative;
            z-index: 2;
            width: 92vw;
            max-width: 600px;
            background: var(--surface);
            border: 1px solid rgba(0, 255, 136, 0.25);
            border-radius: 12px;
            padding: 28px 24px 24px;
            box-shadow:
                0 0 40px rgba(0, 255, 136, 0.06),
                0 0 80px rgba(0, 0, 0, 0.5),
                inset 0 0 0 1px rgba(0, 255, 136, 0.04);
            animation: containerIn 0.6s ease-out;
        }
        @keyframes containerIn {
            from {
                opacity: 0;
                transform: translateY(30px);
            }
            to {
                opacity: 1;
                transform: translateY(0);
            }
        }

        /* 头部 */
        .header {
            display: flex;
            align-items: center;
            justify-content: space-between;
            margin-bottom: 20px;
            padding-bottom: 16px;
            border-bottom: 1px solid rgba(0, 255, 136, 0.15);
            flex-wrap: wrap;
            gap: 10px;
        }
        .logo {
            display: flex;
            align-items: center;
            gap: 10px;
        }
        .logo-icon {
            width: 36px;
            height: 36px;
            border: 2px solid var(--accent);
            border-radius: 6px;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 18px;
            color: var(--accent);
            box-shadow: var(--glow) rgba(0, 255, 136, 0.3);
            animation: logoPulse 2s ease-in-out infinite;
        }
        @keyframes logoPulse {
            0%,
            100% {
                box-shadow: 0 0 8px rgba(0, 255, 136, 0.3);
            }
            50% {
                box-shadow: 0 0 20px rgba(0, 255, 136, 0.6);
            }
        }
        .logo-text {
            font-size: 1.1rem;
            font-weight: bold;
            color: var(--accent);
            letter-spacing: 2px;
        }
        .level-indicator {
            display: flex;
            gap: 8px;
            align-items: center;
        }
        .level-dot {
            width: 10px;
            height: 10px;
            border-radius: 50%;
            background: rgba(255, 255, 255, 0.15);
            transition: all 0.4s ease;
        }
        .level-dot.completed {
            background: var(--accent);
            box-shadow: 0 0 8px rgba(0, 255, 136, 0.6);
        }
        .level-dot.current {
            background: var(--warn);
            box-shadow: 0 0 10px rgba(255, 179, 71, 0.7);
            animation: dotBlink 1s ease-in-out infinite;
        }
        @keyframes dotBlink {
            0%,
            100% {
                box-shadow: 0 0 6px rgba(255, 179, 71, 0.5);
            }
            50% {
                box-shadow: 0 0 16px rgba(255, 179, 71, 1);
            }
        }
        .level-label {
            font-size: 0.75rem;
            color: rgba(255, 255, 255, 0.5);
            letter-spacing: 1px;
        }

        /* 谜题区域 */
        .puzzle-area {
            background: rgba(0, 0, 0, 0.4);
            border: 1px solid rgba(255, 255, 255, 0.08);
            border-radius: 8px;
            padding: 20px 18px;
            margin-bottom: 18px;
            min-height: 120px;
            position: relative;
            transition: all 0.3s ease;
            letter-spacing: 0.5px;
            line-height: 1.7;
        }
        .puzzle-area.shake {
            animation: shake 0.5s ease;
        }
        @keyframes shake {
            0%,
            100% {
                transform: translateX(0);
            }
            20% {
                transform: translateX(-8px);
            }
            40% {
                transform: translateX(8px);
            }
            60% {
                transform: translateX(-5px);
            }
            80% {
                transform: translateX(5px);
            }
        }
        .puzzle-area.success-glow {
            box-shadow: 0 0 30px rgba(0, 255, 136, 0.3);
            border-color: var(--accent);
        }

        /* 终端日志样式(关卡1) */
        .terminal-log {
            font-family: var(--font-mono);
            font-size: 0.9rem;
            color: #8899aa;
            white-space: pre-wrap;
            word-break: break-all;
        }
        .terminal-log .hl {
            color: #bccdd8;
            font-weight: bold;
            transition: color 0.3s;
            cursor: default;
            position: relative;
        }
        .terminal-log .hl:hover {
            color: var(--accent);
            text-shadow: 0 0 8px rgba(0, 255, 136, 0.5);
        }
        .terminal-log .dim {
            color: #556677;
        }
        .terminal-log .warn-line {
            color: var(--warn);
        }
        .terminal-log .info-line {
            color: var(--info);
        }
        .terminal-log .accent-line {
            color: var(--accent);
        }

        /* 密文样式(关卡2) */
        .cipher-display {
            text-align: center;
            font-size: 2.2rem;
            letter-spacing: 12px;
            color: var(--warn);
            text-shadow: 0 0 20px rgba(255, 179, 71, 0.4);
            font-weight: bold;
            padding: 10px 0;
        }

        /* 二进制样式(关卡4) */
        .binary-display {
            text-align: center;
            font-size: 1rem;
            letter-spacing: 2px;
            color: var(--info);
            text-shadow: 0 0 10px rgba(77, 166, 255, 0.3);
            word-break: break-all;
            padding: 8px 0;
            line-height: 2;
        }

        /* 西蒙游戏按钮区 */
        .simon-grid {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 12px;
            max-width: 280px;
            margin: 0 auto;
            padding: 10px 0;
        }
        .simon-btn {
            aspect-ratio: 1;
            border-radius: 12px;
            border: 2px solid rgba(255, 255, 255, 0.2);
            cursor: pointer;
            transition: all 0.15s ease;
            position: relative;
            min-height: 90px;
            -webkit-tap-highlight-color: transparent;
            outline: none;
        }
        .simon-btn:active {
            transform: scale(0.93);
        }
        .simon-btn.red {
            background: #2a1015;
            border-color: rgba(255, 59, 59, 0.5);
        }
        .simon-btn.blue {
            background: #101a2a;
            border-color: rgba(59, 127, 255, 0.5);
        }
        .simon-btn.green {
            background: #102a18;
            border-color: rgba(59, 255, 111, 0.5);
        }
        .simon-btn.yellow {
            background: #2a2410;
            border-color: rgba(255, 204, 59, 0.5);
        }
        .simon-btn.red.lit {
            background: #ff3b3b;
            box-shadow: 0 0 40px rgba(255, 59, 59, 0.8);
            border-color: #ff6b6b;
            transform: scale(1.04);
        }
        .simon-btn.blue.lit {
            background: #3b7fff;
            box-shadow: 0 0 40px rgba(59, 127, 255, 0.8);
            border-color: #6ba0ff;
            transform: scale(1.04);
        }
        .simon-btn.green.lit {
            background: #3bff6f;
            box-shadow: 0 0 40px rgba(59, 255, 111, 0.8);
            border-color: #6fff95;
            transform: scale(1.04);
        }
        .simon-btn.yellow.lit {
            background: #ffcc3b;
            box-shadow: 0 0 40px rgba(255, 204, 59, 0.8);
            border-color: #ffda6b;
            transform: scale(1.04);
        }
        .simon-btn.disabled {
            pointer-events: none;
            opacity: 0.5;
        }
        .simon-status {
            text-align: center;
            margin-top: 10px;
            font-size: 0.85rem;
            color: rgba(255, 255, 255, 0.6);
            letter-spacing: 1px;
            min-height: 20px;
        }

        /* 输入区 */
        .input-row {
            display: flex;
            gap: 10px;
            align-items: center;
            flex-wrap: wrap;
        }
        .input-field {
            flex: 1;
            min-width: 150px;
            background: rgba(0, 0, 0, 0.5);
            border: 1px solid rgba(0, 255, 136, 0.3);
            border-radius: 6px;
            padding: 12px 14px;
            font-family: var(--font-mono);
            font-size: 1rem;
            color: var(--accent);
            letter-spacing: 1px;
            outline: none;
            transition: all 0.3s ease;
            caret-color: var(--accent);
        }
        .input-field:focus {
            border-color: var(--accent);
            box-shadow: 0 0 16px rgba(0, 255, 136, 0.2);
            background: rgba(0, 0, 0, 0.7);
        }
        .input-field::placeholder {
            color: rgba(255, 255, 255, 0.2);
            letter-spacing: 0.5px;
        }
        .btn {
            padding: 12px 20px;
            border-radius: 6px;
            font-family: var(--font-mono);
            font-size: 0.9rem;
            letter-spacing: 1px;
            cursor: pointer;
            border: 1px solid;
            transition: all 0.3s ease;
            white-space: nowrap;
            outline: none;
            -webkit-tap-highlight-color: transparent;
        }
        .btn-submit {
            background: rgba(0, 255, 136, 0.1);
            border-color: var(--accent);
            color: var(--accent);
            font-weight: bold;
        }
        .btn-submit:hover {
            background: rgba(0, 255, 136, 0.2);
            box-shadow: 0 0 20px rgba(0, 255, 136, 0.3);
        }
        .btn-submit:active {
            background: rgba(0, 255, 136, 0.35);
            transform: scale(0.96);
        }
        .btn-hint {
            background: rgba(255, 179, 71, 0.08);
            border-color: rgba(255, 179, 71, 0.4);
            color: var(--warn);
            font-size: 0.8rem;
            padding: 10px 16px;
        }
        .btn-hint:hover {
            background: rgba(255, 179, 71, 0.18);
            box-shadow: 0 0 14px rgba(255, 179, 71, 0.25);
        }
        .btn-hint:active {
            background: rgba(255, 179, 71, 0.3);
            transform: scale(0.96);
        }

        /* 反馈信息 */
        .feedback {
            min-height: 22px;
            margin-top: 10px;
            font-size: 0.8rem;
            letter-spacing: 0.5px;
            transition: all 0.3s ease;
        }
        .feedback.error {
            color: var(--error);
            text-shadow: 0 0 6px rgba(255, 71, 87, 0.4);
        }
        .feedback.success {
            color: var(--accent);
            text-shadow: 0 0 6px rgba(0, 255, 136, 0.4);
        }
        .feedback.info {
            color: var(--info);
        }

        /* 提示面板 */
        .hint-panel {
            margin-top: 8px;
            padding: 10px 14px;
            background: rgba(255, 179, 71, 0.06);
            border: 1px solid rgba(255, 179, 71, 0.2);
            border-radius: 6px;
            font-size: 0.8rem;
            color: #ddb870;
            display: none;
            letter-spacing: 0.5px;
            animation: fadeIn 0.3s ease;
        }
        .hint-panel.visible {
            display: block;
        }
        @keyframes fadeIn {
            from {
                opacity: 0;
                transform: translateY(-6px);
            }
            to {
                opacity: 1;
                transform: translateY(0);
            }
        }

        /* 通关弹窗 */
        .overlay {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.8);
            z-index: 100;
            display: flex;
            align-items: center;
            justify-content: center;
            animation: fadeIn 0.4s ease;
        }
        .overlay.hidden {
            display: none;
        }
        .victory-card {
            background: var(--surface);
            border: 2px solid var(--accent);
            border-radius: 16px;
            padding: 30px 24px;
            text-align: center;
            max-width: 440px;
            width: 90%;
            box-shadow: 0 0 60px rgba(0, 255, 136, 0.25);
            animation: popIn 0.5s cubic-bezier(0.175, 0.885, 0.32, 1.275);
        }
        @keyframes popIn {
            from {
                transform: scale(0.8);
                opacity: 0;
            }
            to {
                transform: scale(1);
                opacity: 1;
            }
        }
        .victory-icon {
            font-size: 4rem;
            margin-bottom: 10px;
            animation: logoPulse 1.5s ease-in-out infinite;
        }
        .victory-title {
            font-size: 1.8rem;
            color: var(--accent);
            letter-spacing: 3px;
            margin-bottom: 8px;
        }
        .victory-sub {
            font-size: 0.85rem;
            color: rgba(255, 255, 255, 0.6);
            letter-spacing: 1px;
            margin-bottom: 20px;
        }
        .btn-restart {
            background: rgba(0, 255, 136, 0.15);
            border: 1px solid var(--accent);
            color: var(--accent);
            padding: 12px 28px;
            border-radius: 6px;
            font-family: var(--font-mono);
            font-size: 1rem;
            cursor: pointer;
            letter-spacing: 1px;
            transition: all 0.3s ease;
        }
        .btn-restart:hover {
            background: rgba(0, 255, 136, 0.3);
            box-shadow: 0 0 25px rgba(0, 255, 136, 0.4);
        }

        /* 响应式 */
        @media (max-width: 480px) {
            .main-container {
                padding: 18px 14px 16px;
                border-radius: 8px;
            }
            .cipher-display {
                font-size: 1.6rem;
                letter-spacing: 8px;
            }
            .binary-display {
                font-size: 0.8rem;
                letter-spacing: 1px;
            }
            .simon-grid {
                max-width: 220px;
                gap: 8px;
            }
            .simon-btn {
                min-height: 65px;
                border-radius: 8px;
            }
            .input-field {
                font-size: 0.85rem;
                padding: 10px 12px;
            }
            .btn {
                font-size: 0.8rem;
                padding: 10px 14px;
            }
            .logo-text {
                font-size: 0.9rem;
            }
            .logo-icon {
                width: 28px;
                height: 28px;
                font-size: 14px;
            }
        }
    </style>
</head>
<body>

    <!-- 粒子背景 -->
    <canvas id="particles"></canvas>
    <!-- 扫描线 -->
    <div class="scanlines"></div>

    <!-- 主容器 -->
    <div class="main-container" id="mainContainer">
        <!-- 头部 -->
        <div class="header">
            <div class="logo">
                <div class="logo-icon">◈</div>
                <span class="logo-text">TERMINAL</span>
            </div>
            <div class="level-indicator" id="levelIndicator">
                <span class="level-label">关卡</span>
                <div class="level-dot current" data-lv="1"></div>
                <div class="level-dot" data-lv="2"></div>
                <div class="level-dot" data-lv="3"></div>
                <div class="level-dot" data-lv="4"></div>
            </div>
        </div>

        <!-- 谜题区 -->
        <div class="puzzle-area" id="puzzleArea"></div>

        <!-- 输入区 -->
        <div id="inputZone"></div>

        <!-- 反馈 -->
        <div class="feedback" id="feedback"></div>

        <!-- 提示面板 -->
        <div class="hint-panel" id="hintPanel"></div>
    </div>

    <!-- 胜利弹窗 -->
    <div class="overlay hidden" id="victoryOverlay">
        <div class="victory-card">
            <div class="victory-icon">🔓</div>
            <div class="victory-title">ACCESS GRANTED</div>
            <div class="victory-sub">你已破解所有谜题，系统完全开放。</div>
            <p style="color:#8899aa;font-size:0.8rem;margin-bottom:16px;letter-spacing:1px;">
                失败次数：<span id="totalFails">0</span> &nbsp;|&nbsp;
                提示使用：<span id="totalHints">0</span>
            </p>
            <button class="btn-restart" onclick="restartGame()">🔄 重新挑战</button>
        </div>
    </div>

    <script>
        (function() {
            // ============ 粒子背景 ============
            const canvas = document.getElementById('particles');
            const ctx = canvas.getContext('2d');
            let particles = [];
            const particleCount = 35;

            function resizeCanvas() {
                canvas.width = window.innerWidth;
                canvas.height = window.innerHeight;
            }
            resizeCanvas();
            window.addEventListener('resize', resizeCanvas);

            class Particle {
                constructor() {
                    this.reset();
                    this.y = Math.random() * canvas.height;
                }
                reset() {
                    this.x = Math.random() * canvas.width;
                    this.y = -10;
                    this.size = Math.random() * 1.8 + 0.4;
                    this.speedY = Math.random() * 0.4 + 0.1;
                    this.speedX = (Math.random() - 0.5) * 0.25;
                    this.opacity = Math.random() * 0.5 + 0.2;
                    this.flickerSpeed = Math.random() * 0.02 + 0.005;
                    this.flickerOffset = Math.random() * Math.PI * 2;
                }
                update() {
                    this.y += this.speedY;
                    this.x += this.speedX;
                    this.opacity += Math.sin(Date.now() * this.flickerSpeed + this.flickerOffset) * 0.008;
                    this.opacity = Math.max(0.08, Math.min(0.7, this.opacity));
                    if (this.y > canvas.height + 10) {
                        this.reset();
                        this.y = -10;
                    }
                    if (this.x < -10) this.x = canvas.width + 10;
                    if (this.x > canvas.width + 10) this.x = -10;
                }
                draw(ctx) {
                    ctx.beginPath();
                    ctx.arc(this.x, this.y, this.size, 0, Math.PI * 2);
                    ctx.fillStyle = `rgba(0,255,136,${this.opacity})`;
                    ctx.fill();
                    // 微光晕
                    if (this.size > 1.2) {
                        ctx.beginPath();
                        ctx.arc(this.x, this.y, this.size * 2.5, 0, Math.PI * 2);
                        ctx.fillStyle = `rgba(0,255,136,${this.opacity * 0.15})`;
                        ctx.fill();
                    }
                }
            }

            for (let i = 0; i < particleCount; i++) {
                const p = new Particle();
                p.y = Math.random() * canvas.height;
                particles.push(p);
            }

            function animateParticles() {
                ctx.clearRect(0, 0, canvas.width, canvas.height);
                particles.forEach(p => {
                    p.update();
                    p.draw(ctx);
                });
                requestAnimationFrame(animateParticles);
            }
            animateParticles();

            // ============ 游戏状态 ============
            const TOTAL_LEVELS = 4;
            let currentLevel = 1;
            let totalFails = 0;
            let totalHintsUsed = 0;
            let hintLevel = 0; // 当前关卡提示使用次数
            let simonSequence = [];
            let simonPlayerIndex = 0;
            let simonRound = 0;
            let simonIsDisplaying = false;
            let simonPlayerCanClick = false;

            // DOM引用
            const puzzleArea = document.getElementById('puzzleArea');
            const inputZone = document.getElementById('inputZone');
            const feedback = document.getElementById('feedback');
            const hintPanel = document.getElementById('hintPanel');
            const levelIndicator = document.getElementById('levelIndicator');
            const victoryOverlay = document.getElementById('victoryOverlay');
            const mainContainer = document.getElementById('mainContainer');

            // ============ 工具函数 ============
            function updateLevelDots() {
                const dots = levelIndicator.querySelectorAll('.level-dot');
                dots.forEach(dot => {
                    const lv = parseInt(dot.dataset.lv);
                    dot.classList.remove('completed', 'current');
                    if (lv < currentLevel) dot.classList.add('completed');
                    if (lv === currentLevel) dot.classList.add('current');
                });
            }

            function showFeedback(msg, type) {
                feedback.textContent = msg;
                feedback.className = 'feedback ' + type;
            }

            function clearFeedback() {
                feedback.textContent = '';
                feedback.className = 'feedback';
            }

            function shakePuzzle() {
                puzzleArea.classList.add('shake');
                setTimeout(() => puzzleArea.classList.remove('shake'), 500);
            }

            function glowPuzzle() {
                puzzleArea.classList.add('success-glow');
                setTimeout(() => puzzleArea.classList.remove('success-glow'), 800);
            }

            function showHint(msg) {
                hintPanel.textContent = '💡 ' + msg;
                hintPanel.classList.add('visible');
            }

            function hideHint() {
                hintPanel.classList.remove('visible');
            }

            function resetHintLevel() {
                hintLevel = 0;
                hideHint();
            }

            // ============ 关卡渲染 ============
            function renderLevel1() {
                resetHintLevel();
                clearFeedback();
                puzzleArea.innerHTML = `
                    <div class="terminal-log">
                        <span class="dim">[SYS]</span> <span class="dim">初始化加密协议...</span>
                        <br><span class="dim">[SYS]</span> <span class="dim">建立安全通道...完成</span>
                        <br><span class="hl">S</span>ecure tunnel established on port 443
                        <br><span class="hl">E</span>ncryption handshake verified (AES-256)
                        <br><span class="hl">C</span>ertificate authority: trusted
                        <br><span class="hl">R</span>outing through anonymous relay nodes
                        <br><span class="hl">E</span>nd-to-end encryption active
                        <br><span class="hl">T</span>erminal session authenticated
                        <br><span class="dim">[OK]</span> <span class="dim">所有模块就绪。</span>
                        <br><span class="warn-line">[WARN]</span> <span class="warn-line">检测到异常流量模式</span>
                        <br><span class="dim">[DBG]</span> <span class="dim">心跳信号正常 @</span> ${new Date().toLocaleTimeString()}
                    </div>
                `;
                inputZone.innerHTML = `
                    <div class="input-row">
                        <input type="text" class="input-field" id="answerInput" placeholder="输入发现的密码..." autocomplete="off" autocorrect="off" autocapitalize="off" spellcheck="false">
                        <button class="btn btn-submit" id="submitBtn">提交 ▸</button>
                        <button class="btn btn-hint" id="hintBtn">提示 ?</button>
                    </div>
                `;
                bindInputEvents(1);
            }

            function renderLevel2() {
                resetHintLevel();
                clearFeedback();
                puzzleArea.innerHTML = `
                    <p style="color:#8899aa;text-align:center;font-size:0.8rem;letter-spacing:1px;margin-bottom:8px;">
                        ⚡ 拦截到加密传输数据 ⚡
                    </p>
                    <div class="cipher-display">ydxow</div>
                    <p style="color:rgba(255,255,255,0.35);text-align:center;font-size:0.7rem;margin-top:8px;letter-spacing:1px;">
                        加密方式：凯撒移位 | 拦截节点：ROME-3
                    </p>
                `;
                inputZone.innerHTML = `
                    <div class="input-row">
                        <input type="text" class="input-field" id="answerInput" placeholder="解密后的明文..." autocomplete="off" autocorrect="off" autocapitalize="off" spellcheck="false">
                        <button class="btn btn-submit" id="submitBtn">解密 ▸</button>
                        <button class="btn btn-hint" id="hintBtn">提示 ?</button>
                    </div>
                `;
                bindInputEvents(2);
            }

            function renderLevel3() {
                resetHintLevel();
                clearFeedback();
                simonSequence = [];
                simonPlayerIndex = 0;
                simonRound = 0;
                simonIsDisplaying = false;
                simonPlayerCanClick = false;
                puzzleArea.innerHTML = `
                    <p style="color:#8899aa;text-align:center;font-size:0.8rem;letter-spacing:1px;margin-bottom:10px;">
                        🧠 记忆复现协议 - 观察闪烁序列并复现
                    </p>
                    <div class="simon-grid">
                        <button class="simon-btn red" data-color="red" disabled></button>
                        <button class="simon-btn blue" data-color="blue" disabled></button>
                        <button class="simon-btn green" data-color="green" disabled></button>
                        <button class="simon-btn yellow" data-color="yellow" disabled></button>
                    </div>
                    <div class="simon-status" id="simonStatus">准备中...</div>
                `;
                inputZone.innerHTML = `
                    <div style="text-align:center;">
                        <button class="btn btn-submit" id="simonStartBtn" style="font-size:1rem;padding:14px 30px;">
                            ▶ 开始观察
                        </button>
                        <button class="btn btn-hint" id="hintBtn" style="margin-left:8px;">提示 ?</button>
                    </div>
                `;
                // 绑定开始按钮
                const startBtn = document.getElementById('simonStartBtn');
                if (startBtn) {
                    startBtn.addEventListener('click', startSimonRound);
                }
                const hintBtn = document.getElementById('hintBtn');
                if (hintBtn) {
                    hintBtn.addEventListener('click', () => useHint(3));
                }
                // 绑定颜色按钮事件
                document.querySelectorAll('.simon-btn').forEach(btn => {
                    btn.addEventListener('click', () => handleSimonClick(btn));
                });
            }

            function renderLevel4() {
                resetHintLevel();
                clearFeedback();
                puzzleArea.innerHTML = `
                    <p style="color:#8899aa;text-align:center;font-size:0.8rem;letter-spacing:1px;margin-bottom:8px;">
                        📡 原始数据流 - 二进制编码
                    </p>
                    <div class="binary-display">
                        01100110 01110010 01100101 01100101
                    </div>
                    <p style="color:rgba(255,255,255,0.3);text-align:center;font-size:0.7rem;margin-top:8px;letter-spacing:1px;">
                        编码标准：ASCII-8bit | 字节数：4
                    </p>
                `;
                inputZone.innerHTML = `
                    <div class="input-row">
                        <input type="text" class="input-field" id="answerInput" placeholder="解码后的单词..." autocomplete="off" autocorrect="off" autocapitalize="off" spellcheck="false">
                        <button class="btn btn-submit" id="submitBtn">解码 ▸</button>
                        <button class="btn btn-hint" id="hintBtn">提示 ?</button>
                    </div>
                `;
                bindInputEvents(4);
            }

            function bindInputEvents(level) {
                const input = document.getElementById('answerInput');
                const submitBtn = document.getElementById('submitBtn');
                const hintBtn = document.getElementById('hintBtn');

                if (input && submitBtn) {
                    const checkFn = () => {
                        const answer = input.value.trim();
                        checkAnswer(level, answer);
                    };
                    submitBtn.addEventListener('click', checkFn);
                    input.addEventListener('keydown', (e) => {
                        if (e.key === 'Enter') checkFn();
                    });
                    // 自动聚焦
                    setTimeout(() => input.focus(), 300);
                }
                if (hintBtn) {
                    hintBtn.addEventListener('click', () => useHint(level));
                }
            }

            // ============ 答案检查 ============
            function checkAnswer(level, answer) {
                const lowerAnswer = answer.toLowerCase().trim();
                let correct = false;
                let expected = '';

                switch (level) {
                    case 1:
                        expected = 'secret';
                        correct = lowerAnswer === 'secret';
                        break;
                    case 2:
                        expected = 'vault';
                        correct = lowerAnswer === 'vault';
                        break;
                    case 3:
                        // 西蒙游戏通过setSimonPass处理，这里不会调用
                        return;
                    case 4:
                        expected = 'free';
                        correct = lowerAnswer === 'free';
                        break;
                }

                if (correct) {
                    showFeedback('✓ 正确！访问授权通过。', 'success');
                    glowPuzzle();
                    totalFails = Math.max(0, totalFails); // keep
                    setTimeout(() => advanceLevel(), 1000);
                } else {
                    totalFails++;
                    showFeedback('✗ 密码错误。拒绝访问。（失败 ' + totalFails + ' 次）', 'error');
                    shakePuzzle();
                    const input = document.getElementById('answerInput');
                    if (input) {
                        input.value = '';
                        input.focus();
                    }
                }
            }

            function advanceLevel() {
                if (currentLevel >= TOTAL_LEVELS) {
                    // 全部通关
                    showVictory();
                } else {
                    currentLevel++;
                    updateLevelDots();
                    clearFeedback();
                    hideHint();
                    loadLevel(currentLevel);
                    // 过渡动画
                    mainContainer.style.animation = 'none';
                    mainContainer.offsetHeight;
                    mainContainer.style.animation = 'containerIn 0.5s ease-out';
                }
            }

            function showVictory() {
                document.getElementById('totalFails').textContent = totalFails;
                document.getElementById('totalHints').textContent = totalHintsUsed;
                victoryOverlay.classList.remove('hidden');
            }

            function restartGame() {
                currentLevel = 1;
                totalFails = 0;
                totalHintsUsed = 0;
                simonSequence = [];
                simonPlayerIndex = 0;
                simonRound = 0;
                simonIsDisplaying = false;
                simonPlayerCanClick = false;
                updateLevelDots();
                clearFeedback();
                hideHint();
                victoryOverlay.classList.add('hidden');
                loadLevel(1);
                mainContainer.style.animation = 'none';
                mainContainer.offsetHeight;
                mainContainer.style.animation = 'containerIn 0.5s ease-out';
            }

            // ============ 提示系统 ============
            function useHint(level) {
                totalHintsUsed++;
                hintLevel++;
                let hintMsg = '';
                switch (level) {
                    case 1:
                        if (hintLevel === 1) hintMsg = '仔细观察系统日志中每行英文的格式...';
                        else if (hintLevel === 2) hintMsg = '每行有效日志的首字母似乎拼成了一个词';
                        else hintMsg = '首字母组合：S-E-C-R-E-T → "secret"';
                        break;
                    case 2:
                        if (hintLevel === 1) hintMsg = '凯撒密码：每个字母在字母表中向后移动了固定步数';
                        else if (hintLevel === 2) hintMsg = '偏移量为3，即密文字母往前移3位得到原文';
                        else hintMsg = 'y→v, d→a, x→u, o→l, w→t → "vault"';
                        break;
                    case 3:
                        hintMsg = '专注观察每个按钮亮起的顺序，不要急于点击。序列长度会逐渐增加。';
                        break;
                    case 4:
                        if (hintLevel === 1) hintMsg = '每8位二进制对应一个ASCII字符';
                        else if (hintLevel === 2)
                        hintMsg = '01100110=102=f, 01110010=114=r, 01100101=101=e, 01100101=101=e';
                        else hintMsg = '四个字母连起来是 "free"';
                        break;
                }
                showHint(hintMsg);
            }

            // ============ 西蒙游戏逻辑(关卡3) ============
            const simonColors = ['red', 'blue', 'green', 'yellow'];
            const simonBtnMap = {};

            function getSimonButtons() {
                const btns = document.querySelectorAll('.simon-btn');
                btns.forEach(b => { simonBtnMap[b.dataset.color] = b; });
                return simonBtnMap;
            }

            function startSimonRound() {
                getSimonButtons();
                simonRound = 0;
                simonPlayerIndex = 0;
                simonSequence = [];
                simonIsDisplaying = false;
                simonPlayerCanClick = false;
                const startBtn = document.getElementById('simonStartBtn');
                if (startBtn) startBtn.style.display = 'none';
                disableSimonButtons();
                nextSimonSequence();
            }

            function nextSimonSequence() {
                simonRound++;
                const seqLength = simonRound + 2; // 第1轮3个, 第2轮4个, 第3轮5个
                simonSequence = [];
                for (let i = 0; i < seqLength; i++) {
                    simonSequence.push(simonColors[Math.floor(Math.random() * 4)]);
                }
                simonPlayerIndex = 0;
                simonIsDisplaying = true;
                simonPlayerCanClick = false;
                disableSimonButtons();
                updateSimonStatus('🔴 观察序列... (第' + simonRound + '轮)');
                displaySimonSequence(0);
            }

            function displaySimonSequence(index) {
                if (index >= simonSequence.length) {
                    // 展示完毕，玩家可以开始
                    simonIsDisplaying = false;
                    simonPlayerCanClick = true;
                    enableSimonButtons();
                    updateSimonStatus('🟢 请复现序列！(' + simonSequence.length + '步)');
                    return;
                }
                const color = simonSequence[index];
                const btn = simonBtnMap[color];
                if (btn) {
                    btn.classList.add('lit');
                    // 短暂蜂鸣模拟 - 通过视觉
                    setTimeout(() => {
                        btn.classList.remove('lit');
                        setTimeout(() => {
                            displaySimonSequence(index + 1);
                        }, 180);
                    }, 400);
                }
            }

            function handleSimonClick(btn) {
                if (!simonPlayerCanClick || simonIsDisplaying) return;
                const clickedColor = btn.dataset.color;
                const expectedColor = simonSequence[simonPlayerIndex];

                // 视觉反馈
                btn.classList.add('lit');
                setTimeout(() => btn.classList.remove('lit'), 200);

                if (clickedColor === expectedColor) {
                    simonPlayerIndex++;
                    if (simonPlayerIndex >= simonSequence.length) {
                        // 本轮完成
                        if (simonRound >= 3) {
                            // 通关！（完成3轮：长度3、4、5）
                            simonPlayerCanClick = false;
                            disableSimonButtons();
                            updateSimonStatus('✅ 序列复现成功！');
                            glowPuzzle();
                            showFeedback('✓ 记忆复现协议通过！', 'success');
                            setTimeout(() => advanceLevel(), 1200);
                        } else {
                            // 进入下一轮
                            simonPlayerCanClick = false;
                            disableSimonButtons();
                            updateSimonStatus('✅ 正确！准备下一轮...');
                            setTimeout(() => nextSimonSequence(), 1000);
                        }
                    } else {
                        updateSimonStatus('🟢 已输入 ' + simonPlayerIndex + '/' + simonSequence.length +
                            ' 步');
                    }
                } else {
                    // 错误
                    totalFails++;
                    simonPlayerCanClick = false;
                    disableSimonButtons();
                    updateSimonStatus('❌ 序列错误！重新展示...');
                    shakePuzzle();
                    showFeedback('✗ 序列不匹配。请重新观察。（失败 ' + totalFails + ' 次）', 'error');
                    // 重新展示当前序列
                    setTimeout(() => {
                        simonPlayerIndex = 0;
                        simonIsDisplaying = true;
                        simonPlayerCanClick = false;
                        disableSimonButtons();
                        updateSimonStatus('🔴 重新观察序列...');
                        displaySimonSequence(0);
                    }, 1200);
                }
            }

            function disableSimonButtons() {
                document.querySelectorAll('.simon-btn').forEach(b => b.classList.add('disabled'));
            }

            function enableSimonButtons() {
                document.querySelectorAll('.simon-btn').forEach(b => b.classList.remove('disabled'));
            }

            function updateSimonStatus(msg) {
                const statusEl = document.getElementById('simonStatus');
                if (statusEl) statusEl.textContent = msg;
            }

            // ============ 加载关卡 ============
            function loadLevel(level) {
                switch (level) {
                    case 1:
                        renderLevel1();
                        break;
                    case 2:
                        renderLevel2();
                        break;
                    case 3:
                        renderLevel3();
                        break;
                    case 4:
                        renderLevel4();
                        break;
                    default:
                        renderLevel1();
                }
                updateLevelDots();
            }

            // ============ 初始化 ============
            function initGame() {
                updateLevelDots();
                loadLevel(1);
            }

            // 暴露restartGame到全局
            window.restartGame = restartGame;

            initGame();
            console.log('%c🔐 终端解密游戏已就绪 %c| %c祝你好运，黑客。',
                'color:#00ff88;font-size:16px;',
                'color:#8899aa;',
                'color:#ffb347;');
            console.log('%c提示：答案都藏在细节里。仔细观察。', 'color:#4da6ff;font-style:italic;');
        })();
    </script>
</body>
</html>
