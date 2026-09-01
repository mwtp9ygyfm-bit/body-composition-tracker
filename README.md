<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Body Composition Tracker</title>
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
            <p class="text-xs text-teal-100">User: Benz | Body Recomposition</p>
        </div>
        <button onclick="switchTab('goals')" class="bg-teal-700 p-2 rounded-full text-sm hover:bg-teal-800 transition">
            <i class="fa-solid fa-user-gear"></i>
        </button>
    </header>

    <main class="p-4 space-y-4">

        <section id="tab-dashboard" class="tab-content active space-y-4">
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
                <div class="flex justify-between items-center mb-3">
                    <h3 class="font-bold text-slate-700 text-sm">Progress Timeline</h3>
                    <button onclick="switchTab('scanner')" class="text-xs bg-teal-50 text-teal-600 font-semibold px-3 py-1.5 rounded-full hover:bg-teal-100 transition">
                        <i class="fa-solid fa-camera mr-1"></i> Scan Fitdays
                    </button>
                </div>
                <div class="relative h-64">
                    <canvas id="progressChart"></canvas>
                </div>
            </div>

            <div class="bg-white p-4 rounded-3xl shadow-sm border border-slate-100">
                <h3 class="font-bold text-slate-700 text-sm mb-3">Recent Logs</h3>
                <div id="history-list" class="space-y-2 max-h-48 overflow-y-auto">
                    </div>
            </div>
        </section>

        <section id="tab-scanner" class="tab-content space-y-4">
            <div class="bg-white p-5 rounded-3xl shadow-sm border border-slate-100 text-center space-y-3">
                <div class="w-16 h-16 bg-teal-50 text-teal-600 rounded-full flex items-center justify-center mx-auto text-2xl shadow-inner">
                    <i class="fa-solid fa-file-invoice"></i>
                </div>
                <h3 class="font-bold text-slate-800 text-base">Fitdays Data Scanner</h3>
                <p class="text-xs text-slate-500 leading-relaxed">Upload a screenshot of your Fitdays app. We'll extract your metrics automatically.</p>
                
                <input type="file" id="fitdaysImageInput" accept="image/*" class="hidden" onchange="handleFitdaysUpload(event)">
                
                <button onclick="document.getElementById('fitdaysImageInput').click()" class="w-full bg-teal-600 text-white py-3 rounded-2xl font-semibold text-sm shadow-md hover:bg-teal-700 transition flex items-center justify-center gap-2">
                    <i class="fa-solid fa-image"></i> Select Screenshot
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
                    <div id="ocrProgressBar" class="bg-teal-500 h-full rounded-full transition-all duration-200" style="width: 0%;"></div>
                </div>
            </div>

            <div id="ocrResultsContainer" class="hidden bg-white p-5 rounded-3xl shadow-sm border border-slate-100 space-y-4">
                <div class="flex justify-between items-center border-b border-slate-100 pb-2">
                    <h3 class="font-bold text-slate-800 text-sm">Verify Scanned Data</h3>
                    <span class="text-[10px] bg-amber-100 text-amber-700 px-2 py-1 rounded-full font-bold">Review Before Saving</span>
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
                    Save to Progress Chart
                </button>
            </div>
        </section>

        <section id="tab-goals" class="tab-content space-y-4">
            <div class="bg-white p-5 rounded-3xl shadow-sm border border-slate-100 space-y-4">
                <h3 class="font-bold text-slate-800 text-base">Calorie & Macro Goals</h3>
                
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
            
            <div class="bg-red-50 p-4 rounded-3xl border border-red-100 text-center">
                <button onclick="clearAllData()" class="text-red-600 text-sm font-semibold">
                    <i class="fa-solid fa-trash mr-1"></i> Clear All Saved Data
                </button>
            </div>
        </section>

    </main>

    <nav class="fixed bottom-0 left-0 right-0 max-w-[480px] mx-auto bg-white border-t border-slate-100 flex justify-around py-3 z-40 shadow-[0_-4px_6px_-1px_rgba(0,0,0,0.05)]">
        <button onclick="switchTab('dashboard')" id="nav-dashboard" class="flex flex-col items-center text-teal-600 transition">
            <i class="fa-solid fa-chart-line text-lg"></i>
            <span class="text-[10px] font-semibold mt-1">Progress</span>
        </button>
        <button onclick="switchTab('scanner')" id="nav-scanner" class="flex flex-col items-center text-slate-400 transition">
            <i class="fa-solid fa-expand text-lg"></i>
            <span class="text-[10px] font-semibold mt-1">Scan App</span>
        </button>
        <button onclick="switchTab('goals')" id="nav-goals" class="flex flex-col items-center text-slate-400 transition">
            <i class="fa-solid fa-bullseye text-lg"></i>
            <span class="text-[10px] font-semibold mt-1">Goals</span>
        </button>
    </nav>

