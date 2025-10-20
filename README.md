<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Thawing Reminder - Alarm Agresif & Persistent</title>
    
    <style>
        /* ==================== 
           CSS: STYLING & RESPONSIVITAS 
           ==================== */
        body {
            font-family: Arial, sans-serif;
            background-color: #f4f4ff;
            min-height: 100vh;
            margin: 0; 
            padding: 20px 0;
            display: flex;
            justify-content: center;
            align-items: flex-start;
            transition: background-color 0.2s; /* Untuk Flash Alarm */
        }

        .main-container {
            background-color: white;
            padding: 30px;
            border-radius: 12px;
            box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
            text-align: center;
            width: 100%;
            max-width: 800px;
            box-sizing: border-box; 
        }

        /* Grid default untuk Desktop/Tablet */
        .timer-list {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
            gap: 20px;
            margin-top: 20px;
        }

        .timer-card {
            border: 2px solid #ddd;
            padding: 15px;
            border-radius: 8px;
            background-color: #f9f9f9;
            transition: all 0.3s;
            text-align: left;
        }

        .timer-card h2 {
            color: #007bff;
            padding-bottom: 10px;
            font-size: 1.2em;
        }

        .timer-controls {
            display: flex;
            gap: 10px;
            margin-top: 10px;
            align-items: center;
        }

        .timer-controls input {
            padding: 8px;
            flex-grow: 1; 
            max-width: 80px; 
            text-align: center; 
        }

        .timer-controls button {
            padding: 8px 15px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-weight: bold;
            flex-grow: 1; 
            white-space: nowrap;
        }

        .start-btn { background-color: #28a745; color: white; }
        .reset-btn { background-color: #dc3545; color: white; }

        .countdown-display {
            font-size: 2em;
            margin: 15px 0;
            padding: 10px;
            font-weight: bold;
            text-align: center;
            background-color: #e9ecef;
            color: #333;
        }

        /* --- STATE ALERT CSS --- */

        /* WARNING (15 Menit) */
        .timer-card.warning {
            border-color: orange;
            box-shadow: 0 0 10px rgba(255, 165, 0, 0.5);
        }
        .timer-card.warning .countdown-display {
            background-color: #fff3cd; 
            color: #ff8c00; 
            animation: pulse-warn 1s infinite alternate; 
        }

        /* ALERT (Waktu Habis) */
        .timer-card.alert {
            border-color: red;
            background-color: #ffeaea;
            box-shadow: 0 0 10px rgba(220, 53, 69, 0.7);
        }
        .timer-card.alert .countdown-display {
            background-color: #f8d7da; 
            color: #dc3545; 
            font-size: 1.8em;
            animation: none;
        }
        
        /* ALARM GLOBAL AGRESIF (Flash Body) */
        .flash-alarm-red {
            background-color: #ff0000 !important; /* Merah menyala */
            transition: background-color 0.2s; 
        }

        /* ANIMASI */
        @keyframes pulse-warn {
            from { transform: scale(1); }
            to { transform: scale(1.02); }
        }

        /* MEDIA QUERY - OPTIMALISASI MOBILE */
        @media (max-width: 650px) {
            body { padding: 10px 0; }
            .main-container { padding: 15px; box-shadow: none; }
            h1 { font-size: 1.5em; }
            .timer-list { grid-template-columns: 1fr; gap: 10px; } /* Single column di mobile */
            .timer-card { padding: 10px; }
            .timer-card h2 { font-size: 1.1em; }
            .countdown-display { font-size: 1.8em; margin: 10px 0; padding: 8px; }
            
            /* Kontrol Ramping untuk Mobile */
            .timer-controls { flex-wrap: wrap; justify-content: space-between; gap: 5px; }
            .timer-controls label { font-size: 0.9em; flex-basis: 100%; text-align: left;}
            .timer-controls input { padding: 6px; max-width: 60px; }
            .timer-controls button { padding: 8px 10px; font-size: 0.85em; flex-basis: calc(50% - 5px); }
        }
    </style>
</head>
<body>
    <div class="main-container">
        <h1>Timer Thawing Reminder 🧊</h1>
        <p>A.</p>
        
        <div class="timer-list" id="timer-list">
        </div>
    </div>

    <script>
        /* ==================== 
           JAVASCRIPT LOGIC (Gabungan & Optimalisasi)
           ==================== */
        
        // --- KONFIGURASI APLIKASI ---
        const THAWING_ITEMS = [
            { id: 1, name: "ADONAN", defaultTimeMinutes: 40 },
            { id: 2, name: "ACIN", defaultTimeMinutes: 60 },
            { id: 3, name: "MIE", defaultTimeMinutes: 120 },
            { id: 4, name: "PENTOL", defaultTimeMinutes: 120 },
            { id: 5, name: "SURAI NAGA", defaultTimeMinutes: 120 },
            { id: 6, name: "KRUPUK MIE", defaultTimeMinutes: 120 },
        ];
        
        const WARNING_TIME_SECONDS = 15 * 60; // 15 menit
        const STORAGE_KEY_PREFIX = 'thawingTimerStatus_'; 
        const STORAGE_INPUT_PREFIX = 'thawingTimerInput_';

        // --- VARIABEL GLOBAL ALARM AGRESIF ---
        let activeIntervals = {}; 
        let audioCtx;
        let notificationPermission = Notification.permission;
        
        let titleInterval = null;
        let flashInterval = null;
        const originalTitle = document.title;
        const WARNING_TITLE_PREFIX = '🚨 HABIS! - ';
        const FLASH_COLOR_CLASS = 'flash-alarm-red'; 

        // --- UTILITY: FORMAT WAKTU ---
        function formatTime(totalSeconds) {
            const hours = Math.floor(totalSeconds / 3600);
            const minutes = Math.floor((totalSeconds % 3600) / 60);
            const seconds = totalSeconds % 60;

            const h = String(hours).padStart(2, '0');
            const m = String(minutes).padStart(2, '0');
            const s = String(seconds).padStart(2, '0');

            return `${h}:${m}:${s}`;
        }
        
        // --- FUNGSI AUDIO & ALARM AGRESIF ---
        function resumeAudioContext() {
             if (audioCtx && audioCtx.state === 'suspended') {
                 audioCtx.resume().catch(e => console.error("Failed to resume AudioContext:", e));
             }
        }
        
        function playBeep(frequency, durationMs) {
            try {
                if (!audioCtx) {
                    audioCtx = new (window.AudioContext || window.webkitAudioContext)();
                }
                resumeAudioContext(); 
                
                const oscillator = audioCtx.createOscillator();
                const gainNode = audioCtx.createGain();

                gainNode.gain.value = 2.0; 

                oscillator.connect(gainNode);
                gainNode.connect(audioCtx.destination);

                oscillator.type = 'square';
                oscillator.frequency.setValueAtTime(frequency, audioCtx.currentTime);

                oscillator.start();
                
                gainNode.gain.exponentialRampToValueAtTime(
                    0.00001, audioCtx.currentTime + durationMs / 1000
                );
                oscillator.stop(audioCtx.currentTime + durationMs / 1000);
            } catch (e) {
                console.error("Error playing beep:", e);
            }
        }

        // Alarm berulang untuk Time-up dan Peringatan
        function playAlarm(times = 8) { 
            let count = 0;
            const interval = setInterval(() => {
                playBeep(440, 400); 
                count++;
                if (count >= times) {
                    clearInterval(interval);
                }
            }, 500); 
        }

        // Notifikasi Desktop (Popup & Suara Sistem)
        function sendNotification(itemName) {
            if (notificationPermission === 'granted') {
                new Notification("⏰ WAKTU THAWING HABIS!", {
                    body: `🚨 Segera ambil bahan: ${itemName}. Alarm ini berbunyi di background Android.`,
                    tag: 'thawing-alarm',
                    renotify: true, 
                    requireInteraction: true // Notifikasi menetap di layar (sulit diabaikan)
                }).onclick = function() {
                    window.focus(); 
                    this.close();
                    stopAggressiveAlarm(); // Hentikan alarm saat user klik notifikasi
                };
            }
        }

        // Vibration API (Hanya berfungsi jika dipicu oleh user interaction/klik notifikasi)
        function startVibrationAlert() {
            if ('vibrate' in navigator) {
                const pattern = [1000, 500, 1000]; // Getar 1s, jeda 0.5s, Getar 1s
                navigator.vibrate(pattern);
            }
        }

        // Flash Background
        function startFlashAlarm() {
            if (flashInterval) return;
            let isFlashing = false;
            flashInterval = setInterval(() => {
                isFlashing = !isFlashing;
                document.body.classList.toggle(FLASH_COLOR_CLASS, isFlashing);
            }, 200);
        }

        // Kedipan Judul Halaman
        function startTitleAlert(itemName) {
            if (titleInterval) return;
            let isAlertState = false;
            titleInterval = setInterval(() => {
                isAlertState = !isAlertState;
                document.title = isAlertState 
                    ? WARNING_TITLE_PREFIX + itemName 
                    : originalTitle;
            }, 800);
        }

        // Hentikan Semua Alarm Agresif Global
        function stopAggressiveAlarm() {
            if (titleInterval) {
                clearInterval(titleInterval);
                titleInterval = null;
            }
            if (flashInterval) {
                clearInterval(flashInterval);
                flashInterval = null;
            }
            document.title = originalTitle;
            document.body.classList.remove(FLASH_COLOR_CLASS);
            // Catatan: Tidak ada clearInterval untuk playAlarm karena sifatnya non-looping.
        }
        
        // --- FUNGSI UTAMA LOGIKA ---
        
        function tick(itemId, endTimeMs) {
            const timerCard = document.getElementById(`card-${itemId}`);
            const display = document.getElementById(`display-${itemId}`);
            const alarmMessage = document.getElementById(`msg-${itemId}`);
            const itemName = timerCard.querySelector('h2').textContent;
            
            const now = Date.now();
            let duration = Math.floor((endTimeMs - now) / 1000); 
            
            clearTimeout(activeIntervals[itemId]);

            // 1. Update tampilan
            if (duration >= 0) {
                 display.textContent = formatTime(duration);
            }
           
            // 2. Transisi ke mode WARNING (15 Menit)
            if (duration <= WARNING_TIME_SECONDS && duration > 0) {
                if (!timerCard.classList.contains('warning')) {
                    timerCard.classList.add('warning');
                    timerCard.classList.remove('alert'); 
                    const remainingMinutes = Math.ceil(duration / 60);
                    alarmMessage.textContent = `🔔 PERINGATAN! ${itemName} tersisa ${remainingMinutes} menit.`;
                    alarmMessage.style.display = 'block';
                }
            } else if (duration > WARNING_TIME_SECONDS) {
                 timerCard.classList.remove('warning');
                 alarmMessage.style.display = 'none';
            }

            // 3. Bunyikan alarm di mode WARNING (setiap 30 detik)
            if (duration <= WARNING_TIME_SECONDS && duration > 0 && duration % 30 === 0) {
                playAlarm(1); 
            }
            
            // 4. Kondisi Waktu Habis: (Logic utama notifikasi dan alarm terkuat)
            if (duration <= 0) {
                delete activeIntervals[itemId];
                localStorage.removeItem(STORAGE_KEY_PREFIX + itemId); 
                
                display.textContent = "WAKTU HABIS!";
                timerCard.classList.remove('warning');
                timerCard.classList.add('alert'); 
                alarmMessage.textContent = `✅ SELESAI! Bahan ${itemName} butuh penanganan.`;
                alarmMessage.style.display = 'block';
                
                // --- AKTIVASI ALARM AGRESIF DAN PERSISTENT ---
                sendNotification(itemName);
                startVibrationAlert(); 
                startTitleAlert(itemName);
                startFlashAlarm();
                playAlarm(15); // Suara alarm lebih panjang
                
                // Reset UI kontrol
                const inputTime = document.getElementById(`time-input-${itemId}`);
                document.getElementById(`start-btn-${itemId}`).style.display = 'block';
                document.getElementById(`reset-btn-${itemId}`).style.display = 'none';
                inputTime.readOnly = false;
                return; 
            }
            
            // 5. Jadwalkan tick berikutnya
            activeIntervals[itemId] = setTimeout(() => tick(itemId, endTimeMs), 1000);
        }

        // Fungsi utama untuk menjalankan countdown
        function startCountdown(itemId) {
            resumeAudioContext(); 
            
            // Minta Izin Notifikasi Pop-up 
            Notification.requestPermission().then(permission => {
                notificationPermission = permission; 
            });
            
            const timerCard = document.getElementById(`card-${itemId}`);
            const inputTime = document.getElementById(`time-input-${itemId}`);
            
            if (activeIntervals[itemId]) {
                const confirmRestart = confirm(`Timer untuk ${timerCard.querySelector('h2').textContent} sedang berjalan. Apakah Anda ingin memulai ulang?`);
                if (!confirmRestart) return; 
                resetTimer(itemId); // Reset bersih sebelum memulai
            }

            const durationMinutes = parseInt(inputTime.value);
            if (isNaN(durationMinutes) || durationMinutes <= 0) {
                alert(`Mohon masukkan waktu thawing yang valid.`);
                return;
            }
            
            // LOGIKA PENYIMPANAN STATE
            const durationMs = durationMinutes * 60 * 1000;
            const endTimeMs = Date.now() + durationMs; 
            localStorage.setItem(STORAGE_KEY_PREFIX + itemId, endTimeMs);
            localStorage.setItem(STORAGE_INPUT_PREFIX + itemId, durationMinutes); // Simpan input baru
            
            // Perubahan UI
            document.getElementById(`start-btn-${itemId}`).style.display = 'none';
            document.getElementById(`reset-btn-${itemId}`).style.display = 'block';
            inputTime.readOnly = true;
            timerCard.classList.remove('alert', 'warning');
            document.getElementById(`msg-${itemId}`).style.display = 'none';
            document.getElementById(`display-${itemId}`).textContent = formatTime(durationMinutes * 60);

            // Jalankan tick
            tick(itemId, endTimeMs);
        }

        // Fungsi reset
        function resetTimer(itemId) {
            clearTimeout(activeIntervals[itemId]);
            delete activeIntervals[itemId];
            
            localStorage.removeItem(STORAGE_KEY_PREFIX + itemId); 

            const item = THAWING_ITEMS.find(i => i.id === itemId);

            const timerCard = document.getElementById(`card-${itemId}`);
            const inputTime = document.getElementById(`time-input-${itemId}`);
            const display = document.getElementById(`display-${itemId}`);
            const alarmMessage = document.getElementById(`msg-${itemId}`);
            const startButton = document.getElementById(`start-btn-${itemId}`);
            const resetButton = document.getElementById(`reset-btn-${itemId}`);
            
            // Hentikan alarm global saat ada timer yang direset
            stopAggressiveAlarm(); 

            if (!timerCard || !inputTime || !display) return; 

            // Hapus kelas state & Tampilan
            timerCard.classList.remove('alert', 'warning');
            inputTime.readOnly = false;
            startButton.style.display = 'block';
            resetButton.style.display = 'none';
            alarmMessage.style.display = 'none';

            // Ambil durasi input yang tersimpan atau default
            const savedInput = localStorage.getItem(STORAGE_INPUT_PREFIX + itemId) || item.defaultTimeMinutes;
            
            inputTime.value = savedInput;
            display.textContent = formatTime(savedInput * 60);

            inputTime.focus();
        }


        // Fungsi untuk membuat elemen HTML timer
        function createTimerCard(item) {
            const timerListContainer = document.getElementById('timer-list');
            
            const card = document.createElement('div');
            card.className = 'timer-card';
            card.id = `card-${item.id}`;
            
            const savedEndTime = localStorage.getItem(STORAGE_KEY_PREFIX + item.id);
            const savedInput = localStorage.getItem(STORAGE_INPUT_PREFIX + item.id) || item.defaultTimeMinutes;
            
            let initialDisplayTime = formatTime(savedInput * 60);
            let isRunning = false;

            if (savedEndTime) {
                const remainingMs = parseInt(savedEndTime) - Date.now();
                if (remainingMs > 1000) { 
                    initialDisplayTime = formatTime(Math.ceil(remainingMs / 1000));
                    isRunning = true;
                } else {
                    localStorage.removeItem(STORAGE_KEY_PREFIX + item.id);
                }
            }
            
            card.innerHTML = `
                <h2>${item.name}</h2>
                <div id="display-${item.id}" class="countdown-display">
                    ${initialDisplayTime}
                </div>
                
                <div id="msg-${item.id}" class="alarm-message"></div>

                <div class="timer-controls">
                    <label for="time-input-${item.id}">Durasi (mnt):</label>
                    <input type="number" id="time-input-${item.id}" value="${savedInput}" min="1" max="180" ${isRunning ? 'readonly' : ''}>
                    <button id="start-btn-${item.id}" class="start-btn" style="display: ${isRunning ? 'none' : 'block'};">START</button>
                    <button id="reset-btn-${item.id}" class="reset-btn" style="display: ${isRunning ? 'block' : 'none'};">RESET</button>
                </div>
            `;
            
            timerListContainer.appendChild(card);
            
            // Tambahkan Event Listener
            document.getElementById(`start-btn-${item.id}`).addEventListener('click', () => startCountdown(item.id));
            document.getElementById(`reset-btn-${item.id}`).addEventListener('click', () => resetTimer(item.id));

            document.getElementById(`time-input-${item.id}`).addEventListener('input', (event) => {
                if (!document.getElementById(`time-input-${item.id}`).readOnly) {
                    const minutes = parseInt(event.target.value) || 0;
                    const display = document.getElementById(`display-${item.id}`);
                    display.textContent = formatTime(minutes * 60);
                }
            });

            // Jalankan kembali tick jika ada status yang tersimpan
            if (isRunning) {
                // Gunakan setTimeout kecil untuk memastikan DOM dirender sempurna sebelum tick
                setTimeout(() => {
                    tick(item.id, parseInt(savedEndTime));
                }, 50); 
            }
        }


        // Inisialisasi Aplikasi
        document.addEventListener('DOMContentLoaded', () => {
            THAWING_ITEMS.forEach(item => {
                createTimerCard(item);
            });
            notificationPermission = Notification.permission;
        });
    </script>
</body>
</html>
