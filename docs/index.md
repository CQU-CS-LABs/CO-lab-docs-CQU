# 重庆大学硬件综合设计实验文档

## 教学资源

[2022MIPS组资料包的使用](https://www.bilibili.com/video/BV19G4y1E7jj)

[2019硬件综合设计讲解](https://www.bilibili.com/video/BV1XJ411k7kR)

[2020硬件综合设计讲解](https://www.bilibili.com/video/BV1TA411x7T7)

## 基本实验步骤

1. 扩充实验4，实现基本的52条指令5级流水线MIPS处理器。
2. 加入CP0和特权指令，实现精确异常处理。至此完成完整57条指令，应该能跑通下发的功能测试并上板。
3. 实现基本Cache

## 扩展任务

- 优化流水线
- 实现分支预测
- 实现多发射
- 优化Cache（多路组相连，写回，AXI Burst）
- 实现MMU（TLB）
- 运行操作系统（需要一些软硬件协同设计）