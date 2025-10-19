<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Thawing Reminder - Optimized Beep Alarm</title>
    
    <style>
        /* ==================== 
           CSS STYLING (Sama seperti sebelumnya)
           ==================== */
        body {
            font-family: Arial, sans-serif;
            background-color: #f4f4ff;
            display: flex;
            justify-content: center;
            align-items: flex-start;
            min-height: 100vh;
            margin: 20px;
        }

        .main-container {
            background-color: white;
            padding: 30px;
            border-radius: 12px;
            box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
            text-align: center;
            width: 100%;
            max-width: 800px;
        }

        h1 {
            color: #333;
            margin-bottom: 5px;
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
        
        .timer-controls input {
            padding: 8px;
            width: 80px;
            border: 1px solid #ccc;
            border-radius: 4px;
        }

        .timer-controls button {
            padding: 8px 15px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-weight: bold;
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

        /* Warna Peringatan */
        .timer-card.warning .countdown-display {
            background-color: #fff3cd; 
            color: #856404; 
            animation: flash 1s infinite alternate; 
        }
        .timer-card.warning {
            border-color: #ffc107;
            box-shadow: 0 0 10px rgba(255, 193, 7, 0.5);
        }

        /* Warna Alarm (Waktu Habis) */
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

        /* Animasi Kedip untuk Peringatan */
        @keyframes flash {
            from { opacity: 1; }
            to { opacity: 0.5; }
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
           JAVASCRIPT LOGIC (Optimized)
           ==================== */
        
        // Data Bahan Thawing
        const THAWING_ITEMS = [
            { id: 1, name: "ADONAN", defaultTimeMinutes: 40 },
            { id: 2, name: "ACIN", defaultTimeMinutes: 60 },
            { id: 3, name: "MIE", defaultTimeMinutes: 120 },
            { id: 4, name: "PENTOL", defaultTimeMinutes: 120 },
            { id: 5, name: "SURAI NAGA", defaultTimeMinutes: 120 },
            { id: 6, name: "KRUPUK MIE", defaultTimeMinutes: 120 },
        ];

        const timerListContainer = document.getElementById('timer-list');
        
        // Konstanta untuk optimalisasi dan keterbacaan
        const WARNING_TIME_SECONDS = 15 * 60; // 15 menit dalam detik
        const CLASS_WARNING = 'warning';
        const CLASS_ALERT = 'alert';
        const DISPLAY_NONE = 'none';
        const DISPLAY_BLOCK = 'block';
        
        // Mengganti activeIntervals menjadi penyimpan ID Timeout
        let activeIntervals = {}; 
        
        // Inisialisasi AudioContext untuk Beep Tone
        let audioCtx;

        // Fungsi untuk membuat suara "beep" sederhana menggunakan AudioContext
        function playBeep(frequency, durationMs) {
            if (!audioCtx) {
                audioCtx = new (window.AudioContext || window.webkitAudioContext)();
            }
            
            const oscillator = audioCtx.createOscillator();
            const gainNode = audioCtx.createGain();

            oscillator.connect(gainNode);
            gainNode.connect(audioCtx.destination);

            oscillator.type = 'square';
            oscillator.frequency.setValueAtTime(frequency, audioCtx.currentTime);

            oscillator.start();
            
            gainNode.gain.exponentialRampToValueAtTime(
                0.00001, audioCtx.currentTime + durationMs / 1000
            );
            oscillator.stop(audioCtx.currentTime + durationMs / 1000);
        }

        // Fungsi umum untuk memainkan alarm (memanggil Beep)
        function playAlarm(times = 1) {
            let count = 0;
            const interval = setInterval(() => {
                playBeep(440, 200); 
                count++;
                if (count >= times) {
                    clearInterval(interval);
                }
            }, 300); 
        }
        
        // PENTING: Perlu interaksi pengguna untuk mengaktifkan AudioContext
        function resumeAudioContext() {
             if (audioCtx && audioCtx.state === 'suspended') {
                 audioCtx.resume();
             }
        }


        // Fungsi untuk memformat total detik menjadi HH:MM:SS
        function formatTime(totalSeconds) {
            const hours = Math.floor(totalSeconds / 3600);
            const minutes = Math.floor((totalSeconds % 3600) / 60);
            const seconds = totalSeconds % 60;

            const h = String(hours).padStart(2, '0');
            const m = String(minutes).padStart(2, '0');
            const s = String(seconds).padStart(2, '0');

            return `${h}:${m}:${s}`;
        }
        
        // Fungsi REKURSIF inti untuk menjalankan countdown
        function tick(itemId, duration) {
            const timerCard = document.getElementById(`card-${itemId}`);
            const display = document.getElementById(`display-${itemId}`);
            const alarmMessage = document.getElementById(`msg-${itemId}`);
            const itemName = timerCard.querySelector('h2').textContent;
            
            duration--;
            
            // --- Logika Update & Alarm ---
            
            // 1. Update tampilan di setiap tick
            display.textContent = formatTime(duration);
            
            // 2. Transisi ke mode WARNING
            if (duration <= WARNING_TIME_SECONDS && duration > 0 && !timerCard.classList.contains(CLASS_WARNING)) {
                timerCard.classList.add(CLASS_WARNING);
                timerCard.classList.remove(CLASS_ALERT); // Pastikan alert dihapus
                const remainingMinutes = Math.ceil(duration / 60);
                alarmMessage.textContent = `🔔 PERINGATAN! ${itemName} tersisa ${remainingMinutes} menit!`;
                alarmMessage.style.display = DISPLAY_BLOCK;
            }

            // 3. Bunyikan alarm di mode WARNING (setiap 30 detik)
            if (duration <= WARNING_TIME_SECONDS && duration > 0 && duration % 30 === 0) {
                playAlarm(1);
            }
            
            // 4. Kondisi Waktu Habis:
            if (duration <= 0) {
                // Hentikan timer dengan menghapus ID timeout dari penyimpanan
                clearTimeout(activeIntervals[itemId]);
                delete activeIntervals[itemId];
                
                display.textContent = "00:00:00";
                timerCard.classList.remove(CLASS_WARNING);
                timerCard.classList.add(CLASS_ALERT); 
                alarmMessage.textContent = `✅ WAKTU ${itemName} SELESAI! Segera Ambil Bahan.`;
                alarmMessage.style.display = DISPLAY_BLOCK;
                playAlarm(5); // Bunyikan 5 kali untuk penekanan
                
                // Reset UI kontrol
                const inputTime = document.getElementById(`time-input-${itemId}`);
                document.getElementById(`start-btn-${itemId}`).style.display = DISPLAY_BLOCK;
                document.getElementById(`reset-btn-${itemId}`).style.display = DISPLAY_NONE;
                inputTime.readOnly = false;
                return; // Hentikan rekursi
            }
            
            // 5. Jadwalkan tick berikutnya (recursively call setTimeout)
            activeIntervals[itemId] = setTimeout(() => tick(itemId, duration), 1000);
        }


        // Fungsi utama untuk menjalankan countdown
        function startCountdown(itemId) {
            resumeAudioContext(); 
            
            const timerCard = document.getElementById(`card-${itemId}`);
            const inputTime = document.getElementById(`time-input-${itemId}`);
            const itemName = timerCard.querySelector('h2').textContent;

            // --- Peningkatan UX: Konfirmasi Restart ---
            if (activeIntervals[itemId]) {
                const confirmRestart = confirm(`Timer untuk ${itemName} sedang berjalan. Apakah Anda ingin menghentikannya dan MEMULAI ULANG dengan waktu yang baru?`);
                
                if (!confirmRestart) {
                    return; 
                }
                // Jika Ya, hentikan timer yang lama
                clearTimeout(activeIntervals[itemId]);
                delete activeIntervals[itemId];
            }
            // ------------------------------------------

            const display = document.getElementById(`display-${itemId}`);
            const alarmMessage = document.getElementById(`msg-${itemId}`);
            const startButton = document.getElementById(`start-btn-${itemId}`);
            const resetButton = document.getElementById(`reset-btn-${itemId}`);
            
            let duration = parseInt(inputTime.value) * 60; 
            
            if (isNaN(duration) || duration <= 0) {
                alert(`Mohon masukkan waktu thawing yang valid untuk ${itemName}.`);
                return;
            }
            
            // Perubahan UI saat timer dimulai
            startButton.style.display = DISPLAY_NONE;
            resetButton.style.display = DISPLAY_BLOCK;
            inputTime.readOnly = true;
            timerCard.classList.remove(CLASS_ALERT, CLASS_WARNING);
            display.textContent = formatTime(duration);
            alarmMessage.style.display = DISPLAY_NONE;


            // Jalankan tick pertama. Fungsi tick akan menjadwalkan dirinya sendiri.
            activeIntervals[itemId] = setTimeout(() => tick(itemId, duration), 1000);
        }

        // Fungsi reset
        function resetTimer(itemId) {
            // Hentikan timer (menggunakan clearTimeout karena kita menggunakan setTimeout)
            clearTimeout(activeIntervals[itemId]);
            delete activeIntervals[itemId];

            const item = THAWING_ITEMS.find(i => i.id === itemId);

            const timerCard = document.getElementById(`card-${itemId}`);
            const inputTime = document.getElementById(`time-input-${itemId}`);
            const display = document.getElementById(`display-${itemId}`);
            const alarmMessage = document.getElementById(`msg-${itemId}`);
            const startButton = document.getElementById(`start-btn-${itemId}`);
            const resetButton = document.getElementById(`reset-btn-${itemId}`);
            
            timerCard.classList.remove(CLASS_ALERT, CLASS_WARNING);
            inputTime.readOnly = false;
            startButton.style.display = DISPLAY_BLOCK;
            resetButton.style.display = DISPLAY_NONE;
            alarmMessage.style.display = DISPLAY_NONE;

            // Atur ulang nilai input dan tampilan ke waktu default
            inputTime.value = item.defaultTimeMinutes;
            display.textContent = formatTime(item.defaultTimeMinutes * 60);

            // Fokus ke input setelah reset untuk UX yang lebih cepat
            inputTime.focus();
        }


        // Fungsi untuk membuat dan menambahkan elemen HTML timer
        function createTimerCard(item) {
            const card = document.createElement('div');
            card.className = 'timer-card';
            card.id = `card-${item.id}`;
            
            card.innerHTML = `
                <h2>${item.name}</h2>
                <div id="display-${item.id}" class="countdown-display">
                    ${formatTime(item.defaultTimeMinutes * 60)}
                </div>
                
                <div id="msg-${item.id}" class="alarm-message"></div>

                <div class="timer-controls">
                    <label>Durasi (mnt):</label>
                    <input type="number" id="time-input-${item.id}" value="${item.defaultTimeMinutes}" min="1" max="180">
                    <button id="start-btn-${item.id}" class="start-btn">START</button>
                    <button id="reset-btn-${item.id}" class="reset-btn" style="display: none;">RESET</button>
                </div>
            `;
            
            timerListContainer.appendChild(card);
            
            // Tambahkan Event Listener
            document.getElementById(`start-btn-${item.id}`).addEventListener('click', () => startCountdown(item.id));
            document.getElementById(`reset-btn-${item.id}`).addEventListener('click', () => resetTimer(item.id));

            // --- Peningkatan UX: Update tampilan real-time saat input berubah ---
            document.getElementById(`time-input-${item.id}`).addEventListener('input', (event) => {
                const minutes = parseInt(event.target.value) || 0;
                const display = document.getElementById(`display-${item.id}`);
                display.textContent = formatTime(minutes * 60);
            });
            // -------------------------------------------------------------------
        }


        // Inisialisasi: Loop data dan buat semua timer
        document.addEventListener('DOMContentLoaded', () => {
            THAWING_ITEMS.forEach(item => {
                createTimerCard(item);
            });
        });
    </script>
</body>
</html>
