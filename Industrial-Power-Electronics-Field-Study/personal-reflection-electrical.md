# Personal Technical Reflection: What I Learned About Electronics, Engineering, and Myself at Tianyi Welding

## 1. Before This Internship: What I Thought Engineering Was

Before I walked into the Tianyi Welding workshop, I thought I understood electronics. I had built a PID motor controller from scratch on an STM32. I had wired a TB6612 driver, configured timer peripherals, tuned a control loop, and solved a signal integrity problem with pull-up resistors and software filtering. I had designed a custom serial protocol, validated it with a self-test script, and pivoted to Bluetooth when the wired link failed. I had deployed YOLO and MiDaS, fused their outputs, and closed the perception-decision-action loop on a real robot.

I was proud of these accomplishments, and I still am. But if you had asked me, at that point, what the difference was between my desktop project and an industrial electronic system, I would have given you a superficial answer. Higher power. Bigger components. More redundancy. I did not yet understand that the difference was not one of degree, but one of kind.

## 2. What I Saw at Tianyi Welding

The first thing I saw on the production floor was a welding robot. A six-axis mechanical arm moving a welding torch along a seam with precise, fluid motion. The second thing I saw was the control cabinet beside it. I asked if I could look inside.

What I found inside that cabinet was not fundamentally different from what I had built on my desk. There were power supplies, circuit breakers, contactors, a PLC, terminal blocks, and miles of neatly routed wire. The components were larger and more expensive, but the functional blocks — power conversion, signal processing, control logic, communication — were the same functional blocks I had been working with.

But as I studied the schematics more carefully, and as I talked to the engineers who designed and maintained these systems, I began to understand that the similarity was only superficial. The way these functional blocks were implemented — the design philosophy behind every component selection, every routing decision, every protection mechanism — was fundamentally different from how I had approached my own projects.

## 3. The Four Gaps

Over the course of this internship, I identified four critical gaps between how I designed electronics on my desk and how electronics are designed for industrial deployment.

### 3.1 Protection: From Reactive to Proactive

In my desktop projects, protection was always reactive. I added a capacitor across the motor power rail after the TB6612 burned. I added current-limiting resistors on the control signal lines after the STM32 was fried by backfeed. Each protection measure was a scar — a response to a failure that had already occurred.

In the industrial cabinet, protection was proactive. Every circuit breaker was sized before the system was built. Every contactor was part of a safety loop that was designed into the schematic from the very first draft. Every I/O port was isolated with optocouplers or isolation amplifiers — not because a failure had occurred, but because the engineers knew that one day, a failure would occur, and when it did, the isolation barrier would be there to stop it.

This was the most important engineering lesson of my internship. It is not enough to fix problems after they happen. You must design as though problems will happen — because in any complex system operating continuously in a harsh environment, problems are not anomalies. They are certainties. The only question is whether you have anticipated them.

### 3.2 Signal Integrity: From Workarounds to Standards

When my encoder readings spiked to 14,000 RPM, I debugged the problem empirically. I swapped signal lines, observed the noise pattern, and eventually arrived at a solution: pull-up resistors plus a software outlier filter. It worked, and I was satisfied.

But when I studied how Tianyi Welding handled signal integrity in their sensor networks — the 4-20mA current loops, the shielded twisted-pair cables, the 30-centimeter minimum clearance from power lines, the single-point grounding, the 90-degree crossing rule — I realized that my empirical approach was not a methodology. It was a workaround. A successful workaround, but a workaround nonetheless.

Industrial signal integrity is not achieved through trial and error. It is achieved through the systematic application of standards that have been developed over decades, validated in thousands of installations, and encoded in engineering practice. The current loop is not "one way to transmit a signal" — it is the standard way, chosen because it solves specific, well-understood failure modes that voltage signaling cannot address. The 30-centimeter clearance is not a rough guideline — it is a calculated distance, derived from electromagnetic coupling equations, that provides a quantified safety margin.

I had solved my signal integrity problem. The industrial engineers had designed a system where that class of problem could not occur in the first place. The difference is the difference between treating symptoms and building immunity.

### 3.3 Communication: From Function to Reliability

My custom 7-byte protocol worked, in the limited sense that it could transport data from one device to another when everything was functioning correctly. But when the CH340 RX channel failed, I had no tools to diagnose it. My protocol had no sequence numbers, so I could not tell if packets were being lost. It had no standard error codes, so I could not determine whether the failure was on the PC side or the STM32 side. It had no timeout mechanism, so I could not distinguish between "no command sent" and "command sent but not received."

