# Standard Pin Names Porting Guide

This guide aims to help vendors to comply with the new pin naming guidelines.

There is a set of generic guidelines that apply to all boards, and further sets of guidelines that additionally apply to boards that have certain compatible connectors. The current guideline documents can be found here:
* [General Pin Names Design Document](https://github.com/ARMmbed/mbed-os/blob/feature-std-pin-name/docs/design-documents/hal/0004-pin-names-general-guidelines.md)
* [Arduino Uno Pin Names Design Document](https://github.com/ARMmbed/mbed-os/blob/feature-std-pin-name/docs/design-documents/hal/0005-pin-names-Arduino-Uno-standard.md)

## Generic Pin Names

The generic guidelines currently define rules for the naming of LED pins, button pins and UART pins. In summary, the rules are:
* LED pins are defined as `LEDx` (e.g. `LED0`, `LED1`)
* Button pins are defined as `BUTTONx` (e.g. `BUTTON0`, `BUTTON1`)
* UART pins for communication with host are defined as `USBTX` and `USBRX`
* Pin aliases are allowed (e.g. `#define RED_LED LED1`)
* Pin definitions must be a `#define`, not an `enum`
* A physical MCU pin can be assigned to no more than one LED, button or UART pin definition
* A pin cannot be assigned to itself
* A pin cannot be defined as `NC` (not connected)

Please refer to the design document for the full specification.

## Arduino Pin Names
The Arduino guidelines currently define rules for the naming of the Arduino Uno connector pins, the functionality that each pin must support and the definition of Arduino Uno pin aliases. In summary, rules that apply to all Arduino pins are:
* All ARDUINO_UNO_XXX pins are defined
* A pin cannot be assigned to itself
* A pin cannot be defined as `NC` (not connected)

Also, every board that has the Arduino Uno connector must include `ARDUINO_UNO` in `targets.json` in the `supported_form_factors` list.

There are additional rules for different types of pins. Below is a summary of those rules.

### Digital and Analog pins
* Pins must be defined in a `PinName` enum
* Digital pins must be defined as `ARDUINO_UNO_Dxx`
* Analog pins must be defined as `ARDUINO_UNO_Ax`
* Analog pins must provide ADC functionality
* A physical MCU pin can be assigned to no more than one Arduino Uno pin

### I2C, SPI and UART pins
* Pins D14, D15 must provide I2C functionality (SDA, SCL)
* Pins D10, D11, D12, D13 must provide SPI functionality (CS, MOSI, MISO, SCK)
* Pins D0, D1 must provide UART functionality (RX, TX) 

### Other pins
* Mbed boards may provide PWM and timer functionality on certain Dx pins
* Applications cannot assume consistent availability of PWM and timer pins across Mbed boards
* The reset pin must be wired correctly in hardware but is not required to be defined in `PinNames.h`

Please refer to the design document for the full specification.

## Compliance Testing

After you have updated the `PinNames.h` files of your targets according to the above guidelines, you can check the files for compliance using two different methods.
* Python script: performs syntax checks on pin definitions according pre-defined coding style rules
* Greentea: performs build and run time checks on Mbed boards to make sure pins names and integration with Drivers work as expected

### Using pinvalidate<span>.py</span> Python script
This is a Python script which you can use to quickly check the compliance of `PinNames.h` files with the above rules.

The script can be found in `mbed-os/hal/tests/TESTS/pin_names`.

The script has a number of options which you can see in the help output (`pinvalidate.py -h`).

You can validate `PinNames.h` files either by passing the path to the files with the `-p` flag, or by passing the name of the targets with the `-t` flag and letting the script automatically discover the corresponding `PinNames.h` file (this feature is currently experimental). You can also pass partial names and the script will try to match the intended target.

The following examples would validate the same file:
`pinvalidate.py -t L4S5I`
`pinvalidate.py -p targets/TARGET_STM/TARGET_STM32L4/TARGET_STM32L4S5xI/TARGET_B_L4S5I_IOT01A/PinNames.h`

Multiple targets or paths can be separated by comma:
`pinvalidate.py -t L475,L4S5I,K22F,NUCLEO_F411RE`

You can also pass the `-a` flag, instead of `-p` or `-t`, to validate all `PinNames.h` files. The script will process every `PinNames.h` file that is discovered in the `mbed-os` directory (and subdirectories).

There are currently two test suites that you can run against a `PinNames.h` file:
* The Generic Pin Names test suite (`generic`)
* The Arduino Uno Pin Names test suite (`arduino_uno`)

When you are passing `PinNames.h` files by path, the script will only execute the `generic` test suite by default. When you are passing targets by name, the script will attempt to determine if the target has the Arduino Uno form factor by looking at the information available in `targets.json`. If it can find the target, and determine that it has the Arduino Uno form factor, it will also, by default, run the `arduino_uno` test suite.

You can manually choose which test suites to run using the `-n` flag:
`pinvalidate.py -t L475 -n generic`
`pinvalidate.py -t L475 -n generic,arduino`

The scripts supports several output formats. The default output format is `prettytext`. You can choose the output format with the `-o` flag:
`pinvalidate.py -t L4S5I -o json`
`pinvalidate.py -t L4S5I -o html`

You may also want to use the `-w` flag to write the output to a file when choosing HTML or JSON output:
`pinvalidate.py -t L4S5I -o html -w output.html`

The `prettytext` format supports four levels of detail/verbosity, selected with the usual `-v` flag:
`pinvalidate.py -t L4S5I` (level 0)
`pinvalidate.py -t L4S5I -v` (level 1)
`pinvalidate.py -t L4S5I -vvv` (level 3)
* At the default level (0), the script will output a short summary table showing a PASS/FAIL outcome for each of the validated files/targets.
* At level 1, the script will output a summary table showing a PASS/FAIL outcome for each test suite ran against a target.
* At level 2, the script will output a table showing a PASS/FAIL outcome for each test case ran against a target.
* At level 3, the script will output all of the above information, with details of each individual error.

When HTML output is chosen, the script will output the report as an interactive HTML table that you can view in a web browser. Each row is the output from one test case, with a PASS/FAIL outcome. You can view the details of any errors by expanding the last cell of the row.

JSON and HTML output always include the highest level of detail.

### Using Greentea

There is also a Greentea test to facilitate compliance testing on the target itself.

After setting up your environment to be able to run Greentea tests, you can run the compliance test:
`mbed test -t GCC_ARM -m B_L4S5I_IOT01A -n *pin_names* --run -v`