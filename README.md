[index.html](https://github.com/user-attachments/files/23841029/index.html)
import tkinter as tk
from tkinter import ttk, messagebox
import pandas as pd
import os
from datetime import datetime

# --- CONFIGURATION ---
EXCEL_FILE = r"C:\Users\sonus\Downloads\materialbot\materials.xlsx"

# --- DATA OPTIONS ---
DATA = {
    "RFO1": [
        "FAT_OTB_related", "Fiber_cut_OH_cable_restored", "Fiber_cut_UG_cable_restored",
        "NFF_Wrong_TT", "ODF_related", "OLT_related", "Planned_Cutover_Rehab",
        "Auto_TT_Activity_Completed", "Copper_Cable_Cut", "Copper_Cable_Faulty", "Proactive"
    ],
    "CUT": ["Inside FAT", "Civil Work", "Rat Bite", "Tree Fall", "Vehicle Hit"],
    "AUTH": ["KSEB", "PWD", "Corporation", "Private", "None"],
    "FAT": ["FAT Related", "OTB Related", "Both", "N/A"],
    "DELAY": ["No Delay", "Rain", "Traffic", "Night Work", "Access Issue"],
    "DEFAULT_RFO2": ["Single Fiber Break", "Multi Fiber Break", "Patch Cord", "Splitter"]
}

RFO2_MAPPING = {
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
}

MATERIALS_LIST = [
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
]

COLUMNS = ["Date", "User", "SR NO", "RFO1", "RFO2", "REASON FOR CUT", "AUTHORITY", "FAT/OTB/ODF/OLT", "DELAY REASON"] + MATERIALS_LIST

class MaterialApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Material Data Entry")
        self.root.geometry("600x800")
        
        # Initialize Excel
        self.init_excel()

        # --- UI LAYOUT ---
        main_frame = ttk.Frame(root, padding="20")
        main_frame.pack(fill=tk.BOTH, expand=True)
        
        # Canvas for scrolling
        canvas = tk.Canvas(main_frame)
        scrollbar = ttk.Scrollbar(main_frame, orient="vertical", command=canvas.yview)
        self.scrollable_frame = ttk.Frame(canvas)

        self.scrollable_frame.bind(
            "<Configure>",
            lambda e: canvas.configure(scrollregion=canvas.bbox("all"))
        )

        canvas.create_window((0, 0), window=self.scrollable_frame, anchor="nw")
        canvas.configure(yscrollcommand=scrollbar.set)

        canvas.pack(side="left", fill="both", expand=True)
        scrollbar.pack(side="right", fill="y")

        # --- FORM FIELDS ---
        self.create_label("SR NO (Separate multiple with comma):")
        self.entry_sr = ttk.Entry(self.scrollable_frame, width=50)
        self.entry_sr.pack(pady=5)

        self.create_label("RFO 1:")
        self.var_rfo1 = tk.StringVar()
        self.combo_rfo1 = ttk.Combobox(self.scrollable_frame, textvariable=self.var_rfo1, values=DATA["RFO1"], width=47, state="readonly")
        self.combo_rfo1.pack(pady=5)
        self.combo_rfo1.bind("<<ComboboxSelected>>", self.update_rfo2)

        self.create_label("RFO 2 (Sub Reason):")
        self.var_rfo2 = tk.StringVar()
        self.combo_rfo2 = ttk.Combobox(self.scrollable_frame, textvariable=self.var_rfo2, width=47, state="readonly")
        self.combo_rfo2.pack(pady=5)

        self.create_label("Reason For Cut:")
        self.var_cut = tk.StringVar()
        self.combo_cut = ttk.Combobox(self.scrollable_frame, textvariable=self.var_cut, values=DATA["CUT"], width=47, state="readonly")
        self.combo_cut.pack(pady=5)

        self.create_label("Authority:")
        self.var_auth = tk.StringVar()
        self.combo_auth = ttk.Combobox(self.scrollable_frame, textvariable=self.var_auth, values=DATA["AUTH"], width=47, state="readonly")
        self.combo_auth.pack(pady=5)

        self.create_label("FAT/OTB:")
        self.var_fat = tk.StringVar()
        self.combo_fat = ttk.Combobox(self.scrollable_frame, textvariable=self.var_fat, values=DATA["FAT"], width=47, state="readonly")
        self.combo_fat.pack(pady=5)

        self.create_label("Delay Reason:")
        self.var_delay = tk.StringVar()
        self.combo_delay = ttk.Combobox(self.scrollable_frame, textvariable=self.var_delay, values=DATA["DELAY"], width=47, state="readonly")
        self.combo_delay.pack(pady=5)

        # --- MATERIALS SECTION ---
        ttk.Separator(self.scrollable_frame, orient='horizontal').pack(fill='x', pady=20)
        self.create_label("MATERIALS USED (Check to add quantity):")
        
        self.material_vars = {} # Stores checkboxes (IntVar)
        self.material_entries = {} # Stores quantity entries (Entry)

        for mat in MATERIALS_LIST:
            frame = ttk.Frame(self.scrollable_frame)
            frame.pack(fill='x', pady=2)
            
            var = tk.IntVar()
            chk = ttk.Checkbutton(frame, text=mat, variable=var, command=lambda m=mat: self.toggle_material_entry(m))
            chk.pack(side='left')
            self.material_vars[mat] = var
            
            entry = ttk.Entry(frame, width=10, state='disabled')
            entry.pack(side='right')
            self.material_entries[mat] = entry

        # --- BUTTONS ---
        btn_frame = ttk.Frame(self.scrollable_frame)
        btn_frame.pack(pady=20, fill='x')
        
        save_btn = tk.Button(btn_frame, text="✅ SAVE ENTRY", bg="green", fg="white", font=("Arial", 12, "bold"), command=self.save_data)
        save_btn.pack(side='left', fill='x', expand=True, padx=5)
        
        clear_btn = tk.Button(btn_frame, text="🗑 CLEAR", bg="red", fg="white", font=("Arial", 12), command=self.clear_form)
        clear_btn.pack(side='right', fill='x', expand=True, padx=5)

    def create_label(self, text):
        ttk.Label(self.scrollable_frame, text=text, font=("Arial", 10, "bold")).pack(anchor="w", pady=(10, 0))

    def init_excel(self):
        directory = os.path.dirname(EXCEL_FILE)
        if not os.path.exists(directory):
            os.makedirs(directory)
        if not os.path.exists(EXCEL_FILE):
            df = pd.DataFrame(columns=COLUMNS)
            try:
                df.to_excel(EXCEL_FILE, index=False)
            except Exception as e:
                messagebox.showerror("Error", f"Could not create Excel file: {e}")

    def update_rfo2(self, event):
        selection = self.var_rfo1.get()
        options = RFO2_MAPPING.get(selection, DATA["DEFAULT_RFO2"])
        self.combo_rfo2['values'] = options
        self.combo_rfo2.set('')

    def toggle_material_entry(self, mat):
        if self.material_vars[mat].get() == 1:
            self.material_entries[mat].config(state='normal')
            self.material_entries[mat].focus()
        else:
            self.material_entries[mat].delete(0, tk.END)
            self.material_entries[mat].config(state='disabled')

    def save_data(self):
        sr_raw = self.entry_sr.get()
        if not sr_raw:
            messagebox.showwarning("Missing Data", "Please enter SR Number(s)")
            return

        sr_list = [s.strip() for s in sr_raw.replace('\n', ',').split(',') if s.strip()]
        
        rows = []
        user = os.getlogin() # Get PC Username
        
        for sr in sr_list:
            row_data = {
                "Date": datetime.now().strftime('%Y-%m-%d %H:%M:%S'),
                "User": user,
                "SR NO": sr,
                "RFO1": self.var_rfo1.get(),
                "RFO2": self.var_rfo2.get(),
                "REASON FOR CUT": self.var_cut.get(),
                "AUTHORITY": self.var_auth.get(),
                "FAT/OTB/ODF/OLT": self.var_fat.get(),
                "DELAY REASON": self.var_delay.get()
            }
            
            # Add materials
            for mat in MATERIALS_LIST:
                if self.material_vars[mat].get() == 1:
                    qty = self.material_entries[mat].get()
                    row_data[mat] = qty if qty else "1"
                else:
                    row_data[mat] = ""
            
            rows.append(row_data)

        try:
            if os.path.exists(EXCEL_FILE):
                df = pd.read_excel(EXCEL_FILE)
            else:
                df = pd.DataFrame(columns=COLUMNS)
            
            new_rows = pd.DataFrame(rows)
            df = pd.concat([df, new_rows], ignore_index=True)
            df.to_excel(EXCEL_FILE, index=False)
            
            messagebox.showinfo("Success", f"Saved {len(rows)} entry/entries successfully!")
            self.clear_form()
            
        except PermissionError:
            messagebox.showerror("Error", "Excel file is open. Please close 'materials.xlsx' and try again.")
        except Exception as e:
            messagebox.showerror("Error", f"Failed to save: {e}")

    def clear_form(self):
        self.entry_sr.delete(0, tk.END)
        self.combo_rfo1.set('')
        self.combo_rfo2.set('')
        self.combo_cut.set('')
        self.combo_auth.set('')
        self.combo_fat.set('')
        self.combo_delay.set('')
        
        for mat in MATERIALS_LIST:
            self.material_vars[mat].set(0)
            self.material_entries[mat].delete(0, tk.END)
            self.material_entries[mat].config(state='disabled')

if __name__ == "__main__":
    root = tk.Tk()
    app = MaterialApp(root)
    root.mainloop()
