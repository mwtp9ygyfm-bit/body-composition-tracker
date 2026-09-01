<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>FitMetrics Pro</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/tesseract.js@4/dist/tesseract.min.js"></script>
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
        <button onclick="switchTab('goals')" class="bg-teal-700 p-2 rounded-full text-sm hover:bg-teal-800 transition">
            <i class="fa-solid fa-user-gear"></i>
        </button>
    </header>

    <main class="p-4 space-y-4">

        <section id="tab-dashboard" class="tab-content active space-y-4">
            
            <div class="bg-gradient-to-br from-teal-500 to-emerald-600 text-white p-5 rounded-3xl shadow-md">
                <div class="flex justify-between items-center mb-3">
                    <h3 class="font-semibold text-sm uppercase tracking-wider text-teal-100">Daily Energy</h3>
                    <span class="bg-teal-700/60 text-xs px-2.5 py-1 rounded-full">Bulking (+2.7kg Goal)</span>
                </div>
                <div class="flex justify-between items-end mb-2">
                    <div>
                        <span class="text-3xl font-extrabold" id="consumed-calories">0</span>
                        <span class="text-xs text-teal-100">/ <span id="target-calories">2300</span> kcal</span>
                    </div>
                    <div class="text-right">
                        <span class="text-xs text-teal-100 block">Remaining</span>
                        <span class="text-lg font-bold" id="remaining-calories">2300 kcal</span>
                    </div>
                </div>
                <div class="w-full bg-teal-900/40 h-3 rounded-full overflow-hidden">
                    <div id="calorie-progress-bar" class="bg-white h-full rounded-full transition-all duration-500" style="width: 0%;"></div>
                </div>
            </div>

            <div class="grid grid-cols-3 gap-3">
                <div class="bg-white p-3 rounded-2xl shadow-sm border border-slate-100 text-center">
                    <span class="text-xs text-slate-400 block font-medium">Weight</span>
                    <span class="text-lg font-bold text-slate-800" id="dash-weight">--</span>
                    <span class="text-[10px] text-teal-600 block font-semibold">kg</span>
                </div>
                <div class="bg-white p-3 rounded-2xl shadow-sm border border-slate-100 text-center">
                    <span class="text-xs text-slate-400 block font-medium">Body Fat</span>
                    <span class="text-lg font-bold text-slate-800" id="dash-fat">--</span>
                    <span class="text-[10px] text-teal-600 block font-semibold">%</span>
                </div>
                <div class="bg-white p-3 rounded-2xl shadow-sm border border-slate-100 text-center">
                    <span class="text-xs text-slate-400 block font-medium">Muscle Mass</span>
                    <span class="text-lg font-bold text-slate-800" id="dash-muscle">--</span>
                    <span class="text-[10px] text-teal-600 block font-semibold">kg</span>
                </div>
            </div>

            <div class="bg-white p-4 rounded-3xl shadow-sm border border-slate-100">
                <h3 class="font-bold text-slate-700 text-sm mb-3">Composition Progress</h3>
                <div class="relative h-56">
                    <canvas id="progressChart"></canvas>
                </div>
            </div>
        </section>

        <section id="tab-food" class="tab-content space-y-4">
            <div class="bg-white p-5 rounded-3xl shadow-sm border border-slate-100 text-center space-y-3">
                <div class="w-16 h-16 bg-teal-50 text-teal-600 rounded-full flex items-center justify-center mx-auto text-2xl shadow-inner">
                    <i class="fa-solid fa-camera-retro"></i>
                </div>
                <h3 class="font-bold text-slate-800 text-base">Food AI Estimator</h3>
                <p class="text-xs text-slate-500 leading-relaxed">Upload a photo of your meal to estimate calories and macros.</p>
                
                <input type="file" id="mealImageInput" accept="image/*" class="hidden" onchange="handleMealUpload(event)">
                
                <button onclick="document.getElementById('mealImageInput').click()" class="w-full bg-teal-600 text-white py-3 rounded-2xl font-semibold text-sm shadow-md hover:bg-teal-700 transition flex items-center justify-center gap-2">
                    <i class="fa-solid fa-upload"></i> Take / Upload Meal Photo
                </button>
            </div>

            <div id="mealResultContainer" class="hidden bg-white p-4 rounded-3xl shadow-sm border border-slate-100 space-y-3">
                <div class="relative rounded-2xl overflow-hidden h-48 bg-slate-100 flex items-center justify-center">
                    <img id="uploadedMealPreview" src="" alt="Meal Preview" class="object-cover w-full h-full">
                </div>
                <div id="mealLoading" class="text-center py-4 text-xs text-slate-400 animate-pulse">
                    <i class="fa-solid fa-spinner fa-spin mr-1"></i> AI is analyzing food items...
                </div>
                <div id="mealDetails" class="hidden space-y-3">
                    <div class="flex justify-between items-center border-b pb-2">
                        <span class="font-bold text-slate-700 text-sm" id="detectedFoodName">--</span>
                        <span class="bg-teal-100 text-teal-800 text-xs font-bold px-2.5 py-1 rounded-full" id="detectedCalories">-- kcal</span>
                    </div>
                    <button onclick="logMealToDiary()" class="w-full bg-slate-900 text-white py-3 rounded-2xl font-semibold text-sm hover:bg-slate-800 transition">
                        Add to Daily Intake
                    </button>
                </div>
            </div>
        </section>

        <section id="tab-fitdays" class="tab-content space-y-4">
            <div class="bg-white p-5 rounded-3xl shadow-sm border border-slate-100 text-center space-y-3">
                <div class="w-16 h-16 bg-blue-50 text-blue-600 rounded-full flex items-center justify-center mx-auto text-2xl shadow-inner">
                    <i class="fa-solid fa-mobile-screen"></i>
                </div>
                <h3 class="font-bold text-slate-800 text-base">Fitdays Data Scanner</h3>
                <p class="text-xs text-slate-500 leading-relaxed">Upload a screenshot of your Fitdays app to extract your body metrics.</p>
                
                <input type="file" id="fitdaysImageInput" accept="image/*" class="hidden" onchange="handleFitdaysUpload(event)">
                
                <button onclick="document.getElementById('fitdaysImageInput').click()" class="w-full bg-blue-600 text-white py-3 rounded-2xl font-semibold text-sm shadow-md hover:bg-blue-700 transition flex items-center justify-center gap-2">
                    <i class="fa-solid fa-image"></i> Upload Fitdays Screenshot
                </button>
            </div>

            <div id="ocrLoadingContainer" class="hidden bg-white p-4 rounded-3xl shadow-sm border border-slate-100 space-y-3">
                <div class="relative rounded-2xl overflow-hidden h-40 bg-slate-100 flex items-center justify-center">
                    <img id="uploadedFitdaysPreview" src="" alt="Preview" class="max-h-full object-contain">
                </div>
                <div id="ocrStatus" class="text-center py-2 text-xs text-slate-500 font-medium">
                    <i class="fa-solid fa-spinner fa-spin mr-1"></i> Reading image data...
                </div>
                <div class="w-full bg-slate-100 h-2 rounded-full overflow-hidden">
                    <div id="ocrProgressBar" class="bg-blue-500 h-full rounded-full transition-all duration-200" style="width: 0%;"></div>
                </div>
            </div>

            <div id="ocrResultsContainer" class="hidden bg-white p-5 rounded-3xl shadow-sm border border-slate-100 space-y-4">
                <div class="flex justify-between items-center border-b border-slate-100 pb-2">
                    <h3 class="font-bold text-slate-800 text-sm">Verify Data</h3>
                    <span class="text-[10px] bg-amber-100 text-amber-700 px-2 py-1 rounded-full font-bold">Review</span>
                </div>
                
                <div class="space-y-3 text-sm">
                    <div>
                        <label class="font-semibold text-slate-600 block mb-1">Weight (kg)</label>
                        <input type="number" id="scannedWeight" step="0.1" class="w-full bg-slate-50 border border-slate-200 p-3 rounded-xl font-bold text-slate-800">
                    </div>
                    <div>
                        <label class="font-semibold text-slate-600 block mb-1">Body Fat (%)</label>
                        <input type="number" id="scannedFat" step="0.1" class="w-full bg-slate-50 border border-slate-200 p-3 rounded-xl font-bold text-slate-800">
                    </div>
                    <div>
                        <label class="font-semibold text-slate-600 block mb-1">Muscle Mass (kg)</label>
                        <input type="number" id="scannedMuscle" step="0.1" class="w-full bg-slate-50 border border-slate-200 p-3 rounded-xl font-bold text-slate-800">
                    </div>
                    <div>
                        <label class="font-semibold text-slate-600 block mb-1">Date</label>
                        <input type="date" id="scannedDate" class="w-full bg-slate-50 border border-slate-200 p-3 rounded-xl font-bold text-slate-800">
                    </div>
                </div>

                <button onclick="saveScannedData()" class="w-full bg-slate-900 text-white py-3 rounded-2xl font-semibold text-sm shadow-md hover:bg-slate-800 transition">
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
                        <input type="number" id="settingTargetWeight" step="0.1" class="w-full bg-slate-50 border border-slate-200 p-3 rounded-xl font-bold text-slate-700">
                    </div>
                    <div>
                        <label class="font-semibold text-slate-600 block mb-1">Daily Calorie Target (kcal)</label>
                        <input type="number" id="settingTargetCalories" class="w-full bg-slate-50 border border-slate-200 p-3 rounded-xl font-bold text-slate-700">
                    </div>
                </div>
                <button onclick="saveSettings()" class="w-full bg-teal-600 text-white py-3 rounded-2xl font-semibold text-sm shadow-md hover:bg-teal-700 transition">
                    Save Goals
                </button>
            </div>
            
            <div class="bg-white p-4 rounded-3xl border border-slate-100 text-center">
                <button onclick="resetDailyCalories()" class="text-amber-600 text-sm font-semibold w-full mb-3 pb-3 border-b border-slate-100">
                    <i class="fa-solid fa-rotate-left mr-1"></i> Reset Today's Calories
                </button>
                <button onclick="clearAllData()" class="text-red-600 text-sm font-semibold w-full">
                    <i class="fa-solid fa-trash mr-1"></i> Delete All History
                </button>
            </div>
        </section>

    </main>

    <nav class="fixed bottom-0 left-0 right-0 max-w-[480px] mx-auto bg-white border-t border-slate-100 flex justify-around py-3 z-40 shadow-[0_-4px_6px_-1px_rgba(0,0,0,0.05)]">
        <button onclick="switchTab('dashboard')" id="nav-dashboard" class="flex flex-col items-center text-teal-600 transition w-1/4">
            <i class="fa-solid fa-chart-line text-lg mb-1"></i>
            <span class="text-[10px] font-semibold">Progress</span>
        </button>
        <button onclick="switchTab('food')" id="nav-food" class="flex flex-col items-center text-slate-400 transition w-1/4">
            <i class="fa-solid fa-utensils text-lg mb-1"></i>
            <span class="text-[10px] font-semibold">Food AI</span>
        </button>
        <button onclick="switchTab('fitdays')" id="nav-fitdays" class="flex flex-col items-center text-slate-400 transition w-1/4">
            <i class="fa-solid fa-mobile-screen text-lg mb-1"></i>
            <span class="text-[10px] font-semibold">Scan App</span>
        </button>
        <button onclick="switchTab('goals')" id="nav-goals" class="flex flex-col items-center text-slate-400 transition w-1/4">
            <i class="fa-solid fa-bullseye text-lg mb-1"></i>
            <span class="text-[10px] font-semibold">Goals</span>
        </button>
    </nav>

