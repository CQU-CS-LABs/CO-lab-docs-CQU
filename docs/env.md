# 环境配置

## 下载Vivado

我们的实验均采用Vivado 2021.2进行，大家可以到[Xilinx官网](https://www.xilinx.com/support/download.html)下载Vivado。

也可以使用我们的[内网下载地址](http://172.20.106.26/Xilinx_Unified_2021.2_1021_0703.tar.gz)。

## 安装Vivado

在安装Vivado时，可以只保留以下选项，确保最小安装：

![vivado_install](img/vivado_install.png)

*目前有接到同学反馈Vivado 2021.2磁盘占用空间过大而自己电脑装不下的问题，可以尝试借同学一个外置的硬盘存放解压后的安装文件，然后从外置硬盘安装，这样在自己电脑上的空间占用只需要74G左右，若仍有同学无法解决可以联系一下助教，我们可以考虑再做一份2019.2兼容的版本，但依然非常建议使用更快速的最新版本来提高开发效率。*

## 获取实验包

源址：[https://github.com/cyyself/CO-lab-material-CQU](https://github.com/cyyself/CO-lab-material-CQU)

镜像：
[https://gitee.com/cyyself/CO-lab-material-CQU](https://gitee.com/cyyself/CO-lab-material-CQU)

推荐同学们先安装[git](https://git-scm.com/)，然后打开你的命令行，输入以下命令下载实验包：

```shell
git clone https://gitee.com/cyyself/CO-lab-material-CQU.git
```

每次更新实验包时，先将命令行目录切换到该文件夹中，使用以下命令即可增量更新：

```shell
git pull
```