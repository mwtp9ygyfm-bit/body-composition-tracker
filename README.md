<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Body Composition & Calorie Tracker</title>
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
    </style>
</head>
<body>

<div class="app-container relative">
    
    <header class="bg-teal-600 text-white p-4 sticky top-0 z-50 shadow-md flex justify-between items-center">
        <div>
            <h1 class="text-xl font-bold tracking-wide">FitMetrics Pro</h1>
            <p class="text-xs text-teal-100">User: Benz | BMR: 1,661 kcal</p>
        </div>
        <button onclick="openModal('profileModal')" class="bg-teal-700 p-2 rounded-full text-sm hover:bg-teal-800 transition">
            <i class="fa-solid fa-user-gear"></i>
        </button>
    </header>

    <main class="p-4 space-y-4">

        <section id="tab-dashboard" class="tab-content active space-y-4">
            <div class="grid grid-cols-3 gap-3">
                <div class="bg-white p-3 rounded-2xl shadow-sm border border-slate-100 text-center">
                    <span class="text-xs text-slate-400 block font-medium">Weight</span>
                    <span class="text-lg font-bold text-slate-800" id="dash-weight">67.4</span>
                    <span class="text-[10px] text-teal-600 block font-semibold">kg (BMI 21.5)</span>
                </div>
                <div class="bg-white p-3 rounded-2xl shadow-sm border border-slate-100 text-center">
                    <span class="text-xs text-slate-400 block font-medium">Body Fat</span>
                    <span class="text-lg font-bold text-slate-800" id="dash-fat">11.3</span>
                    <span class="text-[10px] text-teal-600 block font-semibold">% (7.6 kg)</span>
                </div>
                <div class="bg-white p-3 rounded-2xl shadow-sm border border-slate-100 text-center">
                    <span class="text-xs text-slate-400 block font-medium">Muscle Mass</span>
                    <span class="text-lg font-bold text-slate-800" id="dash-muscle">55.7</span>
                    <span class="text-[10px] text-teal-600 block font-semibold">kg (High)</span>
                </div>
            </div>

            <div class="bg-gradient-to-br from-teal-500 to-emerald-600 text-white p-5 rounded-3xl shadow-md">
                <div class="flex justify-between items-center mb-3">
                    <h3 class="font-semibold text-sm uppercase tracking-wider text-teal-100">Daily Energy Balance</h3>
                    <span id="calorie-status-badge" class="bg-teal-700/60 text-xs px-2.5 py-1 rounded-full">Bulking (+2.7kg Goal)</span>
                </div>
                <div class="flex justify-between items-end mb-2">
                    <div>
                        <span class="text-3xl font-extrabold" id="consumed-calories">1,850</span>
                        <span class="text-xs text-teal-100">/ <span id="target-calories">2,300</span> kcal</span>
                    </div>
                    <div class="text-right">
                        <span class="text-xs text-teal-100 block">Remaining</span>
                        <span class="text-lg font-bold" id="remaining-calories">450 kcal</span>
                    </div>
                </div>
                <div class="w-full bg-teal-900/40 h-3 rounded-full overflow-hidden">
                    <div id="calorie-progress-bar" class="bg-white h-full rounded-full transition-all duration-500" style="width: 80%;"></div>
                </div>
            </div>

            <div class="bg-white p-4 rounded-3xl shadow-sm border border-slate-100">
                <div class="flex justify-between items-center mb-3">
                    <h3 class="font-bold text-slate-700 text-sm">Composition History</h3>
                    <button onclick="openModal('addDataModal')" class="text-xs bg-teal-50 text-teal-600 font-semibold px-3 py-1.5 rounded-full hover:bg-teal-100 transition">
                        <i class="fa-solid fa-plus mr-1"></i> Add Log
                    </button>
                </div>
                <div class="relative h-56">
                    <canvas id="progressChart"></canvas>
                </div>
            </div>

            <div class="bg-white p-4 rounded-3xl shadow-sm border border-slate-100">
                <h3 class="font-bold text-slate-700 text-sm mb-3">Fitdays Segmental Status</h3>
                <div class="grid grid-cols-2 gap-2 text-xs">
                    <div class="bg-slate-50 p-3 rounded-xl border border-slate-100">
                        <span class="text-slate-400 block font-medium">Left/Right Arms</span>
                        <span class="font-bold text-slate-700 text-sm">0.2 kg Fat <span class="text-emerald-600 font-normal">(Low)</span></span>
                    </div>
                    <div class="bg-slate-50 p-3 rounded-xl border border-slate-100">
                        <span class="text-slate-400 block font-medium">Trunk Fat</span>
                        <span class="font-bold text-slate-700 text-sm">4.5 kg <span class="text-teal-600 font-normal">(Standard)</span></span>
                    </div>
                    <div class="bg-slate-50 p-3 rounded-xl border border-slate-100">
                        <span class="text-slate-400 block font-medium">Lower Limbs Muscle</span>
                        <span class="font-bold text-slate-700 text-sm">9.8 kg <span class="text-blue-600 font-normal">(High)</span></span>
                    </div>
                    <div class="bg-slate-50 p-3 rounded-xl border border-slate-100">
                        <span class="text-slate-400 block font-medium">Visceral Fat</span>
                        <span class="font-bold text-slate-700 text-sm">1.0 <span class="text-teal-600 font-normal">(Standard)</span></span>
                    </div>
                </div>
            </div>
        </section>

        <section id="tab-scanner" class="tab-content space-y-4">
            <div class="bg-white p-5 rounded-3xl shadow-sm border border-slate-100 text-center space-y-3">
                <div class="w-16 h-16 bg-teal-50 text-teal-600 rounded-full flex items-center justify-center mx-auto text-2xl shadow-inner">
                    <i class="fa-solid fa-camera-retro"></i>
                </div>
                <h3 class="font-bold text-slate-800 text-base">AI Meal Estimator</h3>
                <p class="text-xs text-slate-500 leading-relaxed">Take a photo of your meal or upload an image from your iPhone library to estimate calories and macronutrients instantly.</p>
                
                <input type="file" id="mealImageInput" accept="image/*" class="hidden" onchange="handleImageUpload(event)">
                
                <div class="flex gap-2 pt-2">
                    <button onclick="document.getElementById('mealImageInput').click()" class="flex-1 bg-teal-600 text-white py-3 rounded-2xl font-semibold text-sm shadow-md hover:bg-teal-700 transition flex items-center justify-center gap-2">
                        <i class="fa-solid fa-upload"></i> Upload / Camera
                    </button>
                </div>
            </div>

            <div id="scannerResultContainer" class="hidden bg-white p-4 rounded-3xl shadow-sm border border-slate-100 space-y-3">
                <div class="relative rounded-2xl overflow-hidden h-48 bg-slate-100">
                    <img id="uploadedMealPreview" src="" alt="Meal Preview" class="w-full h-full object-cover">
                </div>
                <div id="loadingAnalysis" class="text-center py-4 text-xs text-slate-400 animate-pulse">
                    <i class="fa-solid fa-spinner fa-spin mr-1"></i> AI is analyzing food items and estimating nutrition...
                </div>
                <div id="analysisDetails" class="hidden space-y-3">
                    <div class="flex justify-between items-center border-b pb-2">
                        <span class="font-bold text-slate-700 text-sm" id="detectedFoodName">Grilled Chicken & Rice Bowl</span>
                        <span class="bg-teal-100 text-teal-800 text-xs font-bold px-2.5 py-1 rounded-full" id="detectedCalories">580 kcal</span>
                    </div>
                    <div class="grid grid-cols-3 gap-2 text-center text-xs">
                        <div class="bg-slate-50 p-2 rounded-xl">
                            <span class="text-slate-400 block">Protein</span>
                            <span class="font-bold text-slate-700" id="detectedProtein">45g</span>
                        </div>
                        <div class="bg-slate-50 p-2 rounded-xl">
                            <span class="text-slate-400 block">Carbs</span>
                            <span class="font-bold text-slate-700" id="detectedCarbs">52g</span>
                        </div>
                        <div class="bg-slate-50 p-2 rounded-xl">
                            <span class="text-slate-400 block">Fat</span>
                            <span class="font-bold text-slate-700" id="detectedFat">12g</span>
                        </div>
                    </div>
                    <button onclick="logMealToDiary()" class="w-full bg-slate-900 text-white py-3 rounded-2xl font-semibold text-sm hover:bg-slate-800 transition">
                        Add to Daily Intake
                    </button>
                </div>
            </div>
        </section>

        <section id="tab-goals" class="tab-content space-y-4">
            <div class="bg-white p-5 rounded-3xl shadow-sm border border-slate-100 space-y-4">
                <h3 class="font-bold text-slate-800 text-base">Body Transformation Targets</h3>
                
                <div class="space-y-3 text-xs">
                    <div>
                        <label class="font-medium text-slate-600 block mb-1">Target Weight Goal (kg)</label>
                        <input type="number" id="settingTargetWeight" value="70.1" step="0.1" class="w-full bg-slate-50 border border-slate-200 p-3 rounded-xl font-semibold text-slate-700">
                    </div>
                    <div>
                        <label class="font-medium text-slate-600 block mb-1">Daily Calorie Target (kcal)</label>
                        <input type="number" id="settingTargetCalories" value="2300" class="w-full bg-slate-50 border border-slate-200 p-3 rounded-xl font-semibold text-slate-700">
                    </div>
                    <div class="grid grid-cols-3 gap-2">
                        <div>
                            <label class="font-medium text-slate-600 block mb-1">Protein (g)</label>
                            <input type="number" id="settingProtein" value="150" class="w-full bg-slate-50 border border-slate-200 p-3 rounded-xl font-semibold text-slate-700">
                        </div>
                        <div>
                            <label class="font-medium text-slate-600 block mb-1">Carbs (g)</label>
                            <input type="number" id="settingCarbs" value="250" class="w-full bg-slate-50 border border-slate-200 p-3 rounded-xl font-semibold text-slate-700">
                        </div>
                        <div>
                            <label class="font-medium text-slate-600 block mb-1">Fat (g)</label>
                            <input type="number" id="settingFat" value="70" class="w-full bg-slate-50 border border-slate-200 p-3 rounded-xl font-semibold text-slate-700">
                        </div>
                    </div>
                </div>

                <button onclick="saveSettings()" class="w-full bg-teal-600 text-white py-3 rounded-2xl font-semibold text-sm shadow-md hover:bg-teal-700 transition">
                    Save Goals
                </button>
            </div>
        </section>

    </main>

    <nav class="fixed bottom-0 left-0 right-0 max-w-[480px] mx-auto bg-white border-t border-slate-100 flex justify-around py-3 z-40 shadow-lg">
        <button onclick="switchTab('dashboard')" id="nav-dashboard" class="flex flex-col items-center text-teal-600 transition">
            <i class="fa-solid fa-chart-pie text-lg"></i>
            <span class="text-[10px] font-semibold mt-1">Progress</span>
        </button>
        <button onclick="switchTab('scanner')" id="nav-scanner" class="flex flex-col items-center text-slate-400 transition">
            <i class="fa-solid fa-camera text-lg"></i>
            <span class="text-[10px] font-semibold mt-1">AI Scanner</span>
        </button>
        <button onclick="switchTab('goals')" id="nav-goals" class="flex flex-col items-center text-slate-400 transition">
            <i class="fa-solid fa-bullseye text-lg"></i>
            <span class="text-[10px] font-semibold mt-1">Goals</span>
        </button>
    </nav>

