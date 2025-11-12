# Nia IPG Application Firmware Readme

## Overview
Nia Implantable Pulse Generator (IPG) is part of the two device system that makes up the Nia product. It sits inside the patient's head and communicates via wireless link to the earpiece outside the patient's head. Earpiece provides power and communication to the IPG. This firmware runs on the MCU on the IPG. 
This firmware is closely tied with the IPG bootloader firmware as they run on the same MCU, and designed to work directly with the earpiece firmware

IPG main tasks:
1. Communication
    1. Cryptography
    2. SPI with FPGA that communicates with earpiece over inductive link
2. Managing Nia NeuroModulation Integrated Circuit (NMIC)
3. Stimulation
4. Impedance measurements
5. And more to come!  

## Device details
MCU used is STM32U585OIY6Q
IPG hardware version 1.5b

## Environment and Initial Setup
This project is developed in STM32CubeIDE. Download from https://www.st.com/en/development-tools/stm32cubeide.html.
1. Clone the firmware repository with SSH
2. Open STM32CubeIDE with the workspace of ../firmware/im
3. File->Import->General->Existing Projects into Workspace and select root directory ../firmware/im
4. Choose the correct project, i.e. ipg_app, and select Finish
5. Check that you can build the project with Project->Clean and/or Project->Build

## Files
1. No build artifacts or doxygen documentation are stored in git.
2. .ld files are linker files
3. image_builder.exe is a custom script used to post process binary firmware file after it is build in release configuration. See later.

## Configurations
1. Debug
2. Release: Note this configuration has some special build steps with custom build tools to handle things like crc calculation, and the addresses in the linker are modified to account for that. See project properties, bootloader project, linker file, image_header.h, and other resources. 

## Deployment
### Debug
The easiest way to test this firmware is via SWD. Build the firmware in Debug configuration, attach a STlink device to the earpiece, make sure it has power (via usb cable), and debug away! 
Use STMCubeProgrammer to wake up the device if it isn't responding. You may need to use different (i.e. power down) connection modes.
You will have to set up a debug configuration in the IDE and ensure your project is set up for debugging. You can do this when you press the Debug button the first time. Note that some functionality is different while debugging! See the #ifdef DEBUG preprocessor defines.

### Release
#### Overview
The other way to deploy this firmware requires use of the IPG Bootloader and apploader project and follows a more complex procedure. 

Code is laid out in the flash as follows. See memory_map.h for addresses :  

Bootloader  
Apploader                   
Application Active   
FPGA 1 active
FPGA 2 active
Signature active    

The apploader decides which version of the application to jump to. When updating we must update the nonactive partition.  
   
Code is the same for each of these partitions. This is done by:

#### Build process
Just build with the release configuration! Build configurations and automated build steps handle the rest.  

#### Initial Flashing process
Use STMCubeProgrammer. You may need to use different (i.e. power down) connection modes
1. Choose the appropriate MCU
2. Erase the flash completely
3. Create a signature for the firmware package (apploader, app, fpga1, fpga2)
3. Burn the images to the correct addresses

#### Firmware Upgrade Process
Once the whole device firmware package has been flashed, use ICD commands and harness.exe to update firmware. If this means nothing to you, ask for help! There are more detailed guides for this elsewhere.  
1. Build a new signature image with the new firmware application image
2. Use the UPDATE_FIRMWARE macro to update the application update partition. Internally this handles all the required steps. (As an example from a .cif file: { "macro": "UPDATE_FIRMWARE", "key": "C2E", "args": {"file": "C:/Users/Josh/Desktop/chronic_firmware/im_app_fw0_1_0.nif" } })
3. Once firmware is updated, also update the signature image.
  
What this does:  
1. Updates firmware with the "UPDATE_FIRMWARE" ICD command piece by piece. The MCU application handles writing the update, and the apploader handles the transition to a new firmware version.

Nia Therapeutics  
August 2023  
"# colab" 
