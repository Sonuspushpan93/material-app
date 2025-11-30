[index.html](https://github.com/user-attachments/files/23841944/index.html)
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Material Entry</title>
    <script src="https://telegram.org/js/telegram-web-app.js"></script>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        .no-scrollbar::-webkit-scrollbar { display: none; }
        .no-scrollbar { -ms-overflow-style: none; scrollbar-width: none; }
        
        /* Dark Mode Colors */
        body.dark { background-color: #0f0f11; color: #ededed; }
        .dark .bg-white { background-color: #212121; }
        .dark .bg-gray-50 { background-color: #18181b; }
        .dark .text-gray-800 { color: #ededed; }
        .dark .text-gray-700 { color: #d4d4d8; }
        .dark .text-gray-500 { color: #a1a1aa; }
        .dark .border-gray-200 { border-color: #3f3f46; }
        .dark input, .dark select, .dark textarea { background-color: #2c2c2e; border-color: #3f3f46; color: white; }
        .dark .shadow-sm { box-shadow: none; }
        
        /* Modal Styles */
        .modal { transition: opacity 0.25s ease; }
        body.modal-active { overflow-x: hidden; overflow-y: visible !important; }
    </style>
</head>
<body class="bg-gray-50 text-gray-800 pb-24 font-sans select-none transition-colors duration-200">

    <div class="bg-white p-4 sticky top-0 shadow-sm z-50 border-b border-gray-200">
        <h1 class="text-lg font-bold text-blue-600 flex items-center gap-2">
            📝 Material Entry Form
        </h1>
    </div>

    <form id="mainForm" class="p-4 space-y-4 max-w-md mx-auto">
        
        <!-- SR NO -->
        <div class="bg-white p-3 rounded-xl shadow-sm border border-gray-200">
            <label class="block text-xs font-bold text-gray-500 uppercase mb-1 ml-1">SR Number(s)</label>
            <input type="text" id="sr_no" class="w-full p-3 bg-gray-50 border border-gray-200 rounded-lg focus:ring-2 focus:ring-blue-500 outline-none transition-all" placeholder="e.g. 12345, 67890">
        </div>

        <!-- DROPDOWNS -->
        <div class="space-y-3">
            <div class="bg-white p-3 rounded-xl shadow-sm border border-gray-200">
                <label class="block text-xs font-bold text-gray-500 uppercase mb-1 ml-1">RFO 1</label>
                <select id="rfo1" class="w-full p-3 bg-gray-50 border border-gray-200 rounded-lg focus:ring-2 focus:ring-blue-500 outline-none"></select>
            </div>
            
            <div class="bg-white p-3 rounded-xl shadow-sm border border-gray-200" id="rfo2_container">
                <label class="block text-xs font-bold text-gray-500 uppercase mb-1 ml-1">RFO 2 (Sub Reason)</label>
                <select id="rfo2" class="w-full p-3 bg-gray-50 border border-gray-200 rounded-lg focus:ring-2 focus:ring-blue-500 outline-none">
                    <option value="">Select RFO 1 First</option>
                </select>
            </div>

            <!-- CUT REASON with Manual Option -->
            <div class="bg-white p-3 rounded-xl shadow-sm border border-gray-200">
                <label class="block text-xs font-bold text-gray-500 uppercase mb-1 ml-1">Reason for Cut</label>
                <select id="cut" class="w-full p-3 bg-gray-50 border border-gray-200 rounded-lg focus:ring-2 focus:ring-blue-500 outline-none mb-2" onchange="toggleManualInput('cut')"></select>
                <input type="text" id="cut_manual" class="hidden w-full p-3 bg-gray-50 border border-gray-200 rounded-lg focus:ring-2 focus:ring-yellow-500 outline-none" placeholder="Type Reason for Cut manually...">
            </div>

            <div class="bg-white p-3 rounded-xl shadow-sm border border-gray-200">
                <label class="block text-xs font-bold text-gray-500 uppercase mb-1 ml-1">Authority</label>
                <select id="auth" class="w-full p-3 bg-gray-50 border border-gray-200 rounded-lg focus:ring-2 focus:ring-blue-500 outline-none"></select>
            </div>

            <div class="bg-white p-3 rounded-xl shadow-sm border border-gray-200">
                <label class="block text-xs font-bold text-gray-500 uppercase mb-1 ml-1">FAT / OTB</label>
                <select id="fat" class="w-full p-3 bg-gray-50 border border-gray-200 rounded-lg focus:ring-2 focus:ring-blue-500 outline-none"></select>
            </div>

            <!-- DELAY REASON with Manual Option -->
            <div class="bg-white p-3 rounded-xl shadow-sm border border-gray-200">
                <label class="block text-xs font-bold text-gray-500 uppercase mb-1 ml-1">Delay Reason</label>
                <select id="delay" class="w-full p-3 bg-gray-50 border border-gray-200 rounded-lg focus:ring-2 focus:ring-blue-500 outline-none mb-2" onchange="toggleManualInput('delay')"></select>
                <input type="text" id="delay_manual" class="hidden w-full p-3 bg-gray-50 border border-gray-200 rounded-lg focus:ring-2 focus:ring-yellow-500 outline-none" placeholder="Type Delay Reason manually...">
            </div>
        </div>

        <!-- MATERIALS -->
        <div class="bg-white p-4 rounded-xl shadow-sm border border-gray-200">
            <div class="flex justify-between items-center mb-3 cursor-pointer select-none" onclick="toggleMaterials()">
                <label class="text-sm font-bold text-gray-700 cursor-pointer uppercase">📦 Materials Used</label>
                <div id="matToggleIcon" class="w-8 h-8 flex items-center justify-center bg-blue-100 text-blue-600 rounded-full font-bold text-lg hover:bg-blue-200 transition-colors">+</div>
            </div>
            
            <div id="materialsList" class="hidden space-y-3 pt-2 border-t border-gray-100">
                <!-- Materials populated by JS -->
                
                 <!-- Manual Material Entry -->
                <div class="flex flex-col gap-2 p-2 bg-yellow-50 dark:bg-yellow-900/20 rounded-lg border border-yellow-200 dark:border-yellow-700/30">
                    <div class="flex items-center gap-3">
                        <input type="checkbox" id="chk_manual" class="w-5 h-5 text-yellow-600 rounded focus:ring-yellow-500 cursor-pointer" onchange="toggleManualMat()">
                        <label for="chk_manual" class="flex-1 text-sm font-bold text-yellow-700 dark:text-yellow-500 cursor-pointer select-none">Other / Manual Type</label>
                    </div>
                    <div id="manualInputContainer" class="hidden flex gap-2 mt-1">
                        <input type="text" id="manual_name" class="flex-1 p-2 text-sm border border-gray-300 dark:border-zinc-600 rounded-md focus:ring-2 focus:ring-yellow-500 outline-none dark:bg-zinc-700 dark:text-white" placeholder="Item Name">
                        <input type="number" id="manual_qty" class="w-20 p-2 text-sm text-center border border-gray-300 dark:border-zinc-600 rounded-md focus:ring-2 focus:ring-yellow-500 outline-none dark:bg-zinc-700 dark:text-white" placeholder="Qty" min="1">
                    </div>
                </div>
            </div>
        </div>

    </form>

    <!-- CONFIRMATION MODAL -->
    <div id="confirmModal" class="fixed inset-0 z-50 hidden overflow-y-auto" aria-labelledby="modal-title" role="dialog" aria-modal="true">
        <div class="flex items-end justify-center min-h-screen px-4 pt-4 pb-20 text-center sm:block sm:p-0">
            <div class="fixed inset-0 transition-opacity bg-gray-500 bg-opacity-75 dark:bg-black dark:bg-opacity-80" aria-hidden="true"></div>
            <span class="hidden sm:inline-block sm:align-middle sm:h-screen" aria-hidden="true">&#8203;</span>
            <div class="inline-block align-bottom bg-white dark:bg-zinc-800 rounded-lg text-left overflow-hidden shadow-xl transform transition-all sm:my-8 sm:align-middle sm:max-w-lg w-full">
                <div class="bg-white dark:bg-zinc-800 px-4 pt-5 pb-4 sm:p-6 sm:pb-4">
                    <div class="sm:flex sm:items-start">
                        <div class="mt-3 text-center sm:mt-0 sm:ml-4 sm:text-left w-full">
                            <h3 class="text-lg leading-6 font-medium text-gray-900 dark:text-white" id="modal-title">Confirm Entry Details</h3>
                            <div class="mt-4 text-sm text-gray-500 dark:text-gray-300 space-y-2" id="summaryContent">
                                <!-- Content injected via JS -->
                            </div>
                        </div>
                    </div>
                </div>
                <div class="bg-gray-50 dark:bg-zinc-900 px-4 py-3 sm:px-6 sm:flex sm:flex-row-reverse gap-2">
                    <button type="button" onclick="submitFinal()" class="w-full inline-flex justify-center rounded-md border border-transparent shadow-sm px-4 py-2 bg-green-600 text-base font-medium text-white hover:bg-green-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-green-500 sm:ml-3 sm:w-auto sm:text-sm">Confirm & Send</button>
                    <button type="button" onclick="closeModal()" class="mt-3 w-full inline-flex justify-center rounded-md border border-gray-300 shadow-sm px-4 py-2 bg-white text-base font-medium text-gray-700 hover:bg-gray-50 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500 sm:mt-0 sm:ml-3 sm:w-auto sm:text-sm dark:bg-zinc-700 dark:text-white dark:border-zinc-600 dark:hover:bg-zinc-600">Edit</button>
                </div>
            </div>
        </div>
    </div>

    <script>
        const tg = window.Telegram.WebApp;
        tg.expand();

        // Force Dark Mode if Telegram is in dark mode
        if (tg.colorScheme === 'dark') document.body.classList.add('dark');

        const DATA = {
            RFO1: ["FAT_OTB_related", "Fiber_cut_OH_cable_restored", "Fiber_cut_UG_cable_restored", "NFF_Wrong_TT", "ODF_related", "OLT_related", "Planned_Cutover_Rehab", "Auto_TT_Activity_Completed", "Copper_Cable_Cut", "Copper_Cable_Faulty", "Proactive"],
            CUT: ["Inside FAT", "Civil Work", "Rat Bite", "Tree Fall", "Vehicle Hit", "Other (Type Manually)"], // Added Option
            AUTH: ["KSEB", "PWD", "Corporation", "Private", "None"],
            FAT: ["FAT Related", "OTB Related", "Both", "N/A"],
            DELAY: ["No Delay", "Rain", "Traffic", "Night Work", "Access Issue", "Other (Type Manually)"], // Added Option
            DEFAULT_RFO2: ["Single Fiber Break", "Multi Fiber Break", "Patch Cord", "Splitter"]
        };

        const RFO2_MAP = {
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

        const MATERIALS = ["6F Cable (mtr)", "12F Cable (mtr)", "24F Cable (mtr)", "48F Cable (mtr)", "96F Cable (mtr)", "INS TAPE (Nos)", "CLAMP 4-7 MM (Nos)", "CLAMP 10-14 MM2 (Nos)", "CLOSER -12F(Nos)", "CLOSER -24F(Nos)", "CLOSER -48F(Nos)", "CLOSER -96F(Nos)", "1:8 SPLITTER(Nos)", "1:16 SPLITTER(Nos)", "2:4 SPLITTER(Nos)", "2:8 SPLITTER(Nos)", "2:16 SPLITER(Nos)", "FAT BOX-16F(Nos)", "ODP (Nos)", "96F FMS (Nos)", "PICKTEL-GREEN (Nos)", "PICKTEL-BLUE(Nos)", "DUCT (Mtr)", "GI POL (Nos)", "PACTH CORD(Nos)", "SLEEVES (Nos)", "JC HUB (Nos)", "5P Cable(Mtr)", "10P Copper Cable (mtr)", "20P Copper Cable (mtr)", "50P Copper Cable (mtr)", "100P Copper Cable (mtr)", "200P Copper Cable (mtr)", "400P Copper Cable (mtr)", "UY CONNECTOR", "MODULE 10", "VCC", "10/20 PAIR DP BOX", "50 PAIR DP BOX", "TSF-1", "TSF-2", "TSF-3", "TSF-4", "Marking Lable"];

        function populate(id, items) {
            const el = document.getElementById(id);
            while(el.options.length > 1) el.remove(1);
            items.forEach(i => { const o = document.createElement("option"); o.value = i; o.text = i; el.appendChild(o); });
        }
        populate("rfo1", DATA.RFO1); populate("cut", DATA.CUT); populate("auth", DATA.AUTH); populate("fat", DATA.FAT); populate("delay", DATA.DELAY);

        const matList = document.getElementById("materialsList");
        MATERIALS.forEach((m, i) => {
            const d = document.createElement("div");
            d.className = "flex items-center gap-2 p-2 bg-gray-50 dark:bg-zinc-800 rounded border border-gray-200 dark:border-zinc-700";
            d.innerHTML = `<input type="checkbox" id="c${i}" class="w-5 h-5 rounded text-blue-600 focus:ring-blue-500 cursor-pointer"><label for="c${i}" class="flex-1 text-sm dark:text-white cursor-pointer select-none">${m}</label><input type="number" id="q${i}" class="hidden w-16 p-1 border rounded dark:bg-zinc-700 dark:text-white" placeholder="Qty">`;
            matList.appendChild(d);
            d.querySelector("input[type=checkbox]").onchange = function() { document.getElementById(`q${i}`).classList.toggle("hidden", !this.checked); };
        });

        document.getElementById("rfo1").onchange = function() {
            const s = document.getElementById("rfo2"); s.innerHTML = '<option value="">Select Option</option>';
            (RFO2_MAP[this.value] || DATA.DEFAULT_RFO2).forEach(i => { const o = document.createElement("option"); o.value = i; o.text = i; s.appendChild(o); });
        };

        window.toggleMaterials = () => {
            const l = document.getElementById("materialsList");
            l.classList.toggle("hidden");
            document.getElementById("matToggleIcon").textContent = l.classList.contains("hidden") ? "+" : "-";
        };

        window.toggleManualMat = () => {
            const c = document.getElementById('chk_manual'), container = document.getElementById('manualInputContainer');
            container.classList.toggle("hidden", !c.checked);
            if(c.checked) document.getElementById('manual_name').focus();
        };

        window.toggleManualInput = (id) => {
            const select = document.getElementById(id);
            const input = document.getElementById(id + "_manual");
            if (select.value === "Other (Type Manually)") {
                input.classList.remove("hidden");
                input.focus();
            } else {
                input.classList.add("hidden");
                input.value = "";
            }
        };

        tg.MainButton.setText("REVIEW & SUBMIT");
        tg.MainButton.show();

        let finalData = {};

        tg.MainButton.onClick(() => {
            const sr = document.getElementById("sr_no").value;
            if(!sr.trim()) return tg.showAlert("⚠️ Please enter SR Number");

            // Gather Data
            let cutVal = document.getElementById("cut").value;
            if(cutVal === "Other (Type Manually)") cutVal = document.getElementById("cut_manual").value;

            let delayVal = document.getElementById("delay").value;
            if(delayVal === "Other (Type Manually)") delayVal = document.getElementById("delay_manual").value;

            const mats = {};
            MATERIALS.forEach((m, i) => { if(document.getElementById(`c${i}`).checked) mats[m] = document.getElementById(`q${i}`).value || "1"; });
            
            if(document.getElementById('chk_manual').checked) {
                const mName = document.getElementById('manual_name').value.trim();
                const mQty = document.getElementById('manual_qty').value.trim() || "1";
                if(mName) mats[mName] = mQty;
            }

            finalData = {
                SR_NO: sr, 
                RFO1: document.getElementById("rfo1").value, 
                RFO2: document.getElementById("rfo2").value,
                CUT: cutVal, 
                AUTH: document.getElementById("auth").value,
                FAT: document.getElementById("fat").value, 
                DELAY: delayVal, 
                MATERIALS: mats
            };

            // Show Modal
            let summaryHtml = `
                <p><strong>SR NO:</strong> ${finalData.SR_NO}</p>
                <p><strong>RFO 1:</strong> ${finalData.RFO1}</p>
                <p><strong>RFO 2:</strong> ${finalData.RFO2}</p>
                <p><strong>Cut Reason:</strong> ${finalData.CUT}</p>
                <p><strong>Authority:</strong> ${finalData.AUTH}</p>
                <p><strong>FAT/OTB:</strong> ${finalData.FAT}</p>
                <p><strong>Delay:</strong> ${finalData.DELAY}</p>
            `;

            if (Object.keys(mats).length > 0) {
                summaryHtml += `<p class="mt-2 font-bold">Materials:</p><ul class="list-disc ml-5">`;
                for (const [key, value] of Object.entries(mats)) {
                    summaryHtml += `<li>${key}: ${value}</li>`;
                }
                summaryHtml += `</ul>`;
            }

            document.getElementById("summaryContent").innerHTML = summaryHtml;
            document.getElementById("confirmModal").classList.remove("hidden");
        });

        window.submitFinal = () => {
            tg.sendData(JSON.stringify(finalData));
            tg.close();
        };

        window.closeModal = () => {
            document.getElementById("confirmModal").classList.add("hidden");
        };
    </script>
</body>
</html>
