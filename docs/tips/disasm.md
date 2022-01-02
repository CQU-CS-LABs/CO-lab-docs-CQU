# 反汇编

有时候我们需要反汇编来得到当前正在运行的指令，方便观察前后的数据依赖关系，找到

## 自带反汇编

对于lab4、功能测试和性能测试，大家可以在soft文件夹中找到以`.S`结尾的文件，这些包含了他们的反汇编结果，同时还有他们的地址。

## 手动反汇编

1. 在线查询

    [https://www.eg.bucknell.edu/~csci320/mips_web/](https://www.eg.bucknell.edu/~csci320/mips_web/)

2. 手动使用objdump

    可以使用[https://github.com/cyyself/disasm-mips](https://github.com/cyyself/disasm-mips)这个小工具，该工具支持交互查询以及从coe中读取。