---
title: 使用 Python 监控系统信息
top: false
cover: false
toc: true
mathjax: true
date: 2020-08-04 14:25:55
password:
summary: 10分钟完成一个系统 CPU、RAM 占用监控程序。
tags: 
    - Python
    - 运维
    - psutil
categories:
    - Python
    - 运维
---

10分钟完成一个系统 CPU、RAM 占用监控程序。

## 准备工作

* 已经安装 Python (版本不限，已测试 2.7、3.6、3.7 可以正常使用)；
* 可以使用 pip install 安装依赖，测试方法：

  ```bash
  pip -V
  ```

  可以正常返回版本号即可。

## 安装

### 新建终端，安装 psutil

```bash
pip install psutil
```

### 新建并编辑 sys_info.py

```bash
vim sys_info.py
```

将以下内容添加至新文件中

```python
import psutil
import logging
import time
import argparse


parser = argparse.ArgumentParser(description="Get system RAM & CPU usage.")

parser.add_argument('--file', '-f', help='Specify where to save log file. Default: /var/log/sys_info.log ',
                    default='/var/log/sys_info.log')
parser.add_argument('--show', '-s', action='store_true', help='Will be printed on the console.', default=False)

args = parser.parse_args()

log_path = args.file
formatter = logging.Formatter('%(asctime)s - %(message)s')
logger = logging.getLogger()
logger.setLevel(logging.INFO)
fh = logging.FileHandler(log_path)
fh.setLevel(logging.DEBUG)
fh.setFormatter(formatter)
logger.addHandler(fh)
if args.show:
    sh = logging.StreamHandler()
    sh.setLevel(logging.INFO)
    sh.setFormatter(formatter)
    logger.addHandler(sh)

# Wait for psutil to refresh information
time.sleep(1)

# write log
logger.info("RAM {0}% - CPU {1}%".format(psutil.virtual_memory().percent, psutil.cpu_percent()))
## Python 3.6 f-string
# logger.info(f'RAM {psutil.virtual_memory().percent}% - CPU {psutil.cpu_percent()}%')
```

### 测试脚本

```bash
python sys_info.py
```

无报错代表运行成功，检查 log_paht 指定位置是否生成日志。

### 编写 crontab 定时任务

编辑当前用户的定时任务

```bash
crontab -e
```

设置脚本每 30 分钟运行一次(使用 `i` 键进入编辑模式，编辑完成后敲击`esc`键，并输入 `:wq` 保存退出)

```bash
*/30 * * * * python ~/sys_info.py
```

## 附录 crontab 语法

```crontab
# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed
```

## pyinstaller 打包可执行程序

可以使用 Pyinstall 打包可执行程序，方便安装在没有 Python 环境的系统中，但不同的平台需要单独打包(例如，如果想打包为 *.exe 的 Windwos 可执行程序，就需要现在 Windows 平台中运行 PyInstaller)。[官方文档](https://pyinstaller.readthedocs.io/en/stable/)

> 参考资料：
[psutil documentation](https://psutil.readthedocs.io/en/latest/)
