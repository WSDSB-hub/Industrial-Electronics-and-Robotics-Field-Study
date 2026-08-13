# Personal Technical Reflection: What I Learned About Robotics, Perception, and Myself at Tianyi Welding

## 1. Before This Internship: What I Thought a Robot Was

Before I walked into the Tianyi Welding workshop, my understanding of what a robot is came entirely from my own hands. I had built a two-wheel differential-drive robot. I had written a PID controller that kept the wheels spinning at a target speed. I had mounted a USB camera on it, loaded YOLO and MiDaS, and watched it steer around chairs and backpacks in my room. I had even run ROS2 nodes and trained a reinforcement learning agent in a physics simulator. I thought I understood, at least in outline, what a robot was and how it worked.

I was wrong. Not entirely — the principles I had learned were real, and they did transfer. But the scale of my misunderstanding was not in the details of control loops or neural networks. It was in the very concept of what it means for a robot to work.

A desktop robot works when it completes a task once. An industrial robot works when it completes the same task ten thousand times without failure. The gap between those two definitions of "working" is where most of real robotics engineering lives. I had never seen that gap before. Now I have.

## 2. The First Thing I Noticed: The Robot Was Never Alone

The first time I stood in front of the FANUC M-20iD/25, what struck me most was not the robot itself. It was everything around it. The control cabinet humming quietly beside it. The Panasonic welding power supply with its digital display. The wire feeder spooling steadily. The shielding gas cylinder with its pressure gauge. The Siemens PLC coordinating everything. The safety fences. The emergency stop buttons. The fume extraction hood. The laser seam tracking sensor mounted just ahead of the torch.

In my room, my robot was a single machine. I turned it on, ran the code, and it moved. When something went wrong — a dropped Bluetooth connection, an erratic encoder — the failure was isolated. The robot stopped moving, and I fixed it.

In the welding workshop, the robot is not a machine. It is a node in a network. Every movement it makes is coordinated with the welding power supply, the wire feeder, the gas solenoid, the vision system, and the PLC. Every failure it experiences propagates to every other node. A single loose communication cable does not just stop the robot — it freezes the entire workstation, and the diagnostic question is never "what's wrong with the robot?" but "where in the network did the failure originate?"

This was the first and most important lesson of my internship: **robots are systems, not machines.** The boundary of a robot does not end at its mechanical structure. It extends through every cable, every connector, every communication protocol, every safety interlock, every human operator. I had designed my desktop robot as a machine. I want to learn how to design robots as systems.

## 3. The Perception Problem I Had Never Considered

My desktop robot's perception task was trivial. The camera looked ahead. The room was clean and well-lit. The obstacles were large and visually distinct. YOLO drew boxes around them, MiDaS estimated their depth, and my avoidance logic made a left-or-right decision. The whole pipeline worked at 20 frames per second, and the worst thing that could happen if it failed was that the robot bumped into a chair.

The welding robot's perception task is of a completely different nature. The seam it must track may be only a few millimeters wide. The gap between workpieces may vary from one part to the next. The arc light is bright enough to blind an ordinary camera. The spatter, fume, and slag obscure everything. And the sensor must maintain sub-millimeter accuracy not once, but continuously, for every weld cycle, on every workpiece.

And then there is the physical problem I had never imagined: what if the sensor physically cannot reach the seam?

This was the most instructive observation I made during my entire internship. The laser seam tracking sensor is mounted adjacent to the welding torch. In confined spaces — narrow channels, deep grooves, tight corners — the combined envelope of the torch and the sensor is simply too bulky to fit. The workaround is to scan from a greater standoff distance. But this degrades accuracy, because the laser triangulation principle depends on a specific geometric relationship between the emitter, the camera, and the target surface. From further away, the measurement precision degrades, and the vision software may fail to reconstruct the weld model entirely.

This problem crystallized something for me. **Perception in the real world is not just an algorithm problem — it is a physical problem.** The sensor has a size, a weight, a mounting bracket, a field of view, a minimum standoff distance, and a susceptibility to environmental interference. All of these physical constraints determine what the perception system can and cannot do, before any code is ever written. In my desktop project, none of these constraints existed. In the industrial world, they dominate the engineering decisions.

## 4. The Communication Problem I Had Already Met — In a Different Form

When my desktop robot's Bluetooth connection dropped intermittently, I blamed the software. I assumed the problem was in my code — a bug in the serial protocol, a race condition in the threading logic, some subtle issue that I could debug by staring at the terminal output.

The field engineers at Tianyi Welding taught me something different. The most common cause of intermittent communication failures on the production floor is not software. It is the physical layer. A loose connector. A cable that has been flexed one too many times. A shield that has been abraded by a drag chain. A connector that has accumulated dust and oil. These are unglamorous, invisible, and often intermittent — and they are the leading cause of production downtime.

The solutions are equally unglamorous. High-flex industrial cable. Protective drag chains. Strain relief on every connector. Periodic inspection and replacement of cables that show wear. Status LEDs on communication modules for quick link-loss diagnosis. Timestamped error logs that allow maintenance personnel to correlate failures with specific motion sequences.

