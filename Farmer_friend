import flet as ft
import sqlite3
from datetime import datetime
import os

# إعداد قاعدة البيانات المحلية (تُحفظ في هاتف كل مستخدم بشكل مستقل)
def setup_db():
    conn = sqlite3.connect("farmer_friend.db", check_same_thread=False)
    c = conn.cursor()
    c.execute('''CREATE TABLE IF NOT EXISTS income (id INTEGER PRIMARY KEY, date TEXT, item TEXT, amount REAL, z10 REAL, z5 REAL, z25 REAL)''')
    c.execute('''CREATE TABLE IF NOT EXISTS expenses (id INTEGER PRIMARY KEY, date TEXT, water REAL, fert REAL, pest REAL, workers REAL, other REAL, other_desc TEXT)''')
    c.execute('''CREATE TABLE IF NOT EXISTS custom_fields (id INTEGER PRIMARY KEY, field_name TEXT, type TEXT, date TEXT, item TEXT, amount REAL)''')
    conn.commit()
    return conn

def main(page: ft.Page):
    # إعدادات شاشة التطبيق
    page.title = "صديق المزارع"
    page.rtl = True  # تفعيل دعم اللغة العربية من اليمين لليسار
    page.theme_mode = ft.ThemeMode.DARK
    page.window_width = 400  # محاكاة عرض الهاتف في الكمبيوتر
    page.window_height = 800
    page.scroll = ft.ScrollMode.ADAPTIVE

    conn = setup_db()
    c = conn.cursor()

    # ==========================
    # 1. واجهة المدخول العام
    # ==========================
    irrig_type = ft.Dropdown(
        label="مصدر السقي",
        options=[ft.dropdown.Option("غيث (10%)"), ft.dropdown.Option("مكينة (5%)"), ft.dropdown.Option("نقل (2.5%)")],
        value="غيث (10%)"
    )
    inc_item = ft.TextField(label="البيان / المحصول", text_align=ft.TextAlign.RIGHT)
    inc_amount = ft.TextField(label="المبلغ", keyboard_type=ft.KeyboardType.NUMBER)
    inc_totals = ft.Text("إجمالي المدخول: 0", size=16, color=ft.colors.GREEN_400, weight=ft.FontWeight.BOLD)
    inc_list = ft.Column(spacing=10) # قائمة عرض البطاقات

    def load_income():
        inc_list.controls.clear()
        c.execute("SELECT * FROM income ORDER BY id DESC")
        tot_amt = tot_z10 = tot_z5 = tot_z25 = 0
        for r in c.fetchall():
            tot_amt += r[3]; tot_z10 += r[4]; tot_z5 += r[5]; tot_z25 += r[6]
            # تصميم بطاقة العرض لكل عملية
            card = ft.Card(
                color=ft.colors.SURFACE_VARIANT,
                content=ft.Container(padding=15, content=ft.Column([
                    ft.Text(f"التاريخ: {r[1]} | البيان: {r[2]}", weight=ft.FontWeight.BOLD),
                    ft.Text(f"المبلغ: {r[3]:,} ريال", size=18, color=ft.colors.GREEN_300),
                    ft.Text(f"غيث: {r[4]} | مكينة: {r[5]} | نقل: {r[6]}", color=ft.colors.ORANGE_300, size=12)
                ]))
            )
            inc_list.controls.append(card)
        inc_totals.value = f"الإجمالي العام: {tot_amt:,} ريال\n(غيث:{tot_z10:,} | مكينة:{tot_z5:,} | نقل:{tot_z25:,})"
        page.update()

    def add_income(e):
        if not inc_item.value or not inc_amount.value: return
        try:
            amt = float(inc_amount.value)
            z10 = z5 = z25 = 0.0
            if "غيث" in irrig_type.value: z10 = amt * 0.10
            elif "مكينة" in irrig_type.value: z5 = amt * 0.05
            elif "نقل" in irrig_type.value: z25 = amt * 0.025
            
            date = datetime.now().strftime("%Y/%m/%d")
            c.execute("INSERT INTO income (date, item, amount, z10, z5, z25) VALUES (?, ?, ?, ?, ?, ?)", (date, inc_item.value, amt, z10, z5, z25))
            conn.commit()
            inc_item.value = ""; inc_amount.value = ""
            load_income()
            page.snack_bar = ft.SnackBar(ft.Text("تمت إضافة المدخول بنجاح!"), bgcolor=ft.colors.GREEN)
            page.snack_bar.open = True
            page.update()
        except ValueError: pass

    tab_income = ft.Tab(text="المدخول", content=ft.Container(padding=10, content=ft.Column([
        irrig_type, inc_item, inc_amount,
        ft.ElevatedButton("إضافة المدخول وحساب الزكاة", on_click=add_income, width=400, bgcolor=ft.colors.BLUE_700, color=ft.colors.WHITE),
        ft.Divider(), inc_totals, ft.Divider(), inc_list
    ])))

    # ==========================
    # 2. واجهة المصروفات العامة
    # ==========================
    exp_w = ft.TextField(label="المياه", value="0", keyboard_type=ft.KeyboardType.NUMBER, expand=1)
    exp_f = ft.TextField(label="الأسمدة", value="0", keyboard_type=ft.KeyboardType.NUMBER, expand=1)
    exp_p = ft.TextField(label="المبيدات", value="0", keyboard_type=ft.KeyboardType.NUMBER, expand=1)
    exp_wk = ft.TextField(label="العمال", value="0", keyboard_type=ft.KeyboardType.NUMBER, expand=1)
    exp_o = ft.TextField(label="أخرى", value="0", keyboard_type=ft.KeyboardType.NUMBER, expand=1)
    exp_desc = ft.TextField(label="تفاصيل أخرى (بترول...)", expand=1)
    
    exp_totals = ft.Text("إجمالي المصروفات: 0", size=16, color=ft.colors.RED_400, weight=ft.FontWeight.BOLD)
    exp_list = ft.Column(spacing=10)

    def load_expenses():
        exp_list.controls.clear()
        c.execute("SELECT * FROM expenses ORDER BY id DESC")
        tw=tf=tp=twk=to=0
        for r in c.fetchall():
            tw+=r[2]; tf+=r[3]; tp+=r[4]; twk+=r[5]; to+=r[6]
            card = ft.Card(content=ft.Container(padding=15, content=ft.Column([
                ft.Text(f"التاريخ: {r[1]}", weight=ft.FontWeight.BOLD),
                ft.Text(f"مياه:{r[2]} | أسمدة:{r[3]} | مبيدات:{r[4]}", size=12),
                ft.Text(f"عمال:{r[5]} | أخرى:{r[6]} ({r[7]})", size=12),
                ft.Text(f"إجمالي العملية: {r[2]+r[3]+r[4]+r[5]+r[6]:,} ريال", color=ft.colors.RED_300, weight=ft.FontWeight.BOLD)
            ])))
            exp_list.controls.append(card)
        tall = tw+tf+tp+twk+to
        exp_totals.value = f"الإجمالي الكلي: {tall:,} ريال\nمياه:{tw:,} | أسمدة:{tf:,} | مبيدات:{tp:,} | عمال:{twk:,} | أخرى:{to:,}"
        page.update()

    def add_expense(e):
        try:
            w, f, p, wk, o = float(exp_w.value), float(exp_f.value), float(exp_p.value), float(exp_wk.value), float(exp_o.value)
            if sum([w,f,p,wk,o]) == 0: return
            date = datetime.now().strftime("%Y/%m/%d")
            c.execute("INSERT INTO expenses (date, water, fert, pest, workers, other, other_desc) VALUES (?,?,?,?,?,?,?)", (date, w, f, p, wk, o, exp_desc.value))
            conn.commit()
            exp_w.value = exp_f.value = exp_p.value = exp_wk.value = exp_o.value = "0"; exp_desc.value = ""
            load_expenses()
            page.snack_bar = ft.SnackBar(ft.Text("تم تسجيل المصروف!"), bgcolor=ft.colors.GREEN)
            page.snack_bar.open = True
            page.update()
        except ValueError: pass

    tab_expenses = ft.Tab(text="المصروفات", content=ft.Container(padding=10, content=ft.Column([
        ft.Row([exp_w, exp_f]), ft.Row([exp_p, exp_wk]), ft.Row([exp_o, exp_desc]),
        ft.ElevatedButton("تسجيل المصروفات", on_click=add_expense, width=400, bgcolor=ft.colors.RED_700, color=ft.colors.WHITE),
        ft.Divider(), exp_totals, ft.Divider(), exp_list
    ])))

    # ==========================
    # 3. واجهة الحقول المخصصة (مبسطة للهاتف)
    # ==========================
    c_field = ft.TextField(label="اسم الحقل المخصص (مثال: حقل الذرة)")
    c_type = ft.Dropdown(label="نوع العملية", options=[ft.dropdown.Option("مدخول"), ft.dropdown.Option("مصروف")], value="مدخول")
    c_item = ft.TextField(label="البيان / التفاصيل")
    c_amt = ft.TextField(label="المبلغ", keyboard_type=ft.KeyboardType.NUMBER)
    
    c_list = ft.Column(spacing=10)

    def load_custom():
        c_list.controls.clear()
        c.execute("SELECT * FROM custom_fields ORDER BY id DESC")
        for r in c.fetchall():
            color = ft.colors.GREEN_300 if r[2] == "مدخول" else ft.colors.RED_300
            card = ft.Card(content=ft.Container(padding=15, content=ft.Column([
                ft.Text(f"الحقل: {r[1]} | {r[2]}", weight=ft.FontWeight.BOLD, color=color),
                ft.Text(f"التاريخ: {r[3]} | البيان: {r[4]}"),
                ft.Text(f"المبلغ: {r[5]:,} ريال", size=16)
            ])))
            c_list.controls.append(card)
        page.update()

    def add_custom(e):
        if not c_field.value or not c_amt.value: return
        try:
            date = datetime.now().strftime("%Y/%m/%d")
            c.execute("INSERT INTO custom_fields (field_name, type, date, item, amount) VALUES (?, ?, ?, ?, ?)", 
                      (c_field.value, c_type.value, date, c_item.value, float(c_amt.value)))
            conn.commit()
            c_item.value = ""; c_amt.value = ""
            load_custom()
        except ValueError: pass

    tab_custom = ft.Tab(text="حقول مخصصة", content=ft.Container(padding=10, content=ft.Column([
        c_field, c_type, c_item, c_amt,
        ft.ElevatedButton("إضافة للحقل المخصص", on_click=add_custom, width=400, bgcolor=ft.colors.ORANGE_700, color=ft.colors.WHITE),
        ft.Divider(), c_list
    ])))

    # ==========================
    # إعدادات الواجهة الرئيسية
    # ==========================
    tabs = ft.Tabs(selected_index=0, animation_duration=300, tabs=[tab_income, tab_expenses, tab_custom], expand=1)
    
    # حل مشكلة الـ PDF في الهواتف: تصدير كـ HTML يمكن طباعته كـ PDF من المتصفح
    def export_report(e):
        html = f"<html dir='rtl'><head><meta charset='utf-8'><title>تقرير المزرعة</title></head><body style='font-family: Arial; padding: 20px;'>"
        html += "<h2>تقرير نظام صديق المزارع</h2><hr><h3>المدخولات</h3><table border='1' width='100%' style='border-collapse:collapse; text-align:center;'>"
        html += "<tr style='background:#ddd;'><th>التاريخ</th><th>البيان</th><th>المبلغ</th></tr>"
        c.execute("SELECT date, item, amount FROM income"); records = c.fetchall()
        for r in records: html += f"<tr><td>{r[0]}</td><td>{r[1]}</td><td>{r[2]}</td></tr>"
        html += "</table><br><h3>المصروفات</h3><table border='1' width='100%' style='border-collapse:collapse; text-align:center;'>"
        html += "<tr style='background:#ddd;'><th>التاريخ</th><th>مياه</th><th>أسمدة</th><th>مبيدات</th><th>عمال</th><th>أخرى</th></tr>"
        c.execute("SELECT date, water, fert, pest, workers, other FROM expenses"); records = c.fetchall()
        for r in records: html += f"<tr><td>{r[0]}</td><td>{r[1]}</td><td>{r[2]}</td><td>{r[3]}</td><td>{r[4]}</td><td>{r[5]}</td></tr>"
        html += "</table></body></html>"
        
        filepath = os.path.join(os.getcwd(), f"Farm_Report_{datetime.now().strftime('%Y%m%d')}.html")
        with open(filepath, "w", encoding="utf-8") as f:
            f.write(html)
            
        page.snack_bar = ft.SnackBar(ft.Text(f"تم الحفظ في: {filepath} (افتحه بمتصفحك واطبعه كـ PDF)"), bgcolor=ft.colors.BLUE)
        page.snack_bar.open = True
        page.update()

    page.appbar = ft.AppBar(
        title=ft.Text("صديق المزارع 🌾", weight=ft.FontWeight.BOLD),
        center_title=True, bgcolor=ft.colors.GREEN_800,
        actions=[ft.IconButton(ft.icons.PRINT, on_click=export_report, tooltip="تصدير التقرير")]
    )

    page.add(tabs)
    load_income()
    load_expenses()
    load_custom()

ft.app(target=main)
