# DNK230D 开发板支持 - 修改说明文档

## 一、修改概述

本次修改为 K230 Linux SDK 添加正点原子 DNK230D 开发板支持，主要涉及以下内容：

- 新增 U-Boot 板级配置
- 新增 Buildroot defconfig
- 新增 Linux kernel 补丁
- 更新系统启动配置
- 删除旧的 patch 文件

---

## 二、新增文件清单

### 2.1 U-Boot 相关文件

| 文件路径 | 说明 |
|---------|------|
| `buildroot-overlay/boot/uboot/u-boot-2022.10-overlay/board/canaan/k230d_canmv_atk_dnk230d/Kconfig` | 板级 Kconfig 配置 |
| `buildroot-overlay/boot/uboot/u-boot-2022.10-overlay/board/canaan/k230d_canmv_atk_dnk230d/Makefile` | 板级 Makefile |
| `buildroot-overlay/boot/uboot/u-boot-2022.10-overlay/board/canaan/k230d_canmv_atk_dnk230d/k230d_canmv_atk_dnk230d.c` | 板级初始化代码 |
| `buildroot-overlay/boot/uboot/u-boot-2022.10-overlay/configs/k230d_canmv_atk_dnk230d_defconfig` | U-Boot defconfig |
| `buildroot-overlay/boot/uboot/u-boot-2022.10-overlay/arch/riscv/dts/k230d_canmv_atk_dnk230d.dts` | U-Boot 设备树 |
| `buildroot-overlay/boot/uboot/u-boot-2022.10-overlay/include/dt-bindings/pinctrl/k230-pinctrl.h` | Pinctrl 头文件 |

### 2.2 Buildroot 相关文件

| 文件路径 | 说明 |
|---------|------|
| `buildroot-overlay/configs/k230d_canmv_atk_dnk230d_defconfig` | Buildroot defconfig |

### 2.3 Linux Kernel 补丁

| 文件路径 | 说明 |
|---------|------|
| `buildroot-overlay/linux/0020-add-alientek-dnk230d-board.patch` | DNK230D 板子 DTS 支持 |
| `buildroot-overlay/linux/0021-add-ili9881-lcd.patch` | ILI9881 LCD 屏幕支持 |
| `buildroot-overlay/linux/0022-fix-emmc-tuning-failed-printf.patch` | eMMC tuning 修复 |
| `buildroot-overlay/linux/0023-01studio-base-emmc-hdmi.patch` | 01studio 基础配置 |
| `buildroot-overlay/linux/display-st7701-480x640.dtsi` | ST7701 LCD 设备树头文件 |

---

## 三、修改文件详情

### 3.1 default.env - 启动命令修复

**文件:** `buildroot-overlay/board/canaan/k230-soc/default.env`

**修改内容:** 修复 Linux 启动命令，添加 `mmc dev` 初始化

```diff
- blinux=ext4load mmc ${mmc_boot_dev_num}:1 0x3000000 /fw_jump_add_uboot_head.bin && ext4load mmc ${mmc_boot_dev_num}:1 0x200000 /${k} && ext4load mmc ${mmc_boot_dev_num}:1 0x2200000 /k.dtb && bootm 0x3000000 - 0x2200000;
+ blinux=mmc dev ${mmc_boot_dev_num} && ext4load mmc ${mmc_boot_dev_num}:1 0x3000000 fw_jump_add_uboot_head.bin && ext4load mmc ${mmc_boot_dev_num}:1 0x200000 ${k} && ext4load mmc ${mmc_boot_dev_num}:1 0x2200000 k.dtb && bootm 0x3000000 - 0x2200000;
```

**修改说明:**
- 添加 `mmc dev ${mmc_boot_dev_num}` 初始化 MMC 设备
- 移除文件路径前导 `/`，使用相对路径

### 3.2 linux.fragment - 启用启动 Logo

**文件:** `buildroot-overlay/board/canaan/k230-soc/fragment/linux.fragment`

**修改内容:** 启用内核启动 Logo

```diff
  CONFIG_PINCTRL_K230_IOMUX=y
+ CONFIG_LOGO=y
+ CONFIG_LOGO_LINUX_CLUT224=y
```

### 3.3 post-image.sh - DNK230D 配置支持

**文件:** `buildroot-overlay/board/canaan/k230-soc/post-image.sh`

**修改内容:** 添加 DNK230D 板子配置处理

```bash
elif [ ${CONF} == "k230d_canmv_atk_dnk230d_defconfig" ]; then
    sed -i 's/^bootcmd=.*$/bootcmd=run blinux;/g' ${default_env_file}
    sed -i 's/^mmc_boot_dev_num=.*$/mmc_boot_dev_num=0/g' ${default_env_file}
```

**修改说明:**
- 设置 DNK230D 的 bootcmd 为 `run blinux`
- 设置 mmc_boot_dev_num 为 0（使用 SD 卡启动）

### 3.4 Kconfig - U-Boot 板子选项

**文件:** `buildroot-overlay/boot/uboot/u-boot-2022.10-overlay/arch/riscv/Kconfig`

**修改内容:**

```diff
  config TARGET_K230D_CANMV
      bool "Support k230D_CANMV(K230PI zero)"
      select SYS_CACHE_SHIFT_6

+ config TARGET_K230D_CANMV_ATK_DNK230D
+     bool "Support k230D_CANMV ATK DNK230D (Alientek)"
+     select SYS_CACHE_SHIFT_6
+

  source "board/canaan/k230d_canmv/Kconfig"
+ source "board/canaan/k230d_canmv_atk_dnk230d/Kconfig"
```

