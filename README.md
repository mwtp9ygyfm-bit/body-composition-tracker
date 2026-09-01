<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>FitMetrics Pro</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        body {
            background-color: #f3f4f6;
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
            -webkit-tap-highlight-color: transparent;
        }
        .app-container {
            max-width: 480px;
            margin: 0 auto;
            min-height: 100vh;
            background: #f8fafc;
            box-shadow: 0 4px 20px rgba(0,0,0,0.05);
            padding-bottom: 90px;
        }
        .tab-content { display: none; }
        .tab-content.active { display: block; }
        /* Remove arrows from number inputs for a cleaner look */
        input[type=number]::-webkit-inner-spin-button, 
        input[type=number]::-webkit-outer-spin-button { 
            -webkit-appearance: none; 
            margin: 0; 
        }
    </style>
</head>
<body>

<div class="app-container relative">
    
    <header class="bg-teal-600 text-white p-4 sticky top-0 z-50 shadow-md flex justify-between items-center">
        <div>
            <h1 class="text-xl font-bold tracking-wide">FitMetrics Pro</h1>
            <p class="text-xs text-teal-100">User: Benz | Body Recomposition</p>
        </div>
        <div class="bg-teal-700 px-3 py-1 rounded-full text-xs font-bold border border-teal-500 shadow-inner">
            <span id="header-date"></span>
        </div>
    </header>

    <main class="p-4 space-y-4">

        <section id="tab-dashboard" class="tab-content active space-y-4">
            
            <div class="bg-gradient-to-br from-slate-800 to-slate-900 text-white p-5 rounded-3xl shadow-md">
                <div class="flex justify-between items-center mb-3">
                    <h3 class="font-semibold text-sm tracking-wider text-slate-300">Daily Energy Balance</h3>
                    <button onclick="switchTab('log')" class="bg-teal-500 hover:bg-teal-400 text-white text-xs px-3 py-1 rounded-full font-bold transition">
                        <i class="fa-solid fa-plus mr-1"></i> Log Food
                    </button>
                </div>
                <div class="flex justify-between items-end mb-2">
                    <div>
                        <span class="text-4xl font-extrabold text-teal-400" id="consumed-calories">0</span>
                        <span class="text-sm text-slate-400">/ <span id="target-calories">2300</span> kcal</span>
                    </div>
                    <div class="text-right">
                        <span class="text-xs text-slate-400 block mb-1">Remaining</span>
                        <span class="text-lg font-bold bg-slate-700 px-2 py-1 rounded-lg" id="remaining-calories">2300</span>
                    </div>
                </div>
                <div class="w-full bg-slate-700 h-2 rounded-full overflow-hidden mt-3">
                    <div id="calorie-progress-bar" class="bg-teal-400 h-full rounded-full transition-all duration-500" style="width: 0%;"></div>
                </div>
            </div>

            <div class="grid grid-cols-3 gap-3">
                <div class="bg-white p-4 rounded-2xl shadow-sm border border-slate-100 text-center">
                    <span class="text-xs text-slate-400 block font-medium mb-1">Weight</span>
                    <span class="text-xl font-bold text-slate-800" id="dash-weight">--</span>
                    <span class="text-[10px] text-teal-600 block font-bold mt-1">kg</span>
                </div>
                <div class="bg-white p-4 rounded-2xl shadow-sm border border-slate-100 text-center">
                    <span class="text-xs text-slate-400 block font-medium mb-1">Body Fat</span>
                    <span class="text-xl font-bold text-slate-800" id="dash-fat">--</span>
                    <span class="text-[10px] text-teal-600 block font-bold mt-1">%</span>
                </div>
                <div class="bg-white p-4 rounded-2xl shadow-sm border border-slate-100 text-center">
                    <span class="text-xs text-slate-400 block font-medium mb-1">Muscle</span>
                    <span class="text-xl font-bold text-slate-800" id="dash-muscle">--</span>
                    <span class="text-[10px] text-teal-600 block font-bold mt-1">kg</span>
                </div>
            </div>

            <div class="bg-white p-4 rounded-3xl shadow-sm border border-slate-100">
                <div class="flex justify-between items-center mb-3">
                    <h3 class="font-bold text-slate-700 text-sm">Composition History</h3>
                </div>
                <div class="relative h-56">
                    <canvas id="progressChart"></canvas>
                </div>
            </div>
        </section>

        <section id="tab-log" class="tab-content space-y-5">
            
            <div class="bg-white p-5 rounded-3xl shadow-sm border border-slate-100 space-y-4">
                <div class="flex items-center gap-2 border-b border-slate-100 pb-2">
                    <div class="w-8 h-8 bg-teal-100 text-teal-600 rounded-full flex items-center justify-center text-sm">
                        <i class="fa-solid fa-fire"></i>
                    </div>
                    <h3 class="font-bold text-slate-800 text-base">Log Calories</h3>
                </div>
                
                <div class="grid grid-cols-4 gap-2">
                    <button onclick="quickAddCalories(100)" class="bg-slate-50 border border-slate-200 text-slate-600 py-3 rounded-xl font-bold text-sm hover:bg-teal-50 hover:text-teal-600 hover:border-teal-200 active:scale-95 transition">+100</button>
                    <button onclick="quickAddCalories(300)" class="bg-slate-50 border border-slate-200 text-slate-600 py-3 rounded-xl font-bold text-sm hover:bg-teal-50 hover:text-teal-600 hover:border-teal-200 active:scale-95 transition">+300</button>
                    <button onclick="quickAddCalories(500)" class="bg-slate-50 border border-slate-200 text-slate-600 py-3 rounded-xl font-bold text-sm hover:bg-teal-50 hover:text-teal-600 hover:border-teal-200 active:scale-95 transition">+500</button>
                    <button onclick="quickAddCalories(800)" class="bg-slate-50 border border-slate-200 text-slate-600 py-3 rounded-xl font-bold text-sm hover:bg-teal-50 hover:text-teal-600 hover:border-teal-200 active:scale-95 transition">+800</button>
                </div>

                <div class="flex gap-2">
                    <input type="number" id="customCalorieInput" inputmode="numeric" pattern="[0-9]*" placeholder="Custom kcal..." class="flex-1 bg-slate-50 border border-slate-200 p-3 rounded-xl font-bold text-slate-800 focus:outline-none focus:ring-2 focus:ring-teal-500 focus:border-transparent">
                    <button onclick="addCustomCalories()" class="bg-slate-800 text-white px-5 rounded-xl font-bold text-sm hover:bg-slate-700 transition">
                        Add
                    </button>
                </div>
            </div>

            <div class="bg-white p-5 rounded-3xl shadow-sm border border-slate-100 space-y-4">
                <div class="flex items-center justify-between border-b border-slate-100 pb-2">
                    <div class="flex items-center gap-2">
                        <div class="w-8 h-8 bg-blue-100 text-blue-600 rounded-full flex items-center justify-center text-sm">
                            <i class="fa-solid fa-weight-scale"></i>
                        </div>
                        <h3 class="font-bold text-slate-800 text-base">Fitdays Metrics</h3>
                    </div>
                    <span class="text-[10px] bg-slate-100 text-slate-500 px-2 py-1 rounded-full font-bold">Auto-filled with latest</span>
                </div>
                
                <div class="grid grid-cols-3 gap-3">
                    <div>
                        <label class="font-semibold text-slate-500 text-xs block mb-1">Weight (kg)</label>
                        <input type="number" id="logWeight" step="0.1" inputmode="decimal" class="w-full bg-slate-50 border border-slate-200 p-3 rounded-xl font-bold text-slate-800 text-center text-lg focus:outline-none focus:ring-2 focus:ring-teal-500">
                    </div>
                    <div>
                        <label class="font-semibold text-slate-500 text-xs block mb-1">Fat (%)</label>
                        <input type="number" id="logFat" step="0.1" inputmode="decimal" class="w-full bg-slate-50 border border-slate-200 p-3 rounded-xl font-bold text-slate-800 text-center text-lg focus:outline-none focus:ring-2 focus:ring-teal-500">
                    </div>
                    <div>
                        <label class="font-semibold text-slate-500 text-xs block mb-1">Muscle (kg)</label>
                        <input type="number" id="logMuscle" step="0.1" inputmode="decimal" class="w-full bg-slate-50 border border-slate-200 p-3 rounded-xl font-bold text-slate-800 text-center text-lg focus:outline-none focus:ring-2 focus:ring-teal-500">
                    </div>
                </div>

                <div>
                    <label class="font-semibold text-slate-500 text-xs block mb-1">Date</label>
                    <input type="date" id="logDate" class="w-full bg-slate-50 border border-slate-200 p-3 rounded-xl font-bold text-slate-800 focus:outline-none focus:ring-2 focus:ring-teal-500">
                </div>

                <button onclick="saveFitdaysData()" class="w-full bg-teal-600 text-white py-3.5 rounded-2xl font-bold text-sm shadow-md hover:bg-teal-700 active:scale-95 transition">
                    Save Progress
                </button>
            </div>
        </section>

        <section id="tab-goals" class="tab-content space-y-4">
            <div class="bg-white p-5 rounded-3xl shadow-sm border border-slate-100 space-y-4">
                <h3 class="font-bold text-slate-800 text-base">Goals</h3>
                <div class="space-y-3 text-sm">
                    <div>
                        <label class="font-semibold text-slate-600 block mb-1">Target Weight (kg)</label>
                        <input type="number" id="settingTargetWeight" step="0.1" inputmode="decimal" class="w-full bg-slate-50 border border-slate-200 p-3 rounded-xl font-bold text-slate-700">
                    </div>
                    <div>
                        <label class="font-semibold text-slate-600 block mb-1">Daily Calorie Target (kcal)</label>
                        <input type="number" id="settingTargetCalories" inputmode="numeric" pattern="[0-9]*" class="w-full bg-slate-50 border border-slate-200 p-3 rounded-xl font-bold text-slate-700">
                    </div>
                </div>
                <button onclick="saveSettings()" class="w-full bg-slate-800 text-white py-3 rounded-2xl font-bold text-sm shadow-md hover:bg-slate-700 transition">
                    Save Goals
                </button>
            </div>
            
            <div class="bg-white p-4 rounded-3xl border border-slate-100 text-center">
                <button onclick="resetDailyCalories()" class="text-amber-600 text-sm font-semibold w-full mb-3 pb-3 border-b border-slate-100">
                    <i class="fa-solid fa-rotate-left mr-1"></i> Reset Today's Calories to 0
                </button>
                <button onclick="clearAllData()" class="text-red-600 text-sm font-semibold w-full">
                    <i class="fa-solid fa-trash mr-1"></i> Delete All History
                </button>
            </div>
        </section>

    </main>

    <nav class="fixed bottom-0 left-0 right-0 max-w-[480px] mx-auto bg-white border-t border-slate-100 flex justify-around py-2 z-40 shadow-[0_-4px_15px_-3px_rgba(0,0,0,0.05)]">
        <button onclick="switchTab('dashboard')" id="nav-dashboard" class="flex flex-col items-center text-teal-600 transition w-1/3 py-2">
            <i class="fa-solid fa-chart-line text-xl mb-1"></i>
            <span class="text-[10px] font-bold">Progress</span>
        </button>
        <button onclick="switchTab('log')" id="nav-log" class="flex flex-col items-center text-slate-400 transition w-1/3 py-2 relative">
            <div class="absolute -top-6 bg-teal-500 text-white w-12 h-12 rounded-full flex items-center justify-center shadow-lg border-4 border-white">
                <i class="fa-solid fa-plus text-xl"></i>
            </div>
            <span class="text-[10px] font-bold mt-6">Log Data</span>
        </button>
        <button onclick="switchTab('goals')" id="nav-goals" class="flex flex-col items-center text-slate-400 transition w-1/3 py-2">
            <i class="fa-solid fa-bullseye text-xl mb-1"></i>
            <span class="text-[10px] font-bold">Goals</span>
        </button>
    </nav>