</div>

<div id="addDataModal" class="fixed inset-0 bg-black/50 z-50 hidden flex items-end sm:items-center justify-center p-0 sm:p-4">
    <div class="bg-white w-full max-w-md rounded-t-3xl sm:rounded-3xl p-5 space-y-4 animate-in fade-in slide-in-from-bottom duration-200">
        <div class="flex justify-between items-center border-b pb-3">
            <h3 class="font-bold text-slate-800 text-base">Add Fitdays Log</h3>
            <button onclick="closeModal('addDataModal')" class="text-slate-400 hover:text-slate-600"><i class="fa-solid fa-xmark text-lg"></i></button>
        </div>
        <div class="space-y-3 text-xs">
            <div>
                <label class="font-medium text-slate-600 block mb-1">Weight (kg)</label>
                <input type="number" id="newLogWeight" value="67.4" step="0.1" class="w-full bg-slate-50 border border-slate-200 p-3 rounded-xl font-semibold text-slate-700">
            </div>
            <div>
                <label class="font-medium text-slate-600 block mb-1">Body Fat (%)</label>
                <input type="number" id="newLogFat" value="11.3" step="0.1" class="w-full bg-slate-50 border border-slate-200 p-3 rounded-xl font-semibold text-slate-700">
            </div>
            <div>
                <label class="font-medium text-slate-600 block mb-1">Muscle Mass (kg)</label>
                <input type="number" id="newLogMuscle" value="55.7" step="0.1" class="w-full bg-slate-50 border border-slate-200 p-3 rounded-xl font-semibold text-slate-700">
            </div>
        </div>
        <button onclick="submitNewLog()" class="w-full bg-teal-600 text-white py-3 rounded-2xl font-semibold text-sm shadow-md hover:bg-teal-700 transition">
            Save Entry
        </button>
    </div>
