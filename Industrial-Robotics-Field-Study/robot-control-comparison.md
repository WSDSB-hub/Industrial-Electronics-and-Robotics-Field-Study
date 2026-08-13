# From Two-Wheel PID to Six-Axis Trajectory Control: A Comparative Analysis of My Desktop Robot and the FANUC M-20iD/25

## Background

In Project A and Project B, I implemented closed-loop speed control on a differential-drive robot. The system used two DC geared motors with Hall-effect encoders, driven by a TB6612 H-bridge module and controlled by an STM32F103C8T6 running a hand-written positional PID algorithm. The control loop ran at 100 Hz (10 ms period), maintaining wheel speed at 90–98 RPM against a target of 100 RPM, with a steady-state error of less than 10% and disturbance recovery within one second. To me, at the time, this represented a complete and successful motor control implementation.

During my internship at Tianyi Welding, I had the opportunity to study the motion control architecture of a FANUC M-20iD/25 six-axis industrial robot arm. By systematically comparing its control system with the one I had built myself, I came to understand that the gap between the two is not merely one of scale or complexity — it is a difference in the fundamental question being asked. My controller asked: "How fast should the wheels turn?" The FANUC controller asks: "Where should the tool tip be in three-dimensional space, what orientation should it hold, what path should it follow, and how fast should it move along that path?"

This report documents that comparison across eight technical dimensions: degrees of freedom, control objective, feedback resolution, control period, trajectory generation, coordinate systems, error correction, and load constraints. Each dimension reveals a specific lesson about what it takes to move from joint-level speed control to task-level trajectory control.

## 1. Degrees of Freedom: Two Wheels vs. Six Axes

My differential-drive robot had exactly two controllable degrees of freedom: the rotational speed of the left wheel and the rotational speed of the right wheel. Every possible motion — forward, backward, turning, arc trajectories — was a combination of these two speeds. The control problem was therefore inherently two-dimensional.

The FANUC M-20iD/25 has six independently controlled rotary joints. The tool tip — where the welding torch is mounted — moves through three-dimensional space with three degrees of positional freedom and three degrees of rotational freedom. A welding seam that curves through space in three dimensions, while requiring the torch to maintain a specific angle relative to the workpiece, demands simultaneous control of all six joints.

The difference between two degrees of freedom and six is not arithmetic; it is qualitative. With two wheels, the mapping between control inputs and robot motion is direct and intuitive: to go forward, spin both wheels forward; to turn left, slow the left wheel and speed up the right. With six joints, there is no intuitive mapping between individual joint rotations and the desired tool motion. Moving the tool tip in a straight line through space requires all six joints to rotate in a precisely coordinated manner, with each joint following its own time-varying trajectory that is not at all obvious from the desired tool path.

This is the first and most fundamental lesson: multi-axis coordination is not a generalization of single-axis control. It is a fundamentally different problem that requires a fundamentally different architecture — one where the control objective is expressed in Cartesian task space, and the controller is responsible for translating that objective into the coordinated motion of six joints.

## 2. Control Objective: Speed Loop vs. Position-and-Trajectory Loop

My PID controller was a speed loop. The control objective was a scalar value: the wheel speed in RPM. The feedback signal was also scalar: the encoder-derived speed measurement. The error — the difference between target speed and actual speed — was used to compute a PWM duty cycle. The loop was simple, effective, and entirely adequate for a robot whose task was "move forward and avoid obstacles."

The FANUC controller's objective is fundamentally different. It is not controlling joint speeds; it is controlling the pose of the tool tip in three-dimensional space. The objective includes not only where the tool tip should be at any given moment, but also what orientation it should hold, and how it should transition between successive points along a trajectory.

The control loop in the FANUC system is therefore cascaded across multiple levels. The outermost level plans a Cartesian trajectory — a sequence of positions, orientations, and velocities that describe the desired tool motion. The next level performs inverse kinematics, translating the Cartesian trajectory into six joint-space trajectories. The next level performs interpolation, generating intermediate points at the controller's cycle rate. The innermost level — the servo loop — drives each joint to follow its trajectory, using a three-loop cascade of position control, velocity control, and current control.

This cascade architecture is not something I had implemented or even fully appreciated in my own project. My PID speed loop was a single layer. The FANUC system has at least four layers of control, each operating at a different rate, each closing a different feedback loop, each abstracting the complexity below it.

## 3. Feedback Resolution: 1,456 PPR vs. 1,048,576 PPR

The feedback resolution of the two systems differs by three orders of magnitude.

In my project, each motor was equipped with a Hall-effect encoder producing 13 pulses per revolution. With 4× quadrature decoding and a 1:28 gearbox, the effective resolution at the wheel shaft was 1,456 pulses per revolution. This translated to roughly 0.25° of angular resolution at the wheel — sufficient for speed control of a small differential-drive robot, but entirely inadequate for the positional precision required in industrial welding.