### 3.5 dts/Makefile - 设备树编译

**文件:** `buildroot-overlay/boot/uboot/u-boot-2022.10-overlay/arch/riscv/dts/Makefile`

**修改内容:**

```diff
  dtb-$(CONFIG_TARGET_K230_CANMV_01STUDIO) += k230_canmv_01studio_mmc.dtb  k230_canmv_01studio.dtb
+ dtb-$(CONFIG_TARGET_K230D_CANMV_ATK_DNK230D) += k230d_canmv_atk_dnk230d.dtb
```

### 3.6 .gitignore - 忽略临时文件

**修改内容:**

```diff
  /output
  .last_conf
  /dl
+ build.log
+ *.bak
```

---

## 四、删除文件清单

删除以下旧的 Linux kernel patch 文件（已被新 patch 替代）：

| 文件 | 说明 |
|------|------|
| `0010-add-ili9881-lcd.patch` | 已由 0021 替代 |
| `0010-fix-emmc-tuning-failed-printf.patch` | 已由 0022 替代 |
| `0014-01studio-base-emmc-hdmi.patch` | 已由 0023 替代 |
| `0014-Add-clock-and-power-domain-support-for-AI-components.patch` | 已禁用 |
| `0015-dts-add-sensor-reset.patch` | 已禁用 |
| `0016-01studio-xcam-evt1-board-bluetooth.patch` | 已禁用 |
| `0017-clk-k230-Fix-AI-clock-rate-setting-and-multiplexer-c.patch` | 已禁用 |
| `0018-dtc-riscv-k230d-canmv-Add-I2C4-and-MIPI-CSI2-support.patch` | 已禁用 |
| `0018-pinctrl-canaan-Add-K230-IOMUX-driver.patch` | 已禁用 |
| `0019-dts-add-nonai2d.patch` | 已禁用 |
| `0019-pinctrl-k230-iomux-Simplify-driver-by-using-pinctrl_.patch` | 已禁用 |
| `Config.in` | 配置文件 |
| `riscv-Introduce-system-suspend-support.patch` | 已禁用 |

---

## 五、补丁制作方法

### 5.1 补丁文件格式

补丁文件使用标准 git diff 格式，包含以下部分：

```patch
From <commit-hash> Mon Sep 17 00:00:00 2001
From: Author Name <email@example.com>
Date: Date String
Subject: [PATCH] Patch Title

---
diff --git a/path/to/file b/path/to/file
...
```

### 5.2 制作补丁步骤

**方法一：从 git commit 生成补丁**

```bash
# 假设已在 kernel 源码目录做了修改并提交
git format-patch -1 HEAD -o output_dir/

# 或者从特定 commit 生成
git format-patch -1 <commit-hash> -o output_dir/
```

**方法二：手动创建补丁**

```bash
# 1. 备份原始文件
cp original_file original_file.bak

# 2. 修改文件
vim original_file

# 3. 生成 diff
diff -u original_file.bak original_file > new_patch.patch

# 或者使用 git diff（如果有 git 管理）
git diff > new_patch.patch
```

### 5.3 补丁应用方法

```bash
# 在 kernel 源码目录应用补丁
patch -p1 < ../patches/0020-add-alientek-dnk230d-board.patch

# 检查补丁是否能成功应用（不实际修改）
patch -p1 --dry-run < ../patches/0020-add-alientek-dnk230d-board.patch
```

### 5.4 Buildroot 补丁应用流程

Buildroot 会自动应用 `buildroot-overlay/linux/` 目录下的 `.patch` 文件：

1. Buildroot 解压 kernel 源码到 `output/<config>/build/linux-<version>/`
2. 按照 patch 文件名排序依次应用
3. 文件名格式：`<编号>-<描述>.patch`，如 `0020-add-alientek-dnk230d-board.patch`

### 5.5 补丁命名规范

建议使用以下命名格式：

```
<编号>-<功能描述>.patch
```

- 编号：从 0020 开始（保留原仓库编号空间）
- 描述：简短英文描述，如 `add-alientek-dnk230d-board`

---

## 六、编译指南

### 6.1 编译 DNK230D 镜像

```bash
# 安装工具链
sudo make toolchain_and_depend

# 编译 DNK230D 配置
make CONF=k230d_canmv_atk_dnk230d_defconfig
```

### 6.2 输出文件位置

```bash
output/k230d_canmv_atk_dnk230d_defconfig/images/sysimage-sdcard.img.gz
```

### 6.3 烧录方法

解压镜像文件，烧录到 SD 卡：

```bash
gunzip sysimage-sdcard.img.gz
# 使用 dd 或烧录工具写入 SD 卡
```

---

## 七、Git 操作记录

### 7.1 Commit 历史

```
f407d01 Update system config for DNK230D board
1285347 Remove old Linux kernel patches
10e54ef Add Alientek DNK230D board support
```

### 7.2 Remote 配置

```bash
# origin - 官方仓库
git@github.com:kendryte/k230_linux_sdk.git

# myfork - 个人 fork
git@github.com:zhonghui3/k230_linux_sdk.git
```

### 7.3 推送命令

```bash
git push myfork dev
```