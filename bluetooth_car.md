# 树莓派：蓝牙小车遥控器



## 说明

用 Python 实现手机蓝牙控制小车。手机端通过蓝牙串口发送方向指令，树莓派解析后驱动 L298N 电机。



## 硬件

- 树莓派 ×1

- L298N 电机驱动 ×1

- 小车底盘+电机 ×1

- 18650 电池组



## 代码

```python

import RPi.GPIO as GPIO

import bluetooth

import json



# 电机引脚 (L298N)

IN1, IN2, IN3, IN4 = 17, 27, 22, 23

ENA, ENB = 18, 13  # PWM 调速



GPIO.setmode(GPIO.BCM)

for pin in [IN1, IN2, IN3, IN4, ENA, ENB]:

    GPIO.setup(pin, GPIO.OUT)



pwm_a = GPIO.PWM(ENA, 100)  # 100Hz

pwm_b = GPIO.PWM(ENB, 100)

pwm_a.start(0)

pwm_b.start(0)



speed = 50  # 默认速度



def move(direction):

    global speed

    if direction == "forward":

        GPIO.output(IN1, GPIO.HIGH); GPIO.output(IN2, GPIO.LOW)

        GPIO.output(IN3, GPIO.HIGH); GPIO.output(IN4, GPIO.LOW)

    elif direction == "backward":

        GPIO.output(IN1, GPIO.LOW); GPIO.output(IN2, GPIO.HIGH)

        GPIO.output(IN3, GPIO.LOW); GPIO.output(IN4, GPIO.HIGH)

    elif direction == "left":

        GPIO.output(IN1, GPIO.LOW); GPIO.output(IN2, GPIO.HIGH)

        GPIO.output(IN3, GPIO.HIGH); GPIO.output(IN4, GPIO.LOW)

    elif direction == "right":

        GPIO.output(IN1, GPIO.HIGH); GPIO.output(IN2, GPIO.LOW)

        GPIO.output(IN3, GPIO.LOW); GPIO.output(IN4, GPIO.HIGH)

    elif direction == "stop":

        GPIO.output(IN1, GPIO.LOW); GPIO.output(IN2, GPIO.LOW)

        GPIO.output(IN3, GPIO.LOW); GPIO.output(IN4, GPIO.LOW)



    pwm_a.ChangeDutyCycle(speed)

    pwm_b.ChangeDutyCycle(speed)



def main():

    # 蓝牙 SPP 服务

    server_sock = bluetooth.BluetoothSocket(bluetooth.RFCOMM)

    server_sock.bind(("", bluetooth.PORT_ANY))

    server_sock.listen(1)



    port = server_sock.getsockname()[1]

    uuid = "00001101-0000-1000-8000-00805F9B34FB"



    bluetooth.advertise_service(

        server_sock, "CarServer",

        service_id=uuid,

        service_classes=[uuid, bluetooth.SERIAL_PORT_CLASS],

        profiles=[bluetooth.SERIAL_PORT_PROFILE]

    )



    print(f"等待蓝牙连接 (通道 {port})...")

    client_sock, client_info = server_sock.accept()

    print(f"已连接: {client_info}")



    try:

        while True:

            data = client_sock.recv(16).decode("utf-8").strip().lower()

            if data:

                print(f"收到指令: {data}")

                if data in ["f", "forward"]:     move("forward")

                elif data in ["b", "backward"]:  move("backward")

                elif data in ["l", "left"]:      move("left")

                elif data in ["r", "right"]:     move("right")

                elif data in ["s", "stop"]:      move("stop")

                elif data.startswith("speed"):

                    speed = int(data.split()[1])

                    speed = max(0, min(100, speed))

    except KeyboardInterrupt:

        pass

    finally:

        move("stop")

        client_sock.close()

        server_sock.close()

        GPIO.cleanup()



if __name__ == "__main__":

    main()

```



## 教学重点

- `pybluez` 库实现蓝牙 SPP（串口仿真）协议

- L298N 双路 H 桥控制 2 个直流电机正反转

- PWM（脉宽调制）控制电机速度，`ChangeDutyCycle(0-100)`

- 手机安装蓝牙串口 App 即可遥控



## 常见错误

- pybluez 安装复杂：`sudo apt install libbluetooth-dev && pip3 install pybluez`

- 电机电源必须独立供电（不能从 GPIO 取电）

- 速度范围 0-100（占空比百分比）

- 同时改变方向和未降速可能损坏电机

