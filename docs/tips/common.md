# 常见问题

## iram/dram在多次IP核升级后出现离奇问题（Lab4）

解决方法：删除IP核，并按照以下设置重新设置存储器：

![block_memory_1](../img/block_memory_1.png)

![block_memory_2](../img/block_memory_2.png)

之后在Other Options中载入coe即可。

还需要注意的是，如果Lab4使用老版本的文件，建议把top文件中inst_ram和data_ram连接改为以下形式（注意保留你的时钟取反方式）：

```verilog
inst_ram inst_ram
(
    .clka  (clk             ),   
    .wea   (4'd0            ),
    .addra (cpu_inst_addr   ),
    .dina  (32'd0           ),
    .douta (cpu_inst_rdata  ) 
);

data_ram data_ram
(
    .clka  (clk                 ),   
    .wea   (cpu_data_wea        ),
    .addra (cpu_data_addr       ),
    .dina  (cpu_data_wdata      ),
    .douta (cpu_data_rdata      ) 
);
```

这里还需要注意的是，Lab4中我们没有实现按位写入，这里我们需要将wen扩展成4位的wea，来控制每4字节的写入情况。

## 跳转指令测试通过，但跑功能测试炸一片

这个细节卓越班上课有提到过，可以先看看资料包里最近更新的卓越班上课PPT，为了提高大家有限时间内的开发效率，推荐大家把跳转指令从LAB4的ID阶段移动到MEM阶段。因为我们在后续57条指令的时候一样要处理精确异常的跳转，因此这么做其实不会增加工作量，反而可以减少很多跳转指令在ID阶段可能遇到但LAB4没有考虑的坑。