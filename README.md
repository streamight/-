# -
网安数据库期末大作业
-- 患者表
CREATE TABLE Patient (
    patient_id VARCHAR(20) PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    gender CHAR(2) CHECK (gender IN ('男','女')),
    age INT CHECK (age > 0),
    id_card VARCHAR(18) UNIQUE NOT NULL,
    phone VARCHAR(11),
    address VARCHAR(200)
);

-- 科室表
CREATE TABLE Dept (
    dept_id VARCHAR(10) PRIMARY KEY,
    dept_name VARCHAR(50) UNIQUE NOT NULL,
    room_num INT,
    manager_id VARCHAR(20)
);

-- 医生表
CREATE TABLE Doctor (
    doctor_id VARCHAR(20) PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    gender CHAR(2),
    title VARCHAR(20),
    dept_id VARCHAR(10),
    specialty VARCHAR(100),
    schedule_status VARCHAR(10) DEFAULT '空闲',
    FOREIGN KEY (dept_id) REFERENCES Dept(dept_id)
);

-- 预约表
CREATE TABLE Appointment (
    appt_id VARCHAR(20) PRIMARY KEY,
    patient_id VARCHAR(20),
    doctor_id VARCHAR(20),
    appt_date DATE NOT NULL,
    appt_time VARCHAR(20) NOT NULL,
    appt_status VARCHAR(10) DEFAULT '待就诊',
    FOREIGN KEY (patient_id) REFERENCES Patient(patient_id),
    FOREIGN KEY (doctor_id) REFERENCES Doctor(doctor_id)
);

-- 就诊记录表
CREATE TABLE MedicalRecord (
    record_id VARCHAR(20) PRIMARY KEY,
    patient_id VARCHAR(20),
    doctor_id VARCHAR(20),
    visit_date DATE NOT NULL,
    symptoms VARCHAR(500),
    diagnosis VARCHAR(500),
    prescription_id VARCHAR(20),
    FOREIGN KEY (patient_id) REFERENCES Patient(patient_id),
    FOREIGN KEY (doctor_id) REFERENCES Doctor(doctor_id)
);

-- 缴费记录表
CREATE TABLE Payment (
    payment_id VARCHAR(20) PRIMARY KEY,
    patient_id VARCHAR(20),
    record_id VARCHAR(20) UNIQUE,
    amount DECIMAL(10,2) NOT NULL,
    pay_time DATETIME DEFAULT CURRENT_TIMESTAMP,
    pay_method VARCHAR(20),
    pay_status VARCHAR(10) DEFAULT '未缴费',
    FOREIGN KEY (patient_id) REFERENCES Patient(patient_id),
    FOREIGN KEY (record_id) REFERENCES MedicalRecord(record_id)
);

-- 排班表
CREATE TABLE Schedule (
    schedule_id VARCHAR(20) PRIMARY KEY,
    doctor_id VARCHAR(20),
    dept_id VARCHAR(10),
    schedule_date DATE NOT NULL,
    schedule_time VARCHAR(20) NOT NULL,
    room_num VARCHAR(10),
    FOREIGN KEY (doctor_id) REFERENCES Doctor(doctor_id),
    FOREIGN KEY (dept_id) REFERENCES Dept(dept_id)
);


-- 科室表
INSERT INTO Dept (dept_id, dept_name, room_num, manager_id)
VALUES 
('D001', '内科', 5, 'DOC001'),
('D002', '外科', 3, 'DOC002'),
('D003', '儿科', 2, 'DOC003'),
('D004', '妇产科', 4, 'DOC004'),
('D005', '皮肤科', 2, 'DOC005'),
('D006', '眼科', 3, 'DOC006');

-- 医生表
INSERT INTO Doctor (doctor_id, name, gender, title, dept_id, specialty, schedule_status)
VALUES 
('DOC001', '张医生', '男', '主任医师', 'D001', '高血压、糖尿病', '空闲'),
('DOC002', '李医生', '女', '副主任医师', 'D002', '骨折、外伤', '空闲'),
('DOC003', '王医生', '女', '主治医师', 'D003', '小儿感冒、肺炎', '空闲');

