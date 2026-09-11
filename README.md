# 🏎️ RPi-Vision-Mecanum-Robot
> **基于树莓派的全向移动视觉智能小车系统 | A Raspberry Pi-based Omnidirectional Vision Robot with Mecanum Kinematics & Multi-threaded Architecture**

[![Platform](https://img.shields.io/badge/Platform-Raspberry%20Pi%20(Linux)-red.svg)](https://www.raspberrypi.org/)
[![Language](https://img.shields.io/badge/Language-Python%203-blue.svg)](https://www.python.org/)
[![Computer Vision](https://img.shields.io/badge/Vision-OpenCV-green.svg)](https://opencv.org/)
[![Architecture](https://img.shields.io/badge/Concurrency-Multi--Threading-orange.svg)]()
[![Protocol](https://img.shields.io/badge/Protocol-JSON--RPC%20%7C%20HTTP-purple.svg)]()
[![License](https://img.shields.io/badge/License-MIT-brightgreen.svg)](LICENSE)

---

## 项目简介 (Overview)

本项目是一套基于 **树莓派（Linux 嵌入式平台）** 开发的智能全向移动视觉机器人系统。

项目以四轮麦克纳姆轮底盘为载体，通过独立开发的**底盘运动学逆解算法**，实现了平面内的全向平移、旋转及任意航向角巡航。软件架构上针对 Linux 非实时系统的调度特点，设计了**多线程异步解耦并发架构**，将高计算负载的 OpenCV 图像处理、低延迟视频流服务、远程 JSON-RPC 通信与底层高实时电机控制彻底分离，控制响应延迟控制在 **10ms 以内**。

---

##  核心特性与技术亮点 (Key Highlights)

-  **麦克纳姆轮运动学逆解算法**：建立整车几何与运动学模型，将上层输入的任意三自由度速度指令 $(V_x, V_y, \omega_z)$ 毫秒级解算为四个轮组的独立 PWM 驱动参数，实现平滑的全向移动与自旋控制。
-  **多线程异步解耦并发架构**：
  - 采用生产者-消费者模型，将“高耗时图像采集/处理”与“高频底层控制/电压检测”独立在不同线程运行。
  - 引入互斥锁（Thread Lock）与共享内存保护，消除了复杂图像运算对电机控制造成的抖动与卡顿。
-  **嵌入式计算机视觉 (OpenCV)**：
  - 实现了图像畸变矫正、色彩空间转换（HSV）、轮廓提取与目标质心跟踪。
  - 搭配双自由度舵机云台，实现闭环视觉自动跟踪与画面稳定。
-  **高内聚低耦合的通信与控制 API**：
  - 搭建轻量级 **MJPG 视频流服务器**，通过 HTTP 协议提供局域网内超低延迟（<100ms）的实时画面回传。
  - 基于 **JSON-RPC 协议** 封装统一的软硬件控制 API，上层业务与底层驱动完全解耦，支持终端无缝跨平台调用。

---

## 核心控制算法：麦克纳姆轮运动学模型 (Kinematics)

针对 O-shape 布局的四轮麦克纳姆轮底盘，建立运动学坐标系，设小车几何半长为 $a$，半宽为 $b$，车轮半径为 $R$。

```text
       Front
    W1 //-----\\ W2
       |  +x  |
       |  |   |
  +y <-+--o   |   (Coordinate Definition)
       |      |
    W3 \\-----// W4
        Rear
