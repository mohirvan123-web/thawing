<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Thawing Reminder - Alarm Agresif & Persistent (MLGJAK)</title>
    
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
            transition: background-color 0.2s; 
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

        h1 {
            color: #333;
        }

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
        
        .alarm-message {
            margin-bottom: 10px;
            padding: 5px;
            font-weight: bold;
            color: #dc3545;
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
            background-color: #ff0000 !important; 
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
            .timer-list { grid-template-columns: 1fr; gap: 10px; } 
            .timer-card { padding: 10px; }
            .timer-card h2 { font-size: 1.1em; }
            .countdown-display { font-size: 1.8em; margin: 10px 0; padding: 8px; }
            
            .timer-controls { flex-wrap: wrap; justify-content: space-between; gap: 5px; }
            .timer-controls label { font-size: 0.9em; flex-basis: 100%; text-align: left;}
            .timer-controls input { padding: 6px; max-width: 60px; }
            .timer-controls button { padding: 8px 10px; font-size: 0.85em; flex-basis: calc(50% - 5px); }
        }
    </style>
    
    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-app.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-database.js"></script>
</head>
<body>
    <div class="main-container">
        <h1>Timer Thawing Reminder 🧊 (SHARED)</h1>
        <p>Status timer dibagikan dan disinkronkan secara real-time ke semua perangkat.</p>
        
        <div class="timer-list" id="timer-list">
            </div>
    </div>

    <script>
       // ===================================
        // FIREBASE CONFIGURATION 
        // ===================================
        const firebaseConfig = {
          apiKey: "AIzaSyBtUlghTw806GuGuwOXGNgoqN6Rkcg0IMM", // Ganti dengan kunci API Anda yang sebenarnya
          authDomain: "thawing-ec583.firebaseapp.com",
          databaseURL: "https://thawing-ec583-default-rtdb.asia-southeast1.firebasedatabase.app",
          projectId: "thawing-ec583",
          storageBucket: "thawing-ec583.firebasestorage.app", 
          messagingSenderId: "1043079332713",
          appId: "1:1043079332713:web:6d289ad2b7c13a222bb3f8",
          measurementId: "G-WLBFJ6V7QT"            
        };


        // Initialize Firebase (PENTING: Hanya panggil sekali)
        try {
            firebase.initializeApp(firebaseConfig);
        } catch (e) {
            console.error("Firebase Initialization Failed. Check your config.", e);
        }
        
        // Dapatkan referensi database
        const dbRef = firebase.database().ref('thawingTimers');
        
        /* ==================== 
           JAVASCRIPT LOGIC 
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

        // --- VARIABEL GLOBAL ALARM AGRESIF ---
        let activeIntervals = {}; // Menyimpan interval tick LOKAL
        let audioCtx;
        let notificationPermission = Notification.permission;
        
        let titleInterval = null;
        let flashInterval = null;
        const originalTitle = document.title;
        const WARNING_TITLE_PREFIX = '🚨 HABIS! - ';
        const FLASH_COLOR_CLASS = 'flash-alarm-red'; 

        // Fungsi Audio & Alarm
        function formatTime(totalSeconds) {
            const hours = Math.floor(totalSeconds / 3600);
            const minutes = Math.floor((totalSeconds % 3600) / 60);
            const seconds = totalSeconds % 60;

            const h = String(hours).padStart(2, '0');
            const m = String(minutes).padStart(2, '0');
            const s = String(seconds).padStart(2, '0');

            return `${h}:${m}:${s}`;
        }
        
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

        function playAlarm(times = 8) { 
            let count = 0;
            // Interval 500ms adalah jeda antara beep
            const interval = setInterval(() => { 
                playBeep(440, 400); // Beep 400ms
                count++;
                if (count >= times) {
                    clearInterval(interval);
                }
            }, 500); 
        }

        // FUNGSI NOTIFIKASI WAKTU HABIS (Aggressive)
        function sendNotification(itemName) {
            if (notificationPermission === 'granted') {
                new Notification("⏰ WAKTU THAWING HABIS!", {
                    body: `🚨 Segera ambil bahan: ${itemName}.`,
                    tag: 'thawing-alarm',
                    renotify: true, 
                    requireInteraction: true 
                }).onclick = function() {
                    window.focus(); 
                    this.close();
                    stopAggressiveAlarm(); 
                };
            }
        }
        
        // FUNGSI NOTIFIKASI 15 MENIT (Peringatan)
        function sendWarningNotification(itemName) {
            if (notificationPermission === 'granted') {
                new Notification("⏳ WAKTU THAWING MENDEKAT!", {
                    body: `Peringatan: ${itemName} tersisa kurang dari 15 menit.`,
                    tag: 'thawing-warning',
                    renotify: false, // Notifikasi sekali saja
                    requireInteraction: false // Boleh ditutup oleh sistem
                }).onclick = function() {
                    window.focus(); 
                    this.close();
                };
            }
        }

        function startVibrationAlert() {
            if ('vibrate' in navigator) {
                const pattern = [1000, 500, 1000]; 
                navigator.vibrate(pattern);
            }
        }

        function startFlashAlarm() {
            if (flashInterval) return;
            let isFlashing = false;
            flashInterval = setInterval(() => {
                isFlashing = !isFlashing;
                document.body.classList.toggle(FLASH_COLOR_CLASS, isFlashing);
            }, 200);
        }

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
            
            // Hentikan getaran
            if ('vibrate' in navigator) {
                navigator.vibrate(0);
            }
        }
        
        // --- LOGIKA UTAMA (Dipicu oleh Firebase) ---
        
        function tick(itemId, endTimeMs, inputMinutes) {
            const timerCard = document.getElementById(`card-${itemId}`);
            if (!timerCard) return; 

            const display = document.getElementById(`display-${itemId}`);
            const alarmMessage = document.getElementById(`msg-${itemId}`);
            const itemName = timerCard.querySelector('h2').textContent;
            
            clearTimeout(activeIntervals[itemId]);

            const now = Date.now();
            let duration = Math.floor((endTimeMs - now) / 1000); 
            
            // Perbarui UI Kontrol ke mode RUNNING
            const inputTime = document.getElementById(`time-input-${itemId}`);
            const startButton = document.getElementById(`start-btn-${itemId}`);
            const resetButton = document.getElementById(`reset-btn-${itemId}`);
            if (inputTime) inputTime.value = inputMinutes;
            if (inputTime) inputTime.readOnly = true;
            if (startButton) startButton.style.display = 'none';
            if (resetButton) resetButton.style.display = 'block';

            // 1. Update tampilan
            if (duration >= 0) {
                 display.textContent = formatTime(duration);
            }
           
            // 2. Transisi ke mode WARNING (15 Menit)
            if (duration <= WARNING_TIME_SECONDS && duration > 0) {
                if (!timerCard.classList.contains('warning')) {
                    // Masuk ke status WARNING untuk pertama kali
                    timerCard.classList.add('warning');
                    timerCard.classList.remove('alert'); 
                    const remainingMinutes = Math.ceil(duration / 60);
                    alarmMessage.textContent = `🔔 PERINGATAN! ${itemName} tersisa ${remainingMinutes} menit.`;
                    alarmMessage.style.display = 'block';
                    
                    // 🚨 PENGEMBANGAN: KIRIM NOTIFIKASI BUBBLE 15 MENIT
                    sendWarningNotification(itemName);
                }
            } else if (duration > WARNING_TIME_SECONDS) {
                 timerCard.classList.remove('warning');
                 timerCard.classList.remove('alert');
                 alarmMessage.style.display = 'none';
            }

            // 3. Bunyikan alarm di mode WARNING (setiap 30 detik)
            if (duration <= WARNING_TIME_SECONDS && duration > 0 && duration % 30 === 0) {
                playAlarm(3); 
            }
            
            // 4. Kondisi Waktu Habis: (Alarm & Hapus dari database)
            if (duration <= 0) {
                clearTimeout(activeIntervals[itemId]);
                delete activeIntervals[itemId];
                
                dbRef.child(itemId).remove().catch(e => console.log('Hapus item gagal, mungkin sudah dihapus perangkat lain.'));
                
                display.textContent = "WAKTU HABIS!";
                timerCard.classList.remove('warning');
                timerCard.classList.add('alert'); 
                alarmMessage.textContent = `✅ SELESAI! Bahan ${itemName} butuh penanganan.`;
                alarmMessage.style.display = 'block';
                
                // --- AKTIVASI ALARM AGRESIF (Lokal di perangkat ini) ---
                sendNotification(itemName); // Notifikasi waktu habis
                startVibrationAlert(); 
                startTitleAlert(itemName);
                startFlashAlarm();
                playAlarm(15); 
                
                return;
            }
            
            // 5. Jadwalkan tick berikutnya (Lokal)
            activeIntervals[itemId] = setTimeout(() => tick(itemId, endTimeMs, inputMinutes), 1000);
        }

        // --- FUNGSI PUBLIK (Dipanggil oleh User) ---
        
        function startCountdown(itemId) {
            resumeAudioContext(); 
            Notification.requestPermission().then(permission => {
                notificationPermission = permission; 
            });
            
            const inputTime = document.getElementById(`time-input-${itemId}`);
            const durationMinutes = parseInt(inputTime.value);

            if (isNaN(durationMinutes) || durationMinutes <= 0) {
                alert(`Mohon masukkan waktu thawing yang valid.`);
                return;
            }
            
            const durationMs = durationMinutes * 60 * 1000;
            const endTimeMs = Date.now() + durationMs; 
            
            dbRef.child(itemId).set({ 
                endTime: endTimeMs, 
                inputMinutes: durationMinutes
            })
            .then(() => {
                console.log(`Timer ${itemId} started and synced.`);
            })
            .catch(error => {
                alert("Gagal memulai timer. Periksa koneksi atau aturan Firebase.");
                console.error(error);
            });
        }

        function resetTimer(itemId) {
            stopAggressiveAlarm(); 
            
            dbRef.child(itemId).remove()
            .then(() => {
                console.log(`Timer ${itemId} reset and synced.`);
            })
            .catch(error => {
                alert("Gagal mereset timer. Periksa koneksi atau aturan Firebase.");
                console.error(error);
            });
            
            localResetUI(itemId); 
        }

        // Mereset UI lokal berdasarkan item default
        function localResetUI(itemId, inputMinutes = null) {
            const item = THAWING_ITEMS.find(i => i.id === itemId);
            if (!item) return;

            const finalInput = inputMinutes !== null ? inputMinutes : item.defaultTimeMinutes;
            
            clearTimeout(activeIntervals[itemId]);
            delete activeIntervals[itemId];

            const timerCard = document.getElementById(`card-${itemId}`);
            const inputTime = document.getElementById(`time-input-${itemId}`);
            const display = document.getElementById(`display-${itemId}`);
            const alarmMessage = document.getElementById(`msg-${itemId}`);
            const startButton = document.getElementById(`start-btn-${itemId}`);
            const resetButton = document.getElementById(`reset-btn-${itemId}`);
            
            if (!timerCard || !inputTime || !display || !startButton || !resetButton) return; 

            timerCard.classList.remove('alert', 'warning');
            inputTime.readOnly = false;
            startButton.style.display = 'block';
            resetButton.style.display = 'none';
            alarmMessage.style.display = 'none';
            
            inputTime.value = finalInput;
            display.textContent = formatTime(finalInput * 60);

            inputTime.focus();
        }

        // Fungsi untuk membuat elemen HTML timer
        function createTimerCard(item) {
            const timerListContainer = document.getElementById('timer-list');
            
            const card = document.createElement('div');
            card.className = 'timer-card';
            card.id = `card-${item.id}`;
            
            card.innerHTML = `
                <h2>${item.name}</h2>
                <div id="display-${item.id}" class="countdown-display">
                    ${formatTime(item.defaultTimeMinutes * 60)}
                </div>
                
                <div id="msg-${item.id}" class="alarm-message" style="display: none;"></div>

                <div class="timer-controls">
                    <label for="time-input-${item.id}">Durasi (mnt):</label>
                    <input type="number" id="time-input-${item.id}" value="${item.defaultTimeMinutes}" min="1" max="180">
                    <button id="start-btn-${item.id}" class="start-btn">START</button>
                    <button id="reset-btn-${item.id}" class="reset-btn" style="display: none;">RESET</button>
                </div>
            `;
            
            timerListContainer.appendChild(card);
            
            document.getElementById(`start-btn-${item.id}`).addEventListener('click', () => startCountdown(item.id));
            document.getElementById(`reset-btn-${item.id}`).addEventListener('click', () => resetTimer(item.id));

            document.getElementById(`time-input-${item.id}`).addEventListener('input', (event) => {
                if (!document.getElementById(`time-input-${item.id}`).readOnly) {
                    const minutes = parseInt(event.target.value) || 0;
                    const display = document.getElementById(`display-${item.id}`);
                    display.textContent = formatTime(minutes * 60);
                }
            });
            
            localResetUI(item.id, item.defaultTimeMinutes);
        }

        // ===================================
        // LOGIKA SINKRONISASI REAL-TIME
        // ===================================
        
        document.addEventListener('DOMContentLoaded', () => {
            THAWING_ITEMS.forEach(item => {
                createTimerCard(item);
            });
            notificationPermission = Notification.permission;
            
            dbRef.on('value', (snapshot) => {
                const timersData = snapshot.val() || {}; 

                THAWING_ITEMS.forEach(item => {
                    const itemId = item.id;
                    const timerState = timersData[itemId];
                    
                    if (timerState && timerState.endTime) {
                        const endTime = timerState.endTime;
                        const inputMinutes = timerState.inputMinutes || item.defaultTimeMinutes;
                        
                        if (endTime < Date.now()) {
                            tick(itemId, endTime, inputMinutes); 
                        } else {
                            tick(itemId, endTime, inputMinutes);
                        }
                    } else {
                        clearTimeout(activeIntervals[itemId]);
                        delete activeIntervals[itemId];

                        localResetUI(itemId, item.defaultTimeMinutes); 
                    }
                });
            });
        });
    </script>
</body>
</html>