The FANUC M-20iD/25, by contrast, is equipped with 20-bit absolute pulse encoders on all six axes. A 20-bit encoder provides 2^20 = 1,048,576 counts per revolution — a resolution of approximately 0.00034°. Combined with the RV reducers on J1-J3 and harmonic reducers on J4-J6, this encoder resolution provides the hardware foundation for the robot's ±0.02 mm repeatability specification.

The difference in resolution reflects a fundamental difference in measurement philosophy. My incremental encoder counted relative motion: the controller knew how many counts had elapsed since the last reading, but had no way of knowing the absolute position unless it was driven to a known reference point at startup. The FANUC absolute encoder, by contrast, always knows exactly where it is — even after power has been off, even after the arm has been manually moved. The battery-backed position memory eliminates an entire class of startup errors and enables the robot to resume operation immediately after power-on.

## 4. Control Period: 10 ms vs. 4-8 ms Interpolation and 250 μs Servo Sampling

My control loop ran at 100 Hz — a 10 ms period. This was adequate for wheel speed control because the mechanical dynamics of a small differential-drive robot are relatively slow. The wheels' rotational inertia is tiny, the load is light, and the consequence of a slightly late speed correction is negligible.

The FANUC R-30iB Plus Mate controller operates on a much faster timescale, with a layered timing architecture. The standard interpolation period is 8 ms; in optimized configurations, this can be reduced to 4 ms, corresponding to a path update frequency of 125–250 Hz. Below this, the servo drive's innermost control loops — current, velocity, and position — operate with a closed-loop sampling period as short as 250 μs. The FSSB (FANUC Serial Servo Bus) provides high-speed optical communication between the controller and the servo amplifiers, ensuring millisecond-level data exchange across all six axes.

The significance of this timing hierarchy goes beyond raw speed. The 4–8 ms interpolation period determines how frequently the Cartesian path is updated — how often a new target point is computed and sent to the servo layer. The 250 μs servo period determines how tightly each joint follows the trajectory between interpolation points. The combination of a fast interpolation rate and an even faster servo rate is what enables the robot to maintain precise path tracking even at high tool speeds and under varying dynamic loads.

In my project, the control period was a single number: 10 ms. In the FANUC system, the control period is a hierarchy of rates, each matched to the dynamics of the layer it controls. This is a design pattern I had not previously recognized: control loops must be layered, and each layer must operate at a rate appropriate to its dynamics, not at a uniform rate imposed by a single timer interrupt.

## 5. Trajectory Generation: PID Output vs. Interpolation and Kinematic Transformation

My PID controller generated a scalar output: a PWM duty cycle. The conversion from "desired wheel speed" to "PWM duty cycle" was direct and could be computed in a few lines of C code. There was no path planning, no trajectory interpolation, no kinematic transformation — because the robot's motion space was so simple that none of those things were needed.

The FANUC controller's motion generation pipeline is structured very differently. It supports three fundamental interpolation types:

- **Joint interpolation** moves each axis from its current position to the target position with synchronized start and stop, but makes no attempt to constrain the tool's path in Cartesian space. It is used for rapid positioning moves where the path is irrelevant.

- **Linear interpolation** forces the tool center point (TCP) to follow a straight line in Cartesian space, at a constant speed. This is the primary interpolation method for straight weld seams, because welding requires a constant, predictable tool velocity.

- **Circular interpolation** defines a circular arc through three points — a start point, a transition point, and an end point. It is used for ring-shaped and rounded corner welds. For complex three-dimensional curves, spline interpolation can be extended.

Each of these interpolation types requires the controller to continuously solve the inverse kinematics problem: given the desired TCP trajectory, compute the joint angles required at each interpolation point. This computation must complete within the interpolation period — every 4–8 ms — while simultaneously managing acceleration profiles, velocity limits, and the synchronization of all six joints.

This is the second fundamental lesson: the distance between my PID speed loop and the FANUC trajectory controller is not primarily a matter of algorithm complexity. It is a matter of layering and abstraction. My controller worked directly in the space of the actuators. The FANUC controller works in a hierarchy of spaces — Cartesian task space at the top, joint configuration space in the middle, actuator space at the bottom — with systematic transformations between each layer.

## 6. Coordinate Systems: Two Dimensional vs. Multi-Frame

My robot moved in a single, self-evident coordinate frame. Forward and backward were defined by the wheels' orientation; left and right were defined by the robot's facing. There was no possibility of confusion about what "move forward" meant, because the robot existed in a two-dimensional plane with a fixed forward direction.