</div>

<script>
    // App State Management (Stored locally in iPhone browser)
    let appData = JSON.parse(localStorage.getItem('fitMetricsBenz')) || {
        weight: 67.4,
        bodyFat: 11.3,
        muscleMass: 55.7,
        consumedCalories: 1850,
        targetCalories: 2300,
        history: [
            { date: 'Aug 25', weight: 67.0, fat: 11.6, muscle: 55.2 },
            { date: 'Aug 28', weight: 67.2, fat: 11.4, muscle: 55.5 },
            { date: 'Aug 31', weight: 67.4, fat: 11.3, muscle: 55.7 }
        ]
    };

    let progressChartInstance = null;

    // Initialize UI on load
    window.addEventListener('DOMContentLoaded', () => {
        updateDashboardUI();
        initChart();
    });

    // Tab Switching Logic
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
    }

    function openModal(modalId) {
        document.getElementById(modalId).classList.remove('hidden');
    }

    function closeModal(modalId) {
        document.getElementById(modalId).classList.add('hidden');
    }

    // Update Dashboard Metrics
    function updateDashboardUI() {
        document.getElementById('dash-weight').innerText = appData.weight;
        document.getElementById('dash-fat').innerText = appData.bodyFat;
        document.getElementById('dash-muscle').innerText = appData.muscleMass;
        
        document.getElementById('consumed-calories').innerText = appData.consumedCalories.toLocaleString();
        document.getElementById('target-calories').innerText = appData.targetCalories.toLocaleString();
        
        let remaining = appData.targetCalories - appData.consumedCalories;
        document.getElementById('remaining-calories').innerText = `${Math.abs(remaining)} kcal ${remaining >= 0 ? 'left' : 'over'}`;
        
        let percentage = Math.min(100, Math.round((appData.consumedCalories / appData.targetCalories) * 100));
        document.getElementById('calorie-progress-bar').style.width = `${percentage}%`;
    }

    // Initialize Chart.js Lively Graphics
    function initChart() {
        const ctx = document.getElementById('progressChart').getContext('2d');
        const labels = appData.history.map(item => item.date);
        const weights = appData.history.map(item => item.weight);
        const muscles = appData.history.map(item => item.muscle);

        progressChartInstance = new Chart(ctx, {
            type: 'line',
            data: {
                labels: labels,
                datasets: [
                    {
                        label: 'Weight (kg)',
                        data: weights,
                        borderColor: '#0d9488',
                        backgroundColor: 'rgba(13, 148, 136, 0.1)',
                        borderWidth: 3,
                        tension: 0.3,
                        fill: true
                    },
                    {
                        label: 'Muscle (kg)',
                        data: muscles,
                        borderColor: '#3b82f6',
                        borderWidth: 2,
                        borderDash: [4, 4],
                        tension: 0.3
                    }
                ]
            },
            options: {
                responsive: true,
                maintainAspectRatio: false,
                plugins: {
                    legend: { position: 'bottom', labels: { boxWidth: 12, font: { size: 11 } } }
                },
                scales: {
                    y: { grid: { color: '#f1f5f9' }, ticks: { font: { size: 10 } } },
                    x: { grid: { display: false }, ticks: { font: { size: 10 } } }
                }
            }
        });
    }

    function refreshChart() {
        if (!progressChartInstance) return;
        progressChartInstance.data.labels = appData.history.map(item => item.date);
        progressChartInstance.data.datasets[0].data = appData.history.map(item => item.weight);
        progressChartInstance.data.datasets[1].data = appData.history.map(item => item.muscle);
        progressChartInstance.update();
    }

    // Handle New Fitdays Log Submission
    function submitNewLog() {
        const w = parseFloat(document.getElementById('newLogWeight').value);
        const f = parseFloat(document.getElementById('newLogFat').value);
        const m = parseFloat(document.getElementById('newLogMuscle').value);

        appData.weight = w;
        appData.bodyFat = f;
        appData.muscleMass = m;

        const todayStr = 'Sep ' + new Date().getDate();
        appData.history.push({ date: todayStr, weight: w, fat: f, muscle: m });

        localStorage.setItem('fitMetricsBenz', JSON.stringify(appData));
        updateDashboardUI();
        refreshChart();
        closeModal('addDataModal');
    }

    // Save Goals
    function saveSettings() {
        appData.targetCalories = parseInt(document.getElementById('settingTargetCalories').value);
        localStorage.setItem('fitMetricsBenz', JSON.stringify(appData));
        updateDashboardUI();
        alert('Goals saved successfully!');
        switchTab('dashboard');
    }

    // Mock AI Image Upload and Nutrition Estimation
    function handleImageUpload(event) {
        const file = event.target.files[0];
        if (!file) return;

        const reader = new FileReader();
        reader.onload = function(e) {
            document.getElementById('uploadedMealPreview').src = e.target.result;
            document.getElementById('scannerResultContainer').classList.remove('hidden');
            document.getElementById('loadingAnalysis').classList.remove('hidden');
            document.getElementById('analysisDetails').classList.add('hidden');

            // Simulate AI inference delay
            setTimeout(() => {
                document.getElementById('loadingAnalysis').classList.add('hidden');
                document.getElementById('analysisDetails').classList.remove('hidden');
                
                // Randomized realistic values for demo interaction
                const mockMeals = [
                    { name: 'Teriyaki Chicken Rice Bowl', cal: 620, p: 48, c: 65, f: 14 },
                    { name: 'Salmon Avocado Quinoa Salad', cal: 510, p: 36, c: 32, f: 22 },
                    { name: 'Beef Steak with Sweet Potato', cal: 680, p: 54, c: 42, f: 18 }
                ];
                const selected = mockMeals[Math.floor(Math.random() * mockMeals.length)];
                
                document.getElementById('detectedFoodName').innerText = selected.name;
                document.getElementById('detectedCalories').innerText = `${selected.cal} kcal`;
                document.getElementById('detectedProtein').innerText = `${selected.p}g`;
                document.getElementById('detectedCarbs').innerText = `${selected.c}g`;
                document.getElementById('detectedFat').innerText = `${selected.f}g`;
                
                window.lastScannedMealCalories = selected.cal;
            }, 1500);
        };
        reader.readAsDataURL(file);
    }

    function logMealToDiary() {
        appData.consumedCalories += (window.lastScannedMealCalories || 500);
        localStorage.setItem('fitMetricsBenz', JSON.stringify(appData));
        updateDashboardUI();
        switchTab('dashboard');
        alert('Meal logged successfully to your daily total!');
    }
</script>

</body>
</html>