Modbus, Profinet, EtherCAT — these protocols are not merely faster or more sophisticated versions of what I built. They address a fundamentally different set of requirements. They are designed for environments where a communication failure is not an inconvenience but a safety hazard. They provide sequence numbers, CRC checks, timeout detection, standard error codes, and deterministic latency — not as optional features, but as integral components of the protocol specification.

I had built a protocol that could transmit data. The industrial protocols are built on the assumption that transmission will fail, and they provide the tools to detect, diagnose, and recover from those failures gracefully.

### 3.4 Lifecycle: From Prototype to Product

My desktop projects had no concept of a lifecycle. The TB6612 module was expected to work until it didn't. When it burned out, I replaced it. There was no maintenance schedule, no service life specification, no preventive replacement plan.

In the industrial cabinet, every component had a design life. The PLC was rated for 10 to 15 years. The contactors were rated for millions of mechanical operations. The welding power supply had a 60% duty cycle rating and a specified service life of 7 to 10 years. There was a maintenance schedule: daily E-stop tests, weekly terminal tightening, monthly sensor calibration, semi-annual deep cleaning, annual comprehensive inspection.

This was not bureaucracy. This was the recognition that in an industrial setting, downtime costs money. A failed component does not just stop one machine — it can stop an entire production line. The maintenance schedule, the service life specifications, the scheduled replacement of wear parts — these are not optional. They are the difference between a system that sometimes works and a system that reliably works.

## 4. The Electronic Waste Question

There was one more lesson from this internship, and it was the most personal one.

During my projects, I accumulated a small collection of dead electronics. A burned TB6612 module. A fried STM32 board. A failed CH340 module. Clipped wire leads. Discarded resistors and capacitors. I had no clear idea what to do with them. I knew I should not throw them in the general trash, but I did not know where else they should go. My university lab did not have a designated e-waste collection point. There were no obvious channels for a student to responsibly dispose of small quantities of electronic waste.

I am not proud of what happened to those components. I suspect they ended up in the general waste stream, simply because the alternative was unclear and inconvenient.

Standing in the Tianyi Welding workshop, watching the fume extraction system do its work — a system that had been designed, budgeted for, and integrated into the workstation from the very beginning — I realized that this was the same engineering problem at two different scales. The welding fume and my dead TB6612 modules are both unintended consequences of making things. The fume extraction system and the e-waste recycling chain are both engineered mitigations — attempts to manage the downstream effects of industrial activity.

The difference is that the fume extraction system was part of the design. My e-waste disposal pathway was an afterthought — entirely unaccounted for in the project's planning or execution.

This internship did not give me a solution to the e-waste problem. But it gave me something that may be equally valuable: a genuine awareness that the problem exists, and a commitment to address it. In my future projects — and eventually, I hope, in the products and systems I help design — I will think about the entire lifecycle of the materials I use, not just their function during the brief window when they are useful to me.

## 5. What I Take Forward

I came to Tianyi Welding hoping to see how industrial electronics were different from what I had built on my desk. I got that answer, and it was far more detailed and specific than I had expected. The four gaps — protection philosophy, signal integrity standards, communication reliability, and lifecycle management — are now part of my mental framework for evaluating any electronic system, including my own.

But I also came away with something I had not expected. I realized that the engineering I had been doing — building things on my desk, breaking them, fixing them, learning from each failure — was not wrong. It was just incomplete. Every failure I experienced in my projects — the burned TB6612, the fried STM32, the encoder EMI spikes, the serial communication breakdown — was a miniature version of a failure mode that industrial engineers have been managing for decades. I had discovered each of these failure modes through painful personal experience. The industrial engineers had learned about them from standards, from textbooks, from the accumulated experience of their profession.

The difference is not that they are smarter than me. It is that they are standing on a body of knowledge that I had not yet accessed. This internship gave me a first glimpse of that body of knowledge. It showed me that the gap between my desktop projects and industrial systems is bridgeable — not through genius, but through discipline. Through standards. Through designing for failure rather than reacting to it. Through thinking about the entire lifecycle of a system, not just the moment when it first powers on and works.

I am not yet the engineer I want to be. But after this internship, I have a much clearer picture of what that engineer looks like, and a much better understanding of what I need to learn to become that person.

## 6. A Note of Gratitude

I want to thank the engineers at Tianyi Welding who took the time to walk me through their schematics, explain their design decisions, and share their hard-won field experience. Every question I asked was answered with patience and depth. Every time I pointed at a component and asked "what does this do?", I received not just a functional description, but an explanation of why that component was chosen, what failure mode it protected against, and what the consequences of omitting it would be.

This kind of mentorship cannot be replicated in a classroom. It can only be received in the field, from people who have spent years building, maintaining, and troubleshooting real systems. I am deeply grateful for their generosity, and I hope to pay it forward someday.
