# Blinky Example for Zephyr + Ambiq Apollo4 Plus EVB (Windows VSCode Setup Guide)

**Author**: Tony Fu
**Date**: 2025/6/5

This guide describes how to set up a development environment for Zephyr RTOS on Windows using Visual Studio Code, specifically for the Ambiq Apollo4 Plus Evaluation Board (EVB).

---

## 1. Create a Base Directory

Create a top-level directory to store all Zephyr-related content. Example:

```plaintext
C:\zephyr\
```

This will hold your source code, virtual environment, and related tools.

---

## 2. Install Required Tools

### 2.1 Install Software Dependencies
Install the following tools:

* JLink
* Visual Studio Code
* Git
* CMake
* Python
* [Ninja](https://github.com/ninja-build/ninja/releases)

> **Note:** Ninja does not come with an installer. Simply extract the `ninja.exe` file and place it in a directory that is on your `PATH` (e.g., `C:\tools\ninja\`). Do the same for any other tools if not installed via Chocolatey.

You may also use [Chocolatey](https://chocolatey.org/install) to install them in one go:

```powershell
choco install git cmake python ninja
```

### 2.2 Download GNU Arm Embedded Toolchain

Download the [GNU Arm Embedded Toolchain](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-rm) and extract it to a directory. arm-gnu-toolchain-13.2.Rel1 seems to work well with Zephyr as of this writing.

---

## 3. (Optional) Create an Isolated Python Environment

Creating a virtual environment helps avoid conflicts with global Python packages.

```powershell
cd C:\zephyr
python -m venv env\zephyr
```

Then activate the environment:

```powershell
.\env\zephyr\Scripts\Activate.ps1
```

> If you see a permission error, open PowerShell as **Administrator** and run:
>
> ```powershell
> Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
> ```
>
> Then try activation again.

---

## 4. Install `west` (Zephyr's meta-tool)

Upgrade pip and install west:

```powershell
python -m pip install --upgrade pip
pip install west
```

---

## 5. Initialize Zephyr Project

We will clone the Zephyr source tree and its related modules using `west`. This creates a folder typically called `zephyrproject`. It will contain the core Zephyr RTOS source and its modules. We generally **do not modify** this directly unless we're contributing to Zephyr itself. Our application code should live outside or in a separate folder inside this tree. This following commands will take some time.

```powershell
west init zephyrproject
cd zephyrproject
west update
west zephyr-export
```

---

## 6. Install Zephyr Python Dependencies

Install all Python packages required by Zephyr:

```powershell
pip install -r zephyr/scripts/requirements.txt
```

---

## 7. Create a New Zephyr Application

### 7.1 Create an Application Folder

Inside the `zephyrproject` directory, create a subfolder to organize your applications. For example:

```
C:\zephyr\zephyrproject\myApps\
```

**Note:** If you have cloned this repository, you don't need to create a new application from scratch. Just move this repository to the `myApps` folder and ignore steps 7.2 and 7.3 below.

### 7.2 Create the Blinky Application

Now create a new application called `blinky`:

```
C:\zephyr\zephyrproject\myApps\blinky\
```

Inside this directory, create the following structure:

```
blinky/
├── CMakeLists.txt
├── prj.conf
└── src/
    └── main.c
```

### 7.3 File Contents

#### `CMakeLists.txt`

```cmake
cmake_minimum_required(VERSION 3.20.0)
find_package(Zephyr REQUIRED HINTS $ENV{ZEPHYR_BASE})
project(blinky)
target_sources(app PRIVATE src/main.c)
```

#### `prj.conf`

```plaintext
# Minimal configuration
CONFIG_GPIO=y
```

#### `src/main.c`

```c
#include <zephyr/kernel.h>
#include <zephyr/drivers/gpio.h>

#define LED_NODE DT_ALIAS(led0)
static const struct gpio_dt_spec led = GPIO_DT_SPEC_GET(LED_NODE, gpios);

int main(void)
{
    gpio_pin_configure_dt(&led, GPIO_OUTPUT_ACTIVE);

    while (1) {
        gpio_pin_toggle_dt(&led);
        k_msleep(500);
    }

    return 0;
}
```

This example toggles `led0`, which is defined in the devicetree for the board. The actual LED pin will depend on how it's mapped in the board's hardware description.

---

## 8. Build for Apollo4 Plus EVB

To cross-compile for the Ambiq Apollo4 Plus board, you need to specify the toolchain. If you're using PowerShell, set these environment variables:

```powershell
$env:ZEPHYR_TOOLCHAIN_VARIANT = "gnuarmemb"
$env:GNUARMEMB_TOOLCHAIN_PATH = "C:\toolchain\arm-gnu-toolchain-13.2.Rel1"
```

> ⚠️ Make sure that path points to the **root of the toolchain**, not the `bin` directory. The toolchain should contain the `bin`, `include`, and `lib` directories directly under it.

Then navigate to your application directory and build:

```powershell
cd C:\zephyr\zephyrproject\myApps\blinky
west build -b ambiq_apollo4p_evb .
```

If everything is set up correctly, the build system will generate the firmware in the `build\zephyr\` folder, including `zephyr.elf`, which you can later flash to the board.

---

## 9. Flash the Application

You should install the following extensions in Visual Studio Code:

- `Cortex-Debug` by marus25
- `C/C++` by Microsoft

And other extensions you find useful for C/C++ development.

To flash the application, connect your Ambiq Apollo4 Plus EVB to your computer via USB. Then, in VSCode, go to Run > Start Debugging. This will use the .vscode/launch.json configuration to flash the firmware.

Then click the green play button in the debug panel. You should now see the LED on the board blinking.