</div>

<script>
    // --- APP STATE MANAGEMENT ---
    const defaultData = {
        targetCalories: 2300,
        consumedCalories: 0,
        targetWeight: 70.1,
        history: [
            { date: '2026-08-31', weight: 67.4, fat: 11.3, muscle: 55.7 }
        ]
    };

    let appData = JSON.parse(localStorage.getItem('fitMetricsFast')) || defaultData;
    let progressChartInstance = null;

    // --- INITIALIZATION ---
    window.addEventListener('DOMContentLoaded', () => {
        // Set Header Date
        const today = new Date();
        document.getElementById('header-date').innerText = today.toLocaleDateString('en-US', { month: 'short', day: 'numeric' });
        
        // Setup Date Input
        document.getElementById('logDate').valueAsDate = today;

        // Load Settings
        document.getElementById('settingTargetWeight').value = appData.targetWeight;
        document.getElementById('settingTargetCalories').value = appData.targetCalories;
        
        updateDashboardUI();
        prefillLogData();
        initChart();
    });

    // --- NAVIGATION ---
    function switchTab(tabId) {
        document.querySelectorAll('.tab-content').forEach(el => el.classList.remove('active'));
        document.querySelectorAll('nav button').forEach(btn => {
            btn.classList.remove('text-teal-600');
            btn.classList.add('text-slate-400');
        });

        document.getElementById(`tab-${tabId}`).classList.add('active');
        const activeNavBtn = document.getElementById(`nav-${tabId}`);
        activeNavBtn.classList.remove('text-slate-400');
        activeNavBtn.classList.add('text-teal-600');
        
        // Refresh prefill when opening log tab
        if(tabId === 'log') {
            prefillLogData();
            // Reset custom calorie input
            document.getElementById('customCalorieInput').value = "";
        }
    }

    // --- DASHBOARD UI UPDATES ---
    function updateDashboardUI() {
        // Body Comp
        if (appData.history.length > 0) {
            appData.history.sort((a, b) => new Date(a.date) - new Date(b.date));
            const latest = appData.history[appData.history.length - 1];
            document.getElementById('dash-weight').innerText = latest.weight.toFixed(1);
            document.getElementById('dash-fat').innerText = latest.fat.toFixed(1);
            document.getElementById('dash-muscle').innerText = latest.muscle.toFixed(1);
        } else {
            document.getElementById('dash-weight').innerText = "--";
            document.getElementById('dash-fat').innerText = "--";
            document.getElementById('dash-muscle').innerText = "--";
        }

        // Calories
        document.getElementById('consumed-calories').innerText = appData.consumedCalories.toLocaleString();
        document.getElementById('target-calories').innerText = appData.targetCalories.toLocaleString();
        
        let remaining = appData.targetCalories - appData.consumedCalories;
        document.getElementById('remaining-calories').innerText = remaining;
        
        let percentage = Math.min(100, Math.round((appData.consumedCalories / appData.targetCalories) * 100));
        document.getElementById('calorie-progress-bar').style.width = `${percentage}%`;
    }

    // --- FAST LOGGING LOGIC ---
    function prefillLogData() {
        if (appData.history.length > 0) {
            const latest = appData.history[appData.history.length - 1];
            document.getElementById('logWeight').value = latest.weight;
            document.getElementById('logFat').value = latest.fat;
            document.getElementById('logMuscle').value = latest.muscle;
        }
    }

    function quickAddCalories(amount) {
        appData.consumedCalories += amount;
        saveToLocal();
        updateDashboardUI();
        alert(`Added ${amount} kcal!`);
        switchTab('dashboard');
    }

    function addCustomCalories() {
        const input = document.getElementById('customCalorieInput');
        const amount = parseInt(input.value);
        if (amount && amount > 0) {
            appData.consumedCalories += amount;
            saveToLocal();
            updateDashboardUI();
            alert(`Added ${amount} kcal!`);
            input.value = "";
            switchTab('dashboard');
        } else {
            alert("Please enter a valid number of calories.");
        }
    }

    function saveFitdaysData() {
        const w = parseFloat(document.getElementById('logWeight').value);
        const f = parseFloat(document.getElementById('logFat').value);
        const m = parseFloat(document.getElementById('logMuscle').value);
        const d = document.getElementById('logDate').value;

        if (!w || !f || !m || !d) {
            alert("Please fill in all body metrics.");
            return;
        }

        const existingIndex = appData.history.findIndex(item => item.date === d);
        if (existingIndex >= 0) {
            appData.history[existingIndex] = { date: d, weight: w, fat: f, muscle: m };
        } else {
            appData.history.push({ date: d, weight: w, fat: f, muscle: m });
        }

        saveToLocal();
        updateDashboardUI();
        refreshChart();
        alert("Progress saved!");
        switchTab('dashboard');
    }

    // --- CHART.JS ---
    function initChart() {
        const ctx = document.getElementById('progressChart').getContext('2d');
        progressChartInstance = new Chart(ctx, {
            type: 'line',
            data: getChartData(),
            options: {
                responsive: true,
                maintainAspectRatio: false,
                plugins: { legend: { position: 'bottom', labels: { font: { size: 10 } } } },
                scales: {
                    y: { type: 'linear', position: 'left', grid: { color: '#f1f5f9' } },
                    y1: { type: 'linear', position: 'right', grid: { drawOnChartArea: false } },
                    x: { grid: { display: false } }
                }
            }
        });
    }

    function getChartData() {
        const sorted = [...appData.history].sort((a, b) => new Date(a.date) - new Date(b.date));
        return {
            labels: sorted.map(item => {
                const d = new Date(item.date);
                return `${d.getMonth()+1}/${d.getDate()}`;
            }),
            datasets: [
                { label: 'Weight (kg)', data: sorted.map(i => i.weight), borderColor: '#0f766e', borderWidth: 2, yAxisID: 'y', tension: 0.3 },
                { label: 'Muscle (kg)', data: sorted.map(i => i.muscle), borderColor: '#3b82f6', borderWidth: 2, borderDash: [5,5], yAxisID: 'y', tension: 0.3 },
                { label: 'Fat (%)', data: sorted.map(i => i.fat), borderColor: '#f59e0b', borderWidth: 2, yAxisID: 'y1', tension: 0.3 }
            ]
        };
    }

    function refreshChart() {
        if (!progressChartInstance) return;
        progressChartInstance.data = getChartData();
        progressChartInstance.update();
    }

    // --- DATA MANAGEMENT ---
    function saveSettings() {
        appData.targetWeight = parseFloat(document.getElementById('settingTargetWeight').value);
        appData.targetCalories = parseInt(document.getElementById('settingTargetCalories').value);
        saveToLocal();
        updateDashboardUI();
        alert('Goals saved!');
    }

    function resetDailyCalories() {
        if(confirm("Reset today's calories to zero?")) {
            appData.consumedCalories = 0;
            saveToLocal();
            updateDashboardUI();
        }
    }

    function clearAllData() {
        if(confirm("Delete all history? This cannot be undone.")) {
            appData.history = [];
            appData.consumedCalories = 0;
            saveToLocal();
            updateDashboardUI();
            refreshChart();
        }
    }

    function saveToLocal() {
        localStorage.setItem('fitMetricsFast', JSON.stringify(appData));
    }
</script>

</body>
</html>
