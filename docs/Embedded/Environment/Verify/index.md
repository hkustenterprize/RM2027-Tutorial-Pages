# 嵌入式 环境验证

本教程通过克隆、生成、编译和下载示例工程，验证以下工具能否正常协作：

- Git
- Visual Studio Code
- Make
- Arm GNU Toolchain
- STM32CubeMX
- J-Link
- Ozone

验证工程地址：
https://github.com/hkustenterprize/RM2027-Tutorial-Embedded-Env-Verify

## 1. 克隆验证工程

打开终端，执行：

```sh
git clone https://github.com/hkustenterprize/RM2027-Tutorial-Embedded-Env-Verify.git
```

```sh
cd RM2027-Tutorial-Embedded-Env-Verify
```

克隆完成后，工程根目录中应包含：

- `Core/`
- `Core/Src/main.c`
- `RM2027-Tutorial-Embedded-Env-Verify.ioc`

![终端中的命令和成功输出](RM%20Embedded%20Tutorial%20环境验证.assets/image.png)
![文件夹内容](RM%20Embedded%20Tutorial%20环境验证.assets/image-1.png)

## 2. 使用 STM32CubeMX 生成代码

### 2.1 打开 IOC 工程

打开 STM32CubeMX，选择 `File` → `Load Project`，然后打开仓库根目录中的：

```text
RM2027-Tutorial-Embedded-Env-Verify.ioc
```

工程打开后，应能看到芯片型号 `STM32F103C8Tx` 和已经配置的引脚。

![使用 STM32CubeMX 打开 IOC 工程](RM%20Embedded%20Tutorial%20环境验证.assets/Screenshot%202026-09-01%20161213.png)

### 2.2 生成代码

点击右上角的 `GENERATE CODE`。

如果 STM32CubeMX 提示是否打开工程目录或生成报告，可以直接关闭提示并返回工程目录。
![STM32CubeMX 弹窗提示](RM%20Embedded%20Tutorial%20环境验证.assets/Screenshot%202026-09-01%20185613.png)

生成完成后，工程根目录中应出现 `Drivers/`、`Makefile`、`startup_stm32f103xb.s` 和 `STM32F103xx_FLASH.ld`。

![生成后的工程目录](RM%20Embedded%20Tutorial%20环境验证.assets/image-2.png)

## 3. 使用 Visual Studio Code 打开工程

打开 Visual Studio Code，点击左上角菜单栏 `File` → `Open Folder`。

![在vscode中选择 Open Folder](RM%20Embedded%20Tutorial%20环境验证.assets/Screenshot%202026-09-01%20152840.png)

在弹出的对话框中选中克隆得到的 `RM2027-Tutorial-Embedded-Env-Verify` 文件夹，然后打开。

Visual Studio Code 的 Explorer 中应能看到 `Core/`、`Drivers/`、`Makefile` 和 `.ioc` 文件。

![在vscode中打开验证工程](RM%20Embedded%20Tutorial%20环境验证.assets/Screenshot%202026-09-01%20192749.png)

工程打开后，点击 `Terminal` → `New Terminal`。
![在菜单栏打开终端](RM%20Embedded%20Tutorial%20环境验证.assets/Screenshot%202026-09-01%20193251.png)

窗口下方将出现 Terminal 窗格。
![vscode终端已打开](RM%20Embedded%20Tutorial%20环境验证.assets/image-3.png)


## 4. 编译验证固件

在 Visual Studio Code 终端中执行：

```sh
make -j
```

该命令使用 Arm GNU Toolchain 编译 STM32CubeMX 生成的 Makefile 工程。

![make结果](RM%20Embedded%20Tutorial%20环境验证.assets/Screenshot%202026-09-01%20193341.png)

编译成功后，终端中不应出现 error，并会在 `build/` 中生成：

```text
RM2027-Tutorial-Embedded-Env-Verify.elf
RM2027-Tutorial-Embedded-Env-Verify.bin
RM2027-Tutorial-Embedded-Env-Verify.hex
```

下图中，终端已经完成编译；Explorer 中的 `build/` 目录也已出现。

![make 编译成功并生成 ELF 固件](RM%20Embedded%20Tutorial%20环境验证.assets/Screenshot%202026-09-01%20193445.png)

## 5. 连接调试器和目标板

![连接调试器和目标板](RM%20Embedded%20Tutorial%20环境验证.assets/IMG_20260905_215152.jpg)

## 6. 创建 Ozone 工程

### 6.1 选择目标芯片

打开 Ozone，选择 `File` → `New` → `New Project Wizard`。

![确认 Ozone 目标设备](RM%20Embedded%20Tutorial%20环境验证.assets/Screenshot%202026-09-01%20160349.png)

