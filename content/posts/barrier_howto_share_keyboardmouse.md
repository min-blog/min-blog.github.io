+++
date = '2026-05-12T11:56:30-04:00'
draft = false
title = '使用 Barrier 实现 Linux 与 Windows 共享键鼠'

+++

# 使用 Barrier 实现 Linux 与 Windows 共享键鼠

适用环境：

- Windows 11（Server）
- Ubuntu X11（Client）

---

# Windows 端

## 版本

下载 Barrier 2.3.3：

https://github.com/debauchee/barrier/releases

---

## 创建配置文件

创建：

```text
screen.conf
```

内容：

```conf
section: screens
    win11:
    bsa-lab:
end

section: links
    win11:
        left = bsa-lab
    bsa-lab:
        right = win11
end

section: options
    switchDelay = 300
    screenSaverSync = false
end
```

说明：

- Linux 在左
- Windows 在右

---

## 启动 Server

进入 Barrier 目录：

```cmd
barriers.exe -f --debug INFO --name win11 --config screen.conf
```

等待 Linux 连接。

---

# Linux 端

## 检查 X11

```bash
echo $XDG_SESSION_TYPE
```

输出应为：

```text
x11
```

---

## 下载源码

```bash
git clone https://github.com/debauchee/barrier.git
```

---

## 安装依赖

```bash
sudo apt install \
libavahi-compat-libdnssd-dev \
libcurl4-openssl-dev \
libssl-dev \
libx11-dev \
libxtst-dev \
libxinerama-dev \
libxrandr-dev \
libxi-dev \
qtbase5-dev \
qttools5-dev
```

---

## 初始化子模块

```bash
cd barrier

git submodule update --init --recursive
```

---

## 修复 GCC 编译问题

查找：

```bash
grep -R "std::uint8_t" -n ~/Downloads/barrier/src
```

例如修复：

```bash
sed -i '/#include <vector>/a #include <cstdint>' ~/Downloads/barrier/src/lib/base/String.h
```

若其他头文件也报：

```text
std::uint8_t
```

同样补：

```cpp
#include <cstdint>
```

---

## 编译

```bash
mkdir build
cd build

cmake ..

make -j$(nproc)
```

---

## 启动 Client

将 IP 替换为 Windows 主机 IP：

```bash
./barrierc -f --debug INFO --disable-crypto --name bsa-lab 192.168.5.10
```
