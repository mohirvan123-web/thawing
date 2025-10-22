<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Thawing Reminder - Alarm Agresif & Persistent (MLGJAK)</title>
    
    <style>
        /* ==================== 
           CSS: STYLING & RESPONSIVITAS (Dibiarkan sama)
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
// FIREBASE CONFIGURATION (SUDAH DIPERBAIKI)
// ===================================
const firebaseConfig = {
  apiKey: "AIzaSyBtUlghTw806GuGuwOXGNgoqN6Rkcg0IMM",
  authDomain: "thawing-ec583.firebaseapp.com",
  databaseURL: "https://thawing-ec583-default-rtdb.asia-southeast1.firebasedatabase.app",
  projectId: "thawing-ec583",
  storageBucket: "thawing-ec583.firebasestorage.app", // Pastikan ada koma
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

        // Fungsi Audio & Alarm (Tidak diubah, karena alarm bersifat lokal pada setiap perangkat)
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
            const interval = setInterval(() => {
                playBeep(440, 400); 
                count++;
                if (count >= times) {
                    clearInterval(interval);
                }
            }, 500); 
        }

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
        }
        
        // --- LOGIKA UTAMA (Sama, namun dipicu oleh Firebase) ---
        
        /**
         * Fungsi tick sekarang dipicu oleh listener Firebase, bukan hanya tombol START.
         * Ia menghentikan dirinya sendiri (cleartimeout) dan menjadwalkan yang baru.
         */
        function tick(itemId, endTimeMs, inputMinutes) {
            const timerCard = document.getElementById(`card-${itemId}`);
            if (!timerCard) return; // Exit jika card belum ter-render (safety check)

            const display = document.getElementById(`display-${itemId}`);
            const alarmMessage = document.getElementById(`msg-${itemId}`);
            const itemName = timerCard.querySelector('h2').textContent;
            
            clearTimeout(activeIntervals[itemId]);

            const now = Date.now();
            let duration = Math.floor((endTimeMs - now) / 1000); 
            
            // Perbarui UI Kontrol
            const inputTime = document.getElementById(`time-input-${itemId}`);
            const startButton = document.getElementById(`start-btn-${itemId}`);
            const resetButton = document.getElementById(`reset-btn-${itemId}`);
            if (inputTime) inputTime.value = inputMinutes; // Sinkronkan input
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
            
            // 4. Kondisi Waktu Habis: (Alarm & Hapus dari database)
            if (duration <= 0) {
                // Hentikan interval lokal
                clearTimeout(activeIntervals[itemId]);
                delete activeIntervals[itemId];
                
                // Hapus entry dari Firebase (Hanya satu perangkat yang berhasil menghapus)
                dbRef.child(itemId).remove().catch(e => console.log('Hapus item gagal, mungkin sudah dihapus perangkat lain.'));
                
                display.textContent = "WAKTU HABIS!";
                timerCard.classList.remove('warning');
                timerCard.classList.add('alert'); 
                alarmMessage.textContent = `✅ SELESAI! Bahan ${itemName} butuh penanganan.`;
                alarmMessage.style.display = 'block';
                
                // --- AKTIVASI ALARM AGRESIF (Lokal di perangkat ini) ---
                sendNotification(itemName);
                startVibrationAlert(); 
                startTitleAlert(itemName);
                startFlashAlarm();
                playAlarm(15); 
                
                // UI akan direset oleh listener Firebase saat data dihapus
                return; 
            }
            
            // 5. Jadwalkan tick berikutnya (Lokal)
            activeIntervals[itemId] = setTimeout(() => tick(itemId, endTimeMs, inputMinutes), 1000);
        }

        // --- FUNGSI PUBLIK (Dipanggil oleh User) ---
        
        // FUNGSI INI MENGIRIM PERUBAHAN KE FIREBASE
        function startCountdown(itemId) {
            resumeAudioContext(); 
            Notification.requestPermission().then(permission => {
                notificationPermission = permission; 
            });
            
            const timerCard = document.getElementById(`card-${itemId}`);
            const inputTime = document.getElementById(`time-input-${itemId}`);
            const durationMinutes = parseInt(inputTime.value);

            if (isNaN(durationMinutes) || durationMinutes <= 0) {
                alert(`Mohon masukkan waktu thawing yang valid.`);
                return;
            }
            
            const durationMs = durationMinutes * 60 * 1000;
            const endTimeMs = Date.now() + durationMs; 
            
            // 🚨 LOGIKA UTAMA: TULIS KE DATABASE 
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

        // FUNGSI INI MENGHAPUS DATA DARI FIREBASE
        function resetTimer(itemId) {
            stopAggressiveAlarm(); 
            
            // 🚨 LOGIKA UTAMA: HAPUS DARI DATABASE
            dbRef.child(itemId).remove()
            .then(() => {
                console.log(`Timer ${itemId} reset and synced.`);
            })
            .catch(error => {
                alert("Gagal mereset timer. Periksa koneksi atau aturan Firebase.");
                console.error(error);
            });
            
            // UI akan direset secara lokal oleh listener Firebase
            localResetUI(itemId);
        }

        // Mereset UI lokal berdasarkan item default
        function localResetUI(itemId, inputMinutes = null) {
            const item = THAWING_ITEMS.find(i => i.id === itemId);
            if (!item) return;

            // Jika inputMinutes tidak disediakan, ambil nilai default
            const finalInput = inputMinutes !== null ? inputMinutes : item.defaultTimeMinutes;
            
            clearTimeout(activeIntervals[itemId]);
            delete activeIntervals[itemId];

            const timerCard = document.getElementById(`card-${itemId}`);
            const inputTime = document.getElementById(`time-input-${itemId}`);
            const display = document.getElementById(`display-${itemId}`);
            const alarmMessage = document.getElementById(`msg-${itemId}`);
            const startButton = document.getElementById(`start-btn-${itemId}`);
            const resetButton = document.getElementById(`reset-btn-${itemId}`);
            
            if (!timerCard || !inputTime || !display) return; 

            timerCard.classList.remove('alert', 'warning');
            inputTime.readOnly = false;
            startButton.style.display = 'block';
            resetButton.style.display = 'none';
            alarmMessage.style.display = 'none';

            inputTime.value = finalInput;
            display.textContent = formatTime(finalInput * 60);

            inputTime.focus();
        }

        // Fungsi untuk membuat elemen HTML timer (Dibiarkan sama)
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
            
            // Panggil reset lokal untuk memastikan UI sesuai default saat dimuat
            localResetUI(item.id, item.defaultTimeMinutes);
        }

        // ===================================
        // 🚨 LOGIKA SINKRONISASI REAL-TIME
        // ===================================
        
        document.addEventListener('DOMContentLoaded', () => {
            // 1. Render semua kartu timer
            THAWING_ITEMS.forEach(item => {
                createTimerCard(item);
            });
            notificationPermission = Notification.permission;
            
            // 2. Pasang Listener Real-time
            // Listener ini akan dipicu setiap kali ada perubahan di node 'thawingTimers'
            dbRef.on('value', (snapshot) => {
                const timersData = snapshot.val() || {}; // Ambil data, atau objek kosong jika tidak ada

                // Loop melalui semua item default
                THAWING_ITEMS.forEach(item => {
                    const itemId = item.id;
                    const timerState = timersData[itemId];
                    
                    if (timerState && timerState.endTime) {
                        // Timer sedang berjalan (Data ditemukan di Firebase)
                        const endTime = timerState.endTime;
                        const inputMinutes = timerState.inputMinutes || item.defaultTimeMinutes;
                        
                        // Periksa apakah timer sudah selesai (hanya untuk kasus ekstrem)
                        if (endTime < Date.now()) {
                            // Jika sudah berakhir, panggil tick untuk memicu alarm dan penghapusan
                            tick(itemId, endTime, inputMinutes); 
                        } else {
                            // Panggil tick untuk memulai/melanjutkan countdown
                            tick(itemId, endTime, inputMinutes);
                        }
                    } else {
                        // Timer TIDAK berjalan (Data tidak ada di Firebase)
                        
                        // Hentikan interval lokal jika ada yang masih aktif
                        clearTimeout(activeIntervals[itemId]);
                        delete activeIntervals[itemId];

                        // Reset tampilan UI ke status default/input terakhir
                        // Gunakan default time dari config, karena input time tidak disimpan setelah reset
                        localResetUI(itemId, item.defaultTimeMinutes); 
                    }
                });
            });
        });
    </script>
</body>
</html>
