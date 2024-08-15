# Pot_Graph_IOT
Pot_Graph_IOT

import asyncio
import websockets
import RPi.GPIO as GPIO
import time

# ตั้งค่า GPIO
GPIO.setmode(GPIO.BCM)

# กำหนดพินสำหรับสวิตช์และไฟ
SWITCH_PIN = 23

GPIO.setup(SWITCH_PIN, GPIO.IN, pull_up_down=GPIO.PUD_UP)

async def monitor_switch(websocket):
    light_status = False  # สถานะของไฟ (เปิด/ปิด)
    last_switch_state = GPIO.input(SWITCH_PIN)
    
    while True:
        current_switch_state = GPIO.input(SWITCH_PIN)
        
        # ตรวจสอบการกดสวิตช์ (transition จาก HIGH ไป LOW)
        if current_switch_state == GPIO.LOW and last_switch_state == GPIO.HIGH:
            # Toggle สถานะของไฟ
            light_status = not light_status

            if light_status:
                await websocket.send("1")  # เปิดไฟ
                print("Switch toggled: Light ON")
            else:
                await websocket.send("0")  # ปิดไฟ
                print("Switch toggled: Light OFF")
            
            # หน่วงเวลาเล็กน้อยเพื่อลดการ debounce ของสวิตช์
            await asyncio.sleep(0.5)

        # อัพเดตสถานะล่าสุดของสวิตช์
        last_switch_state = current_switch_state
        
        await asyncio.sleep(0.1)  # ตรวจสอบสถานะทุก ๆ 100ms

async def main():
    uri = "ws://192.168.188.143:8765"
    async with websockets.connect(uri) as websocket:
        print("Connected to Server")
        
        async def receive_message():
            while True:
                try:
                    response = await websocket.recv()
                    print(f"Server Response: {response}")
                except websockets.exceptions.ConnectionClosed:
                    print("Connection Closed")
                    break
        
        await asyncio.gather(monitor_switch(websocket), receive_message())

if __name__ == "__main__":
    try:
        asyncio.run(main())
    finally:
        GPIO.cleanup()  # ทำความสะอาด GPIO เมื่อหยุดการทำงาน
