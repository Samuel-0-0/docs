# VAST Stealthburner内嵌CAN控制板

这是关于VAST Stealthburner内嵌CAN控制板使用Klipper的相关说明。

!!! success "此文档在以下条件中测试通过:"

    - VAST Stealthburner控制板 v2.0
    - Klipper 0.11.0-267
    - Moonraker 0.8.0-138
    - Mainsail 2.7.1

## VAST Stealthburner内嵌CAN控制板 v2.0

- 原理图
- 尺寸
- 3D模型
- 配置案例

### 硬件

- **MCU:** STM32F072
- **步进电机驱动:** 板载TMC2209，UART模式, UART address: 00, Rsense: 0.11R
- **板载加速度传感器:** ADXL345
- **板载温度转换芯片:** Max31865 2线PT100
- **输入电压:** DC12V-DC24V 6A
- **IO逻辑电压:** DC 3.3V
- **加热端口:** 热端最大输出电流: 5A
- **风扇接口:** 2个可控风扇 (FAN0, FAN1)
- **风扇最大输出电流:** 1A, Peak Value 1.5A
- **接口:** EndStop, Probe, RGB, PT100, USB, CAN
- **可选温度传感器:** 1路100K NTC, 1路PT100
- **额外板载温度传感器** 1路100K NTC
- **USB接口:** USB-Type-C
- **DC 5V最大输出电流:** 2A

### 引脚说明


