## **Background:**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Hello world, as I am new to PCB hardware design, a friend (a senpai) of mine has a garden and is planning to turn his garden into a smart garden, so he gave me requirement design and I will design and make the PCB for him. And I thought it would be a good idea to share my approach, my way of learning of how to be a complete beginner to how to be able to design a PCB full-fledge.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Beginners could view this as a reference for their new journey.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; For guys with experience, constructive feedback is always welcome. 

## **Some good materials before we dive in:**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; These are some good resources that I mainly use for this project:

1. How to use Altium and learn the PCB design process in general: [This youtube video of Robert Feranec](https://www.youtube.com/watch?v=PqFtSpAXB9Q)
2. Places to order, find reliable electrical components: [Digikey](https://www.digikey.com/), [Mouser](https://www.mouser.vn/),
3. Tools to quickly download footprint and schematic symbols: [Altium Library Loader](https://www.samacsys.com/library-loader/); [SnapEDA](https://www.snapeda.com/)
4. Calculate trace width, via diameter, etc: [SaturnPCB toolkit](https://saturnpcb.com/saturn-pcb-toolkit/)
5. To be updated…

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Now finally, we will dive into the project

## 1. **Design requirement**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; A brief requirement for this project can be summarized as follows:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ![image](https://github.com/ntdkhoa0410/garden_pcb_project/assets/63006481/785cd7a2-ae13-461f-8c13-ae102facb33d)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; As can be seen, there would be 3 boards for this project: **Power management board, relay control board,** these two boards would be controlled by **Main controller board.**

## **Let’s analyze the boards:**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Seeing the requirements, some questions arise:

1. How can an ESP32 communicate with the other 2 boards?
2. An ESP32 only contains 48 pins, but some are NC pins (unused pin), GND pins and power pins… the setting up for ESP32 (input power, code pouring, boot, reset buttons…) consume another number of pins. So how will we solve this limited pin problem?

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Let’s start with the first question, there are a variety of communication protocol an ESP32 provides: UART, SPI, I2C, Wifi…

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; We will consider UART, SPI, I2C only because wireless protocols require more modules and components on the board, hence making it bigger and space consuming. Another reason is that 3 boards will be placed near each other, hence wireless technology is unnecessary.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; The advantages and disadvantages of UART, SPI, I2C can be explained in depth on Google (example [here](https://www.seeedstudio.com/blog/2019/09/25/uart-vs-i2c-vs-spi-communication-protocols-and-uses/)). I will eliminate UART as it is only 1 master – 1 slave. And eliminate SPI as it require 4 wires while I2C requires only 2 wires, although the SPI speed is generally faster than I2C.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; The second question can be answered quickly, by using GPIO Extended Module [MCP23017](https://www.microchip.com/en-us/product/mcp23017) which also support I2C communication. 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; And as rule of thumb, always isolate signal module and dynamic module, hence for each relay, the control signal will be isolate by one [PC817X2NIPW](https://www.mouser.vn/ProductDetail/Sharp-Microelectronics/PC817X2NIPW?qs=t7xnP681wgWlfi7h7v0GxQ%3D%3D) optoisolator.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; As ESP32 will be using 3.3VDC power source, the optoisolator should also use a 3.3VDC power source. Therefore, an additional 3.3VDC power source is needed on-board for relay and power boards.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; In conclusion, the controller board will communicate with others using I2C protocol, on the Power and Relay boards, there would be a GPIO Extended Module to receive and send control signal for the relays, each signal is isolated by an optoisolator.

# Schematic Design

## 1. **Power Management Board**

   **Power Source**
   
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Take a safety factor of around 1.3; a 25A fuse is chosen to protect the board from overload input. For the same reason, a varistor is chosen with a protect voltage of 18V

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; As for how to convert 12.6VDC to lower voltage, there are two types of regulators for this job: linear and switching regulators. The working principles of [linear](https://en.wikipedia.org/wiki/Linear_regulator) and [switching](https://www.ablic.com/en/semicon/products/power-management-ic/switching-regulator/intro-2/) regulators can be found by clicking the links.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Simply put, the linear regulator has lower efficiency, higher heat dissipation, therefore choose the switching regulator to step down 12.6VDC to necessary voltage.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; A [TPS5430](https://www.digikey.com/en/products/detail/texas-instruments/TPS5430MDDAREP/2038543) is chosen for this purpose. One point I want to note, instead of designing a fixed voltage divider as shown in the datasheet, I will use potentiometer to manually adjust to have the wanted voltage, after achieving the desired voltage, I will connect the jumper 61300211121 to provide output power source for the board.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ![image](https://github.com/ntdkhoa0410/garden_pcb_project/assets/63006481/5f40c2a4-2ec9-41a3-b704-a15c4b46675c)

   **Relay control**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; The signal from the ESP32 will control the relay through the flow like this:

1. First it get to the GPIO Extended Module through the I2C connector:

![image](https://github.com/ntdkhoa0410/garden_pcb_project/assets/63006481/c19f0fa0-2f38-437c-9d31-41622da75176)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; A2 A1 A0 pin are used to define the address, instead of a fixed address, I use a DIP switch to flexibly change the address of the module. Add resistors to regulates the flow of electrical current.

2. Then the signal goes from the module to the optoisolator:

![image](https://github.com/ntdkhoa0410/garden_pcb_project/assets/63006481/41d04eef-5b06-4a6f-a590-a691e996c7a1)

3. Goes through the transistor, which acts as a switch to allow relay control:

![image](https://github.com/ntdkhoa0410/garden_pcb_project/assets/63006481/75219f4a-06e3-42df-a5ef-39761da22aff)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Add a LED to each relay for easier debugging. The base resistor is calculated from this [tutorial](https://youtu.be/Tv90-yhs0vI?si=ct7tP2PInm-D4Lox).

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  One side note when I search how to design a relay circuit, I came across a knowledge that every inductor should be parallel with a diode (the keyword for this is a flyback diode) to prevent voltage spike. Detail explanation can be found [here](https://spinningnumbers.org/a/inductor-kickback.html#:~:text=If%20we%20place%20a%20diode,biased%20and%20doesn't%20conduct.). So next time you try to control anything with an inductor (motors, relay…) remember this point.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; In conclusion, our schematic for the power management board will look like this:

![image](https://github.com/ntdkhoa0410/garden_pcb_project/assets/63006481/a23fc38d-10b1-4a75-b171-d1a5af87fd54)

## 2. **Relay control board**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; The relay control board is fairly the same as the power management board.

![image](https://github.com/ntdkhoa0410/garden_pcb_project/assets/63006481/b5f212cb-9303-4e5a-b147-2e7728c0b35e)

## 3. **Main controller board**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; The power source on this board is designed similarly to the other boards.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Since this board will mainly use modules, hence the design is fairly easy.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; The things I like to note are:

1. Remember the pins RXD0 and TXD0 is for pouring the code for the ESP32. Don’t use this for other purposes. I almost forgot this and use those two pins to connect with the RS485-UART converter module.
1. The MOSFET to control the fan needs to have two resistors: one pull-down resistor (R12) ensures the state of the MOSFET. One resistor (R11) to protect the optoisolator from the current surge from the MOSFET. Detail explanation [here](https://www.youtube.com/watch?v=Wd6NzCY3NgI&t=409s).

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Schematic of the board:
![image](https://github.com/user-attachments/assets/2065faee-ed14-45f9-b3c6-a729a530be7b)


# PCB Design

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; The components of the same module / purpose are placed closed together to reduce noise, emi... (for example bypass capacitor of an IC). When designing PCB, remember to use suitable trace width and via width for their current.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; The power management module:

![image](https://github.com/ntdkhoa0410/garden_pcb_project/assets/63006481/614da038-a0f6-4f3f-9867-d18b95f885c3)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; The relay control module:

![image](https://github.com/ntdkhoa0410/garden_pcb_project/assets/63006481/cde3183b-324d-4e64-b031-797abbf4ef5a)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; The main controller module:

![image](https://github.com/ntdkhoa0410/garden_pcb_project/assets/63006481/d139e983-d4e3-4e5a-9261-209c0d5d7557)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; The boards are currently being manufactured, I will update this document later! Hope it helps you guys somehow, and feedbacks are welcome
