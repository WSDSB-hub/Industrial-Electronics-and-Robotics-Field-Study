# Industrial Communication Protocols and PLC System Integration: From UART/Bluetooth to Industrial Fieldbus

## Background

In Project B (Vision-Based Autonomous Obstacle Avoidance Robot), I designed a custom 7-byte serial communication protocol for transmitting motion commands between the laptop and the STM32. The frame format included a header byte (0xAA), a command byte, two-byte left and right wheel speed values (int16, big-endian), and an XOR checksum. However, during actual debugging, the wired serial link (CH340 USB-TTL) failed in one direction: the STM32 could transmit data to the PC, but could never receive commands from the PC. After extensive debugging — swapping TX/RX lines, replacing CH340 modules, testing on different computers — the root cause could not be identified. I ultimately switched to Bluetooth (HC-05 master-slave paired configuration), which resolved the communication problem and made the robot fully wireless.

This experience left me with a question: how do real industrial production lines handle communication between multiple devices when reliability is non-negotiable? During my internship at Tianyi Welding, I found the answer — and the gap between my desktop approach and industrial practice turned out to be far larger than I had imagined.

## 1. Core Communication Equipment on the Production Line

### 1.1 Main Controller PLC

The brain of the entire welding automation workstation is a Siemens SIMATIC S7-1215C DC/DC/DC PLC — a mainstream industrial model widely adopted for small and medium-sized welding special-purpose machines, and the standard configuration for Tianyi Welding's automated workstations.

Its communication interface configuration is as follows. It natively provides two RJ45 Profinet industrial Ethernet ports with built-in switch functionality, allowing simultaneous connection to the host computer, the robot control cabinet, and the welding power supply. Both Profinet IO controller and intelligent device operating modes are supported. An optional CB 1241 communication board can be added to provide RS485/RS232 serial ports, supporting Modbus RTU and free-port protocols for connecting peripheral auxiliary sensing devices. An optional CM 1243-5 module can be added to enable Profibus DP bus communication for compatibility with legacy industrial peripherals. It also integrates multiple digital I/O and analog I/O channels for basic switching control and analog signal acquisition.

The core role of the PLC is to serve as the control brain of the entire welding workstation, coordinating the robot arm's motion trajectory, welding power supply parameter calls, wire feeder control, shielding gas on/off control, sensor signal acquisition, and safety interlock logic.

### 1.2 Welding Power Supply Communication Interface

The Panasonic YD-500GP5HGK robot-specific welding machine is the model deployed at the site. Its standard external terminal block supports digital I/O signals (arc start, arc stop, fault output, etc.) as well as 0–15V / 0–12V / 0–10V analog inputs for current and voltage setpoints, with the range switchable via the machine's P23 menu. A robot-specific communication port supports Profinet, DeviceNet, and other industrial bus adapters via an optional dedicated communication board, enabling fully digital process parameter interaction. A D-sub communication connector is used for dedicated control signal interaction with the wire feeder and the robot control cabinet.

### 1.3 Robot Arm Control Cabinet Communication Interface

It comes standard with a Profinet Ethernet port, connecting as a Profinet slave to the PLC bus system to receive motion commands and report real-time position and equipment status. A dedicated robot welding interface allows direct linked communication with the Panasonic welding machine, enabling arc voltage feedback and real-time seam tracking adjustment. Reserved RS485, Ethernet, and other expansion interfaces are available for integration with vision positioning and laser seam tracking sensors.

## 2. Communication Between the Robot Arm and the Welding Power Supply

In a welding workstation, the interaction between the robot arm and the welding power supply follows one of two mainstream approaches. The Tianyi Welding rock drilling rig automation project adopts the fully digital bus approach.

### 2.1 Traditional Analog + Digital I/O Approach (Entry-Level Fixed-Process Workstations)

The control logic is as follows. The robot control cabinet controls the welding machine's arc start, arc stop, gas flow, and wire feed start/stop via digital output signals. Welding current and voltage setpoints are sent to the welding machine via 0–10V or 4–20mA analog signals. The welding machine reports its ready status, in-welding status, and fault alarms back to the robot via digital signals. This approach features simple wiring and low hardware cost, but parameter accuracy is significantly affected by line interference and cable resistance. Process adjustments are inflexible, real-time parameter switching for complex welding tasks is impossible, and it is only suitable for simple welding stations with fixed processes.

### 2.2 Profinet Fully Digital Bus Approach (Mainstream Automation Configuration at Tianyi Welding)

The control logic is as follows. The Siemens PLC acts as the Profinet master, while the robot arm control cabinet, the welding power supply, and the laser tracking sensor are all connected as Profinet slaves on the same bus network. All process parameters — welding current, voltage, pulse parameters, and crater-fill specifications — are transmitted via digital telegrams. The welding machine's real-time output current and voltage, internal temperature, fault codes, and wire feed speed are reported back as digital telegrams. The robot arm's motion position and welding parameters can be linked in real time, enabling automatic process parameter switching across different welding segments. This approach offers strong anti-interference capability, zero-attenuation and zero-drift parameter transmission, support for remote invocation of hundreds of welding procedure specifications, precise fault localization and tracing, and adaptability to complex automated welding processes. It is the standard configuration for current industrial welding robots.

## 3. Core Differences Between Industrial Communication Protocols and UART/Bluetooth

This represents the key cognitive gap between embedded board-level development and industrial field applications, reflected in four fundamental aspects.

### 3.1 Real-Time Performance and Determinism

Industrial fieldbuses such as EtherCAT and Profinet IRT can achieve microsecond-level deterministic latency, with synchronization accuracy across all station devices reaching the nanosecond level. The transmission time of each telegram is fixed and controllable, ensuring precise synchronization between mechanical motion and welding output. In contrast, UART and Bluetooth exhibit millisecond-level latency with significant jitter and no temporal determinism, making it impossible to guarantee synchronized multi-device actions. They are only suitable for non-real-time, simple data transmission.

