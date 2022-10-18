# XMC&trade; MCU: Math CORDIC

This code example uses the Math CORDIC block to perform circular, hyperbolic, and logarithmic operations. The example demonstrates the blocking, non-blocking, and direct register write operations of the CORDIC block.

## Requirements

- [ModusToolbox&trade; software](https://www.infineon.com/modustoolbox) v3.0
- [SEGGER J-Link software](https://www.segger.com/downloads/jlink/#J-LinkSoftwareAndDocumentationPack)
- Programming language: C
- Associated parts: [XMC1000 MCU](https://www.infineon.com/cms/en/product/microcontroller/32-bit-industrial-microcontroller-based-on-arm-cortex-m/32-bit-xmc1000-industrial-microcontroller-arm-cortex-m0/) parts

## Supported toolchains (make variable 'TOOLCHAIN')

- GNU Arm&reg; embedded compiler v10.3.1 (`GCC_ARM`) - Default value of `TOOLCHAIN`
- Arm&reg; compiler v6.16 (`ARM`)
- IAR C/C++ compiler v9.30.1 (`IAR`)

## Supported kits (make variable 'TARGET')

- [XMC1400 boot kit](https://www.infineon.com/cms/en/product/evaluation-boards/kit_xmc14_boot_001/) (`KIT_XMC14_BOOT_001`) - Default value of `TARGET`

## Hardware setup

This example uses the board's default configuration. See the kit user guide to ensure that the board is configured correctly.

## Software setup

Install a terminal emulator if you do not have one. Instructions in this document use [Tera Term](https://ttssh2.osdn.jp/index.html.en).

## Theory

The CORDIC coprocessor computes trigonometric, linear, hyperbolic, and related functions using the CORDIC algorithm. The CORDIC algorithm is a useful convergence method, which performs the mathematical operations through an iterative process. The main advantage of using this algorithm is the fast calculation speed compared to software, and high accuracy. These operations are essential in applications such as the following:

- Electric power steering and motor control: Angle computation

- Motor control: Park transformation

- Digital filtering and PI control loop:Linear operations

The X and Y data input for the CORDIC coprocessor can be sent directly in 'Qm.n' format. 'Q' is a binary fixed-point number format, where the number of integer bits (indicated by 'm') and the number of fractional bits (indicated by 'n') are specified. The 'Qm.n' format is generally used in hardware that does not have a floating-point unit.

In this code example, 'Q0.23' and 'Q1.22' formats are used:

- `XMC_MATH_Q0_23_t` – 1 signed bit, 0 integer bits, 23 fraction bits

- `XMC_MATH_Q1_22_t` – 1 signed bit, 1 integer bits, 22 fraction bits

The functions to convert the Q format to float are implemented in the code example. The theory behind the conversion is to right-shift the fractional bits to obtain the float value.

The Z input and output data also follow the same procedure for linear operations. However, for circular and hyperbolic functions, the accessible Z result data and input data are normalized. The Z input data needs to be scaled up by ((2^23)/pi). For example, if a Z input of pi/2 needs to be provided, the actual Z input that is sent to the CORDIC coprocessor is as follows:

```
((pi/2) * ((2^23)/pi)) = 0x400000
```

On the other hand, the Z result from the CORDIC block for circular and hyperbolic functions need to be scaled down by ((2^23)/pi). For example, if the CORDIC Z output is (2^23)/pi (example considered for better understanding), the actual Z result is as follows:

```
(((2^23)/pi) / ((2^23)/pi)) = 1
```

See the following for more details of the CORDIC coprocessor:

- [AP32307 - Math Coprocessor application note](https://www.infineon.com/cms/en/product/microcontroller/32-bit-industrial-microcontroller-based-on-arm-cortex-m/32-bit-xmc1000-industrial-microcontroller-arm-cortex-m0/#!?fileId=5546d4624e765da5014ed91d36911f60)

- [XMC1000 family technical reference manuals](https://www.infineon.com/cms/en/product/microcontroller/32-bit-industrial-microcontroller-based-on-arm-cortex-m/32-bit-xmc1000-industrial-microcontroller-arm-cortex-m0/#document-group-myInfineon-44)


## Using the code example

Create the project and open it using one of the following:

<details open><summary><b>In Eclipse IDE for ModusToolbox&trade; software</b></summary>

1. Click the **New Application** link in the **Quick Panel** (or, use **File** > **New** > **ModusToolbox&trade; Application**). This launches the [Project Creator](https://www.infineon.com/ModusToolboxProjectCreator) tool.

2. Pick a kit supported by the code example from the list shown in the **Project Creator - Choose Board Support Package (BSP)** dialog.

   When you select a supported kit, the example is reconfigured automatically to work with the kit. To work with a different supported kit later, use the [Library Manager](https://www.infineon.com/ModusToolboxLibraryManager) to choose the BSP for the supported kit. You can use the Library Manager to select or update the BSP and firmware libraries used in this application. To access the Library Manager, click the link from the **Quick Panel**.

   You can also just start the application creation process again and select a different kit.

   If you want to use the application for a kit not listed here, you may need to update the source files. If the kit does not have the required resources, the application may not work.

3. In the **Project Creator - Select Application** dialog, choose the example by enabling the checkbox.

4. (Optional) Change the suggested **New Application Name**.

5. The **Application(s) Root Path** defaults to the Eclipse workspace which is usually the desired location for the application. If you want to store the application in a different location, you can change the *Application(s) Root Path* value. Applications that share libraries should be in the same root path.

6. Click **Create** to complete the application creation process.

For more details, see the [Eclipse IDE for ModusToolbox&trade; software user guide](https://www.infineon.com/MTBEclipseIDEUserGuide) (locally available at *{ModusToolbox&trade; software install directory}/ide_{version}/docs/mt_ide_user_guide.pdf*).

</details>

<details open><summary><b>In command-line interface (CLI)</b></summary>

ModusToolbox&trade; software provides the Project Creator as both a GUI tool and the command line tool, "project-creator-cli". The CLI tool can be used to create applications from a CLI terminal or from within batch files or shell scripts. This tool is available in the *{ModusToolbox&trade; software install directory}/tools_{version}/project-creator/* directory.

Use a CLI terminal to invoke the "project-creator-cli" tool. On Windows, use the command line "modus-shell" program provided in the ModusToolbox&trade; software installation instead of a standard Windows command-line application. This shell provides access to all ModusToolbox&trade; software tools. You can access it by typing `modus-shell` in the search box in the Windows menu. In Linux and macOS, you can use any terminal application.

This tool has the following arguments:

Argument | Description | Required/optional
---------|-------------|-----------
`--board-id` | Defined in the `<id>` field of the [BSP](https://github.com/Infineon?q=bsp-manifest&type=&language=&sort=) manifest | Required
`--app-id`   | Defined in the `<id>` field of the [CE](https://github.com/Infineon?q=ce-manifest&type=&language=&sort=) manifest | Required
`--target-dir`| Specify the directory in which the application is to be created if you prefer not to use the default current working directory | Optional
`--user-app-name`| Specify the name of the application if you prefer to have a name other than the example's default name | Optional

<br>

The following example will clone the "[Math Cordic](https://github.com/Infineon/mtb-example-xmc-math-cordic)" application with the desired name "MathCordic" configured for the *KIT_XMC14_BOOT_001* BSP into the specified working directory, *C:/mtb_projects*:

   ```
   project-creator-cli --board-id KIT_XMC14_BOOT_001 --app-id mtb-example-xmc-math-cordic --user-app-name MathCordic --target-dir "C:/mtb_projects"
   ```

**Note:** The project-creator-cli tool uses the `git clone` and `make getlibs` commands to fetch the repository and import the required libraries. For details, see the "Project creator tools" section of the [ModusToolbox&trade; software user guide](https://www.infineon.com/ModusToolboxUserGuide) (locally available at *{ModusToolbox&trade; software install directory}/docs_{version}/mtb_user_guide.pdf*).

</details>

<details open><summary><b>In third-party IDEs</b></summary>

**Note:** Only VS Code is supported.

1. Follow the instructions from the **In command-line interface (CLI)** section to create the application, and import the libraries using the `make getlibs` command.

2. Export the application to a supported IDE using the `make <ide>` command.

   For a list of supported IDEs and more details, see the "Exporting to IDEs" section of the [ModusToolbox&trade; software user guide](https://www.infineon.com/ModusToolboxUserGuide) (locally available at *{ModusToolbox&trade; software install directory}/docs_{version}/mtb_user_guide.pdf*).

3. Follow the instructions displayed in the terminal to create or import the application as an IDE project.

</details>


## Operation

1. Connect the board to your PC using a micro-USB cable through the debug USB connector.

2. Program the board using Eclipse IDE for ModusToolbox&trade; software:

   1. Select the application project in the Project Explorer.

   2. In the **Quick Panel**, scroll down, and click **\<Application Name> Program (JLink)**.

3. Once the device is programmed, the blocking `cos()`, non-blocking `sinh()`, and `ln()` CORDIC operations are performed and the result is displayed on the serial terminal:

   **Figure 1. Serial terminal log**

   ![](images/serial_terminal_log.jpg)


## Debugging

You can debug the example to step through the code. In the IDE, use the **\<Application Name> Debug (JLink)** configuration in the **Quick Panel**. For more details, see the "Program and debug" section in the [Eclipse IDE for ModusToolbox&trade; user guide](https://www.infineon.com/MTBEclipseIDEUserGuide).


## Design and implementation

This code example is divided into three steps:

- **Step 1:** This step demonstrates the use of the CORDIC coprocessor to perform blocking operations using the `cos()` operation. The angle, pi/3 in this case, is first scaled up by `((2^23)/pi)` because it is the input to the CORDIC Z value (check the `XMC_MATH_CORDIC_Cos()` implementation in *xmc_math.c*). The result obtained from the CORDIC block is converted from the Q0_23 format to float to obtain `cos(pi/3)`.

- **Step 2:** This step demonstrates the procedure to use non-blocking CORDIC operations. Here, `sinh()` of pi/4 is calculated. The CORDIC interrupt is configured and enabled. The angle is again scaled up by `((2^23)/pi)` and fed into the CORDIC block. Once the CORDIC operation is started, the CPU can perform other actions instead of polling such as in blocking operations. After the CORDIC operation completion, an interrupt is triggered and the result is read. The result is converted from Q1_22 format to float to obtain `sinh(pi/4)`.

- **Step 3:** This step uses direct register writes to perform a natural logarithmic operation. The fundamental equation behind this process is `ln(x) = 2*atanh[(x-1)/(x+1)]`. Because the `atanh()` functionality is required, the CORDIC block is used in hyperbolic vectoring mode. The initial inputs are fed into CORDIC data registers. Setting the *Start Mode* to '0' ensures that the CORDIC operation starts automatically after the X data register is written. Once the operation is complete, the final result is scaled down by `((2^23)/pi)` to obtain the result of `ln(3)`.

The result of each step and debug messages are displayed on the serial terminal.


### Resources and settings

The project uses a custom *design.modus* file because the following settings were modified in the default *design.modus* file.

**USIC (UART) settings**

![](images/uart_settings.jpg)

<br>

**USIC interrupt settings (For XMC1400 devices only)**

![](images/interrupt_settings.jpg)

<br>

**UART Tx pin settings**

![](images/tx_pin_settings.jpg)

<br>

**UART Rx pin settings**

![](images/rx_pin_settings.jpg)

<br>

## Related resources

Resources | Links
--------------------|----------------------
Code examples | [Using ModusToolbox&trade; software](https://github.com/Infineon/Code-Examples-for-ModusToolbox-Software) on GitHub
Device documentation | [XMC1000 family datasheets](https://www.infineon.com/cms/en/product/microcontroller/32-bit-industrial-microcontroller-based-on-arm-cortex-m/32-bit-xmc1000-industrial-microcontroller-arm-cortex-m0/#document-group-myInfineon-49) <br> [XMC1000 family technical reference manuals](https://www.infineon.com/cms/en/product/microcontroller/32-bit-industrial-microcontroller-based-on-arm-cortex-m/32-bit-xmc1000-industrial-microcontroller-arm-cortex-m0/#document-group-myInfineon-44) 
Development kits |[XMC eval boards](https://www.infineon.com/cms/en/product/microcontroller/32-bit-industrial-microcontroller-based-on-arm-cortex-m/#boards)
Libraries on GitHub | [mtb-xmclib-cat3](https://github.com/Infineon/mtb-xmclib-cat3) – XMC&trade; MCU peripheral library (XMCLib)
Tools | [Eclipse IDE for ModusToolbox&trade; software](https://www.infineon.com/modustoolbox) – ModusToolbox&trade; software is a collection of easy-to-use software and tools enabling rapid development with Infineon MCUs, covering applications from embedded sense and control to wireless and cloud-connected systems using AIROC&trade; Wi-Fi and Bluetooth&reg; connectivity devices.

## Other resources

Infineon provides a wealth of data at www.infineon.com to help you select the right device, and quickly and effectively integrate it into your design.

For XMC&trade; MCU devices, see [32-bit XMC™ industrial microcontroller based on Arm® Cortex®-M](https://www.infineon.com/cms/en/product/microcontroller/32-bit-industrial-microcontroller-based-on-arm-cortex-m/).

## Document history

Document title: *CE232787* – *XMC&trade; MCU: Math CORDIC*

 Version | Description of change
 ------- | ---------------------
 1.0.0   | New code example
 1.1.0   | Added support for new kits
 2.0.0   | Updated to support ModusToolbox&trade; software v3.0; CE will not be backward compatible with previous versions of ModusToolbox&trade; software
------

All other trademarks or registered trademarks referenced herein are the property of their respective owners.

© 2022 Infineon Technologies AG

All Rights Reserved.

### Legal disclaimer

The information given in this document shall in no event be regarded as a guarantee of conditions or characteristics. With respect to any examples or hints given herein, any typical values stated herein and/or any information regarding the application of the device, Infineon Technologies hereby disclaims any and all warranties and liabilities of any kind, including without limitation, warranties of non-infringement of intellectual property rights of any third party.

### Information

For further information on technology, delivery terms and conditions and prices, please contact the nearest Infineon Technologies Office (www.infineon.com).

### Warnings

Due to technical requirements, components may contain dangerous substances. For information on the types in question, please contact the nearest Infineon Technologies Office.

Infineon Technologies components may be used in life-support devices or systems only with the express written approval of Infineon Technologies, if a failure of such components can reasonably be expected to cause the failure of that life-support device or system or to affect the safety or effectiveness of that device or system. Life support devices or systems are intended to be implanted in the human body or to support and/or maintain and sustain and/or protect human life. If they fail, it is reasonable to assume that the health of the user or other persons may be endangered.

-------------------------------------------------------------------------------
