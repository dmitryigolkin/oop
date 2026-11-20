# -*- coding: utf-8 -*-


import tkinter as tk
from tkinter import ttk, messagebox
import json
import os
import re

DATA_FILE = "marshruty.json"
TIME_RE = re.compile(r"^\d{1,2}:\d{2}$")


class PunktMarshruta:
    

    def __init__(self, name, ptype, t_arr, t_dep):
        self.name = name          
        self.ptype = ptype        
        self.t_arr = t_arr        
        self.t_dep = t_dep       

    def to_dict(self):
        return {"name": self.name, "ptype": self.ptype,
                "t_arr": self.t_arr, "t_dep": self.t_dep}

    @staticmethod
    def from_dict(d):
        return PunktMarshruta(d["name"], d["ptype"], d["t_arr"], d["t_dep"])

    def __str__(self):
        return f"{self.name} [{self.ptype}] {self.t_arr} - {self.t_dep}"


class Marshrut:
  

    def __init__(self, number, title):
        self.number = number      
        self.title = title       
        self.points = []       

    @property
    def label(self):
        return f"№ {self.number} — {self.title}"

    def add_point(self, p):
        self.points.append(p)

    def remove_point(self, idx):
        if 0 <= idx < len(self.points):
            self.points.pop(idx)
            return True
        return False

    def edit_point(self, idx, p):
        if 0 <= idx < len(self.points):
            self.points[idx] = p
            return True
        return False

    def to_dict(self):
        return {"number": self.number, "title": self.title,
                "points": [p.to_dict() for p in self.points]}

    @staticmethod
    def from_dict(d):
        m = Marshrut(d["number"], d["title"])
        m.points = [PunktMarshruta.from_dict(x) for x in d["points"]]
        return m


def check_time(value, allow_dash=False):
    """Проверка формата времени ЧЧ:ММ (00:00 - 23:59)."""
    if allow_dash and value == "--":
        return True
    if not TIME_RE.match(value):
        return False
    h, m = map(int, value.split(":"))
    return 0 <= h < 24 and 0 <= m < 60


