import threading
import time
from datetime import datetime, timezone, timedelta
def get_thai_time():
    thai_timezone = timezone(timedelta(hours=7))
    return datetime.now(thai_timezone).strftime('%H:%M:%S')
def schedule_exit(table_number, duration):
    global tables
    time.sleep(duration)  # รอเวลาที่กำหนด
    if table_number in tables:
        del tables[table_number]
        print(f"[{get_thai_time()}] (ระบบ) โต๊ะ {table_number} ออกไป ตอนนี้เหลือ {len(tables)} โต๊ะ")
responses = {
    "เข้า": "[{time}] ลูกค้าเข้าไปที่โต๊ะ {table} และจะออกใน {duration} วินาที",
    "ไป": "[{time}] ลูกค้าโต๊ะ {table} ออกไป"
}
tables = {}  # เก็บโต๊ะที่มีลูกค้าอยู่
MAX_TABLES = 30  # กำหนดจำนวนโต๊ะสูงสุด
DEFAULT_DURATION = 15  # ค่าเริ่มต้นของเวลาที่ลูกค้าจะอยู่ในร้าน
while True:
    text = input("การมาถึงของลูกค้า (พิมพ์ 'ออก' เพื่อหยุด): ")
   
    if text == "ออก":
        print("จบการทำงาน")
        break
    elif text.startswith("เข้า"):
        parts = text.split()
        try:
            table_number = int(parts[0][4:])  # ดึงหมายเลขโต๊ะหลัง "เข้า"
            duration = int(parts[1]) if len(parts) > 1 else DEFAULT_DURATION  # ใช้ค่าเริ่มต้นถ้าไม่มีระบุ
        except ValueError:
            print("กรุณาระบุหมายเลขโต๊ะและเวลาที่ถูกต้อง!")
            continue
        if table_number in tables:
            print(f"โต๊ะ {table_number} มีลูกค้าอยู่แล้ว!")
        elif len(tables) >= MAX_TABLES:
            print(f"ขออภัย โต๊ะเต็มแล้ว! ตอนนี้มี {len(tables)} โต๊ะ (สูงสุด {MAX_TABLES})")
        else:
            tables[table_number] = True
            print(responses["เข้า"].format(time=get_thai_time(), table=table_number, duration=duration))
            threading.Thread(target=schedule_exit, args=(table_number, duration), daemon=True).start()  # ตั้งเวลาลูกค้าออก
    elif text.startswith("ออก"):
        try:
            table_number = int(text[3:])  # ดึงหมายเลขโต๊ะหลัง "ออก"
        except ValueError:
            print("กรุณาระบุหมายเลขโต๊ะที่ถูกต้อง!")
            continue
        if table_number in tables:
            del tables[table_number]  # ลบโต๊ะออกทันที
            print(responses["ไป"].format(time=get_thai_time(), table=table_number))
        else:
            print(f"ไม่มีลูกค้าที่โต๊ะ {table_number}!")
    else:
        print("คำสั่งไม่ถูกต้อง กรุณาลองใหม่!")
