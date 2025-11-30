[index.html](https://github.com/user-attachments/files/23841501/index.html)
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Material Entry</title>
    <script src="https://telegram.org/js/telegram-web-app.js"></script>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        /* Hide scrollbar for cleaner look */
        .no-scrollbar::-webkit-scrollbar { display: none; }
        .no-scrollbar { -ms-overflow-style: none; scrollbar-width: none; }
    </style>
</head>
<body class="bg-gray-100 text-gray-800 pb-24 select-none font-sans">

    <!-- Header -->
    <div class="bg-white p-4 sticky top-0 shadow-sm z-50 border-b border-gray-200">
        <h1 class="text-lg font-bold text-blue-600 flex items-center gap-2">
            📝 Material Entry Form
        </h1>
    </div>

    <form id="mainForm" class="p-4 space-y-4 max-w-md mx-auto">
        
        <!-- SR NO -->
        <div class="bg-white p-3 rounded-xl shadow-sm border border-gray-200">
            <label class="block text-xs font-bold text-gray-500 uppercase mb-1 ml-1">SR Number(s)</label>
            <input type="text" id="sr_no" class="w-full p-3 bg-gray-50 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:bg-white outline-none transition-all" placeholder="e.g. 12345, 67890">
        </div>

        <!-- RFO 1 -->
        <div class="bg-white p-3 rounded-xl shadow-sm border border-gray-200">
            <label class="block text-xs font-bold text-gray-500 uppercase mb-1 ml-1">RFO 1</label>
            <select id="rfo1" class="w-full p-3 bg-gray-50 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 outline-none">
                <option value="">Select Option</option>
            </select>
        </div>

        <!-- RFO 2 (Sub Reason) -->
        <div class="bg-white p-3 rounded-xl shadow-sm border border-gray-200 transition-all" id="rfo2_container">
            <label class="block text-xs font-bold text-gray-500 uppercase mb-1 ml-1">RFO 2 (Sub Reason)</label>
            <select id="rfo2" class="w-full p-3 bg-gray-50 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 outline-none">
                <option value="">Select RFO 1 First</option>
            </select>
        </div>

        <!-- CUT REASON -->
        <div class="bg-white p-3 rounded-xl shadow-sm border border-gray-200">
            <label class="block text-xs font-bold text-gray-500 uppercase mb-1 ml-1">Reason for Cut</label>
            <select id="cut" class="w-full p-3 bg-gray-50 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 outline-none">
                <option value="">Select Option</option>
            </select>
        </div>

        <!-- AUTHORITY -->
        <div class="bg-white p-3 rounded-xl shadow-sm border border-gray-200">
            <label class="block text-xs font-bold text-gray-500 uppercase mb-1 ml-1">Authority</label>
            <select id="auth" class="w-full p-3 bg-gray-50 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 outline-none">
                <option value="">Select Option</option>
            </select>
        </div>

        <!-- FAT/OTB -->
        <div class="bg-white p-3 rounded-xl shadow-sm border border-gray-200">
            <label class="block text-xs font-bold text-gray-500 uppercase mb-1 ml-1">FAT / OTB</label>
            <select id="fat" class="w-full p-3 bg-gray-50 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 outline-none">
                <option value="">Select Option</option>
            </select>
        </div>

        <!-- DELAY -->
        <div class="bg-white p-3 rounded-xl shadow-sm border border-gray-200">
            <label class="block text-xs font-bold text-gray-500 uppercase mb-1 ml-1">Delay Reason</label>
            <select id="delay" class="w-full p-3 bg-gray-50 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 outline-none">
                <option value="">Select Option</option>
            </select>
        </div>

        <!-- MATERIALS -->
        <div class="bg-white p-4 rounded-xl shadow-sm border border-gray-200">
            <div class="flex justify-between items-center mb-3 cursor-pointer select-none" onclick="toggleMaterials()">
                <label class="text-sm font-bold text-gray-700 cursor-pointer uppercase">📦 Materials Used</label>
                <div id="matToggleIcon" class="w-8 h-8 flex items-center justify-center bg-blue-100 text-blue-600 rounded-full font-bold text-lg hover:bg-blue-200 transition-colors">+</div>
            </div>
            
            <div id="materialsList" class="hidden space-y-3 pt-2 border-t border-gray-100">
                <!-- Materials will be populated here by JavaScript -->
            </div>
        </div>

    </form>

    <script>
        // Initialize Telegram WebApp
        const tg = window.Telegram.WebApp;
        tg.expand(); // Request full screen

        // --- DATA CONFIGURATION ---
        // (Must match Python Backend Data)
        const DATA = {
            RFO1: [
                "FAT_OTB_related", "Fiber_cut_OH_cable_restored", "Fiber_cut_UG_cable_restored",
                "NFF_Wrong_TT", "ODF_related", "OLT_related", "Planned_Cutover_Rehab",
                "Auto_TT_Activity_Completed", "Copper_Cable_Cut", "Copper_Cable_Faulty", "Proactive"
            ],
            CUT: ["Inside FAT", "Civil Work", "Rat Bite", "Tree Fall", "Vehicle Hit"],
            AUTH: ["KSEB", "PWD", "Corporation", "Private", "None"],
            FAT: ["FAT Related", "OTB Related", "Both", "N/A"],
            DELAY: ["No Delay", "Rain", "Traffic", "Night Work", "Access Issue"],
            DEFAULT_RFO2: ["Single Fiber Break", "Multi Fiber Break", "Patch Cord", "Splitter"]
        };

        const RFO2_MAPPING = {
            "FAT_OTB_related": ["FAT/OTB shifted/ replaced - cable re-spliced", "Pigtail/patch cord cleaned/replaced", "Riser cable damaged due to inbuilding activity", "Secondary splitter replaced", "Single fiber break-re-spliced"],
            "Fiber_cut_OH_cable_restored": ["Cable burnt- electric short Circuit", "Cable cut by vehicle", "cable cut- tree cleaning drive", "Cable damaged- Other Infra works", "Cable damaged-Pole Removal/shifting", "Fiber break inside JC", "Intentional cut by unknown person"],
            "Fiber_cut_UG_cable_restored": ["Cable damaged due to nala/culvert cleaning/infa work/Road expansion", "Fiber break inside JC", "Intentional cut by unknown person", "Rodent cut"],
            "NFF_Wrong_TT": ["TT auto resolved post power restoration", "TT resolved without any action", "Wrong TT Login - Customer Barred", "Wrong TT login - Electricity Failure", "Wrong TT login- Planned activity", "No fault found", "TT Cancelled", "False Alarm"],
            "ODF_related": ["Connector cleaned at ODF", "ODF damaged by Authority/unknown person- Replaced", "patch cord/pigtail damaged - Replaced", "Primary splitter faulty - Replaced"],
            "OLT_related": ["Fiber cut by Rodent inside plant FDMS/patch cord", "PON port fault - Replaced", "PON Port Hang- Resetted", "PON Port swapped", "SFP Faulty- Replaced"],
            "Planned_Cutover_Rehab": ["Cable Routing/Sagging corrected", "Cutover activity- PON Changed", "FAT Changed", "FAT Rehab", "FMS/ODF Changed", "FMS/ODF Rehab", "ODF Cutover", "ODF Rehab", "ODF replaced", "OH cable replaced", "OH to UG", "UG cable replaced", "UG to OH"],
            "Auto_TT_Activity_Completed": ["No Fault found", "Planned actvity", "Drop wire cut"],
            "Copper_Cable_Cut": ["Cable burnt- electric short Circuit", "Cable cut by vehicle", "cable cut- tree cleaning drive", "Cable damaged- Other Infra works", "Cable damaged-Pole Removal/shifting", "Intentional cut by unknown person", "Cable damaged due to nala/culvert cleaning/infa work/Road expansion", "Fiber break inside JC", "Rodent cut"],
            "Copper_Cable_Faulty": ["Copper pair faulty", "Copper joint faulty", "Copper to Ftth"],
            "Proactive": ["ERP clearance", "Cable sagging correction", "FAT Changed", "FAT Rehab", "FMS Changed", "FMS Rehab", "ODF Rehab", "FAT Refix", "OTB Refix", "EB drive", "Door closed", "EB Pole shifting drive", "EB Tree cleaning drive", "Overhead cable replaced", "Overhead to under ground", "underground cable replaced", "Underground to over head", "Power correction", "JC Sagging Correction", "Own Pole shifting", "Planned Actvity"]
        };

        const MATERIALS_LIST = [
            "6F Cable (mtr)", "12F Cable (mtr)", "24F Cable (mtr)", "48F Cable (mtr)", "96F Cable (mtr)",
            "INS TAPE (Nos)", "CLAMP 4-7 MM (Nos)", "CLAMP 10-14 MM2 (Nos)",
            "CLOSER -12F(Nos)", "CLOSER -24F(Nos)", "CLOSER -48F(Nos)", "CLOSER -96F(Nos)",
            "1:8 SPLITTER(Nos)", "1:16 SPLITTER(Nos)", "2:4 SPLITTER(Nos)", "2:8 SPLITTER(Nos)", "2:16 SPLITER(Nos)",
            "FAT BOX-16F(Nos)", "ODP (Nos)", "96F FMS (Nos)",
            "PICKTEL-GREEN (Nos)", "PICKTEL-BLUE(Nos)", "DUCT (Mtr)", "GI POL (Nos)",
            "PACTH CORD(Nos)", "SLEEVES (Nos)", "JC HUB (Nos)",
            "5P Cable(Mtr)", "10P Copper Cable (mtr)", "20P Copper Cable (mtr)",
            "50P Copper Cable (mtr)", "100P Copper Cable (mtr)", "200P Copper Cable (mtr)", "400P Copper Cable (mtr)",
            "UY CONNECTOR", "MODULE 10", "VCC", "10/20 PAIR DP BOX", "50 PAIR DP BOX",
            "TSF-1", "TSF-2", "TSF-3", "TSF-4", "Marking Lable"
        ];

        // --- POPULATE UI FUNCTIONS ---
        function populateSelect(id, items) {
            const el = document.getElementById(id);
            // Clear existing options except the first one
            while (el.options.length > 1) {
                el.remove(1);
            }
            items.forEach(i => {
                const opt = document.createElement("option");
                opt.value = i;
                opt.textContent = i;
                el.appendChild(opt);
            });
        }

        // Initialize Dropdowns
        populateSelect("rfo1", DATA.RFO1);
        populateSelect("cut", DATA.CUT);
        populateSelect("auth", DATA.AUTH);
        populateSelect("fat", DATA.FAT);
        populateSelect("delay", DATA.DELAY);

        // Populate Materials List
        const matContainer = document.getElementById("materialsList");
        MATERIALS_LIST.forEach((mat, idx) => {
            const div = document.createElement("div");
            div.className = "flex items-center gap-3 p-2 bg-gray-50 rounded-lg border border-gray-200 hover:bg-white hover:shadow-sm transition-all";
            div.innerHTML = `
                <input type="checkbox" id="chk_${idx}" class="w-5 h-5 text-blue-600 rounded focus:ring-blue-500 cursor-pointer" onchange="toggleQty(${idx})">
                <label for="chk_${idx}" class="flex-1 text-sm font-medium text-gray-700 cursor-pointer select-none">${mat}</label>
                <input type="number" id="qty_${idx}" class="hidden w-20 p-2 text-sm text-center border border-gray-300 rounded-md focus:ring-2 focus:ring-blue-500 outline-none" placeholder="Qty" min="1">
            `;
            matContainer.appendChild(div);
        });

        // --- EVENT LISTENERS ---
        
        // Dynamic RFO2 Dropdown Logic
        document.getElementById("rfo1").addEventListener("change", function() {
            const rfo2 = document.getElementById("rfo2");
            rfo2.innerHTML = '<option value="">Select Option</option>'; // Reset RFO2
            
            const selection = this.value;
            // Get specific options or default if none found
            const options = RFO2_MAPPING[selection] || DATA.DEFAULT_RFO2;
            
            options.forEach(opt => {
                const el = document.createElement("option");
                el.value = opt;
                el.textContent = opt;
                rfo2.appendChild(el);
            });
        });

        // Toggle Materials Section Visibility
        window.toggleMaterials = function() {
            const list = document.getElementById("materialsList");
            const icon = document.getElementById("matToggleIcon");
            if (list.classList.contains("hidden")) {
                list.classList.remove("hidden");
                icon.textContent = "-";
                icon.className = "w-8 h-8 flex items-center justify-center bg-red-100 text-red-600 rounded-full font-bold text-lg hover:bg-red-200 transition-colors";
            } else {
                list.classList.add("hidden");
                icon.textContent = "+";
                icon.className = "w-8 h-8 flex items-center justify-center bg-blue-100 text-blue-600 rounded-full font-bold text-lg hover:bg-blue-200 transition-colors";
            }
        };

        // Toggle Quantity Input Visibility
        window.toggleQty = function(idx) {
            const chk = document.getElementById(`chk_${idx}`);
            const qty = document.getElementById(`qty_${idx}`);
            if (chk.checked) {
                qty.classList.remove("hidden");
                qty.focus();
                qty.value = "1"; // Default quantity
            } else {
                qty.classList.add("hidden");
                qty.value = ""; // Clear value
            }
        };

        // --- TELEGRAM MAIN BUTTON LOGIC ---
        tg.MainButton.setText("SAVE ENTRY"); // Set button text
        tg.MainButton.show(); // Make sure it's visible

        // Handle Main Button Click
        tg.MainButton.onClick(function() {
            const sr_no = document.getElementById("sr_no").value;
            
            // Validation
            if (!sr_no.trim()) {
                tg.showAlert("⚠️ Please enter SR Number(s)");
                return;
            }

            // Gather Form Data
            const data = {
                SR_NO: sr_no,
                RFO1: document.getElementById("rfo1").value,
                RFO2: document.getElementById("rfo2").value,
                CUT: document.getElementById("cut").value,
                AUTH: document.getElementById("auth").value,
                FAT: document.getElementById("fat").value,
                DELAY: document.getElementById("delay").value,
                MATERIALS: {}
            };

            // Gather Materials Data
            let hasMaterials = false;
            MATERIALS_LIST.forEach((mat, idx) => {
                const chk = document.getElementById(`chk_${idx}`);
                const qty = document.getElementById(`qty_${idx}`);
                if (chk.checked) {
                    data.MATERIALS[mat] = qty.value || "1";
                    hasMaterials = true;
                }
            });

            // Send Data back to Python Bot
            tg.sendData(JSON.stringify(data));
        });

    </script>
</body>
</html>
