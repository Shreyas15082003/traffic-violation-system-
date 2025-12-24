import tkinter as tk
from tkinter import ttk, messagebox
from datetime import datetime

# ---------------------------
# Utility and validation
# ---------------------------
VIOLATION_TYPES = [
    "Speeding",
    "Signal Jump",
    "Wrong Parking",
    "No Helmet/Seatbelt",
    "Driving Without License",
    "Dangerous Driving",
    "Other",
]

PAYMENT_STATUS = ["Pending", "Paid", "Disputed"]

def validate_vehicle_number(vnum: str) -> bool:
    """
    Basic validation for Indian-style vehicle numbers.
    Example formats: KA01AB1234, KA-01-AB-1234
    """
    v = vnum.replace("-", "").strip().upper()
    if len(v) < 8 or len(v) > 12:
        return False
    # Must start with 2 letters (state), then 2 digits (RTO), then 1-3 letters (series), then 3-4 digits (number)
    # This is lenient; adjust to your region rules
    return v[:2].isalpha() and v[2:4].isdigit() and v[4:].isalnum()

def auto_penalty(violation_type: str) -> int:
    base = {
        "Speeding": 1000,
        "Signal Jump": 500,
        "Wrong Parking": 300,
        "No Helmet/Seatbelt": 500,
        "Driving Without License": 5000,
        "Dangerous Driving": 2500,
        "Other": 300,
    }
    return base.get(violation_type, 300)

