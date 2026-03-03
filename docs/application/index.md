<!DOCTYPE html>
<html lang="bn">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>ChitraJal | Smart TV</title>

    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css" rel="stylesheet">
    <link href="https://fonts.googleapis.com/css2?family=Hind+Siliguri:wght@400;700;800&family=Plus+Jakarta+Sans:wght@500;700&display=swap" rel="stylesheet">

    <script src="https://cdn.jsdelivr.net/npm/hls.js@latest"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/shaka-player/4.7.12/shaka-player.ui.js"></script>

    <style>
        :root { --primary: #E50914; --glass: rgba(15, 15, 20, 0.85); }
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body {
            background-color: #000; color: #fff;
            font-family: 'Hind Siliguri', 'Plus Jakarta Sans', sans-serif;
            height: 100vh; width: 100vw; overflow: hidden;
            -webkit-user-select: none; user-select: none;
        }

        /* Splash Screen */
        #splash-screen {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: linear-gradient(-45deg, #050507, #1a0202, #050507, #0d0d12);
            background-size: 400% 400%;
            animation: gradientBG 8s ease infinite;
            z-index: 10001; display: flex; align-items: center; justify-content: center;
            transition: opacity 1s ease, visibility 1s ease;
        }
        @keyframes gradientBG {
            0% { background-position: 0% 50%; }
            50% { background-position: 100% 50%; }
            100% { background-position: 0% 50%; }
        }
        .splash-content { text-align: center; }
        .snake-svg-logo text { font-family: 'Hind Siliguri', sans-serif; font-weight: 800; font-size: 55px; }
        .base-text { fill: rgba(255, 255, 255, 0.05); stroke: rgba(229, 9, 20, 0.1); stroke-width: 1px; }
        .snake-text {
            fill: transparent; stroke: #E50914; stroke-width: 2px;
            stroke-dasharray: 50 150; animation: slither 3s linear infinite;
            filter: drop-shadow(0 0 15px #E50914);
        }
        @keyframes slither { 100% { stroke-dashoffset: -200; } }

        .loader-bar { width: 320px; height: 5px; background: rgba(255, 255, 255, 0.08); margin: 35px auto; border-radius: 20px; overflow: hidden; position: relative; }
        .loader-progress { width: 0%; height: 100%; background: linear-gradient(90deg, #b80710, #E50914, #ff3d47, #E50914, #b80710); background-size: 200% 100%; animation: progress 4.2s cubic-bezier(0.65, 0, 0.35, 1) forwards, shimmer 1.8s infinite linear; }
        @keyframes progress { 0% { width: 0%; } 100% { width: 100%; } }
        @keyframes shimmer { 0% { background-position: 200% 0; } 100% { background-position: -200% 0; } }
        .tagline { color: #aaa; font-size: 1.2rem; opacity: 0; animation: fadeIn 2s ease forwards 1s; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }

        /* App UI */
        .main-container { width: 100vw; height: 100vh; position: relative; background: #000; overflow: hidden; }
        #liveVideo { width: 100%; height: 100%; object-fit: contain; background: #000; }

        /* 90s CRT TV Switch Effect */
        #tv-switch-overlay {
            position: fixed; inset: 0; z-index: 999;
            pointer-events: none; opacity: 0; visibility: hidden;
            background: #000;
        }
        #tv-switch-overlay.active {
            animation: tv90sSwitch 0.6s steps(10);
        }
        @keyframes tv90sSwitch {
            0% { opacity: 0; visibility: visible; }
            10% { opacity: 1; background: #000; }
            25% { opacity: 1; background: url('https://upload.wikimedia.org/wikipedia/commons/5/5a/Canal_Plus_Static.gif'); filter: contrast(150%) brightness(1.2); }
            45% { opacity: 1; background: #000; transform: scaleY(0.01); } /* Horizontal Collapse */
            65% { opacity: 1; background: url('https://upload.wikimedia.org/wikipedia/commons/5/5a/Canal_Plus_Static.gif'); transform: scaleY(1); }
            85% { opacity: 1; background: #000; }
            100% { opacity: 0; visibility: hidden; }
        }

        /* Professional New Gen Channel Info Overlay */
        #channelInfo {
            position: absolute; bottom: 50px; left: 50px;
            background: var(--glass);
            padding: 15px 30px; border-left: 6px solid var(--primary);
            border-radius: 12px;
            display: flex; align-items: center; gap: 20px;
            transition: all 0.7s cubic-bezier(0.23, 1, 0.32, 1);
            opacity: 0; transform: translateY(100px); z-index: 100;
            backdrop-filter: blur(20px);
            box-shadow: 0 15px 40px rgba(0,0,0,0.6);
            border: 1px solid rgba(255, 255, 255, 0.1);
        }
        #channelInfo.show { opacity: 1; transform: translateY(0); }

        .info-left { display: flex; align-items: center; gap: 20px; }
        #channelLogo { width: 70px; height: 70px; object-fit: contain; border-radius: 10px; background: rgba(255,255,255,0.05); padding: 5px; box-shadow: 0 4px 10px rgba(0,0,0,0.3); }
        #channelNum { font-size: 2.8rem; font-weight: 800; color: var(--primary); font-family: 'Plus Jakarta Sans', sans-serif; text-shadow: 0 0 15px rgba(229, 9, 20, 0.4); }
        #channelName { font-size: 2rem; font-weight: 700; color: #fff; border-left: 2px solid rgba(255,255,255,0.2); padding-left: 20px; line-height: 1.2; }

        /* No Signal Effect */
        #noSignal {
            position: absolute; inset: 0; display: none; flex-direction: column; align-items: center; justify-content: center;
            background: #000; z-index: 50;
        }
        .static-bg {
            position: absolute; inset: 0; background: url('https://upload.wikimedia.org/wikipedia/commons/5/5a/Canal_Plus_Static.gif');
            background-size: cover; opacity: 0.15; pointer-events: none;
        }
        .no-signal-text {
            font-size: 4rem; font-weight: 800; color: #fff; letter-spacing: 5px;
            text-shadow: 0 0 20px rgba(255,255,255,0.5); animation: glitch 1s infinite;
        }
        @keyframes glitch {
            0% { transform: translate(0); }
            20% { transform: translate(-2px, 2px); }
            40% { transform: translate(-2px, -2px); }
            60% { transform: translate(2px, 2px); }
            100% { transform: translate(0); }
        }

        #loading { position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); display: none; z-index: 60; }
        .spinner { width: 80px; height: 80px; border: 6px solid rgba(229, 9, 20, 0.2); border-top: 6px solid var(--primary); border-radius: 50%; animation: spin 1s linear infinite; }
        @keyframes spin { 100% { transform: rotate(360deg); } }

        /* Exit Dialog Styling */
        #exit-dialog-overlay {
            position: fixed; inset: 0; z-index: 20000;
            background: rgba(0, 0, 0, 0.85);
            backdrop-filter: blur(15px);
            display: none; align-items: center; justify-content: center;
            opacity: 0; transition: opacity 0.3s ease;
        }
        #exit-dialog-overlay.show { display: flex; opacity: 1; }
        .exit-dialog {
            background: linear-gradient(135deg, #1a1a24, #0d0d12);
            padding: 40px; border-radius: 30px; text-align: center;
            border: 1px solid rgba(255, 255, 255, 0.1);
            box-shadow: 0 25px 50px rgba(0,0,0,0.8);
            width: 450px; transform: scale(0.8); transition: transform 0.3s cubic-bezier(0.18, 0.89, 0.32, 1.28);
        }
        #exit-dialog-overlay.show .exit-dialog { transform: scale(1); }
        .exit-dialog h2 { font-size: 2.2rem; color: #fff; margin-bottom: 15px; }
        .exit-dialog p { color: #aaa; font-size: 1.1rem; margin-bottom: 30px; }
        .exit-buttons { display: flex; gap: 20px; justify-content: center; }
        .exit-btn {
            padding: 15px 40px; font-size: 1.2rem; font-weight: 700; border: none;
            border-radius: 15px; cursor: pointer; transition: all 0.3s ease;
            min-width: 140px; outline: none;
        }
        .btn-no { background: rgba(255, 255, 255, 0.05); color: #fff; border: 1px solid rgba(255, 255, 255, 0.1); }
        .btn-yes { background: var(--primary); color: #fff; box-shadow: 0 10px 20px rgba(229, 9, 20, 0.3); }
        .exit-btn:focus, .exit-btn:hover { transform: scale(1.1); }
        .btn-yes:focus { box-shadow: 0 0 25px var(--primary); border: 2px solid #fff; }
        .btn-no:focus { background: rgba(255, 255, 255, 0.15); border: 2px solid #fff; }
    </style>
</head>
<body>

    <div id="splash-screen">
        <div class="splash-content">
            <svg class="snake-svg-logo" style="width: 350px; height: 120px;" viewBox="0 0 220 80">
                <text class="base-text" x="50%" y="60%" text-anchor="middle">চিত্রজাল</text>
                <text class="snake-text" x="50%" y="60%" text-anchor="middle">চিত্রজাল</text>
            </svg>
            <div class="loader-bar"><div class="loader-progress"></div></div>
            <div class="tagline">সেরা বিনোদনের ঝকঝকে দুনিয়া, এখন আপনার ঘরের আঙিনায়...</div>
        </div>
    </div>

    <div id="tv-switch-overlay"></div>

    <div class="main-container">
        <video id="liveVideo" playsinline autoplay></video>
        <div id="noSignal"><div class="static-bg"></div><div class="no-signal-text">NO SIGNAL</div></div>
        <div id="loading"><div class="spinner"></div></div>

        <div id="channelInfo">
            <div class="info-left">
                <img id="channelLogo" src="" alt="" onerror="this.style.display='none';">
                <div id="channelNum">01</div>
            </div>
            <div id="channelName">চ্যানেল নাম</div>
        </div>
    </div>

    <!-- Professional Exit Dialog -->
    <div id="exit-dialog-overlay">
        <div class="exit-dialog">
            <div style="font-size: 4rem; color: var(--primary); margin-bottom: 20px;"><i class="fas fa-power-off"></i></div>
            <h2>বিদায় নিচ্ছেন?</h2>
            <p>আপনি কি নিশ্চিত যে আপনি চিত্রজাল অ্যাপটি বন্ধ করতে চান?</p>
            <div class="exit-buttons">
                <button class="exit-btn btn-no" id="exit-no">না</button>
                <button class="exit-btn btn-yes" id="exit-yes">হ্যাঁ</button>
            </div>
        </div>
    </div>

    <script>
        const video = document.getElementById('liveVideo');
        const info = document.getElementById('channelInfo');
        const infoNum = document.getElementById('channelNum');
        const infoName = document.getElementById('channelName');
        const infoLogo = document.getElementById('channelLogo');
        const noSignal = document.getElementById('noSignal');
        const loading = document.getElementById('loading');
        const tvSwitch = document.getElementById('tv-switch-overlay');
        const exitOverlay = document.getElementById('exit-dialog-overlay');
        const btnYes = document.getElementById('exit-yes');
        const btnNo = document.getElementById('exit-no');

        let allChannels = [];
        let currentIndex = 0;
        let hlsPlayer = null;
        let infoTimeout = null;
        let isExitShowing = false;

        window.addEventListener('load', () => {
            setTimeout(() => {
                const splash = document.getElementById('splash-screen');
                splash.style.opacity = '0';
                splash.style.visibility = 'hidden';
                setTimeout(() => splash.remove(), 1000);
            }, 4500);
        });

        async function fetchPlaylists() {
            const playlists = [
                { url: 'https://raw.githubusercontent.com/sm-monirulislam/SM-Live-TV/c39313219bb273dae55f6d200d5d03b7d6b7c9bb/new_sm.m3u', source: 'sm_live' },
                { url: 'https://raw.githubusercontent.com/sm-monirulislam/AynaOTT-auto-update-playlist/refs/heads/main/AynaOTT.m3u', source: 'ayna' },
                { url: 'https://raw.githubusercontent.com/sydul104/main04/refs/heads/main/my', source: 'sydul' },
                { url: 'https://raw.githubusercontent.com/BINOD-XD/Toffee-Auto-Update-Playlist/refs/heads/main/toffee_NS_Player.m3u', source: 'toffee' }
            ];

            for (const item of playlists) {
                try {
                    const res = await fetch(item.url);
                    if (res.ok) parseM3U(await res.text(), item.source);
                } catch (e) {}
            }

            allChannels.sort((a, b) => {
                const getPriority = (ch) => {
                    const n = ch.name.toLowerCase();
                    const isSM = ch.source === 'sm_live';
                    if (isSM && n.includes('fhd')) return 0;
                    if (n.includes('fhd')) return 1;
                    if (isSM && n.includes('hd')) return 2;
                    if (n.includes('hd')) return 3;
                    return 5;
                };
                return getPriority(a) - getPriority(b);
            });

            if (allChannels.length > 0) playChannel(0);
        }

        function parseM3U(data, source) {
            const lines = data.split('\n');
            let temp = {};
            lines.forEach(line => {
                if (line.startsWith('#EXTINF:')) {
                    temp.logo = line.match(/tvg-logo="([^"]+)"/)?.[1] || '';
                    temp.name = line.split(',')[1]?.trim() || 'Unknown';
                    temp.source = source;
                } else if (line.startsWith('http')) {
                    temp.url = line.trim();
                    allChannels.push({...temp});
                    temp = {};
                }
            });
        }

        function playChannel(index) {
            if (index < 0) index = allChannels.length - 1;
            if (index >= allChannels.length) index = 0;
            currentIndex = index;

            tvSwitch.classList.remove('active');
            void tvSwitch.offsetWidth;
            tvSwitch.classList.add('active');

            const ch = allChannels[currentIndex];
            showInfo(currentIndex + 1, ch.name, ch.logo);

            if (hlsPlayer) { hlsPlayer.destroy(); hlsPlayer = null; }
            video.src = "";
            noSignal.style.display = 'none';
            loading.style.display = 'block';

            if (Hls.isSupported() && ch.url.includes('.m3u8')) {
                hlsPlayer = new Hls({ maxBufferLength: 10 });
                hlsPlayer.loadSource(ch.url);
                hlsPlayer.attachMedia(video);
                hlsPlayer.on(Hls.Events.MANIFEST_PARSED, () => { video.play().catch(() => {}); });
                hlsPlayer.on(Hls.Events.ERROR, (e, data) => { if(data.fatal) handleError(); });
            } else {
                video.src = ch.url;
                video.play().catch(handleError);
            }
        }

        function handleError() {
            loading.style.display = 'none';
            noSignal.style.display = 'flex';
        }

        function showInfo(num, name, logo) {
            infoNum.innerText = num.toString().padStart(2, '0');
            infoName.innerText = name;
            if (logo) {
                infoLogo.src = logo;
                infoLogo.style.display = 'block';
            } else {
                infoLogo.style.display = 'none';
            }

            info.classList.add('show');
            clearTimeout(infoTimeout);
            infoTimeout = setTimeout(() => info.classList.remove('show'), 6000);
        }

        // Exit Dialog Handling
        window.showExitDialog = function() {
            exitOverlay.classList.add('show');
            btnNo.focus();
            isExitShowing = true;
        }

        btnNo.onclick = () => {
            exitOverlay.classList.remove('show');
            isExitShowing = false;
        }

        btnYes.onclick = () => {
            if (window.Android) window.Android.exitApp();
        }

        document.addEventListener('keydown', (e) => {
            if (isExitShowing) {
                if (e.key === "ArrowLeft" || e.key === "ArrowRight") {
                    if (document.activeElement === btnNo) btnYes.focus();
                    else btnNo.focus();
                }
                return;
            }

            switch(e.key) {
                case "ArrowUp": case "ChannelUp": playChannel(currentIndex - 1); break;
                case "ArrowDown": case "ChannelDown": playChannel(currentIndex + 1); break;
                case "Enter": case "OK": showInfo(currentIndex + 1, allChannels[currentIndex].name, allChannels[currentIndex].logo); break;
            }
            if (e.key >= '0' && e.key <= '9') handleNumberInput(e.key);
        });

        let numInput = "";
        let numTimeout = null;
        function handleNumberInput(key) {
            numInput += key;
            showInfo(numInput, "...", null);
            clearTimeout(numTimeout);
            numTimeout = setTimeout(() => {
                const target = parseInt(numInput) - 1;
                if (target >= 0 && target < allChannels.length) playChannel(target);
                numInput = "";
            }, 2000);
        }

        video.onplaying = () => { loading.style.display = 'none'; noSignal.style.display = 'none'; };
        video.onerror = handleError;
        video.onwaiting = () => { loading.style.display = 'block'; };

        window.onload = fetchPlaylists;
    </script>
</body>
</html>
