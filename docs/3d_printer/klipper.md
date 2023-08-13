## Klipper近程挤出机校准：
针对已知的挤出机类型，获取rotation_distance的推荐值填入配置文件
待热端在设定的挤出温度下稳定后，先执行：
``` { .yaml .copy .annotate }
M83 ; 挤出机相对当前位置运动
G1 E1 F60 ; 以1mm/s (60mm/min)速度挤出1mm
```
让挤出机做好准备。在耗材距离挤出机入口120毫米的位置做一个标记。执行：
``` { .yaml .copy .annotate }
M83 ; 挤出机相对当前位置运动
G1 E100 F60 ; 以1mm/s (60mm/min)速度挤出100mm
```
测量挤出机入口到之前的标记剩余长度，最终测量的结果理想值是20毫米（120 毫米 - 20 毫米 = 100 毫米）。
如果不是，则使用以下公式更新rotation_distance：
新配置值=旧配置值*（实际挤出长度/目标挤出长度）
注意：配置值越高意味着挤出的耗材越少。使用命令：
``` { .yaml .copy .annotate }
SET_EXTRUDER_ROTATION_DISTANCE EXTRUDER=extruder DISTANCE=新配置值
```
临时指定rotation_distance，再进行测试。
挤出量在目标值的±0.5% 以内（即目标100mm，实际挤出99.5-100.5mm），则已经满足使用。
将新值填入配置文件，保存并重启。
