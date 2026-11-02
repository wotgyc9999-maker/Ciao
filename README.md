<!DOCTYPE html>
<html lang="it">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Agenda 24H - 2026/2036</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/firebase/9.23.0/firebase-app-compat.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/firebase/9.23.0/firebase-database-compat.min.js"></script>
    <style>
        .glass { background: rgba(15, 23, 42, 0.75); backdrop-filter: blur(12px); }
        input[type="date"]::-webkit-calendar-picker-indicator { filter: invert(1); }
    </style>
</head>
<body class="bg-slate-950 text-slate-100 min-h-screen flex flex-col font-sans selection:bg-indigo-500 selection:text-white">

    <!-- HEADER -->
    <header class="sticky top-0 z-50 glass border-b border-slate-800 px-4 py-3 flex flex-wrap justify-between items-center gap-4">
        <div class="flex items-center space-x-3">
            <span class="text-2xl">📅</span>
            <div>
                <h1 class="text-lg font-bold tracking-wide text-indigo-400">Agenda Digitale H24</h1>
                <p class="text-xs text-slate-400">Copertura: 2026 - 2036 (Condivisa)</p>
            </div>
        </div>

        <div class="flex items-center space-x-2">
            <span id="lockStatusBadge" class="px-2.5 py-1 rounded-full text-xs font-semibold bg-amber-500/10 text-amber-400 border border-amber-500/20">🔒 Solo Lettura</span>
            <button id="authBtn" onclick="toggleAuth()" class="bg-indigo-600 hover:bg-indigo-500 text-white px-3.5 py-1.5 rounded-lg text-xs font-medium transition shadow-sm">Sblocca Modifica</button>
            <button onclick="openSettings()" class="bg-slate-800 hover:bg-slate-700 p-1.5 rounded-lg text-slate-300 text-xs transition">⚙️</button>
        </div>
    </header>

    <!-- CONTROLLO DATA & NAVIGAZIONE -->
    <main class="flex-1 max-w-6xl w-full mx-auto p-4 flex flex-col gap-4">
        <div class="glass p-4 rounded-2xl border border-slate-800 flex flex-wrap items-center justify-between gap-4">
            <div class="flex items-center space-x-2">
                <button onclick="changeDay(-1)" class="bg-slate-800 hover:bg-slate-700 px-3 py-2 rounded-xl text-sm font-bold transition">◀</button>
                <input type="date" id="selectedDate" min="2026-01-01" max="2036-12-31" onchange="loadDayData()" class="bg-slate-900 border border-slate-700 rounded-xl px-3 py-2 text-sm text-indigo-200 font-medium focus:outline-none focus:border-indigo-500">
                <button onclick="changeDay(1)" class="bg-slate-800 hover:bg-slate-700 px-3 py-2 rounded-xl text-sm font-bold transition">▶</button>
                <button onclick="goToday()" class="bg-slate-800 hover:bg-slate-700 px-3 py-2 rounded-xl text-xs font-medium transition text-indigo-300">Oggi</button>
            </div>
            
            <div class="relative flex-1 max-w-xs">
                <input type="text" id="searchInput" placeholder="Cerca negli eventi..." oninput="filterEvents()" class="w-full bg-slate-900 border border-slate-700 rounded-xl px-3 py-2 text-xs text-slate-200 placeholder-slate-500 focus:outline-none focus:border-indigo-500">
            </div>
        </div>

        <!-- TABELLA 24 ORE -->
        <div class="glass rounded-2xl border border-slate-800 overflow-hidden shadow-xl flex-1">
            <div class="overflow-x-auto">
                <table class="w-full text-left border-collapse">
                    <thead>
                        <tr class="bg-slate-900/90 text-slate-400 text-xs border-b border-slate-800 uppercase tracking-wider">
                            <th class="p-3 w-24 text-center">Orario</th>
                            <th class="p-3">Evento / Attività</th>
                            <th class="p-3 w-48">Assegnatario / Note</th>
                            <th class="p-3 w-28 text-center">Stato</th>
                        </tr>
                    </thead>
                    <tbody id="hoursTableBody" class="divide-y divide-slate-800/60 text-sm">
                        <!-- Generato via JavaScript -->
                    </tbody>
                </table>
            </div>
        </div>
    </main>

    <!-- MODALE PIN -->
    <div id="authModal" class="fixed inset-0 z-50 bg-black/80 backdrop-blur-sm hidden flex items-center justify-center p-4">
        <div class="glass p-6 rounded-2xl border border-slate-700 max-w-sm w-full space-y-4">
            <h3 class="text-base font-bold text-slate-200">Inserisci PIN di Sblocco</h3>
            <p class="text-xs text-slate-400">Per modificare la tabella è richiesto il PIN autorizzato.</p>
            <input type="password" id="pinInput" placeholder="Inserisci PIN" class="w-full bg-slate-900 border border-slate-700 rounded-xl px-3 py-2.5 text-sm text-center tracking-widest text-slate-100 focus:outline-none focus:border-indigo-500">
            <div class="flex space-x-2 pt-2">
                <button onclick="closeAuthModal()" class="flex-1 bg-slate-800 hover:bg-slate-700 py-2 rounded-xl text-xs font-medium transition">Annulla</button>
                <button onclick="verifyPin()" class="flex-1 bg-indigo-600 hover:bg-indigo-500 py-2 rounded-xl text-xs font-medium transition text-white">Sblocca</button>
            </div>
        </div>
    </div>

    <!-- MODALE IMPOSTAZIONI -->
    <div id="settingsModal" class="fixed inset-0 z-50 bg-black/80 backdrop-blur-sm hidden flex items-center justify-center p-4">
        <div class="glass p-6 rounded-2xl border border-slate-700 max-w-sm w-full space-y-4">
            <h3 class="text-base font-bold text-slate-200">Impostazioni</h3>
            <div class="space-y-3">
                <div>
                    <label class="text-xs text-slate-400 block mb-1">Cambia PIN Amministratore</label>
                    <input type="text" id="newPinInput" placeholder="Nuovo PIN" class="w-full bg-slate-900 border border-slate-700 rounded-xl px-3 py-2 text-xs text-slate-100 focus:outline-none focus:border-indigo-500">
                </div>
                <button onclick="saveNewPin()" class="w-full bg-indigo-600 hover:bg-indigo-500 py-2 rounded-xl text-xs font-medium transition text-white">Salva Nuovo PIN</button>
            </div>
            <hr class="border-slate-800">
            <div class="flex space-x-2">
                <button onclick="exportData()" class="flex-1 bg-slate-800 hover:bg-slate-700 py-2 rounded-xl text-xs font-medium transition">Esporta JSON</button>
                <button onclick="closeSettings()" class="flex-1 bg-slate-800 hover:bg-slate-700 py-2 rounded-xl text-xs font-medium transition">Chiudi</button>
            </div>
        </div>
    </div>

    <script>
        let isAuthorized = false;
        let adminPin = localStorage.getItem('agenda_admin_pin') || 'leovincenzo11';
        let currentDayData = {};

        // Configurazione Firebase (Database Realtime Pubblico Condiviso)
        const firebaseConfig = {
            databaseURL: "https://agenda-h24-default-rtdb.europe-west1.firebasedatabase.app/"
        };
        
        let db = null;
        try {
            firebase.initializeApp(firebaseConfig);
            db = firebase.database();
        } catch (e) {
            console.warn("Modalità offline attiva");
        }

        function initApp() {
            const today = new Date().toISOString().split('T')[0];
            document.getElementById('selectedDate').value = today;
            buildTableStructure();
            loadDayData();
        }

        function buildTableStructure() {
            const tbody = document.getElementById('hoursTableBody');
            tbody.innerHTML = '';
            for (let i = 0; i < 24; i++) {
                const hourStr = String(i).padStart(2, '0') + ':00';
                const nextHourStr = String((i + 1) % 24).padStart(2, '0') + ':00';
                const timeLabel = `${hourStr} - ${nextHourStr}`;
                
                const tr = document.createElement('tr');
                tr.className = "hover:bg-slate-900/40 transition border-b border-slate-900/50";
                tr.innerHTML = `
                    <td class="p-3 text-center font-mono text-xs text-indigo-300 font-semibold bg-slate-900/20">${timeLabel}</td>
                    <td class="p-2">
                        <input type="text" data-hour="${i}" data-field="event" placeholder="Nessun evento..." 
                            ${!isAuthorized ? 'disabled' : ''} oninput="updateData(${i}, 'event', this.value)"
                            class="w-full bg-transparent border border-transparent hover:border-slate-800 focus:border-indigo-500 rounded-lg px-2.5 py-1.5 text-xs text-slate-100 focus:outline-none transition">
                    </td>
                    <td class="p-2">
                        <input type="text" data-hour="${i}" data-field="assignee" placeholder="Note / Autore..." 
                            ${!isAuthorized ? 'disabled' : ''} oninput="updateData(${i}, 'assignee', this.value)"
                            class="w-full bg-transparent border border-transparent hover:border-slate-800 focus:border-indigo-500 rounded-lg px-2.5 py-1.5 text-xs text-slate-300 focus:outline-none transition">
                    </td>
                    <td class="p-2 text-center">
                        <select data-hour="${i}" data-field="status" ${!isAuthorized ? 'disabled' : ''} onchange="updateData(${i}, 'status', this.value)"
                            class="bg-slate-900 border border-slate-800 rounded-lg px-2 py-1 text-xs text-slate-300 focus:outline-none focus:border-indigo-500">
                            <option value="libero">Libero</option>
                            <option value="occupato">Occupato</option>
                            <option value="importante">Importante</option>
                        </select>
                    </td>
                `;
                tbody.appendChild(tr);
            }
        }

        function getSelectedDateKey() {
            return document.getElementById('selectedDate').value || new Date().toISOString().split('T')[0];
        }

        function loadDayData() {
            const dateKey = getSelectedDateKey();
            if (db) {
                db.ref('days/' + dateKey).once('value').then((snapshot) => {
                    currentDayData = snapshot.val() || {};
                    populateTableFields();
                });
            } else {
                currentDayData = JSON.parse(localStorage.getItem('agenda_' + dateKey) || '{}');
                populateTableFields();
            }
        }

        // Ascolto in tempo reale se Firebase è disponibile
        let currentRef = null;
        function setupRealtimeSync() {
            const dateKey = getSelectedDateKey();
            if (currentRef) currentRef.off();
            if (db) {
                currentRef = db.ref('days/' + dateKey);
                currentRef.on('value', (snapshot) => {
                    currentDayData = snapshot.val() || {};
                    populateTableFields();
                });
            }
        }

        document.getElementById('selectedDate').addEventListener('change', () => {
            loadDayData();
            setupRealtimeSync();
        });

        function populateTableFields() {
            for (let i = 0; i < 24; i++) {
                const hourData = currentDayData[i] || { event: '', assignee: '', status: 'libero' };
                const eventInput = document.querySelector(`input[data-hour="${i}"][data-field="event"]`);
                const assigneeInput = document.querySelector(`input[data-hour="${i}"][data-field="assignee"]`);
                const statusSelect = document.querySelector(`select[data-hour="${i}"][data-field="status"]`);
                
                if (eventInput) eventInput.value = hourData.event || '';
                if (assigneeInput) assigneeInput.value = hourData.assignee || '';
                if (statusSelect) statusSelect.value = hourData.status || 'libero';
            }
        }

        function updateData(hour, field, value) {
            if (!isAuthorized) return;
            if (!currentDayData[hour]) {
                currentDayData[hour] = { event: '', assignee: '', status: 'libero' };
            }
            currentDayData[hour][field] = value;
            
            const dateKey = getSelectedDateKey();
            if (db) {
                db.ref('days/' + dateKey + '/' + hour).set(currentDayData[hour]);
            } else {
                localStorage.setItem('agenda_' + dateKey, JSON.stringify(currentDayData));
            }
        }

        function changeDay(days) {
            const dateInput = document.getElementById('selectedDate');
            let currentDate = new Date(dateInput.value);
            currentDate.setDate(currentDate.getDate() + days);
            dateInput.value = currentDate.toISOString().split('T')[0];
            loadDayData();
            setupRealtimeSync();
        }

        function goToday() {
            document.getElementById('selectedDate').value = new Date().toISOString().split('T')[0];
            loadDayData();
            setupRealtimeSync();
        }

        function toggleAuth() {
            if (isAuthorized) {
                isAuthorized = false;
                updateAuthUI();
                buildTableStructure();
                loadDayData();
            } else {
                document.getElementById('authModal').classList.remove('hidden');
                document.getElementById('pinInput').value = '';
                document.getElementById('pinInput').focus();
            }
        }

        function verifyPin() {
            const pin = document.getElementById('pinInput').value;
            if (pin === adminPin) {
                isAuthorized = true;
                closeAuthModal();
                updateAuthUI();
                buildTableStructure();
                loadDayData();
                setupRealtimeSync();
            } else {
                alert('PIN errato!');
            }
        }

        function closeAuthModal() {
            document.getElementById('authModal').classList.add('hidden');
        }

        function updateAuthUI() {
            const badge = document.getElementById('lockStatusBadge');
            const btn = document.getElementById('authBtn');
            if (isAuthorized) {
                badge.className = "px-2.5 py-1 rounded-full text-xs font-semibold bg-emerald-500/10 text-emerald-400 border border-emerald-500/20";
                badge.innerText = "🔓 Sbloccato (Modifica attiva)";
                btn.innerText = "Blocca";
                btn.className = "bg-rose-600 hover:bg-rose-500 text-white px-3.5 py-1.5 rounded-lg text-xs font-medium transition shadow-sm";
            } else {
                badge.className = "px-2.5 py-1 rounded-full text-xs font-semibold bg-amber-500/10 text-amber-400 border border-amber-500/20";
                badge.innerText = "🔒 Solo Lettura";
                btn.innerText = "Sblocca Modifica";
                btn.className = "bg-indigo-600 hover:bg-indigo-500 text-white px-3.5 py-1.5 rounded-lg text-xs font-medium transition shadow-sm";
            }
        }

        function openSettings() { document.getElementById('settingsModal').classList.remove('hidden'); }
        function closeSettings() { document.getElementById('settingsModal').classList.add('hidden'); }

        function saveNewPin() {
            const newPin = document.getElementById('newPinInput').value.trim();
            if (newPin.length > 0) {
                adminPin = newPin;
                localStorage.setItem('agenda_admin_pin', adminPin);
                alert('PIN aggiornato con successo!');
                closeSettings();
            } else {
                alert('Inserisci un PIN valido.');
            }
        }

        function exportData() {
            const dataStr = "data:text/json;charset=utf-8," + encodeURIComponent(JSON.stringify(currentDayData));
            const downloadAnchor = document.createElement('a');
            downloadAnchor.setAttribute("href", dataStr);
            downloadAnchor.setAttribute("download", "agenda_" + getSelectedDateKey() + ".json");
            document.body.appendChild(downloadAnchor);
            downloadAnchor.click();
            downloadAnchor.remove();
        }

        function filterEvents() {
            const query = document.getElementById('searchInput').value.toLowerCase();
            const rows = document.querySelectorAll('#hoursTableBody tr');
            rows.forEach(row => {
                const eventText = row.querySelector('input[data-field="event"]').value.toLowerCase();
                const assigneeText = row.querySelector('input[data-field="assignee"]').value.toLowerCase();
                if (eventText.includes(query) || assigneeText.includes(query)) {
                    row.style.display = '';
                } else {
                    row.style.display = 'none';
                }
            });
        }

        window.onload = () => {
            initApp();
            setupRealtimeSync();
        };
    </script>
</body>
</html>
