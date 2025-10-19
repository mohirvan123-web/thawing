<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Thawing Reminder - Persistent Alarm & Notifications</title>
    
    <style>
        /* ==================== 
           CSS STYLING (Default/Desktop)
           ==================== */
        body {
            font-family: Arial, sans-serif;
            background-color: #f4f4ff;
            display: flex;
            justify-content: center;
            align-items: flex-start;
            min-height: 100vh;
            margin: 0; 
            padding: 20px 0; 
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
            margin-bottom: 5px;
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
            margin-top: 0;
            color: #007bff;
            border-bottom: 1px solid #eee;
            padding-bottom: 10px;
            font-size: 1.2em;
        }

        .timer-controls {
            display: flex;
            gap: 10px;
            margin-top: 10px;
            align-items: center;
        }
        
        .timer-controls label {
            white-space: nowrap; 
        }

        .timer-controls input {
            padding: 8px;
            flex-grow: 1; 
            max-width: 80px; 
            border: 1px solid #ccc;
            border-radius: 4px;
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

        .start-btn {
            background-color: #28a745;
            color: white;
        }
        .start-btn:hover { background-color: #218838; }

        .reset-btn {
            background-color: #dc3545;
            color: white;
        }
        .reset-btn:hover { background-color: #c82333; }


        .countdown-display {
            font-size: 2em;
            margin: 15px 0;
            padding: 10px;
            border-radius: 6px;
            font-weight: bold;
            text-align: center;
            background-color: #e9ecef;
            color: #333;
            letter-spacing: 1px;
        }

        /* Warna Peringatan & Alarm */
        .timer-card.warning .countdown-display {
            background-color: #fff3cd; 
            color: #856404; 
            animation: flash 1s infinite alternate; 
        }
        .timer-card.warning {
            border-color: #ffc107;
            box-shadow: 0 0 10px rgba(255, 193, 7, 0.5);
        }

        .timer-card.alert .countdown-display {
            background-color: #f8d7da; 
            color: #721c24; 
            animation: none;
        }
        .timer-card.alert {
            border-color: #dc3545;
            box-shadow: 0 0 10px rgba(220, 53, 69, 0.7);
        }

        .alarm-message {
            font-size: 0.9em;
            color: #f44336;
            background-color: #ffeaea;
            padding: 8px;
            border-radius: 4px;
            border: 1px solid #f44336;
            margin-top: 10px;
            text-align: center;
            display: none;
        }

        @keyframes flash {
            from { opacity: 1; }
            to { opacity: 0.5; }
        }

        /* ==================== 
           MEDIA QUERY UNTUK MOBILE (Layar Kecil) - TAMPILAN 2 BARIS RINGKAS
           ==================== */
        @media (max-width: 650px) {
            
            body {
                padding: 10px; 
            }

            .main-container {
                padding: 15px;
                box-shadow: none; 
            }

            /* 1. Atur Teks Utama Lebih Kecil */
            h1 {
                font-size: 1.5em; 
            }
            p {
                font-size: 0.9em; 
            }

            /* 2. Grid menjadi 2 Kolom yang Ringkas */
            .timer-list {
                grid-template-columns: repeat(auto-fit, minmax(140px, 1fr)); 
                gap: 10px; 
            }
            
            .timer-card {
                padding: 10px; 
            }

            .timer-card h2 {
                font-size: 1.0em; 
                padding-bottom: 5px;
            }

            /* 3. Tampilan Countdown Lebih Ringkas */
            .countdown-display {
                font-size: 1.5em; 
                margin: 10px 0;
                padding: 5px;
            }

            /* 4. Kontrol Dibuat Horizontal Ringkas */
            .timer-controls {
                flex-direction: row; 
                flex-wrap: wrap; 
                justify-content: space-between; 
                gap: 5px; 
            }
            
            .timer-controls label {
                font-size: 0.8em; 
                order: 1;
            }

            .timer-controls input {
                padding: 4px; 
                max-width: 40px; 
                order: 2;
            }

            .timer-controls button {
                padding: 5px 8px;
                font-size: 0.75em;
                flex-basis: 0; 
                flex-grow: 1; 
                order: 3;
            }

            .alarm-message {
                font-size: 0.7em;
                padding: 5px;
            }
        }
    </style>
</head>
<body>
    <div class="main-container">
        <h1>Timer Thawing Reminder 🧊</h1>
        <p>Atur dan pantau waktu Thawing untuk berbagai bahan secara bersamaan.</p>

        <div class="timer-list" id="timer-list">
            </div>
    </div>

    <script>
        /* ==================== 
           JAVASCRIPT LOGIC (Integrasi LocalStorage & Notifikasi)
           ==================== */
        
        const THAWING_ITEMS = [
            { id: 1, name: "ADONAN", defaultTimeMinutes: 40 },
            { id: 2, name: "ACIN", defaultTimeMinutes: 60 },
            { id: 3, name: "MIE", defaultTimeMinutes: 120 },
            { id: 4, name: "PENTOL", defaultTimeMinutes: 120 },
            { id: 5, name: "SURAI NAGA", defaultTimeMinutes: 120 },
            { id: 6, name: "KRUPUK MIE", defaultTimeMinutes: 120 },
        ];

        const timerListContainer = document.getElementById('timer-list');
        
        const WARNING_TIME_SECONDS = 15 * 60; // 15 menit dalam detik
        const CLASS_WARNING = 'warning';
        const CLASS_ALERT = 'alert';
        const DISPLAY_NONE = 'none';
        const DISPLAY_BLOCK = 'block';
        
        let activeIntervals = {}; 
        let audioCtx;
        
        const STORAGE_KEY = 'thawingTimerStatus_'; 
        
        let notificationPermission = Notification.permission; 

        // Fungsi untuk membuat suara "beep" yang lebih keras (gainNode.gain.value = 2.0)
        function playBeep(frequency, durationMs) {
            try {
                if (!audioCtx) {
                    audioCtx = new (window.AudioContext || window.webkitAudioContext)();
                }
                
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

        // Fungsi untuk memainkan alarm yang lebih lama (8 beeps @ 400ms)
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
        
        function resumeAudioContext() {
             if (audioCtx && audioCtx.state === 'suspended') {
                 audioCtx.resume();
             }
        }

        function formatTime(totalSeconds) {
            const hours = Math.floor(totalSeconds / 3600);
            const minutes = Math.floor((totalSeconds % 3600) / 60);
            const seconds = totalSeconds % 60;

            const h = String(hours).padStart(2, '0');
            const m = String(minutes).padStart(2, '0');
            const s = String(seconds).padStart(2, '0');

            return `${h}:${m}:${s}`;
        }
        
        // Fungsi untuk menampilkan notifikasi pop-up (Ikon eksternal dihapus)
        function sendNotification(title, body) {
            if (notificationPermission === 'granted') {
                try {
                    new Notification(title, {
                        body: body,
                        // Ikon eksternal dihapus untuk menghindari error CORS/blokir browser.
                        // Anda bisa menggantinya dengan ikon base64 atau menghapusnya
                        renotify: true, 
                        tag: 'thawing-alarm' 
                    });
                } catch (e) {
                    console.error("Error sending notification:", e);
                }
            }
        }


        // Fungsi REKURSIF inti (tick)
        function tick(itemId, endTimeMs) {
            
            const timerCard = document.getElementById(`card-${itemId}`);
            if (!timerCard) return; 
            
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
           
            // 2. Transisi ke mode WARNING
            if (duration <= WARNING_TIME_SECONDS && duration > 0) {
                if (!timerCard.classList.contains(CLASS_WARNING)) {
                    timerCard.classList.add(CLASS_WARNING);
                    timerCard.classList.remove(CLASS_ALERT); 
                    const remainingMinutes = Math.ceil(duration / 60);
                    alarmMessage.textContent = `🔔 PERINGATAN! ${itemName} tersisa ${remainingMinutes} menit!`;
                    alarmMessage.style.display = DISPLAY_BLOCK;
                }
            } else if (duration > WARNING_TIME_SECONDS) {
                 timerCard.classList.remove(CLASS_WARNING);
                 alarmMessage.style.display = DISPLAY_NONE;
            }

            // 3. Bunyikan alarm di mode WARNING (setiap 30 detik)
            if (duration <= WARNING_TIME_SECONDS && duration > 0 && duration % 30 === 0) {
                playAlarm(1); 
            }
            
            // 4. Kondisi Waktu Habis: (Logic utama notifikasi dan alarm)
            if (duration <= 0) {
                delete activeIntervals[itemId];
                localStorage.removeItem(STORAGE_KEY + itemId); 
                
                display.textContent = "00:00:00";
                timerCard.classList.remove(CLASS_WARNING);
                timerCard.classList.add(CLASS_ALERT); 
                alarmMessage.textContent = `✅ WAKTU ${itemName} SELESAI! Segera Ambil Bahan.`;
                alarmMessage.style.display = DISPLAY_BLOCK;
                
                // AKSI ALARM DAN NOTIFIKASI POP-UP
                sendNotification("WAKTU THAWING HABIS!", `Segera ambil bahan: ${itemName}.`);
                playAlarm(8); 
                
                // Reset UI kontrol
                const inputTime = document.getElementById(`time-input-${itemId}`);
                document.getElementById(`start-btn-${itemId}`).style.display = DISPLAY_BLOCK;
                document.getElementById(`reset-btn-${itemId}`).style.display = DISPLAY_NONE;
                inputTime.readOnly = false;
                return; 
            }
            
            // 5. Jadwalkan tick berikutnya
            activeIntervals[itemId] = setTimeout(() => tick(itemId, endTimeMs), 1000);
        }


        // Fungsi utama untuk menjalankan countdown
        function startCountdown(itemId) {
            resumeAudioContext(); 
            
            // Minta Izin Notifikasi Pop-up (dipicu oleh interaksi user)
            Notification.requestPermission().then(permission => {
                notificationPermission = permission; 
            });
            
            const timerCard = document.getElementById(`card-${itemId}`);
            const inputTime = document.getElementById(`time-input-${itemId}`);
            
            if (!timerCard || !inputTime) {
                console.error(`DOM Error: Card or Input element not found for ID ${itemId}`);
                return; 
            }
            
            const itemName = timerCard.querySelector('h2').textContent;
            
            if (activeIntervals[itemId]) {
                const confirmRestart = confirm(`Timer untuk ${itemName} sedang berjalan. Apakah Anda ingin menghentikannya dan MEMULAI ULANG dengan waktu yang baru?`);
                if (!confirmRestart) return; 
                clearTimeout(activeIntervals[itemId]);
                delete activeIntervals[itemId];
                localStorage.removeItem(STORAGE_KEY + itemId); 
            }

            const durationMinutes = parseInt(inputTime.value);
            if (isNaN(durationMinutes) || durationMinutes <= 0) {
                alert(`Mohon masukkan waktu thawing yang valid untuk ${itemName}.`);
                return;
            }
            
            // LOGIKA PENYIMPANAN STATE BARU
            const durationMs = durationMinutes * 60 * 1000;
            const endTimeMs = Date.now() + durationMs; 
            localStorage.setItem(STORAGE_KEY + itemId, endTimeMs);
            
            // Perubahan UI
            document.getElementById(`start-btn-${itemId}`).style.display = DISPLAY_NONE;
            document.getElementById(`reset-btn-${itemId}`).style.display = DISPLAY_BLOCK;
            inputTime.readOnly = true;
            timerCard.classList.remove(CLASS_ALERT, CLASS_WARNING);
            document.getElementById(`msg-${itemId}`).style.display = DISPLAY_NONE;

            // Jalankan tick
            tick(itemId, endTimeMs);
        }

        // Fungsi reset
        function resetTimer(itemId) {
            clearTimeout(activeIntervals[itemId]);
            delete activeIntervals[itemId];
            
            localStorage.removeItem(STORAGE_KEY + itemId); 

            const item = THAWING_ITEMS.find(i => i.id === itemId);

            const timerCard = document.getElementById(`card-${itemId}`);
            const inputTime = document.getElementById(`time-input-${itemId}`);
            const display = document.getElementById(`display-${itemId}`);
            const alarmMessage = document.getElementById(`msg-${itemId}`);
            const startButton = document.getElementById(`start-btn-${itemId}`);
            const resetButton = document.getElementById(`reset-btn-${itemId}`);
            
            if (!timerCard || !inputTime || !display) return; // Safeguard

            timerCard.classList.remove(CLASS_ALERT, CLASS_WARNING);
            inputTime.readOnly = false;
            startButton.style.display = DISPLAY_BLOCK;
            resetButton.style.display = DISPLAY_NONE;
            alarmMessage.style.display = DISPLAY_NONE;

            inputTime.value = item.defaultTimeMinutes;
            display.textContent = formatTime(item.defaultTimeMinutes * 60);

            inputTime.focus();
        }


        // Fungsi untuk membuat dan menambahkan elemen HTML timer
        function createTimerCard(item) {
            try {
                const card = document.createElement('div');
                card.className = 'timer-card';
                card.id = `card-${item.id}`;
                
                const savedEndTime = localStorage.getItem(STORAGE_KEY + item.id);
                let initialDisplayTime;
                let isRunning = false;

                if (savedEndTime) {
                    const remainingMs = parseInt(savedEndTime) - Date.now();
                    if (remainingMs > 1000) { 
                        initialDisplayTime = formatTime(Math.ceil(remainingMs / 1000));
                        isRunning = true;
                    } else {
                        localStorage.removeItem(STORAGE_KEY + item.id);
                        initialDisplayTime = formatTime(item.defaultTimeMinutes * 60);
                    }
                } else {
                    initialDisplayTime = formatTime(item.defaultTimeMinutes * 60);
                }
                
                card.innerHTML = `
                    <h2>${item.name}</h2>
                    <div id="display-${item.id}" class="countdown-display">
                        ${initialDisplayTime}
                    </div>
                    
                    <div id="msg-${item.id}" class="alarm-message"></div>

                    <div class="timer-controls">
                        <label>Durasi (mnt):</label>
                        <input type="number" id="time-input-${item.id}" value="${item.defaultTimeMinutes}" min="1" max="180" ${isRunning ? 'readonly' : ''}>
                        <button id="start-btn-${item.id}" class="start-btn" style="display: ${isRunning ? DISPLAY_NONE : DISPLAY_BLOCK};">START</button>
                        <button id="reset-btn-${item.id}" class="reset-btn" style="display: ${isRunning ? DISPLAY_BLOCK : DISPLAY_NONE};">RESET</button>
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

                // 2. Jika timer berjalan, jalankan kembali tick
                if (isRunning) {
                    setTimeout(() => {
                        tick(item.id, parseInt(savedEndTime));
                    }, 50); 
                }
            } catch (e) {
                console.error(`Error creating card for item ${item.id}:`, e);
            }
        }


        // Inisialisasi: Loop data dan buat semua timer
        document.addEventListener('DOMContentLoaded', () => {
            THAWING_ITEMS.forEach(item => {
                createTimerCard(item);
            });
            notificationPermission = Notification.permission;
        });
    </script>
</body>
</html>
