# 树莓派：语音播报（TTS 文字转语音）



## 说明

用 eSpeak/eSpeak-NG 把文字转换为语音输出。可用来报时、读天气、念通知等。学习 subprocess 调用系统命令。



## 代码

```python

import subprocess

import time

from datetime import datetime

import requests

import json



def speak(text, speed=150, voice="zh"):

    """调用 espeak 朗读文字"""

    # speed: 语速（词/分钟）voice: zh/m3=中文男,f2=英文女

    subprocess.run(["espeak", f"-s{speed}", f"-v{voice}", text],

                   stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL)



def speak_time():

    """整点报时"""

    now = datetime.now()

    hour = now.hour

    minute = now.minute

    text = f"现在是{hour}点{minute}分"

    speak(text)

    print(text)



def speak_weather(city="北京"):

    """播报天气"""

    try:

        url = f"http://wttr.in/{city}?format=%C+%t"

        resp = requests.get(url, timeout=5)

        weather = resp.text.strip()

        text = f"{city}天气：{weather}"

    except:

        text = "天气信息获取失败"



    speak(text)

    print(text)



def speak_countdown(seconds):

    """倒数计时语音"""

    for i in range(seconds, 0, -1):

        speak(str(i), speed=200, voice="en")

        time.sleep(1)

    speak("时间到！")



def main():

    print("=== 树莓派语音播报 ===")

    print("1 - 报时")

    print("2 - 天气")

    print("3 - 倒计时 10 秒")

    print("0 - 退出")



    while True:

        try:

            choice = input("选择: ").strip()

            if choice == "1":

                speak_time()

            elif choice == "2":

                speak_weather()

            elif choice == "3":

                speak_countdown(10)

            elif choice == "0":

                speak("再见！")

                break

            else:

                print("无效选择")

        except KeyboardInterrupt:

            break



if __name__ == "__main__":

    main()

```



## 教学重点

- `subprocess.run()` 调用外部程序（espeak），参数用列表传递

- eSpeak 的 `-vzh` / `-ven` 切换中文/英文

- `requests.get()` 获取网络数据

- 语音交互是实现家庭 AI 助手的基础



## 常见错误

- eSpeak 未安装：`sudo apt install espeak -y`

- 中文发音不好：eSpeak 中文效果一般，可换 espeak-ng 或百度 TTS API

- `subprocess.run` 的 `stdout`/`stderr` 未处理导致卡住

- 需要音频输出设备（耳机或 HDMI 音频）