</div>

<script>
    // --- APP STATE MANAGEMENT ---
    let appData = JSON.parse(localStorage.getItem('fitMetricsBenzReal')) || {
        targetCalories: 2300,
        targetWeight: 70.1,
        history: []
    };

    let progressChartInstance = null;

    // --- INITIALIZATION ---
    window.addEventListener('DOMContentLoaded', () => {
        // Set today's date in scanner form
        document.getElementById('scannedDate').valueAsDate = new Date();
        
        // Load Settings
        document.getElementById('settingTargetWeight').value = appData.targetWeight;
        document.getElementById('settingTargetCalories').value = appData.targetCalories;

        updateDashboardUI();
        initChart();
        renderHistoryList();
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

    // --- UI UPDATES ---
    function updateDashboardUI() {
        if (appData.history.length > 0) {
            // Sort history by date chronologically
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
    }

    function renderHistoryList() {
        const list = document.getElementById('history-list');
        list.innerHTML = '';
        
        if (appData.history.length === 0) {
            list.innerHTML = '<p class="text-xs text-slate-400 text-center py-2">No data logged yet.</p>';
            return;
        }

        // Show newest first in the list
        const reversedHistory = [...appData.history].sort((a, b) => new Date(b.date) - new Date(a.date));

        reversedHistory.forEach(item => {
            const dateObj = new Date(item.date);
            const dateString = dateObj.toLocaleDateString('en-US', { month: 'short', day: 'numeric', year: 'numeric' });
            
            const div = document.createElement('div');
            div.className = "flex justify-between items-center bg-slate-50 p-3 rounded-xl";
            div.innerHTML = `
                <div>
                    <span class="block text-xs font-bold text-slate-700">${dateString}</span>
                    <span class="block text-[10px] text-slate-500">Fat: ${item.fat}% | Muscle: ${item.muscle}kg</span>
                </div>
                <div class="text-right">
                    <span class="block text-sm font-bold text-teal-600">${item.weight} kg</span>
                </div>
            `;
            list.appendChild(div);
        });
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
                interaction: { mode: 'index', intersect: false },
                plugins: {
                    legend: { position: 'bottom', labels: { boxWidth: 12, font: { size: 10 } } }
                },
                scales: {
                    y: { 
                        type: 'linear', 
                        display: true, 
                        position: 'left',
                        title: { display: true, text: 'Weight/Muscle (kg)', font: {size: 10} },
                        grid: { color: '#f1f5f9' }
                    },
                    y1: {
                        type: 'linear',
                        display: true,
                        position: 'right',
                        title: { display: true, text: 'Body Fat (%)', font: {size: 10} },
                        grid: { drawOnChartArea: false }
                    },
                    x: { grid: { display: false }, ticks: { font: { size: 10 } } }
                }
            }
        });
    }

    function getChartData() {
        // Sort chronologically for the chart
        const sorted = [...appData.history].sort((a, b) => new Date(a.date) - new Date(b.date));
        
        return {
            labels: sorted.map(item => {
                const d = new Date(item.date);
                return `${d.getMonth()+1}/${d.getDate()}`;
            }),
            datasets: [
                {
                    label: 'Weight (kg)',
                    data: sorted.map(item => item.weight),
                    borderColor: '#0f766e', // teal-700
                    backgroundColor: '#0f766e',
                    borderWidth: 2,
                    yAxisID: 'y',
                    tension: 0.3
                },
                {
                    label: 'Muscle (kg)',
                    data: sorted.map(item => item.muscle),
                    borderColor: '#3b82f6', // blue-500
                    backgroundColor: '#3b82f6',
                    borderWidth: 2,
                    borderDash: [5, 5],
                    yAxisID: 'y',
                    tension: 0.3
                },
                {
                    label: 'Body Fat (%)',
                    data: sorted.map(item => item.fat),
                    borderColor: '#f59e0b', // amber-500
                    backgroundColor: '#f59e0b',
                    borderWidth: 2,
                    yAxisID: 'y1',
                    tension: 0.3
                }
            ]
        };
    }

    function refreshChart() {
        if (!progressChartInstance) return;
        progressChartInstance.data = getChartData();
        progressChartInstance.update();
    }

    // --- TESSERACT OCR INTEGRATION ---
    function handleFitdaysUpload(event) {
        const file = event.target.files[0];
        if (!file) return;

        // Reset UI
        document.getElementById('ocrResultsContainer').classList.add('hidden');
        document.getElementById('ocrLoadingContainer').classList.remove('hidden');
        document.getElementById('ocrProgressBar').style.width = '0%';
        document.getElementById('ocrStatus').innerHTML = '<i class="fa-solid fa-spinner fa-spin mr-1"></i> Initializing AI Scanner...';
        
        // Clear previous inputs
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
                        document.getElementById('ocrStatus').innerHTML = '<i class="fa-solid fa-microchip mr-1"></i> Scanning image...';
                        document.getElementById('ocrProgressBar').style.width = Math.round(m.progress * 100) + '%';
                    }
                }
            });

            await worker.load();
            await worker.loadLanguage('eng');
            await worker.initialize('eng');
            
            // Run OCR
            const { data: { text } } = await worker.recognize(imageSrc);
            console.log("Raw OCR Text Extracted:\n", text);
            await worker.terminate();

            parseFitdaysText(text);

        } catch (error) {
            console.error("OCR Error:", error);
            document.getElementById('ocrStatus').innerText = "Scan failed. Please enter data manually.";
            // Show the form anyway so they can type it in manually
            document.getElementById('ocrLoadingContainer').classList.add('hidden');
            document.getElementById('ocrResultsContainer').classList.remove('hidden');
        }
    }

    function parseFitdaysText(text) {
        document.getElementById('ocrLoadingContainer').classList.add('hidden');
        document.getElementById('ocrResultsContainer').classList.remove('hidden');

        // Regex patterns to find numbers near specific keywords
        // Accounts for potential OCR typos like spaces or missing decimals
        
        const extractMetric = (keywordRegex, textStr) => {
            const match = textStr.match(keywordRegex);
            if (match && match[1]) {
                // Replace commas with dots just in case
                let numStr = match[1].replace(',', '.');
                return parseFloat(numStr);
            }
            return "";
        };

        // Looking for "Weight 67.4kg" or "Weight [newline] 67.4"
        const weightRegex = /Weight\s*[:\-]?\s*(\d{2,3}[\.,]\d)/i;
        // Looking for "Body Fat 11.3%" or similar
        const fatRegex = /Body\s*Fat\s*[:\-]?\s*(\d{1,2}[\.,]\d)/i;
        // Looking for "Muscle mass 55.7kg"
        const muscleRegex = /Muscle\s*mass\s*[:\-]?\s*(\d{2,3}[\.,]\d)/i;

        const w = extractMetric(weightRegex, text);
        const f = extractMetric(fatRegex, text);
        const m = extractMetric(muscleRegex, text);

        if(w) document.getElementById('scannedWeight').value = w;
        if(f) document.getElementById('scannedFat').value = f;
        if(m) document.getElementById('scannedMuscle').value = m;

        // Fallback for user: If OCR misses something, the inputs are blank and they can just type it in.
    }

    // --- SAVING DATA ---
    function saveScannedData() {
        const w = parseFloat(document.getElementById('scannedWeight').value);
        const f = parseFloat(document.getElementById('scannedFat').value);
        const m = parseFloat(document.getElementById('scannedMuscle').value);
        const d = document.getElementById('scannedDate').value;

        if (!w || !f || !m || !d) {
            alert("Please ensure Weight, Body Fat, Muscle Mass, and Date are filled out correctly.");
            return;
        }

        // Check if date already exists and update it, otherwise add new
        const existingIndex = appData.history.findIndex(item => item.date === d);
        if (existingIndex >= 0) {
            appData.history[existingIndex] = { date: d, weight: w, fat: f, muscle: m };
        } else {
            appData.history.push({ date: d, weight: w, fat: f, muscle: m });
        }

        saveToLocal();
        updateDashboardUI();
        refreshChart();
        renderHistoryList();
        
        alert("Progress logged successfully!");
        switchTab('dashboard');
    }

    function saveSettings() {
        appData.targetWeight = parseFloat(document.getElementById('settingTargetWeight').value);
        appData.targetCalories = parseInt(document.getElementById('settingTargetCalories').value);
        saveToLocal();
        alert('Goals saved successfully!');
    }

    function saveToLocal() {
        localStorage.setItem('fitMetricsBenzReal', JSON.stringify(appData));
    }

    function clearAllData() {
        if(confirm("Are you sure you want to delete all your history? This cannot be undone.")) {
            appData.history = [];
            saveToLocal();
            updateDashboardUI();
            refreshChart();
            renderHistoryList();
            alert("All history cleared.");
        }
    }
</script>

</body>
</html>
