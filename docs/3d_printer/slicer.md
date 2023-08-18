## 关于打印件黏结不牢固从热床掉落的几种处理方法

- 尽可能避免用手接触PEI弹簧板表面。手指上的油脂会引起此类问题。
- 用洗洁精和水彻底清洗PEI弹簧板，自然晾干或者用吹风机吹干，不要用纸或布擦。
- 确保首层挤出量合适，有纹理的PEI表面需要更多的挤出量，以便将耗材填入缝隙内。
- 使用合适的首层温度：
    * PLA: 一般60℃
    * PETG: 一般80℃
    * ABS: 一般100℃（部分打印机由于热床铝板的厚度问题，温度设置为105℃~110℃较为合理）
- 首层高度不变的情况下，较大的首层线宽会对PEI弹簧板表面产生更大的压力，从而产生更好的粘合。
- 使用增强粘合的辅助产品，如PVP胶棒（固体胶）与高浓度乙醇的混合溶液、聚维酮K30与高浓度乙醇的混合溶液、LAC喷雾等等。

## 打印开始宏PRINT_START与参数传递

在使用Klipper系统的打印机的时候，通常我们使用附带参数的PRINT_START宏命令来执行打印开始前的准备工作，包括但不限于加热热床、预热打印头、预热仓温、调平等等。为了达到上述目的，通常我们在切片软件中只设置需要执行的宏，并附上相应的参数。至于宏的完整命令则在打印机的配置文件中设定，这样做的目的是为了适应不同的打印机硬件，也便于在不重新切片的情况下编辑调整打印开始前的操作。要实现上述目的，我们需要在切片软件的开始G-CODE填入相应的代码，以下列举了3种常用切片软件的开始G-CODE。

!!! note "需要注意"

    如果切片软件开始G-CODE中已经有填写内容，请删除后再填入。

1、 CURA 5.0：
``` { .yaml .copy }
PRINT_START EXTRUDER={material_print_temperature_layer_0} BED={material_bed_temperature_layer_0} CHAMBER={build_volume_temperature} NOZZLE={machine_nozzle_size} FILAMENT={material_type} PRINT_MIN=%MINX%,%MINY% PRINT_MAX=%MAXX%,%MAXY%
```
2、 SuperSlicer：
``` { .yaml .copy }
M190 S0
M109 S0
PRINT_START EXTRUDER={first_layer_temperature[initial_extruder] + extruder_temperature_offset[initial_extruder]} BED=[first_layer_bed_temperature] CHAMBER=[chamber_temperature] NOZZLE=[nozzle_diameter] FILAMENT=[filament_type] PRINT_MIN={first_layer_print_min[0]},{first_layer_print_min[1]} PRINT_MAX={first_layer_print_max[0]},{first_layer_print_max[1]}
```
3、 OrcaSlicer:
``` { .yaml .copy }
M190 S0
M109 S0
PRINT_START EXTRUDER=[nozzle_temperature_initial_layer] BED=[bed_temperature_initial_layer_single] CHAMBER=[chamber_temperature] NOZZLE={nozzle_diameter[0]} FILAMENT=[filament_type] PRINT_MIN={first_layer_print_min[0]},{first_layer_print_min[1]} PRINT_MAX={first_layer_print_max[0]},{first_layer_print_max[1]}
```

### 参数的说明

* PRINT_START：调用的宏名字
* M190 S0：禁用热床预热，SuperSlicer和OrcaSlicer会默认在切片中加入预热的gcode，用此参数会禁用默认加入
* M109 S0：禁用打印头预热，SuperSlicer和OrcaSlicer会默认在切片中加入预热的gcode，用此参数会禁用默认加入
* EXTRUDER=XXX：切片设置中打印头的温度
* BED=XXX：切片设置中热床的温度
* CHAMBER=XXX：切片设置中仓温的温度
* NOZZLE=XXX：切片设置中喷嘴的直径
* FILAMENT=XXX：切片设置中耗材的类型
* PRINT_MIN=XXX：切片中最小的打印坐标点
* PRINT_MAX=XXX：切片中最大的打印坐标点

### PRINT_START宏案例

以下是一个比较简单的PRINT_START宏案例，前面提到的参数都有实际的对应用途，也加了注释便于理解。

``` { .yaml .copy }
[gcode_macro PRINT_START]
description: 打印开始前进行的操作
gcode:
    CLEAR_PAUSE                                                                                             # 清空暂停缓存
    BED_MESH_CLEAR                                                                                          # 清除网床
    {% set EXTRUDER_TEMP = params.EXTRUDER|default(200)|int %}                                              # 从切片中获取打印头温度
    {% set BED_TEMP = params.BED|default(60)|int %}                                                         # 从切片中获取热床温度
    {% set CHAMBER_TEMP = params.CHAMBER|default(0)|int %}                                                  # 从切片中获取仓温温度
    {% set FILAMENT_TYPE = params.FILAMENT|default("PLA")|string %}                                         # 从切片中获取耗材类型
    {% set NOZZLE_SIZE = params.NOZZLE|default("0.4")|string %}                                             # 从切片中获取喷嘴大小
    M140 S{BED_TEMP}                                                                                        # 设置热床目标温度
    M104 S150                                                                                               # 设置打印头预热温度
    {% if CHAMBER_TEMP > 0 %}                                                                               # 判断如果需要仓温则进行指定操作
        M106 S255                                                                                           # 模型散热风扇开到最大，辅助空气循环
        TEMPERATURE_WAIT SENSOR="temperature_sensor Chamber" MINIMUM={CHAMBER_TEMP-5}                       # 等待仓温到达指定温度
        M107                                                                                                # 关闭模型散热风扇
    {% endif %}                                                                                             # 结束判断
    M190 S{BED_TEMP}                                                                                        # 等待热床到达指定温度
    G28                                                                                                     # XYZ轴归零
    QUAD_GANTRY_LEVEL                                                                                       # 龙门架调平
    G28 Z                                                                                                   # Z轴再次归零，消除龙门架调平造成的Z误差
    ADAPTIVE_BED_MESH_CALIBRATE AREA_START={params.PRINT_MIN} AREA_END={params.PRINT_MAX}                   # 自适应网床探测，需要安装插件 https://github.com/eamars/klipper_adaptive_bed_mesh
    MATERIAL_PA MATERIAL={FILAMENT_TYPE} NOZZLE={NOZZLE_SIZE}                                               # 自动调整PA值
    PURGE_LINE EXTRUDER_TEMP={EXTRUDER_TEMP} PRINT_MIN={params.PRINT_MIN}                                   # 根据区域床尺寸自适应打印测试线
```