-- 患者表
INSERT INTO Patient (patient_id, name, gender, age, id_card, phone, address)
VALUES 
('P001', '张三', '男', 45, '110101198001011234', '13800138000', '北京市朝阳区'),
('P002', '李四', '女', 30, '110101199502021234', '13900139000', '北京市海淀区'),
('P003', '小明', '男', 5, '110101202003031234', '13700137000', '北京市西城区');

-- 排班表
INSERT INTO Schedule (schedule_id, doctor_id, dept_id, schedule_date, schedule_time, room_num)
VALUES 
('SCH001', 'DOC001', 'D001', '2025-12-28', '09:00-11:00', '101'),
('SCH002', 'DOC002', 'D002', '2025-12-28', '14:00-16:00', '201'),
('SCH003', 'DOC003', 'D003', '2025-12-28', '10:00-12:00', '301');

-- 预约表
INSERT INTO Appointment (appt_id, patient_id, doctor_id, appt_date, appt_time, appt_status)
VALUES 
('APP20251228001', 'P001', 'DOC001', '2025-12-28', '09:30', '待就诊'),
('APP20251228002', 'P002', 'DOC002', '2025-12-28', '14:30', '待就诊'),
('APP20251228003', 'P003', 'DOC003', '2025-12-28', '10:30', '待就诊');

-- 就诊记录表
INSERT INTO MedicalRecord (record_id, patient_id, doctor_id, visit_date, symptoms, diagnosis, prescription_id)
VALUES 
('REC20251228001', 'P001', 'DOC001', '2025-12-28', '头晕、血压高', '高血压1级', 'PRE001'),
('REC20251228002', 'P002', 'DOC002', '2025-12-28', '手臂骨折', '桡骨骨折', 'PRE002'),
('REC20251228003', 'P003', 'DOC003', '2025-12-28', '咳嗽、发烧', '小儿急性支气管炎', 'PRE003');

-- 缴费记录表
INSERT INTO Payment (payment_id, patient_id, record_id, amount, pay_time, pay_method, pay_status)
VALUES 
('PAY20251228001', 'P001', 'REC20251228001', 150.50, '2025-12-28 10:00', '微信支付', '已缴费'),
('PAY20251228002', 'P002', 'REC20251228002', 800.00, '2025-12-28 15:00', '支付宝支付', '已缴费'),
('PAY20251228003', 'P003', 'REC20251228003', 200.80, '2025-12-28 11:00', '现金', '已缴费');


import tkinter as tk
from tkinter import ttk, messagebox
import pymysql
import datetime


# -------------------------- 数据库连接函数 --------------------------
def connect_db():
    """建立数据库连接"""
    try:
        conn = pymysql.connect(
            host='localhost',
            user='root',  # 替换为你的MySQL用户名
            password='haoyu.060319',  # 替换为你的MySQL密码
            database='edhw',  # 替换为你的数据库名
            charset='utf8'
        )
        return conn
    except Exception as e:
        messagebox.showerror("数据库错误", f"连接失败：{str(e)}")
        return None


# -------------------------- 新增：患者信息管理 --------------------------
def add_patient():
    """新增患者信息"""
    patient_id = entry_p_id.get()
    name = entry_p_name.get()
    gender = entry_p_gender.get()
    age = entry_p_age.get()
    id_card = entry_p_idcard.get()
    phone = entry_p_phone.get()
    address = entry_p_address.get()
    if not (patient_id and name and gender and age and id_card):
        messagebox.showwarning("输入错误", "患者ID、姓名、性别、年龄、身份证号为必填！")
        return
    try:
        age = int(age)
    except ValueError:
        messagebox.showwarning("输入错误", "年龄必须是数字！")
        return
    conn = connect_db()
    if not conn:
        return
    cursor = conn.cursor()
    try:
        sql = """
        INSERT INTO Patient (patient_id, name, gender, age, id_card, phone, address)
        VALUES (%s, %s, %s, %s, %s, %s, %s)
        """
        cursor.execute(sql, (patient_id, name, gender, age, id_card, phone, address))
        conn.commit()
        messagebox.showinfo("成功", f"患者{name}信息添加成功！")
        # 清空输入框
        for entry in [entry_p_id, entry_p_name, entry_p_gender, entry_p_age, entry_p_idcard, entry_p_phone,
                      entry_p_address]:
            entry.delete(0, tk.END)
    except Exception as e:
        conn.rollback()
        messagebox.showerror("失败", f"添加失败：{str(e)}")
    finally:
        conn.close()