在 `Device` 右侧点击 `...`，搜索并选择 `STM32F103C8`。

![在 Ozone 中选择 STM32F103C8](RM%20Embedded%20Tutorial%20环境验证.assets/Screenshot%202026-09-01%20160331.png)

返回向导后，确认：

- `Device` 为 `STM32F103C8`
- `Register Set` 为 `Cortex-M3`
- Flash 起始地址为 `0x08000000`

点击 `Next`。

### 6.2 设置调试连接

确认以下设置：

- `Target Interface`：`SWD`
- `Target Interface Speed`：`4 MHz`
- `Host Interface`：`USB`

已连接的调试器应出现在下方列表中。

![设置 Ozone 调试连接](RM%20Embedded%20Tutorial%20环境验证.assets/image-4.png)

如果列表为空，请检查目标板供电、USB 连接和驱动，然后重新打开此页面。

点击 `Next`。

### 6.3 选择 ELF 文件

在 `Program File` 页面点击 `...`。

![选择 Ozone Program File](RM%20Embedded%20Tutorial%20环境验证.assets/Screenshot%202026-09-01%20160423.png)

进入工程的 `build/` 目录，选择：

```text
RM2027-Tutorial-Embedded-Env-Verify.elf
```

![在构建目录中选择 ELF 文件](RM%20Embedded%20Tutorial%20环境验证.assets/Screenshot%202026-09-01%20160456.png)

点击 `Next`。

### 6.4 完成可选设置

保持以下默认值：

- `Initial PC`：`ELF Entry Point`
- `Initial Stack Pointer`：`Read from Base Address Vector Table`
- `J-Link Script File`：留空
- `J-Link Log File`：留空

![Ozone 可选设置](RM%20Embedded%20Tutorial%20环境验证.assets/Screenshot%202026-09-01%20160515.png)

点击 `Finish`。

## 7. 下载并运行程序

Ozone 打开工程后，会显示源代码。

![Ozone 工程界面](RM%20Embedded%20Tutorial%20环境验证.assets/Screenshot%202026-09-01%20160820.png)

点击工具栏中的 最左上角的绿色按钮 `Download and Reset Program`。

### 7.1 处理许可提示

如果弹出 `License Missing`，点击 `Yes` 继续评估模式。

![Ozone License Missing 提示](RM%20Embedded%20Tutorial%20环境验证.assets/Screenshot%202026-09-01%20160851.png)

出现 J-Link 使用条款，点击 `Accept`。

![接受 J-Link 使用条款](RM%20Embedded%20Tutorial%20环境验证.assets/Screenshot%202026-09-01%20160919.png)

如果屏幕右下角出现 J-Link 软件更新提示，直接关闭提示。

![J-Link 软件更新提示](RM%20Embedded%20Tutorial%20环境验证.assets/Screenshot%202026-09-01%20160929.png)

### 7.2 确认连接和下载结果

下载完成后，Ozone 底部状态栏应显示目标已经连接，并显示接口速度。CPU 会暂停在程序入口附近（绿色高亮行）。

![Ozone 已连接并暂停 CPU](RM%20Embedded%20Tutorial%20环境验证.assets/Screenshot%202026-09-01%20160935.png)

点击工具栏中的 绿色三角按钮 `Run` 继续运行程序。

## 8. 检查运行结果

程序运行后，与 PB12 和 PC14 对应的两个指示灯应每 0.5s 翻转一次，并始终保持相反状态。观察，确认两个指示灯持续、交替且稳定地闪烁。

## 9. 完成检查

以下项目全部满足时，环境验证完成：

- [ ] 验证工程可以成功克隆。
- [ ] STM32CubeMX 可以打开 `.ioc` 工程并生成代码。
- [ ] `make -j` 可以成功编译验证固件。
- [ ] `build/RM2027-Tutorial-Embedded-Env-Verify.elf` 已生成。
- [ ] Ozone 可以识别调试器并连接 `STM32F103C8`。
- [ ] 固件可以下载并运行。
- [ ] PB12 和 PC14 对应的两个指示灯可以稳定地交替闪烁。

## 常见问题

### Make 找不到 Arm 编译器

重新打开终端并检查：

```text
make --version
arm-none-eabi-gcc --version
```

如果任一命令无法输出版本信息，请返回环境配置教程检查对应组件。

### Ozone 找不到调试器

检查 USB 连接，重新插拔调试器，观察调试器指示灯是否为常亮状态。

### Ozone 无法连接目标芯片

观察指示灯，检查目标板供电、调试器连接情况。

### 下载后程序没有运行

下载完成时 CPU 可能处于暂停状态。确认 Ozone 已连接目标设备，然后点击 `Run` 继续运行。
