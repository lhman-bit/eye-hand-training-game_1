# eye-hand-training-game_1
<!DOCTYPE html>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>手眼協調訓練活動</title>
    <style>
        body, html {
            margin: 0;
            padding: 0;
            width: 100%;
            height: 100%;
            background-color: black;
            overflow: hidden;
            display: flex;
            justify-content: center;
            align-items: center;
        }

        #game-target {
            position: absolute;
            width: 132px; /* 初始約 3.5cm (基於常見 96DPI 螢幕) */
            height: 132px;
            cursor: pointer;
            transition: all 0.5s ease-in-out; /* 平滑滑行效果 */
            display: flex;
            justify-content: center;
            align-items: center;
        }

        #game-target img {
            width: 100%;
            height: 100%;
            object-fit: contain;
        }

        /* 上下震動動畫 */
        @keyframes shake {
            0% { transform: translateY(0); }
            25% { transform: translateY(-10px); }
            75% { transform: translateY(10px); }
            100% { transform: translateY(0); }
        }

        .shaking {
            animation: shake 0.1s ease-in-out 2;
        }

        /* 消失動畫 */
        .fade-out {
            opacity: 0;
            transition: opacity 3s linear !important;
        }
    </style>
</head>
<body>

    <div id="game-target">
        <img id="target-img" src="https://via.placeholder.com/132?text=Breadman" alt="目標圖案">
    </div>

    <script>
        const target = document.getElementById('game-target');
        let clickCount = 0;
        let isMoving = false;
        
        // 設定尺寸：1cm 約等於 37.8px
        const baseSize = 132; // 3.5cm
        const stepSize = 19;  // 0.5cm

        // Web Audio API 產生「叮」一聲
        const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
        function playDingSound() {
            const oscillator = audioCtx.createOscillator();
            const gainNode = audioCtx.createGain();
            oscillator.connect(gainNode);
            gainNode.connect(audioCtx.destination);
            oscillator.type = 'sine';
            oscillator.frequency.setValueAtTime(1000, audioCtx.currentTime); // 高音
            gainNode.gain.setValueAtTime(0.5, audioCtx.currentTime);
            gainNode.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + 0.5);
            oscillator.start();
            oscillator.stop(audioCtx.currentTime + 0.5);
        }

        target.addEventListener('click', () => {
            if (isMoving) return; // 動畫期間禁止點擊

            clickCount++;
            playDingSound();

            // 觸發震動效果
            target.classList.remove('shaking');
            void target.offsetWidth; // 強制重繪
            target.classList.add('shaking');

            if (clickCount < 3) {
                // 放大尺寸
                let newSize = baseSize + (clickCount * stepSize);
                target.style.width = newSize + 'px';
                target.style.height = newSize + 'px';
            } else {
                // 第三次點擊：縮回原大小並開始路徑
                target.style.width = baseSize + 'px';
                target.style.height = baseSize + 'px';
                startPathSequence();
            }
        });

        async function startPathSequence() {
            isMoving = true;
            
            const padding = 20; // 距離邊緣的距離
            const positions = [
                { left: padding + 'px', top: padding + 'px' }, // 左上
                { left: (window.innerWidth - baseSize - padding) + 'px', top: padding + 'px' }, // 右上
                { left: padding + 'px', top: (window.innerHeight - baseSize - padding) + 'px' }, // 左下
                { left: (window.innerWidth - baseSize - padding) + 'px', top: (window.innerHeight - baseSize - padding) + 'px' } // 右下
            ];

            // 依序跳動
            for (let pos of positions) {
                await sleep(500); // 準備滑動
                target.style.left = pos.left;
                target.style.top = pos.top;
                await sleep(3000); // 到達後停頓3秒
            }

            // 回到中央
            target.style.left = '50%';
            target.style.top = '50%';
            target.style.transform = 'translate(-50%, -50%)';
            
            await sleep(3000);
            
            // 慢慢消失
            target.classList.add('fade-out');
        }

        function sleep(ms) {
            return new Promise(resolve => setTimeout(resolve, ms));
        }

        // 初始化位置在中央
        target.style.left = '50%';
        target.style.top = '50%';
        target.style.transform = 'translate(-50%, -50%)';
    </script>
</body>
</html>