### 3.2 Reliability and Error Handling

In industrial protocols, every data frame carries a CRC checksum, a frame sequence number, and timeout detection mechanisms. Some protocols support automatic retransmission. Transmission errors can be immediately identified and safety shutdown mechanisms triggered to prevent equipment malfunction from causing production accidents. In contrast, UART and Bluetooth provide only basic checksums, have no standard error handling or frame sequence mechanisms, and cannot automatically recover after packet loss or errors. They are unsuitable for industrial scenarios with high safety integrity requirements.

### 3.3 Topology and Node Count

Industrial fieldbuses support multiple topologies — bus, star, and others — with a single backbone bus capable of connecting dozens or even hundreds of nodes. Wiring is clean and expansion is convenient. UART only supports point-to-point communication, requiring a dedicated interface and cable for each one-to-one connection. Interconnecting multiple devices requires a large number of interfaces and cables, and scalability is extremely poor.

### 3.4 Anti-Interference Capability

Industrial Ethernet and fieldbus use differential transmission, shielded cables, and standardized grounding practices, enabling stable transmission in the harsh environment of welding with its strong electromagnetic radiation and high-current switching. Bluetooth is a wireless transmission method, susceptible to metal obstruction and electromagnetic radiation interference. The arc light and high-frequency inversion on a welding site can cause Bluetooth disconnection and packet loss, making it entirely unsuitable for industrial welding scenarios.

## 4. Comparison Between the Modbus Protocol and My Custom 7-Byte Protocol

Comparing my short-frame custom protocol with the industrial standard Modbus protocol reveals fundamental differences in engineering completeness.

### 4.1 Standardized Frame Format

Modbus RTU/TCP has a unified international standard frame structure: address code or transaction identifier, function code, data field, and CRC or checksum. All manufacturers' equipment follows the same specification, allowing direct interoperability between PLCs, sensors, and instruments from different brands. My custom protocol only defined a frame header and a checksum, had no unified specification, could only work with a single self-developed device, and was incompatible with any third-party industrial equipment.

### 4.2 Function Code Definition

Modbus has a standard function code system: 03 Read Holding Registers, 06 Write Single Register, 16 Write Multiple Registers, and so on. Read and write operations and parameter access follow unified conventions, and a large number of general-purpose tools are available for development and debugging. My custom protocol had no standard functional partitioning; all logic had to be defined ad hoc, and both generality and debugging convenience were poor.

### 4.3 Exception Response Mechanism

Modbus has standard exception response rules: when a slave device receives an erroneous command or cannot execute an operation, it returns a fixed-format exception code (such as Illegal Function or Illegal Data Address), enabling the master to quickly identify the fault cause. Simple custom protocols typically only have normal data frames with no exception response logic. When an error occurs, the fault type cannot be determined, making field troubleshooting extremely difficult.

### 4.4 Retransmission and Fault Tolerance

Industrial field applications of Modbus are typically accompanied by timeout retransmission and frame sequence verification logic to ensure reliable data delivery. Simple custom protocols generally have no retransmission mechanism; data is directly lost after packet loss. They are unsuitable for long-distance, high-interference industrial welding environments.

## 5. Cognitive Upgrade: From Point-to-Point to Industrial-Grade Multi-Device Networks

In my desktop project, the communication requirement was simple: one laptop sending motion commands to one STM32. A single ASCII character over Bluetooth was sufficient — the consequences of a lost packet were negligible (the robot would simply continue its previous action for another 100 ms). The debugging experience, however, was already painful: the CH340 serial RX direction inexplicably failed, and the root cause was never identified. This was a point-to-point link between only two devices.

On the Tianyi Welding production line, the communication system connects a PLC, a robot control cabinet, a welding power supply, a laser tracking sensor, a vision positioning camera, a wire feeder, and a host computer — all coordinated in real time. Every arc ignition must be precisely synchronized with the robot's motion trajectory. Every fault code must be reported and acted upon within milliseconds. A single communication failure does not merely degrade performance — it can cause a production line shutdown, a batch of scrapped workpieces, or a safety incident.

This comparison made me realize that the communication protocols I used in my projects — UART and Bluetooth — addressed only the most basic question of "can data be transmitted?" In an industrial setting, the questions are fundamentally different: "Can data be transmitted deterministically? Can errors be detected and recovered? Can dozens of devices from different vendors interoperate on the same network? Can the system fail safely when communication is lost?"

The answers to these questions are embedded in the design of Profinet, Modbus, and other industrial protocols — in their CRC checksums, their sequence numbers, their standard function codes, their exception response frames, their timeout handling, their bus topologies, and their differential signaling over shielded twisted-pair cables. These are not features that can be bolted on after the fact; they must be designed in from the start. This was the most important lesson I learned about communication system design during this internship.

## 6. A Concrete Improvement I Can Bring Back to My Own Project

If I were to redesign the communication architecture for Project B, I would replace the custom 7-byte protocol with a standard Modbus RTU frame structure over UART. This would allow any off-the-shelf Modbus debugging tool to monitor, log, and simulate commands — a capability that would have saved me hours of frustration when the CH340 RX direction failed and I had no way to determine whether the problem was on the PC side or the STM32 side. Furthermore, I would add a timeout-based watchdog on the STM32: if no valid command frame is received within a defined interval, the motors would automatically stop. This is the industrial "fail-safe" principle applied to a desktop robot — and it is a direct lesson learned from observing how the Siemens PLC handles communication loss on the Profinet bus at Tianyi Welding.