This was humbling. I had been thinking about communication failures as a software engineer thinks about them — as bugs to be found and fixed in code. The industrial perspective is different: communication failures are physical events, and the engineering response is physical robustness, redundancy, and preventive maintenance. The best communication protocol in the world is useless if the cable carrying it is loose.

## 5. The Precision Gap: What ±0.02 mm Really Means

My PID controller maintained wheel speed within ±5 RPM of the target. The wheels were small DC motors with Hall encoders and a 1:28 gearbox. The encoder resolution was 1,456 pulses per revolution at the output shaft. The control loop ran every 10 milliseconds. The steady-state error was under 10 percent, and I was proud of it.

The FANUC M-20iD/25 maintains a repeatability of ±0.02 millimeters. Its servo motors carry 20-bit absolute encoders — over a million counts per revolution. Its control architecture includes 4 to 8 millisecond interpolation and a 250-microsecond servo sampling period. Its three-stage reducers provide the mechanical rigidity to hold position against varying loads and dynamic forces.

The numbers themselves are less important than what they represent. Every one of those specifications exists because someone, at some point, asked a very specific question: "What precision does this task actually require?" The weld must be within a fraction of a millimeter of the intended path, or the joint will fail. The robot must repeat that precision across ten thousand cycles, with thermal expansion, tool wear, workpiece variation, and environmental interference all trying to push it off course. The precision is not a feature — it is the entire point of the machine.

My robot did not need sub-millimeter precision, because its task did not require it. The lesson is not that my robot was inadequate, but that **precision requirements are task-defined, and the entire architecture of a control system should be derived from the precision its task demands.** For a desktop obstacle-avoidance robot, a 10-millisecond PID loop was the right level of precision. For a welding robot, it would be catastrophically insufficient. The skill I want to develop is the ability to determine, for any given task, exactly how much precision is enough — and then design the simplest system that reliably achieves it.

## 6. What the Engineers Actually Told Me

I asked the field engineers a simple question: what do you worry about most?

Their answers surprised me. They did not mention the robot's inverse kinematics. They did not mention the welding power supply's IGBT topology. They did not mention the neural networks or the control algorithms. They mentioned **seam tracking failure leading to torch collision**, and **contact tip and wire liner wear producing batch defects**.

The seam tracking failure scenario is this: the workpiece is clamped with an offset that exceeds the robot's compensation range. Or the laser sensor is obscured by slag and fume, and the tracking drifts. The result is not a small weld defect — it is a collision between the torch and the fixture, destroying expensive hardware and stopping the production line.

The contact tip wear scenario is subtler. The contact tip is a consumable. It wears gradually over thousands of weld cycles. When it wears too much, the weld quality degrades — porosity, misalignment, uneven bead appearance — but not abruptly. The defects appear gradually, in batches, and by the time anyone notices, dozens of workpieces may have been affected.

This is what real industrial robotics is about. Not elegant algorithms. Not state-of-the-art neural architectures. But the unglamorous, relentless, physical reality of wear, drift, interference, and failure. The engineers who design these systems spend their time thinking not about how to make the robot work once, but about how to ensure that every possible failure mode — from a worn contact tip to a fogged laser lens to a loose cable — is anticipated, monitored, and mitigated.

## 7. What I Want to Learn Next

This internship has changed the direction of my graduate studies. Before Tianyi Welding, I was interested in robotics because I enjoyed building systems and watching them work. That interest is still there, but it has been refined. Now I know what specific problem I want to work on.

I want to work on **perception robustness for industrial robots in unstructured environments.** The seam tracking sensors I observed are capable of sub-millimeter precision under ideal conditions, but they fail in confined spaces, under heavy spatter, and against complex workpiece geometries. This is not a software problem alone, and it is not a hardware problem alone. It is a systems problem — how do you design a perception pipeline that remains accurate and reliable across the full range of real-world conditions it will encounter, and how do you gracefully degrade performance when conditions exceed the sensor's physical limits?

I also want to work on **multi-device system integration.** The coordination between the robot, the welding power supply, the wire feeder, and the gas system is an engineering achievement in its own right — one that requires careful attention to timing, failure modes, and safety interlocking. This is the kind of systems thinking that cannot be learned from a textbook. It can only be learned by studying real systems, in real environments, with real failure modes.

And finally, I want to work on **closing the gap between AI and industrial practice.** I came to this internship with experience in deep learning — YOLO, MiDaS, reinforcement learning. I leave with the recognition that these tools, as powerful as they are, have not yet been fully integrated into industrial robotic workflows. The engineers I spoke to want better seam tracking, better fault diagnosis, better process optimization — and they know that AI could potentially deliver these improvements. But the gap between a deep learning model that works in a lab and a perception system that works on a welding production line is enormous. Bridging that gap is, I believe, where the future of industrial robotics lies.

## 8. A Note of Gratitude

I want to thank the engineers and workers at Tianyi Welding who took the time to explain their systems to me. Every question I asked — and I asked many — was answered with patience and detail. They showed me their control cabinets, walked me through their fault diagnostics, let me watch their welding cycles, and shared their hard-won field experience with a generosity I did not expect.

I came to this internship hoping to see what industrial robotics looks like. I am leaving with a much clearer picture of what I want to contribute to it. For that, I am grateful beyond what this reflection can express.

This is not the end of my learning. It is the beginning of a more focused one. And I am ready for it.