def query_patient_by_id():
    """通过患者ID查询详情（自动填充姓名/性别等）"""
    patient_id = entry_query_p_id.get()
    if not patient_id:
        messagebox.showwarning("输入错误", "请输入患者ID！")
        return
    conn = connect_db()
    if not conn:
        return
    cursor = conn.cursor()
    try:
        sql = "SELECT name, gender, age, phone, address FROM Patient WHERE patient_id = %s"
        cursor.execute(sql, (patient_id,))
        result = cursor.fetchone()
        if result:
            # 填充到展示标签
            lbl_p_info.config(
                text=f"姓名：{result[0]} | 性别：{result[1]} | 年龄：{result[2]} | 电话：{result[3]} | 地址：{result[4]}")
        else:
            lbl_p_info.config(text="未查询到该患者信息！")
    except Exception as e:
        messagebox.showerror("失败", f"查询失败：{str(e)}")
    finally:
        conn.close()


# -------------------------- 原有功能优化（关联姓名展示） --------------------------
def add_appointment():
    patient_id = entry_appt_p_id.get()
    doctor_id = entry_appt_d_id.get()
    appt_date = entry_appt_date.get()
    appt_time = entry_appt_time.get()
    if not (patient_id and doctor_id and appt_date and appt_time):
        messagebox.showwarning("输入错误", "请填写所有预约信息！")
        return
    # 先查询患者/医生姓名（验证ID是否存在）
    conn = connect_db()
    if not conn:
        return
    cursor = conn.cursor()
    try:
        # 查询患者姓名
        cursor.execute("SELECT name FROM Patient WHERE patient_id = %s", (patient_id,))
        p_name = cursor.fetchone()
        if not p_name:
            messagebox.showerror("错误", "患者ID不存在！")
            return
        # 查询医生姓名
        cursor.execute("SELECT name FROM Doctor WHERE doctor_id = %s", (doctor_id,))
        d_name = cursor.fetchone()
        if not d_name:
            messagebox.showerror("错误", "医生ID不存在！")
            return
        # 插入预约记录
        appt_id = f"APP{datetime.datetime.now().strftime('%Y%m%d%H%M%S')}"
        sql = """
        INSERT INTO Appointment (appt_id, patient_id, doctor_id, appt_date, appt_time, appt_status)
        VALUES (%s, %s, %s, %s, %s, '待就诊')
        """
        cursor.execute(sql, (appt_id, patient_id, doctor_id, appt_date, appt_time))
        conn.commit()
        messagebox.showinfo("成功", f"预约成功！\n患者：{p_name[0]} | 医生：{d_name[0]} | 预约ID：{appt_id}")
        # 清空输入框
        entry_appt_p_id.delete(0, tk.END)
        entry_appt_d_id.delete(0, tk.END)
        entry_appt_date.delete(0, tk.END)
        entry_appt_time.delete(0, tk.END)
    except Exception as e:
        conn.rollback()
        messagebox.showerror("失败", f"预约失败：{str(e)}")
    finally:
        conn.close()


def add_payment():
    """优化：缴费时关联患者姓名展示"""
    record_id = entry_pay_record_id.get()
    patient_id = entry_pay_p_id.get()
    amount = entry_pay_amount.get()
    pay_method = entry_pay_method.get()
    if not (record_id and patient_id and amount and pay_method):
        messagebox.showwarning("输入错误", "请填写所有缴费信息！")
        return
    try:
        amount = float(amount)
    except ValueError:
        messagebox.showwarning("输入错误", "缴费金额必须是数字！")
        return
    # 验证患者ID并查询姓名
    conn = connect_db()
    if not conn:
        return
    cursor = conn.cursor()
    try:
        cursor.execute("SELECT name FROM Patient WHERE patient_id = %s", (patient_id,))
        p_name = cursor.fetchone()
        if not p_name:
            messagebox.showerror("错误", "患者ID不存在！")
            return
        # 插入缴费记录
        payment_id = f"PAY{datetime.datetime.now().strftime('%Y%m%d%H%M%S')}"
        pay_time = datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S')
        sql = """
        INSERT INTO Payment (payment_id, patient_id, record_id, amount, pay_time, pay_method, pay_status)
        VALUES (%s, %s, %s, %s, %s, %s, '已缴费')
        """
        cursor.execute(sql, (payment_id, patient_id, record_id, amount, pay_time, pay_method))
        conn.commit()
        messagebox.showinfo("成功", f"缴费成功！\n患者：{p_name[0]} | 缴费金额：{amount:.2f}元 | 缴费ID：{payment_id}")
        # 清空输入框
        entry_pay_record_id.delete(0, tk.END)
        entry_pay_p_id.delete(0, tk.END)
        entry_pay_amount.delete(0, tk.END)
        entry_pay_method.delete(0, tk.END)
    except Exception as e:
        conn.rollback()
        messagebox.showerror("失败", f"缴费失败：{str(e)}")
    finally:
        conn.close()


