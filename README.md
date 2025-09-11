## Coral Wildlife crossings 
The coral-crossings is a subproject of the Wildlife Crossings Guide project. It is shown as "edge ai device" in the system diagram section.

The Wildlife Crossings Guide project proposes a system that directs animals toward designated overpasses and underpasses to ensure safe road crossings, aiming to save the millions of animals that die on Australian roads each year, including koalas, kangaroos, wallabies, and wombats.

note: This project is supported by NextPCB (https://www.nextpcb.com), an one-stop solution for PCB manufacturing, assembly, PCB Prototype, SMD Stencil, and Multilayer PCB.

[![License: GPL v3](https://img.shields.io/badge/License-GPL_v3-blue.svg)](https://github.com/teamprof/github-coral-crossings/blob/main/LICENSE)

<a href="https://www.buymeacoffee.com/teamprof" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/v2/default-yellow.png" alt="Buy Me A Coffee" style="height: 38px !important;width: 128px !important;" ></a>



---
### System diagram

```
         Kaolas ─►


spk-n       spk-1         edge ai       spk+1        spk+n
(on)        (off)         device        (off)        (off)

───────────────────────┐             ┌────────────────────────
                       | overpasses  |
       road            |      /      |         road
                       | underpasses |
───────────────────────┘             └────────────────────────
```

### System operation
There are two core devices in the Wildlife crossings project. The first one is a Edge AI device running on Coral Dev Board Micro, which looks for animal position and whose firmware is available on [coral-crossings](https://github.com/teamprof/github-coral-crossings). 
The second one is a Lora speaker device running on ESP32S3, which receives command from Lora Module and output sound to speaker, its firmware is available on [esp32s3-lora-speaker](https://github.com/teamprof/github-esp32s3-lora-speaker).

When animals (e.g., koalas) are detected to the left of the crossings, the far-left speaker emits the sound of their natural predators (e.g., dingoes). This encourages the animals (e.g., koalas) to move rightward toward the crossings.


### demo system
A simplified system with one edge AI device and two Lora speakers to illustrate the principal idea of the project, i.e., directing animals to the safe road crossings by the sound of their natural enemies.

[![crossings-demo-system](/doc/image/coral-crossings-demo-system.png)](https://github.com/teamprof/github-esp32s3-lora-speaker/blob/main/image/coral-crossings-demo-system.png)




---
## Hardware
The following components are required for this project:
1. [Coral Micro Board](https://coral.ai/products/dev-board-micro/)
2. ESP32S3 development boards
3. Semtech LLCC68 Lora module


---
## Edge AI Device Block Diagram
```
                                                     ┌─────────────┐
                                                     │   LLCC68    │ 
   ┌────────┐                                        └──────▲──────┘
   │ camera │                                               | (SPI)
   ├────────┤   (I2C)   ┌───────────────┐   (I2C)    ┌──────▼──────┐
   │ Coral  │◄─────────►│ level shifter │◄──────────►│  ESP32-S3   │
   └────────┘       1V8 └───────────────┘ 3V3        └─────────────┘

note: ESP32-S3 acts as a bridge between Coral Micro Board and Lora module. For MP product, Coral Micro Board can controls Lora module directly by its SPI.

```

[![edge AI device](/doc/image/coral-crossings.png)](https://github.com/teamprof/github-esp32s3-lora-speaker/blob/main/image/coral-crossings.png)




---
## Software 
1. Install [Coral Dev Board Micro](https://coral.ai/docs/dev-board-micro/get-started)
2. Clone this repository by "git clone https://github.com/teamprof/github-coral-crossings"
---


## Download and build firmwares for the Coral Crossings System
Since there are two MCUs (NXP i.MX RT1176 and ESP32S3) in the "ai edge device", They have to flash separately.

### Download, build and flash firmware on ESP32S3
Please following the instruction on "https://github.com/teamprof/github-esp32s3-lora-speaker" about how to flash the ESP32S3 firmware.

### Download, build and flash firmware on Coral Dev Board Micro
Launch a Ubuntu/Linux terminal and type the following commands to download, build and flash the firmware on the Coral Dev Board Micro
1. git clone https://github.com/teamprof/github-coral-crossings
2. cd github-coral-crossings
3. git clone --recurse-submodules -j8 https://github.com/google-coral/coralmicro 
4. git submodule update --init --recursive
5. bash coralmicro/setup.sh
6. cmake -B out -S .
7. make -C out -j4
8. python3 coralmicro/scripts/flashtool.py --build_dir out --elf_path out/coralmicro-app

For more information about creating a project, either in-tree or out-of-tree, see the guide
to [Build apps with FreeRTOS for the Dev Board Micro](https://coral.ai/docs/dev-board-micro/freertos/).

**Note:** This project depends on [coralmicro](https://github.com/google-coral/coralmicro),
which requires about 2.5 GB.  

## Run the Coral Crossings System
1. Launch a Serial terminal, set baud rate to 115200 and connect it to the USB port of the Coral Dev Board Micro
2. Power on both the Coral Dev Board Micro and ESP32S3. If everything goes smooth, you should see the followings on the serial terminal:
[![serial terminal screen](/doc/image/serial-terminal-.png)](https://github.com/teamprof/github-coral-crossings/blob/main/doc/image/serial-terminal-.png)


## Setup and run Python script
Install Python plugins and run the Python script to see the image and detection results from the Coral Dev Board Micro
1. python3 -m pip install -r py/requirements.txt
2. python3 py/coral_crossings.py
3. Place an animal in front of the camera. If the animal is detected, you should see a box around the animal on the GUI app. The "User LED" on the Coral Dev Board Micro should also be turned on.
4. Move the animal to left, you should see a red-colored rectangle in the GUI app around the left region. The left speaker should output dingoes' sound.
5. Move the animal to middle, you should see a green-colored rectangle in the GUI app around the middle region. All the left speakers should be silent.
6. Move the animal to right, you should see a red-colored rectangle in the GUI app around the right region. The right speaker should output dingoes' sound.

If everything goes smooth, you should see the following GUI app on Ubuntu:
[![gui app screen](/doc/image/gui-app.png)](https://github.com/teamprof/github-coral-crossings/blob/main/doc/image/gui-app.png)
// !!!

---

## Hardware connection between Coral Dev Board Micro and ESP32S3
```
+------------------------+     +---------------------+
| Coral Dev Board Micro  |     |       ESP32S3       |
+-------+----------------+     +------+--------------+
| GPIO  |   description  |     | GPIO |  description |
+-------+----------------+     +------+--------------+
| kSCL1 |   I2C1 SCL     |     |  13  |   I2C SCL    |
| kSDA1 |   I2C1 SDA     |     |  11  |   I2C SDA    |
+-------+----------------+     +------+--------------+
```

## I2C communication format 
```
command: IsLoraReady
                  +----------------+----------------+
                  |     byte 0     |     byte 1     |
                  +----------------+----------------+
 master to slave  |  IsLoraReady   |       0        |
                  +----------------+----------------+
 slave to master  |       0        |     result     |
                  +----------------+----------------+


command: AudioCommand
                  +----------------+----------------+----------------+----------------+----------------+----------------+----------------+
                  |     byte 0     |     byte 1     |     byte 2     |     byte 3     |     byte 4     |     byte 5     |     byte 6     |
                  +----------------+----------------+----------------+----------------+----------------+----------------+----------------+
 master to slave  |  AudioCommand  |     event      |    audio_id    |     volume     |    did (low)   |    did (high)  |        0       |
                  +----------------+----------------+----------------+----------------+----------------+----------------+----------------+
 slave to master  |       0        |       0        |       0        |       0        |       0        |       0        |     result     |
                  +----------------+----------------+----------------+----------------+----------------+----------------+----------------+

```

## Flow of communication between Coral Dev Board Micro and ESP32
```
    coral                                     esp32s3
      |                                          |
      |        I2cCommand::IsLoraReady           |
      | ---------------------------------------> |
      |                                          |
      |        I2cResponse::LoraUnknown          |
      | <--------------------------------------- |
      |                                          |
      |                                          |
      |     (query IsLoraReady until ok)         |
      |                                          |
      |                                          |
      |        I2cCommand::IsLoraReady           |
      | ---------------------------------------> |
      |                                          |
      |        I2cResponse::LoraOk               |
      | <--------------------------------------- |
      |                                          |
      |                                          |
      |  I2cCommand::AudioCommand                |
      |  <dst_id> <event> <audio_id> <volumne>   |
      | ---------------------------------------> |
      |                                          |
      |        I2cResponse::Success              |
      | <--------------------------------------- |
      |                                          |
      |                                          |
      |  I2cCommand::AudioCommand                |
      |  <dst_id> <event> <audio_id> <volumne>   |
      | ---------------------------------------> |
      |                                          |
      |        I2cResponse::Success              |
      | <--------------------------------------- |
      |                                          |

```

## LED
- User LED: on when detected animal 
- Status LED: on when I2cResponse::LoraOk, off otherwise

---

## Code explanation

### I2C address
The I2C slave address of ESP32S3 is defined in the macro "I2C_DEV_ADDR" in the file "./src/app/thread/ThreadReporter.cpp"
Here's an example of setting the I2C slave address to "0x55".
```
#define I2C_DEV_ADDR ((uint8_t)0x55)
```

### ThreadReporter
The I2C communication code is implemented in the file "./src/app/thread/ThreadReporter.cpp". "ThreadReporter" is responsible for the following tasks:
1. Pool ESP32S3 on I2C bus regularly, to check Lora status on ESP32S3
2. listens event from "ThreadInference", sends commands to ESP32S3 via I2C

### ThreadInference
"ThreadInference" (./src/app/thread/ThreadInference.cpp) is responsible for the following tasks:
1. initialize tesnsorflow machine learning code 
2. perform objects detection regularly
3. computes animal location and sends event to "ThreadReporter" and "QueueMainM7"

### QueueMainM7
"QueueMainM7" (./src/app/thread/QueueMainM7.cpp) is responsible for the following tasks:
1. Turn "User LED" on when detected animal 
2. Turn "Status LED" on when I2cResponse::LoraOk is received

### Please refer to source code for details

---

## System image
[![system image](/doc/image/system-img.png)](https://github.com/teamprof/github-coral-crossings/blob/main/doc/image/system-img.png)

## Demo
Video demo is available on [Coral crossings video](https://www.youtube.com/watch?v=)  


---
### Debug
Enable or disable log be defining/undefining macro on "src/LibLog.h"

Debug is disabled by "#undef DEBUG_LOG_LEVEL"
Enable trace debug by "#define DEBUG_LOG_LEVEL Debug"

Example of LibLog.h
```
// enable debug log by defining the following macro
#define DEBUG_LOG_LEVEL Debug
// disable debug log by comment out the macro DEBUG_LOG_LEVEL 
// #undef DEBUG_LOG_LEVEL
```
---
### Troubleshooting
If you get compilation errors, more often than not, you may need to install a newer version of the coralmicro.

Sometimes, the project will only work if you update the board core to the latest version because I am using newly added functions.

---
### Issues
Submit issues to: [Coral crossings issues](https://github.com/teamprof/github-coral-crossings/issues) 

---
### TO DO
1. Search for bug and improvement.
---

### Contributions and Thanks
Many thanks to the following authors who have developed great software and libraries.
1. [NextPCB](https://www.nextpcb.com)
2. [Coral team](https://coral.ai/about-coral/)

#### Many thanks for everyone for bug reporting, new feature suggesting, testing and contributing to the development of this project.
---

### Contributing
If you want to contribute to this project:
- Report bugs and errors
- Ask for enhancements
- Create issues and pull requests
- Tell other people about this library
---

### License
- The project is licensed under GNU GENERAL PUBLIC LICENSE Version 3
---

### Copyright
- Copyright 2025 teamprof.net@gmail.com. All rights reserved.



