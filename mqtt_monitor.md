# 树莓派：MQTT 数据采集节点（Python）



## 说明

用 Paho MQTT 客户端库，把树莓派的 CPU 温度/内存/磁盘使用率发布到 MQTT 服务器，实现系统监控。



## 代码

```python

import paho.mqtt.client as mqtt

import time

import psutil

import json



BROKER = "broker.emqx.io"

PORT = 1883

TOPIC_PREFIX = "raspberry/system"

CLIENT_ID = f"raspi-{time.time_ns() % 100000}"



client = mqtt.Client(client_id=CLIENT_ID)



def on_connect(client, userdata, flags, rc):

    print(f"MQTT 已连接 (rc={rc})")



def get_system_stats():

    """获取系统信息"""

    cpu_temp = 0

    try:

        with open("/sys/class/thermal/thermal_zone0/temp") as f:

            cpu_temp = int(f.read()) / 1000.0

    except:

        cpu_temp = -1



    return {

        "cpu_percent": psutil.cpu_percent(interval=1),

        "cpu_temp": round(cpu_temp, 1),

        "memory_percent": psutil.virtual_memory().percent,

        "disk_percent": psutil.disk_usage("/").percent,

        "uptime_hours": round(psutil.boot_time() / 3600, 1),

    }



def publish_stats():

    """发布数据的推荐做法：JSON + 单 topic 或 多 topic"""

    stats = get_system_stats()



    # 方式 1: 一个 topic 一条数据

    client.publish(f"{TOPIC_PREFIX}/cpu_percent", stats["cpu_percent"])

    client.publish(f"{TOPIC_PREFIX}/cpu_temp", stats["cpu_temp"])

    client.publish(f"{TOPIC_PREFIX}/memory_percent", stats["memory_percent"])

    client.publish(f"{TOPIC_PREFIX}/disk_percent", stats["disk_percent"])



    # 方式 2: JSON 打包（推荐）

    client.publish(f"{TOPIC_PREFIX}/json", json.dumps(stats))



    print(json.dumps(stats, indent=2, ensure_ascii=False))



def main():

    client.on_connect = on_connect

    client.connect(BROKER, PORT, 60)

    client.loop_start()  # 后台线程处理网络



    try:

        while True:

            publish_stats()

            time.sleep(10)

    except KeyboardInterrupt:

        print("退出...")

    finally:

        client.loop_stop()

        client.disconnect()



if __name__ == "__main__":

    main()

```



## 教学重点

- `paho-mqtt` 是 Python 官方 MQTT 库：`pip3 install paho-mqtt`

- `client.loop_start()` 开启后台线程，不阻塞主程序

- `psutil` 跨平台获取系统信息：`pip3 install psutil`

- `json.dumps(stats)` 将字典转为 JSON 字符串发布

- MQTT 发布模式：逐 topic 发布 vs JSON 打包，各有利弊



## 常见错误

- Broker 连接失败 → 检查网络或换 broker (test.mosquitto.org)

- ClientID 重复 → 用时间戳 + 随机数确保唯一

- 免费 broker 不稳定 → 生产环境用自建 Mosquitto 或云服务

- `loop_start()` 忘记配对 `loop_stop()` → 线程残留