def query_income():
    """收入统计（原有功能不变）"""
    query_date = entry_query_date.get()
    if not query_date:
        messagebox.showwarning("输入错误", "请填写查询日期！")
        return
    conn = connect_db()
    if not conn:
        return
    cursor = conn.cursor()
    try:
        sql = """
        SELECT SUM(amount) FROM Payment 
        WHERE DATE(pay_time) = %s AND pay_status = '已缴费'
        """
        cursor.execute(sql, (query_date,))
        total_income = cursor.fetchone()[0] or 0.0
        messagebox.showinfo("统计结果", f"{query_date} 门诊总收入：{total_income:.2f} 元")
        entry_query_date.delete(0, tk.END)
    except Exception as e:
        messagebox.showerror("失败", f"统计失败：{str(e)}")
    finally:
        conn.close()


# -------------------------- 界面重构（含完整信息） --------------------------
# 主窗口
root = tk.Tk()
root.title("社区医院门诊管理系统（完整版）")
root.geometry("900x700")
root.resizable(False, False)

# 标签页容器
tab_control = ttk.Notebook(root)

# 患者信息管理标签页
tab_patient = ttk.Frame(tab_control)
tab_control.add(tab_patient, text="患者信息管理")

# 新增患者信息控件
# 第一行：患者ID、姓名
ttk.Label(tab_patient, text="患者ID：").grid(row=0, column=0, padx=10, pady=10, sticky="e")
entry_p_id = ttk.Entry(tab_patient, width=20)
entry_p_id.grid(row=0, column=1, padx=5, pady=10)

ttk.Label(tab_patient, text="姓名：").grid(row=0, column=2, padx=10, pady=10, sticky="e")
entry_p_name = ttk.Entry(tab_patient, width=20)
entry_p_name.grid(row=0, column=3, padx=5, pady=10)

# 第二行：性别、年龄
ttk.Label(tab_patient, text="性别：").grid(row=1, column=0, padx=10, pady=10, sticky="e")
entry_p_gender = ttk.Entry(tab_patient, width=20)
entry_p_gender.grid(row=1, column=1, padx=5, pady=10)

ttk.Label(tab_patient, text="年龄：").grid(row=1, column=2, padx=10, pady=10, sticky="e")
entry_p_age = ttk.Entry(tab_patient, width=20)
entry_p_age.grid(row=1, column=3, padx=5, pady=10)

# 第三行：身份证号、电话
ttk.Label(tab_patient, text="身份证号：").grid(row=2, column=0, padx=10, pady=10, sticky="e")
entry_p_idcard = ttk.Entry(tab_patient, width=20)
entry_p_idcard.grid(row=2, column=1, padx=5, pady=10)

ttk.Label(tab_patient, text="电话：").grid(row=2, column=2, padx=10, pady=10, sticky="e")
entry_p_phone = ttk.Entry(tab_patient, width=20)
entry_p_phone.grid(row=2, column=3, padx=5, pady=10)

# 第四行：地址
ttk.Label(tab_patient, text="地址：").grid(row=3, column=0, padx=10, pady=10, sticky="e")
entry_p_address = ttk.Entry(tab_patient, width=65)
entry_p_address.grid(row=3, column=1, columnspan=3, padx=5, pady=10)

# 按钮：新增患者
btn_add_patient = ttk.Button(tab_patient, text="新增患者信息", command=add_patient)
btn_add_patient.grid(row=4, column=0, columnspan=2, pady=15)