</div>

<script>
    // --- APP STATE MANAGEMENT ---
    // Pre-populating with Benz's baseline data to make testing easier
    const defaultData = {
        targetCalories: 2300,
        consumedCalories: 0,
        targetWeight: 70.1,
        history: [
            { date: '2026-08-31', weight: 67.4, fat: 11.3, muscle: 55.7 }
        ]
    };

    let appData = JSON.parse(localStorage.getItem('fitMetricsUltimate')) || defaultData;
    let progressChartInstance = null;

    // --- INITIALIZATION ---
    window.addEventListener('DOMContentLoaded', () => {
        document.getElementById('scannedDate').valueAsDate = new Date();
        document.getElementById('settingTargetWeight').value = appData.targetWeight;
        document.getElementById('settingTargetCalories').value = appData.targetCalories;
        
        updateDashboardUI();
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
        document.getElementById('remaining-calories').innerText = `${Math.abs(remaining)} kcal ${remaining >= 0 ? 'left' : 'over'}`;
        
        let percentage = Math.min(100, Math.round((appData.consumedCalories / appData.targetCalories) * 100));
        document.getElementById('calorie-progress-bar').style.width = `${percentage}%`;
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

    // --- FOOD AI SCANNER (MOCK) ---
    function handleMealUpload(event) {
        const file = event.target.files[0];
        if (!file) return;

        const reader = new FileReader();
        reader.onload = function(e) {
            document.getElementById('uploadedMealPreview').src = e.target.result;
            document.getElementById('mealResultContainer').classList.remove('hidden');
            document.getElementById('mealLoading').classList.remove('hidden');
            document.getElementById('mealDetails').classList.add('hidden');

            setTimeout(() => {
                document.getElementById('mealLoading').classList.add('hidden');
                document.getElementById('mealDetails').classList.remove('hidden');
                
                const mockMeals = [
                    { name: 'Chicken & Rice', cal: 620 },
                    { name: 'Protein Shake', cal: 350 },
                    { name: 'Beef Steak', cal: 680 }
                ];
                const selected = mockMeals[Math.floor(Math.random() * mockMeals.length)];
                
                document.getElementById('detectedFoodName').innerText = selected.name;
                document.getElementById('detectedCalories').innerText = `${selected.cal} kcal`;
                window.lastScannedMealCalories = selected.cal;
            }, 1200);
        };
        reader.readAsDataURL(file);
    }

    function logMealToDiary() {
        appData.consumedCalories += (window.lastScannedMealCalories || 500);
        saveToLocal();
        updateDashboardUI();
        alert('Meal logged!');
        switchTab('dashboard');
        // Reset meal scanner
        document.getElementById('mealResultContainer').classList.add('hidden');
        document.getElementById('mealImageInput').value = "";
    }

    // --- FITDAYS OCR SCANNER (TESSERACT) ---
    function handleFitdaysUpload(event) {
        const file = event.target.files[0];
        if (!file) return;

        document.getElementById('ocrResultsContainer').classList.add('hidden');
        document.getElementById('ocrLoadingContainer').classList.remove('hidden');
        document.getElementById('ocrProgressBar').style.width = '0%';
        document.getElementById('ocrStatus').innerHTML = '<i class="fa-solid fa-spinner fa-spin mr-1"></i> Processing...';
        
        document.getElementById('scannedWeight').value = "";
        document.getElementById('scannedFat').value = "";
        document.getElementById('scannedMuscle').value = "";

        const reader = new FileReader();
        reader.onload = (e) => {
            const imageSrc = e.target.result;
            document.getElementById('uploadedFitdaysPreview').src = imageSrc;
            processImageWithTesseract(imageSrc);
        };
        reader.readAsDataURL(file);
    }

    async function processImageWithTesseract(imageSrc) {
        try {
            const worker = Tesseract.createWorker({
                logger: m => {
                    if (m.status === 'recognizing text') {
                        document.getElementById('ocrProgressBar').style.width = Math.round(m.progress * 100) + '%';
                    }
                }
            });
            await worker.load();
            await worker.loadLanguage('eng');
            await worker.initialize('eng');
            
            const { data: { text } } = await worker.recognize(imageSrc);
            await worker.terminate();

            parseFitdaysText(text);
        } catch (error) {
            document.getElementById('ocrStatus').innerText = "Scan failed. Please type data manually.";
            document.getElementById('ocrLoadingContainer').classList.add('hidden');
            document.getElementById('ocrResultsContainer').classList.remove('hidden');
        }
    }

    function parseFitdaysText(text) {
        document.getElementById('ocrLoadingContainer').classList.add('hidden');
        document.getElementById('ocrResultsContainer').classList.remove('hidden');

        const extract = (regex) => {
            const match = text.match(regex);
            return match && match[1] ? parseFloat(match[1].replace(',', '.')) : "";
        };

        const w = extract(/Weight\s*[:\-]?\s*(\d{2,3}[\.,]\d)/i);
        const f = extract(/Body\s*Fat\s*[:\-]?\s*(\d{1,2}[\.,]\d)/i);
        const m = extract(/Muscle\s*mass\s*[:\-]?\s*(\d{2,3}[\.,]\d)/i);

        if(w) document.getElementById('scannedWeight').value = w;
        if(f) document.getElementById('scannedFat').value = f;
        if(m) document.getElementById('scannedMuscle').value = m;
    }

    function saveScannedData() {
        const w = parseFloat(document.getElementById('scannedWeight').value);
        const f = parseFloat(document.getElementById('scannedFat').value);
        const m = parseFloat(document.getElementById('scannedMuscle').value);
        const d = document.getElementById('scannedDate').value;

        if (!w || !f || !m || !d) {
            alert("Please fill Weight, Fat, Muscle, and Date.");
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
        alert("Progress logged!");
        switchTab('dashboard');
        
        // Reset scanner
        document.getElementById('ocrResultsContainer').classList.add('hidden');
        document.getElementById('fitdaysImageInput').value = "";
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
            alert("Calories reset.");
        }
    }

    function clearAllData() {
        if(confirm("Delete all history? This cannot be undone.")) {
            appData.history = [];
            appData.consumedCalories = 0;
            saveToLocal();
            updateDashboardUI();
            refreshChart();
            alert("History cleared.");
        }
    }

    function saveToLocal() {
        localStorage.setItem('fitMetricsUltimate', JSON.stringify(appData));
    }
</script>

</body>
</html>