# ---------------------------
# Main Application
# ---------------------------
class TrafficViolationApp(tk.Tk):
    def __init__(self):
        super().__init__()
        self.title("Traffic Violation Management System")
        self.geometry("1100x700")
        self.minsize(1000, 650)

        # In-memory storage: list of dicts
        self.records = []
        self.selected_id = None
        self.record_id_seq = 1

        self._build_ui()

    def _build_ui(self):
        self.columnconfigure(0, weight=1)
        self.rowconfigure(0, weight=0)
        self.rowconfigure(1, weight=1)

        # Header
        header = ttk.Label(self, text="Traffic Violation Management System", font=("Segoe UI", 16, "bold"))
        header.grid(row=0, column=0, sticky="ew", padx=12, pady=(12, 6))

        container = ttk.Frame(self)
        container.grid(row=1, column=0, sticky="nsew", padx=12, pady=12)
        container.columnconfigure(0, weight=1)
        container.columnconfigure(1, weight=1)
        container.rowconfigure(1, weight=1)

        # ------------- Form -------------
        form_frame = ttk.LabelFrame(container, text="Violation Entry")
        form_frame.grid(row=0, column=0, sticky="nsew", padx=(0, 8), pady=(0, 8))
        for i in range(4):
            form_frame.columnconfigure(i, weight=1)

        # Fields
        self.var_vehicle = tk.StringVar()
        self.var_license = tk.StringVar()
        self.var_owner = tk.StringVar()
        self.var_vehicle_type = tk.StringVar()
        self.var_violation_type = tk.StringVar(value=VIOLATION_TYPES[0])
        self.var_location = tk.StringVar()
        self.var_datetime = tk.StringVar(value=datetime.now().strftime("%Y-%m-%d %H:%M"))
        self.var_officer_id = tk.StringVar()
        self.var_penalty = tk.StringVar(value=str(auto_penalty(self.var_violation_type.get())))
        self.var_payment_status = tk.StringVar(value=PAYMENT_STATUS[0])
        self.var_notes = tk.StringVar()

        # Row 0
        ttk.Label(form_frame, text="Vehicle Number").grid(row=0, column=0, sticky="w", padx=8, pady=6)
        ttk.Entry(form_frame, textvariable=self.var_vehicle).grid(row=0, column=1, sticky="ew", padx=8, pady=6)

        ttk.Label(form_frame, text="Driver License No").grid(row=0, column=2, sticky="w", padx=8, pady=6)
        ttk.Entry(form_frame, textvariable=self.var_license).grid(row=0, column=3, sticky="ew", padx=8, pady=6)

        # Row 1
        ttk.Label(form_frame, text="Owner Name").grid(row=1, column=0, sticky="w", padx=8, pady=6)
        ttk.Entry(form_frame, textvariable=self.var_owner).grid(row=1, column=1, sticky="ew", padx=8, pady=6)

        ttk.Label(form_frame, text="Vehicle Type").grid(row=1, column=2, sticky="w", padx=8, pady=6)
        ttk.Entry(form_frame, textvariable=self.var_vehicle_type).grid(row=1, column=3, sticky="ew", padx=8, pady=6)

        # Row 2
        ttk.Label(form_frame, text="Violation Type").grid(row=2, column=0, sticky="w", padx=8, pady=6)
        vt = ttk.Combobox(form_frame, values=VIOLATION_TYPES, textvariable=self.var_violation_type, state="readonly")
        vt.grid(row=2, column=1, sticky="ew", padx=8, pady=6)
        vt.bind("<<ComboboxSelected>>", self._update_penalty_from_type)

        ttk.Label(form_frame, text="Location").grid(row=2, column=2, sticky="w", padx=8, pady=6)
        ttk.Entry(form_frame, textvariable=self.var_location).grid(row=2, column=3, sticky="ew", padx=8, pady=6)

        # Row 3
        ttk.Label(form_frame, text="Date & Time (YYYY-MM-DD HH:MM)").grid(row=3, column=0, sticky="w", padx=8, pady=6)
        ttk.Entry(form_frame, textvariable=self.var_datetime).grid(row=3, column=1, sticky="ew", padx=8, pady=6)

        ttk.Label(form_frame, text="Officer ID").grid(row=3, column=2, sticky="w", padx=8, pady=6)
        ttk.Entry(form_frame, textvariable=self.var_officer_id).grid(row=3, column=3, sticky="ew", padx=8, pady=6)

        # Row 4
        ttk.Label(form_frame, text="Penalty Amount (₹)").grid(row=4, column=0, sticky="w", padx=8, pady=6)
        ttk.Entry(form_frame, textvariable=self.var_penalty).grid(row=4, column=1, sticky="ew", padx=8, pady=6)

        ttk.Label(form_frame, text="Payment Status").grid(row=4, column=2, sticky="w", padx=8, pady=6)
        ttk.Combobox(form_frame, values=PAYMENT_STATUS, textvariable=self.var_payment_status, state="readonly").grid(
            row=4, column=3, sticky="ew", padx=8, pady=6
        )

        # Row 5
        ttk.Label(form_frame, text="Notes").grid(row=5, column=0, sticky="w", padx=8, pady=6)
        ttk.Entry(form_frame, textvariable=self.var_notes).grid(row=5, column=1, columnspan=3, sticky="ew", padx=8, pady=6)

        # Buttons
        btns = ttk.Frame(form_frame)
        btns.grid(row=6, column=0, columnspan=4, sticky="ew", padx=8, pady=8)
        btns.columnconfigure((0,1,2,3,4), weight=1)

        ttk.Button(btns, text="Add", command=self.add_record).grid(row=0, column=0, sticky="ew", padx=4)
        ttk.Button(btns, text="Update", command=self.update_record).grid(row=0, column=1, sticky="ew", padx=4)
        ttk.Button(btns, text="Delete", command=self.delete_record).grid(row=0, column=2, sticky="ew", padx=4)
        ttk.Button(btns, text="Clear Form", command=self.clear_form).grid(row=0, column=3, sticky="ew", padx=4)
        ttk.Button(btns, text="Load Selected", command=self.load_selected).grid(row=0, column=4, sticky="ew", padx=4)

        # ------------- Search -------------
        search_frame = ttk.LabelFrame(container, text="Search")
        search_frame.grid(row=0, column=1, sticky="nsew", padx=(8, 0), pady=(0, 8))
        for i in range(3):
            search_frame.columnconfigure(i, weight=1)

        self.var_search_vehicle = tk.StringVar()
        self.var_search_type = tk.StringVar()
        self.var_search_status = tk.StringVar()

        ttk.Label(search_frame, text="Vehicle Number").grid(row=0, column=0, sticky="w", padx=8, pady=6)
        ttk.Entry(search_frame, textvariable=self.var_search_vehicle).grid(row=0, column=1, sticky="ew", padx=8, pady=6)

        ttk.Label(search_frame, text="Violation Type").grid(row=1, column=0, sticky="w", padx=8, pady=6)
        ttk.Combobox(search_frame, values=VIOLATION_TYPES + [""], textvariable=self.var_search_type, state="readonly").grid(
            row=1, column=1, sticky="ew", padx=8, pady=6
        )

        ttk.Label(search_frame, text="Payment Status").grid(row=2, column=0, sticky="w", padx=8, pady=6)
        ttk.Combobox(search_frame, values=PAYMENT_STATUS + [""], textvariable=self.var_search_status, state="readonly").grid(
            row=2, column=1, sticky="ew", padx=8, pady=6
        )

        sb = ttk.Frame(search_frame)
        sb.grid(row=3, column=0, columnspan=3, sticky="ew", padx=8, pady=8)
        sb.columnconfigure((0,1,2), weight=1)

        ttk.Button(sb, text="Apply Filters", command=self.apply_filters).grid(row=0, column=0, sticky="ew", padx=4)
        ttk.Button(sb, text="Reset Filters", command=self.reset_filters).grid(row=0, column=1, sticky="ew", padx=4)
        ttk.Button(sb, text="Export CSV", command=self.export_csv).grid(row=0, column=2, sticky="ew", padx=4)

        # ------------- Table -------------
        table_frame = ttk.LabelFrame(container, text="Violations")
        table_frame.grid(row=1, column=0, columnspan=2, sticky="nsew")
        table_frame.columnconfigure(0, weight=1)
        table_frame.rowconfigure(0, weight=1)

        columns = (
            "id", "vehicle", "license", "owner", "type", "location", "dt", "officer",
            "penalty", "status", "notes"
        )
        self.tree = ttk.Treeview(table_frame, columns=columns, show="headings")
        for col, text in [
            ("id", "ID"),
            ("vehicle", "Vehicle No"),
            ("license", "License No"),
            ("owner", "Owner"),
            ("type", "Violation Type"),
            ("location", "Location"),
            ("dt", "Date & Time"),
            ("officer", "Officer ID"),
            ("penalty", "Penalty (₹)"),
            ("status", "Payment Status"),
            ("notes", "Notes"),
        ]:
            self.tree.heading(col, text=text)
            self.tree.column(col, anchor="w", width=120)
        self.tree.column("id", width=60, anchor="center")
        self.tree.grid(row=0, column=0, sticky="nsew")

        # Scrollbars
        vsb = ttk.Scrollbar(table_frame, orient="vertical", command=self.tree.yview)
        hsb = ttk.Scrollbar(table_frame, orient="horizontal", command=self.tree.xview)
        self.tree.configure(yscrollcommand=vsb.set, xscrollcommand=hsb.set)
        vsb.grid(row=0, column=1, sticky="ns")
        hsb.grid(row=1, column=0, sticky="ew")

        # Sample data
        self.add_sample_data()

    # ---------------------------
    # Form and table operations
    # ---------------------------
    def _update_penalty_from_type(self, _event=None):
        self.var_penalty.set(str(auto_penalty(self.var_violation_type.get())))

    def clear_form(self):
        self.selected_id = None
        self.var_vehicle.set("")
        self.var_license.set("")
        self.var_owner.set("")
        self.var_vehicle_type.set("")
        self.var_violation_type.set(VIOLATION_TYPES[0])
        self.var_location.set("")
        self.var_datetime.set(datetime.now().strftime("%Y-%m-%d %H:%M"))
        self.var_officer_id.set("")
        self.var_penalty.set(str(auto_penalty(self.var_violation_type.get())))
        self.var_payment_status.set(PAYMENT_STATUS[0])
        self.var_notes.set("")

    def _validate_form(self) -> bool:
        vnum = self.var_vehicle.get().strip()
        if not vnum or not validate_vehicle_number(vnum):
            messagebox.showwarning("Validation", "Enter a valid vehicle number (e.g., KA01AB1234).")
            return False
        dt_str = self.var_datetime.get().strip()
        try:
            datetime.strptime(dt_str, "%Y-%m-%d %H:%M")
        except ValueError:
            messagebox.showwarning("Validation", "Date & Time must be in format YYYY-MM-DD HH:MM.")
            return False
        try:
            pen = int(self.var_penalty.get())
            if pen < 0:
                raise ValueError
        except ValueError:
            messagebox.showwarning("Validation", "Penalty must be a non-negative integer.")
            return False
        status = self.var_payment_status.get()
        if status not in PAYMENT_STATUS:
            messagebox.showwarning("Validation", "Select a valid payment status.")
            return False
        return True

    def add_record(self):
        if not self._validate_form():
            return
        record = {
            "id": self.record_id_seq,
            "vehicle": self.var_vehicle.get().strip().upper(),
            "license": self.var_license.get().strip().upper(),
            "owner": self.var_owner.get().strip(),
            "vtype": self.var_vehicle_type.get().strip(),
            "type": self.var_violation_type.get(),
            "location": self.var_location.get().strip(),
            "dt": self.var_datetime.get().strip(),
            "officer": self.var_officer_id.get().strip(),
            "penalty": int(self.var_penalty.get()),
            "status": self.var_payment_status.get(),
            "notes": self.var_notes.get().strip(),
        }
        self.records.append(record)
        self.record_id_seq += 1
        self._insert_tree_item(record)
        self.clear_form()
        messagebox.showinfo("Success", "Violation added.")

    def _insert_tree_item(self, r):
        self.tree.insert("", "end", iid=str(r["id"]), values=(
            r["id"], r["vehicle"], r["license"], r["owner"], r["type"], r["location"], r["dt"],
            r["officer"], r["penalty"], r["status"], r["notes"]
        ))

    def load_selected(self):
        sel = self.tree.selection()
        if not sel:
            messagebox.showinfo("Info", "Select a record in the table.")
            return
        rid = int(sel[0])
        rec = next((r for r in self.records if r["id"] == rid), None)
        if not rec:
            messagebox.showerror("Error", "Record not found.")
            return
        self.selected_id = rid
        self.var_vehicle.set(rec["vehicle"])
        self.var_license.set(rec["license"])
        self.var_owner.set(rec["owner"])
        self.var_vehicle_type.set(rec["vtype"])
        self.var_violation_type.set(rec["type"])
        self.var_location.set(rec["location"])
        self.var_datetime.set(rec["dt"])
        self.var_officer_id.set(rec["officer"])
        self.var_penalty.set(str(rec["penalty"]))
        self.var_payment_status.set(rec["status"])
        self.var_notes.set(rec["notes"])

    def update_record(self):
        if self.selected_id is None:
            messagebox.showinfo("Info", "Load a record to update.")
            return
        if not self._validate_form():
            return
        rec = next((r for r in self.records if r["id"] == self.selected_id), None)
        if not rec:
            messagebox.showerror("Error", "Record not found.")
            return
        rec.update({
            "vehicle": self.var_vehicle.get().strip().upper(),
            "license": self.var_license.get().strip().upper(),
            "owner": self.var_owner.get().strip(),
            "vtype": self.var_vehicle_type.get().strip(),
            "type": self.var_violation_type.get(),
            "location": self.var_location.get().strip(),
            "dt": self.var_datetime.get().strip(),
            "officer": self.var_officer_id.get().strip(),
            "penalty": int(self.var_penalty.get()),
            "status": self.var_payment_status.get(),
            "notes": self.var_notes.get().strip(),
        })
        # Refresh tree item
        self.tree.item(str(rec["id"]), values=(
            rec["id"], rec["vehicle"], rec["license"], rec["owner"], rec["type"], rec["location"],
            rec["dt"], rec["officer"], rec["penalty"], rec["status"], rec["notes"]
        ))
        messagebox.showinfo("Success", "Record updated.")

    def delete_record(self):
        sel = self.tree.selection()
        if not sel:
            messagebox.showinfo("Info", "Select a record to delete.")
            return
        rid = int(sel[0])
        if messagebox.askyesno("Confirm", f"Delete record ID {rid}?"):
            self.records = [r for r in self.records if r["id"] != rid]
            self.tree.delete(sel[0])
            if self.selected_id == rid:
                self.selected_id = None
                self.clear_form()

    def apply_filters(self):
        veh = self.var_search_vehicle.get().strip().upper()
        vtype = self.var_search_type.get().strip()
        status = self.var_search_status.get().strip()

        def matches(r):
            if veh and veh not in r["vehicle"]:
                return False
            if vtype and r["type"] != vtype:
                return False
            if status and r["status"] != status:
                return False
            return True

        # Clear tree
        for i in self.tree.get_children():
            self.tree.delete(i)
        # Re-insert filtered
        for r in filter(matches, self.records):
            self._insert_tree_item(r)

    def reset_filters(self):
        self.var_search_vehicle.set("")
        self.var_search_type.set("")
        self.var_search_status.set("")
        # Reload all
        for i in self.tree.get_children():
            self.tree.delete(i)
        for r in self.records:
            self._insert_tree_item(r)

    def export_csv(self):
        import csv
        from tkinter.filedialog import asksaveasfilename
        path = asksaveasfilename(defaultextension=".csv", filetypes=[("CSV Files", "*.csv")])
        if not path:
            return
        cols = ["id","vehicle","license","owner","vtype","type","location","dt","officer","penalty","status","notes"]
        try:
            with open(path, "w", newline="", encoding="utf-8") as f:
                writer = csv.DictWriter(f, fieldnames=cols)
                writer.writeheader()
                for r in self.records:
                    writer.writerow(r)
            messagebox.showinfo("Export", f"CSV exported to:\n{path}")
        except Exception as e:
            messagebox.showerror("Export Error", str(e))

    def add_sample_data(self):
        samples = [
            dict(vehicle="KA01AB1234", license="KA1234567890", owner="Ravi Kumar", vtype="Car",
                 type="Speeding", location="Mysuru Ring Road", dt="2025-12-24 10:30",
                 officer="OF123", penalty=auto_penalty("Speeding"), status="Pending", notes="Radar: 78 km/h"),
            dict(vehicle="KA05MN4321", license="KA0987654321", owner="Anita Rao", vtype="Bike",
                 type="No Helmet/Seatbelt", location="Devaraja Mohalla", dt="2025-12-24 09:15",
                 officer="OF451", penalty=auto_penalty("No Helmet/Seatbelt"), status="Paid", notes="Helmet missing"),
        ]
        for s in samples:
            s["id"] = self.record_id_seq
            self.record_id_seq += 1
            self.records.append(s)
            self._insert_tree_item(s)

if __name__ == "__main__":
    app = TrafficViolationApp()
    # Use ttk theme for cleaner look
    style = ttk.Style(app)
    try:
        style.theme_use("clam")
    except tk.TclError:
        pass
    app.mainloop()