# 患者ID查询
ttk.Label(tab_patient, text="查询患者ID：").grid(row=5, column=0, padx=10, pady=10, sticky="e")
entry_query_p_id = ttk.Entry(tab_patient, width=20)
entry_query_p_id.grid(row=5, column=1, padx=5, pady=10)

btn_query_patient = ttk.Button(tab_patient, text="查询患者信息", command=query_patient_by_id)
btn_query_patient.grid(row=5, column=2, padx=5, pady=10)

# 患者信息展示标签
lbl_p_info = ttk.Label(tab_patient, text="患者信息展示区：未查询", font=("Arial", 10))
lbl_p_info.grid(row=6, column=0, columnspan=4, padx=10, pady=10)

#  预约功能标签页（优化版）
tab_appt = ttk.Frame(tab_control)
tab_control.add(tab_appt, text="患者预约")

ttk.Label(tab_appt, text="患者ID：").grid(row=0, column=0, padx=20, pady=15, sticky="e")
entry_appt_p_id = ttk.Entry(tab_appt, width=30)
entry_appt_p_id.grid(row=0, column=1, padx=10, pady=15)

ttk.Label(tab_appt, text="医生ID：").grid(row=1, column=0, padx=20, pady=15, sticky="e")
entry_appt_d_id = ttk.Entry(tab_appt, width=30)
entry_appt_d_id.grid(row=1, column=1, padx=10, pady=15)

ttk.Label(tab_appt, text="预约日期（YYYY-MM-DD）：").grid(row=2, column=0, padx=20, pady=15, sticky="e")
entry_appt_date = ttk.Entry(tab_appt, width=30)
entry_appt_date.grid(row=2, column=1, padx=10, pady=15)

ttk.Label(tab_appt, text="预约时段：").grid(row=3, column=0, padx=20, pady=15, sticky="e")
entry_appt_time = ttk.Entry(tab_appt, width=30)
entry_appt_time.grid(row=3, column=1, padx=10, pady=15)

btn_appt = ttk.Button(tab_appt, text="提交预约", command=add_appointment)
btn_appt.grid(row=4, column=0, columnspan=2, pady=20)

#  缴费功能标签页（优化版）
tab_pay = ttk.Frame(tab_control)
tab_control.add(tab_pay, text="缴费结算")

ttk.Label(tab_pay, text="就诊记录ID：").grid(row=0, column=0, padx=20, pady=15, sticky="e")
entry_pay_record_id = ttk.Entry(tab_pay, width=30)
entry_pay_record_id.grid(row=0, column=1, padx=10, pady=15)

ttk.Label(tab_pay, text="患者ID：").grid(row=1, column=0, padx=20, pady=15, sticky="e")
entry_pay_p_id = ttk.Entry(tab_pay, width=30)
entry_pay_p_id.grid(row=1, column=1, padx=10, pady=15)

ttk.Label(tab_pay, text="缴费金额：").grid(row=2, column=0, padx=20, pady=15, sticky="e")
entry_pay_amount = ttk.Entry(tab_pay, width=30)
entry_pay_amount.grid(row=2, column=1, padx=10, pady=15)

ttk.Label(tab_pay, text="缴费方式：").grid(row=3, column=0, padx=20, pady=15, sticky="e")
entry_pay_method = ttk.Entry(tab_pay, width=30)
entry_pay_method.grid(row=3, column=1, padx=10, pady=15)

btn_pay = ttk.Button(tab_pay, text="确认缴费", command=add_payment)
btn_pay.grid(row=4, column=0, columnspan=2, pady=20)

#  收入统计标签页
tab_stat = ttk.Frame(tab_control)
tab_control.add(tab_stat, text="收入统计")

ttk.Label(tab_stat, text="查询日期（YYYY-MM-DD）：").grid(row=0, column=0, padx=20, pady=30, sticky="e")
entry_query_date = ttk.Entry(tab_stat, width=30)
entry_query_date.grid(row=0, column=1, padx=10, pady=30)

btn_query = ttk.Button(tab_stat, text="查询当日收入", command=query_income)
btn_query.grid(row=1, column=0, columnspan=2, pady=20)

# 显示标签页
tab_control.pack(expand=1, fill="both")

# 运行主循环
root.mainloop()