class App(tk.Tk):
    def __init__(self):
        super().__init__()
        self.title("Учет маршрутов пассажирских поездов")
        self.geometry("920x560")
        self.routes = []
        self.build_ui()
        self.load_data()


    def build_ui(self):
        top = ttk.LabelFrame(self, text="Новый маршрут")
        top.pack(fill="x", padx=8, pady=6)
        ttk.Label(top, text="Номер поезда:").grid(row=0, column=0, padx=4, pady=4)
        self.e_num = ttk.Entry(top, width=10)
        self.e_num.grid(row=0, column=1, padx=4)
        ttk.Label(top, text="Название маршрута:").grid(row=0, column=2, padx=4)
        self.e_title = ttk.Entry(top, width=40)
        self.e_title.grid(row=0, column=3, padx=4)
        ttk.Button(top, text="Создать маршрут", command=self.create_route)\
            .grid(row=0, column=4, padx=8)

        left = ttk.LabelFrame(self, text="Маршруты")
        left.pack(side="left", fill="y", padx=8, pady=6)
        self.lb = tk.Listbox(left, width=34, height=20)
        self.lb.pack(padx=4, pady=4)
        self.lb.bind("<<ListboxSelect>>", self.on_select_route)
        ttk.Button(left, text="Удалить маршрут", command=self.delete_route)\
            .pack(fill="x", padx=4, pady=(0, 4))

        right = ttk.LabelFrame(self, text="Пункты маршрута")
        right.pack(side="left", fill="both", expand=True, padx=8, pady=6)

        form = ttk.Frame(right)
        form.pack(fill="x", padx=6, pady=4)
        ttk.Label(form, text="Станция:").grid(row=0, column=0, sticky="w")
        self.e_station = ttk.Entry(form, width=24)
        self.e_station.grid(row=1, column=0, padx=4, sticky="we")
        ttk.Label(form, text="Тип пункта:").grid(row=0, column=1, sticky="w")
        self.cb_type = ttk.Combobox(form, width=16, state="readonly",
                                    values=["отправление", "прибытие", "промежуточная"])
        self.cb_type.current(0)
        self.cb_type.grid(row=1, column=1, padx=4)
        ttk.Label(form, text="Прибытие (ЧЧ:ММ):").grid(row=0, column=2, sticky="w")
        self.e_arr = ttk.Entry(form, width=9)
        self.e_arr.grid(row=1, column=2, padx=4)
        ttk.Label(form, text="Отправление (ЧЧ:ММ):").grid(row=0, column=3, sticky="w")
        self.e_dep = ttk.Entry(form, width=9)
        self.e_dep.grid(row=1, column=3, padx=4)

        btns = ttk.Frame(right)
        btns.pack(fill="x", padx=6)
        ttk.Button(btns, text="Добавить пункт", command=self.add_point).pack(side="left", padx=3)
        ttk.Button(btns, text="Изменить пункт", command=self.edit_point).pack(side="left", padx=3)
        ttk.Button(btns, text="Удалить пункт", command=self.delete_point).pack(side="left", padx=3)

        cols = ("station", "ptype", "arr", "dep")
        self.tv = ttk.Treeview(right, columns=cols, show="headings", height=13)
        for c, w, t in (("station", 200, "Станция"), ("ptype", 120, "Тип пункта"),
                        ("arr", 90, "Прибытие"), ("dep", 90, "Отправление")):
            self.tv.heading(c, text=t)
            self.tv.column(c, width=w, anchor="w")
        self.tv.pack(fill="both", expand=True, padx=6, pady=6)
        self.tv.bind("<<TreeviewSelect>>", self.on_select_point)

        bottom = ttk.Frame(self)
        bottom.pack(fill="x", padx=8, pady=(0, 8))
        ttk.Button(bottom, text="Сохранить в файл", command=self.save_data).pack(side="left")

    def current_route(self):
        sel = self.lb.curselection()
        return self.routes[sel[0]] if sel else None

    def refresh_routes(self):
        cur = self.lb.curselection()
        self.lb.delete(0, "end")
        for r in self.routes:
            self.lb.insert("end", r.label)
        if cur and cur[0] < len(self.routes):
            self.lb.selection_set(cur[0])

    def refresh_points(self):
        self.tv.delete(*self.tv.get_children())
        r = self.current_route()
        if r:
            for i, p in enumerate(r.points):
                self.tv.insert("", "end", iid=str(i),
                               values=(p.name, p.ptype, p.t_arr, p.t_dep))

    def create_route(self):
        num = self.e_num.get().strip()
        title = self.e_title.get().strip()
        if not num or not title:
            messagebox.showwarning("Внимание", "Заполните номер поезда и название маршрута")
            return
        if any(r.number == num for r in self.routes):
            messagebox.showwarning("Внимание", "Маршрут с таким номером уже есть")
            return
        self.routes.append(Marshrut(num, title))
        self.refresh_routes()
        self.lb.selection_set("end")
        self.e_num.delete(0, "end")
        self.e_title.delete(0, "end")

    def delete_route(self):
        r = self.current_route()
        if not r:
            messagebox.showinfo("Информация", "Выберите маршрут в списке")
            return
        if messagebox.askyesno("Удаление", f"Удалить маршрут {r.label}?"):
            self.routes.remove(r)
            self.refresh_routes()
            self.refresh_points()


    def read_point_from_form(self):
        name = self.e_station.get().strip()
        ptype = self.cb_type.get()
        t_arr = self.e_arr.get().strip() or "--"
        t_dep = self.e_dep.get().strip() or "--"
        if not name:
            messagebox.showwarning("Внимание", "Введите название станции")
            return None
        for t, txt in ((t_arr, "прибытия"), (t_dep, "отправления")):
            if not check_time(t, allow_dash=True):
                messagebox.showwarning("Внимание",
                                       f"Время {txt} должно быть в формате ЧЧ:ММ")
                return None
        if ptype == "отправление":
            t_arr = "--"
        if ptype == "прибытие":
            t_dep = "--"
        return PunktMarshruta(name, ptype, t_arr, t_dep)

    def add_point(self):
        r = self.current_route()
        if not r:
            messagebox.showinfo("Информация", "Сначала выберите маршрут")
            return
        p = self.read_point_from_form()
        if p:
            r.add_point(p)
            self.refresh_points()

    def edit_point(self):
        r = self.current_route()
        sel = self.tv.selection()
        if not r or not sel:
            messagebox.showinfo("Информация", "Выберите пункт в таблице")
            return
        p = self.read_point_from_form()
        if p:
            r.edit_point(int(sel[0]), p)
            self.refresh_points()

    def delete_point(self):
        r = self.current_route()
        sel = self.tv.selection()
        if not r or not sel:
            messagebox.showinfo("Информация", "Выберите пункт в таблице")
            return
        r.remove_point(int(sel[0]))
        self.refresh_points()

    def on_select_route(self, _e=None):
        self.refresh_points()

    def on_select_point(self, _e=None):
        r = self.current_route()
        sel = self.tv.selection()
        if not r or not sel:
            return
        p = r.points[int(sel[0])]
        self.e_station.delete(0, "end"); self.e_station.insert(0, p.name)
        self.cb_type.set(p.ptype)
        self.e_arr.delete(0, "end")
        if p.t_arr != "--": self.e_arr.insert(0, p.t_arr)
        self.e_dep.delete(0, "end")
        if p.t_dep != "--": self.e_dep.insert(0, p.t_dep)


    def save_data(self):
        with open(DATA_FILE, "w", encoding="utf-8") as f:
            json.dump([r.to_dict() for r in self.routes], f, ensure_ascii=False, indent=2)
        messagebox.showinfo("Сохранение", f"Данные сохранены в файл {DATA_FILE}")

    def load_data(self):
        if os.path.exists(DATA_FILE):
            try:
                with open(DATA_FILE, "r", encoding="utf-8") as f:
                    self.routes = [Marshrut.from_dict(d) for d in json.load(f)]
                self.refresh_routes()
                if self.routes:
                    self.lb.selection_set(0)
                    self.refresh_points()
            except (ValueError, KeyError):
                self.routes = []


if __name__ == "__main__":
    App().mainloop()
