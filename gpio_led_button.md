# 树莓派：GPIO 控制 LED 和按钮



## 说明

用 Python 的 RPi.GPIO 库控制树莓派的 GPIO 引脚：点亮 LED、读取按钮状态。这是树莓派硬件控制的第一步。



## 硬件

- 树莓派 ×1

- LED ×1，按键 ×1，电阻 220Ω + 10KΩ



## 电路

- LED 正极 → GPIO17，负极 → 220Ω → GND

- 按键 → GPIO27 → 10KΩ → GND，按键另一端 → 3.3V



## 代码

```python

import RPi.GPIO as GPIO

import time



# 使用 BCM 编号方式

GPIO.setmode(GPIO.BCM)

GPIO.setwarnings(False)



LED_PIN = 17

BTN_PIN = 27



# 设置引脚模式

GPIO.setup(LED_PIN, GPIO.OUT)

GPIO.setup(BTN_PIN, GPIO.IN, pull_up_down=GPIO.PUD_DOWN)



print("按键控制 LED - 按 Ctrl+C 退出")



try:

    while True:

        if GPIO.input(BTN_PIN):

            GPIO.output(LED_PIN, GPIO.HIGH)

            print("按钮按下 -> LED 亮")

        else:

            GPIO.output(LED_PIN, GPIO.LOW)

        time.sleep(0.05)  # 消抖

except KeyboardInterrupt:

    print("\n程序退出")

finally:

    GPIO.cleanup()  # 释放 GPIO 资源（重要！）

```



## 教学重点

- GPIO 编号有 BCM 和 BOARD 两种，`setmode(BCM)` 选 BCM

- 上拉(PUD_UP) vs 下拉(PUD_DOWN)：决定空闲状态

- `try/finally` 确保 `GPIO.cleanup()` 一定执行

- 树莓派 GPIO 是 3.3V 电平，5V 会烧毁！



## 常见错误

- 忘记 `GPIO.cleanup()` → 下次运行报 GPIO 占用

- 3.3V vs 5V 混淆 → 烧毁 GPIO

- 按键没接上拉/下拉电阻 → 悬空状态不稳定

- `sudo` 权限：某些系统需 `sudo python3` 运行

