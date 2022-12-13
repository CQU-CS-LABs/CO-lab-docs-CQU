# 环境安装

## Vivado

Vivado，硬件仿真、综合上板的工具。
### 下载Vivado

实验资料包采用Vivado 2019.2为基础，在更高的版本运行时，IP核升级选择`With Core Container Disabled`即可。

### 安装Vivado

在安装Vivado时，可以只保留以下选项，确保最小安装：

![vivado_install](../img/vivado_install.png)

## Verilator

Verilator，轻量仿真工具。

### 安装Verilator

Verilator 主要是在 Ubuntu 上进行开发和测试，也额外在 FreeBSD、Apple OS-X、Red Hat Linux和 GNU/Linux-ish 其他平台上进行了测试。Verilator 还可用于 Windows 上的 Linux 子系统 (WSL2) ，Cygwin 下的 Windows和Mingw(GCC-MNO-Cygwin) 下的 Windows。

Windows 用户推荐使用 WSL 中 Ubuntu 下安装 Verilator。其他操作系统的 Verilator 安装可自行谷歌或STFM[^1][^2]。

**注意**：在本实验仿真中，Verilator 版本需要 4.0 以上。但是低版本 Ubuntu （如 Ubuntu 20 ）无法下载高版本的 Verilator。低版本 Ubuntu 建议手动升级至 Ubuntu 22。可用 `lsb_release -a` 查看 Ubuntu 版本和其 Codename。

```shell
➜  CO-lab-material-CQU git:(2022) ✗ lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 22.04.1 LTS
Release:        22.04
Codename:       jammy
➜  CO-lab-material-CQU git:(2022) ✗ verilator --version
Verilator 4.220 2022-03-12 rev v4.220
```

Ubuntu 版本升级：
```shell
sudo nano /etc/apt/sources.list  # 编辑 sources.list
    - 替换所有的 [Codename] 为 jammy，Codename 可通过 lsb_release -a 命令查询
    - 可选：替换所有的 archive.ubuntu.com 为 mirrors.cqu.edu.cn，加快下载速度
    - nano 用法自行谷歌
sudo apt update
sudo apt full-upgrade -y
```

Ubuntu 22 中 Verlator 安装：
```shell
apt-get install verilator
```

> 如果有同学无法使用 apt-get 正常安装 Verilator（这种情况可能发生，尚不清楚原因），可以尝试 [brew 工具](https://brew.sh/)。

## GTKWave
GTKWave, 波形图查看工具。
### 安装 GTKWave

GTKWave, vcd(value change dump)波形图文件查看器。Verilator 在 trace 过程中可以生成 vcd 文件，记录每个时钟下各个变量的数值，可利用 GTKWave 生成波形图进行查看。
```shell
sudo apt update
sudo apt install gtkwave
```

### 使用说明
GTKWave总体界面和Vivado仿真界面相似，功能用法相近。现介绍一下基本用法，其他用法可自行探索或STFM。

在有 vcd 文件的目录中，利用 `gtkwave [filename].vcd` 进行查看。主界面如下：
![GTKWave主界面](../img/gtkwave-main.png)

模块内变量支持正则匹配，如下的 `ctrlBlock.*mem_wen`。
![GTKWave正则搜索](../img/gtkwave-regular-search.png)

可通过工具栏的 `view`，进行自定义设置：
![GTKWave View](../img/gtkwave-view.png)

GTKWave信号处，可右键进行单独设置，可利用 `insert Blank/Comment` 进行分隔/分组查看：
![GTKWave信号设置](../img/gtkwave-signals.png)

[^1]: [Installation — Verilator 5.003 documentation](https://verilator.org/guide/latest/install.html)
[^2]: 不同系统下对应的 verilator 版本：[verilator package versions - Repology](https://repology.org/project/verilator/versions)