The FANUC robot operates with multiple coordinate frames, each serving a specific purpose. The world coordinate system provides a global reference anchored to the factory floor. The tool coordinate system defines the position and orientation of the TCP relative to the tool flange — the actual point that performs the welding. The user coordinate system is anchored to the workpiece itself, allowing weld trajectories to be programmed relative to the part rather than to the floor, which means that if the workpiece is repositioned, the robot can follow the same trajectory simply by updating the user frame.

In actual welding operations, the robot primarily uses the user coordinate system and the tool coordinate system together. The welding trajectory is programmed in the user frame — anchored to the workpiece — and executed with the TCP as the controlled point. The world frame serves only as the global reference for system-level positioning and is rarely used directly for welding path programming.

This multi-frame architecture is the answer to a practical problem I had never faced: how do you program a robot to weld a seam on a workpiece that is not at a fixed, known position relative to the robot's base? The answer is to anchor the trajectory to the workpiece itself via the user frame, so that small variations in the workpiece's placement do not corrupt the entire program. This is the spatial analog of closed-loop feedback control — instead of correcting errors after they occur, you define the control problem in a reference frame where the errors do not exist.

## 7. Error Correction: Blind PID vs. Visual Servo

In my project, the only feedback came from the encoders, and the only error being corrected was wheel speed. The robot had no way of knowing whether it was actually moving in the direction it intended — no sense of its own position in the world, no awareness of obstacles beyond the camera's forward view, and no ability to correct deviations from an intended path.

The FANUC M-20iD/25, when equipped with a visual seam tracking system, implements true visual servoing. The laser vision sensor mounted ahead of the torch detects the actual seam position — including lateral deviation, groove geometry, and gap width — and transmits this information to the robot controller via a fieldbus. The controller then applies corrections to the TCP path in real time, closing the loop between perception and motion at the interpolation rate of 4–8 ms.

Two integration methods are commonly used. With FANUC's native iRVision system, the VOFFSET instruction adds the detected X/Y/Z deviation directly to the motion target point, correcting the weld trajectory segment by segment. With a third-party laser vision system, the real-time deviation values are written into the robot's position registers via Ethernet/IP or DeviceNet, and the OFFSET instruction or dynamic path modification function applies the offset to the current TCP path at each interpolation cycle.

The lead distance between the sensor and the torch — typically 10 to 40 mm — provides the control response time window, ensuring that the correction process is smooth and free of mechanical shock.

The contrast with my own system is stark. My robot's "perception" was a deep learning model that operated at 20–30 fps and whose output was a coarse classification of obstacle positions. The FANUC system's perception is a dedicated laser triangulation sensor that reports sub-millimeter deviations at rates synchronized with the robot's control loop. My system and the FANUC system both "see and act," but they operate at completely different levels of precision, speed, and integration.

## 8. Load Constraints: Ignored vs. Explicitly Managed

In my project, I never gave any thought to the load on the motors. The robot carried a USB camera, a battery, and some wires. The motors were strong enough to move the robot, and that was that.

The FANUC M-20iD/25, by contrast, has explicitly specified load constraints for every axis. The wrist axes J4 and J5 have an allowable inertia of 2.4 kg·m² each, while J6 has an allowable inertia of 1.2 kg·m². These limits exist because the servo motors, reducers, and structural components are designed for specific dynamic loads. If the actual inertia of the end-effector combination — the welding torch, the vision sensor, the cable assembly — exceeds these rated values, the consequences are predictable and well-documented: reduced acceleration and deceleration capability, slower cycle times, joint vibration, trajectory deviation, and degraded path accuracy. In severe cases, the robot will trigger SRVO-series servo overload alarms and shut down for protection. Long-term over-inertia operation accelerates reducer wear, leading to abnormal noise, oil leakage, and progressive precision degradation.

This is a level of mechanical awareness I had never needed. But it is precisely this awareness that separates a system designed for a specific, well-characterized task from a system that simply "works" under the conditions it happens to encounter.

## 9. What This Comparison Taught Me

Building a two-wheel PID robot taught me how to control speed. Studying the FANUC M-20iD/25 taught me what speed control is for.

The most important lesson is not any single technical detail — not the encoder resolution, not the control period, not the interpolation algorithm. It is the recognition that **control is always in service of a task, and the task defines the architecture of the control system.** My robot's task — avoid obstacles and move forward — required nothing more than a speed loop with moderate precision. The FANUC robot's task — weld a seam along a specified trajectory with sub-millimeter precision, in a harsh environment, for thousands of cycles — demands a layered control architecture that spans from Cartesian trajectory planning down to microsecond-level servo actuation.

I now understand that the mark of a good control engineer is not the ability to implement complex algorithms. It is the ability to recognize what level of control precision and architectural sophistication a task actually requires — and to design accordingly. I built my robot at one point on that spectrum. The FANUC system occupies another point. In my future work — including the graduate research I hope to pursue — I want to learn how to navigate that spectrum deliberately.
