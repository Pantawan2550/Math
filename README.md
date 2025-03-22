from flask import Flask, request, jsonify
import threading
import time
from datetime import datetime, timezone, timedelta

app = Flask(__name__)

def get_thai_time():
    thai_timezone = timezone(timedelta(hours=7))
    return datetime.now(thai_timezone).strftime('%H:%M:%S')

def schedule_exit(table_number, duration):
    time.sleep(duration)
    if table_number in tables:
        del tables[table_number]
        print(f"[{get_thai_time()}] (ระบบ) โต๊ะ {table_number} ออกไป ตอนนี้เหลือ {len(tables)} โต๊ะ")

tables = {}
MAX_TABLES = 30  
DEFAULT_DURATION = 15  

@app.route("/webhook", methods=["POST"])
def webhook():
    data = request.json  # รับข้อมูล JSON จาก webhook
    if not data:
        return jsonify({"status": "error", "message": "ไม่มีข้อมูล"}), 400

    action = data.get("action")  # "เข้า" หรือ "ออก"
    table_number = data.get("table")  
    duration = data.get("duration", DEFAULT_DURATION)  

    if action == "เข้า":
        if table_number in tables:
            return jsonify({"status": "error", "message": f"โต๊ะ {table_number} มีลูกค้าอยู่แล้ว!"}), 400
        elif len(tables) >= MAX_TABLES:
            return jsonify({"status": "error", "message": "โต๊ะเต็มแล้ว!"}), 400
        else:
            tables[table_number] = True
            threading.Thread(target=schedule_exit, args=(table_number, duration), daemon=True).start()
            return jsonify({
                "status": "success",
                "message": f"ลูกค้าเข้าโต๊ะ {table_number} และจะออกใน {duration} วินาที",
                "time": get_thai_time()
            })

    elif action == "ออก":
        if table_number in tables:
            del tables[table_number]
            return jsonify({
                "status": "success",
                "message": f"ลูกค้าโต๊ะ {table_number} ออกไป",
                "time": get_thai_time()
            })
        else:
            return jsonify({"status": "error", "message": f"ไม่มีลูกค้าที่โต๊ะ {table_number}!"}), 400

    else:
        return jsonify({"status": "error", "message": "คำสั่งไม่ถูกต้อง"}), 400

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000, debug=True)
