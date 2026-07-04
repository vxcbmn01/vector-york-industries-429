# 树莓派：定时任务自动化（Bash + Python）



## 说明

让树莓派定时自动执行任务：每天拍照、每小时采集温湿度、每周备份。学习 cron 定时任务、bash 脚本、Python syslog 日志。



## 代码



### 脚本 1: 拍照脚本 `capture_bash.sh`

```bash

#!/bin/bash

YEAR=$(date +%Y)

MONTH=$(date +%m)

DAY=$(date +%d)

TIME=$(date +%H%M%S)

DIR="/home/pi/photos/$YEAR/$MONTH/$DAY"

mkdir -p "$DIR"

raspistill -o "$DIR/${TIME}.jpg" -w 640 -h 480 -q 85

logger "树莓派拍照: $DIR/${TIME}.jpg"

```



### 脚本 2: 自动发送邮件 `send_report.py`

```python

#!/usr/bin/env python3

import smtplib

import psutil

from email.mime.text import MIMEText

from datetime import datetime



def send_report():

    cpu = psutil.cpu_percent()

    mem = psutil.virtual_memory().percent

    disk = psutil.disk_usage("/").percent



    body = f"""树莓派状态报告

时间: {datetime.now():%Y-%m-%d %H:%M:%S}

CPU: {cpu}%

内存: {mem}%

磁盘: {disk}%

"""

    msg = MIMEText(body)

    msg["Subject"] = f"树莓派日报 {datetime.now():%Y-%m-%d}"

    msg["From"] = "pi@raspberrypi"

    msg["To"] = "your@email.com"



    # SMTP 配置（以 QQ 邮箱为例）

    with smtplib.SMTP_SSL("smtp.qq.com", 465) as server:

        server.login("your@qq.com", "授权码")

        server.send_message(msg)



    print("日报发送成功")



if __name__ == "__main__":

    send_report()

```



### 脚本 3: 自动备份 `backup.sh`

```bash

#!/bin/bash

BACKUP_DIR="/mnt/usb_backup"

BACKUP_FILE="$BACKUP_DIR/backup_$(date +%Y%m%d).tar.gz"

mkdir -p "$BACKUP_DIR"

tar -czf "$BACKUP_FILE" /home/pi/projects /home/pi/data 2>/dev/null

logger "备份完成: $BACKUP_FILE"



# 保留最近 7 天备份

find "$BACKUP_DIR" -name "*.tar.gz" -mtime +7 -delete

```



### cron 配置

```bash

# 编辑定时任务

crontab -e



# 每天 8:00 拍照

0 8 * * * /home/pi/scripts/capture_bash.sh



# 每小时采集温湿度

0 * * * * python3 /home/pi/projects/monitor.py



# 每天 20:00 发送日报

0 20 * * * python3 /home/pi/scripts/send_report.py



# 每周日凌晨 3:00 备份

0 3 * * 0 /home/pi/scripts/backup.sh

```



## 教学重点

- cron 格式：`分 时 日 月 周 命令`

- `logger` 命令写入 syslog：`tail -f /var/log/syslog` 查看

- SMTP 邮件发送需要授权码（非密码）

- `find -mtime +7` 清理旧备份文件

- `.sh` 脚本需 `chmod +x` 赋予执行权限



## 常见错误

- 脚本未设执行权限 → cron 调用失败

- 使用相对路径 → cron 的工作目录是 `~`，用绝对路径

- 邮件 SMTP 端口错误（587=TLS, 465=SSL）

- 脚本依赖环境变量 → cron 中 PATH 和普通用户不同

