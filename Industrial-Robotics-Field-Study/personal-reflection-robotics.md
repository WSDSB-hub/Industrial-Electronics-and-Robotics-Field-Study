# Personal Technical Reflection: What the Welding Floor Taught Me About the Gap Between Automation and Understanding

## Prologue: Why I Keep Building

For the past year, I have spent most of my free time building a small two-wheeled robot. I wrote a PID controller from scratch on an STM32, tuned it by watching RPM curves on a serial plotter, and cursed at it when the encoder readings spiked to 14,000 for no visible reason. I trained a YOLO model to see chairs and backpacks, fused its output with monocular depth estimates, and watched my robot weave around obstacles in my living room. I burned a motor driver, fried a microcontroller, and learned the hard way that a 10-cent MOSFET module will not tolerate a 12.6V lithium battery. Then I did what every student building robots eventually does: I started asking whether my little machine was actually "intelligent" in any meaningful sense, or whether I had simply built a very obedient device that followed rules I had written for it.

This question followed me into my internship at Tianyi Welding. Standing in front of a FANUC M-20iD/25 six-axis industrial robot, watching it trace a weld seam with ±0.02 mm repeatability, I found the same question written in industrial scale. The machine was magnificent. It was also — in the way that mattered most to me — completely blind to what it was doing.

That tension, between extraordinary precision and total absence of understanding, is what this reflection is about. It is the most honest account I can give of what I saw, what I learned, and what I now want to spend the next several years studying.

## What the FANUC M-20iD/25 Can Do

The numbers are staggering. Twenty-bit absolute encoders on all six axes, yielding over one million counts per revolution. A layered control architecture with 4-to-8-millisecond interpolation and a 250-microsecond servo sampling period. RV reducers on the first three joints and harmonic reducers on the wrist, engineered for stiffness and longevity. The entire system communicating over FSSB fiber optic servo bus, with the controller running a proprietary real-time operating system that guarantees deterministic timing without the overhead of a general-purpose OS.

The result is a robot that can position a welding torch at a point in space, return to that exact point after a year of continuous operation, and hold the torch at a constant speed of 5 to 20 millimeters per second along a seam, with current fluctuations of ±5–10% that the engineers call "normal process variation" rather than error.

My PID controller, by comparison, holds a wheel speed within ±5 RPM of its target. That is roughly ±5% error on a 100 RPM setpoint. For a two-wheeled robot steering around a living room, that is more than adequate. For a welding torch, it would mean the difference between a sound joint and a catastrophic defect.

I understood, in an abstract way, that precision mattered. What I did not understand until the internship was how that precision is achieved — not as a single achievement, but as an architecture of layered decisions, each one made in response to a specific failure mode that had occurred at some point in the past and been engineered out of the design.

## What the FANUC M-20iD/25 Cannot Do

Here is the harder truth. For all that precision, the robot understands nothing.

Every trajectory it follows was taught by a human operator. Every welding parameter it uses was set by a process engineer. Every path it traces was defined in advance. When a workpiece is clamped at a slightly different angle, the robot does not notice. When the gap between two metal plates varies beyond the tolerance built into the program, the robot does not care. It follows its taught path precisely, and the weld fails at the exact point where the physical world refused to conform to the expectation.

The laser seam tracking system is the closest thing the workstation has to perception. It projects a laser stripe onto the seam, triangulates the groove geometry, and sends deviation corrections to the robot controller at the interpolation rate. It is a remarkable piece of engineering. But it is also, in the words of the field engineers, "a correction mechanism, not a judgment mechanism." It keeps the torch on the path that was defined for it. It does not decide what the path should be. It does not know whether the weld it just completed is good or bad. It does not notice the contact tip gradually wearing down over thousands of cycles. It does not see the spatter accumulating on its own lens.

In other words, the industrial robot has the physical capability to act with superhuman precision and the cognitive capability of a servo loop. Everything it does is, in the deepest sense, an execution of human decisions that were made before the robot ever moved.

## The Gap I Want to Work On

The field engineers at Tianyi Welding described this gap better than any textbook. When I asked them to compare robot welding with manual welding, they did not give me a simple answer. In most scenarios, they said, the robot's consistency is far superior. The robot never gets tired. It never rushes. It never has an off day.

But when the workpiece gap varied too much, or the fixture positioning was imperfect, the robot's advantage vanished. A skilled human welder could see that the gap was too wide, adjust the torch angle, widen the weave, slow the travel speed, and rescue the weld. The robot could not. It followed its programmed path exactly, and the weld failed where the real world refused to match the program.

That improvisational ability — grounded in years of visual and tactile experience, in an implicit understanding of how molten metal behaves, in a felt sense of what a good weld looks and sounds like — is precisely what today's robots lack. It is not a matter of more precision. It is not a matter of faster computation. It is a matter of having a model of the world rich enough to recognize when the world is not what you expected, and the capacity to adjust your behavior accordingly.

This is the problem I want to work on. It sits at the intersection of control theory and machine learning, of perception and reasoning, of classical robotics and whatever comes next. It is the problem that programs in robotics, intelligent systems, and agentic AI are beginning to address — not by making industrial robots more precise, but by giving them the capacity to understand what they are doing.

## What I Bring

I came to this realization by building things, not by reading about them.

On the software side, I have built the constituent components of an embodied intelligent system. I deployed YOLO and MiDaS on a real robot, fused their outputs into actionable obstacle data, and closed the perception-decision-action loop. I wrote a PID controller from scratch and tuned it until it resisted disturbance. I refactored a monolithic script into a modular multi-threaded architecture, then validated a ROS2 node-based version in a Linux environment. I trained a PPO reinforcement learning agent in PyBullet and watched it discover a strategy I had not programmed. Each of these pieces works, and I have seen how they fit together.

On the hardware side, I have burned motor drivers and fried microcontrollers, which taught me to respect power electronics in a way no datasheet could. I have diagnosed encoder EMI problems, worked through signal integrity failures, and learned to design circuits defensively — adding isolation resistors and protection capacitors not because a textbook told me to, but because a 12V backfeed killed my first STM32 and I was not going to let it happen twice. My internship extended this into the industrial realm: I studied a 19.5 kW welding power supply with IGBT-based inversion, analyzed a sensor signal chain designed to survive electromagnetic interference from a 500-amp arc, and traced the communication architecture of a multi-device industrial network running Profinet over shielded twisted pair.

I can speak the language of both the AI lab and the factory floor. What I cannot yet do is bring them together. That is what I want to learn.

## What I Want to Study

If I am given the chance to continue into graduate study, these are the questions I will pursue.

**Learning-based perception for manipulation under real-world noise.** The seam tracking sensors I observed degrade in confined spaces, under heavy spatter, and against complex workpiece geometries. I want to explore whether learning-based approaches — trained on real and simulated sensor data, augmented by domain randomization and sim-to-real transfer — can produce perception systems that maintain robustness across the full messiness of a real production environment, rather than only under the narrow conditions for which they were designed.

**Imitation learning and reinforcement learning for contact-rich manipulation.** Welding is an extreme case of the manipulation problem: the robot must maintain a precise relationship between its tool and a deforming, glowing, unpredictable material while a physical process unfolds at thousands of degrees. Human welders learn this through thousands of hours of sensorimotor practice. I want to study whether robots can learn it the same way — from demonstration, from reward, and from their own experience — rather than from hand-tuned programs that break the moment the workpiece is clamped differently.

**The integration of learned components with classical control.** My own projects taught me that a PID controller is robust and predictable in ways a neural network is not. My internship taught me that industrial systems depend on that predictability for safety and repeatability. Embodied AI cannot simply replace classical control; it must learn to live with it. I want to understand how to architect systems in which learned modules make high-level decisions while low-level controllers guarantee stability and safety — and how to ensure that learned modules fail gracefully when they fail.

**Agentic AI in physical systems.** The frontier of AI is moving toward agents that plan, reason about trade-offs, and explain their decisions. Most of this work today lives in software. Bringing it into the physical world — into robots that touch, move, and affect the real environment — is the next step. I want to be part of that step.

## The Kind of Training I Am Looking For

I am applying to programs that span robotics, intelligent systems, electronic engineering, and mechanical engineering because I believe the problem I care about cannot be solved from within any single discipline. It needs control theory for stability, mechanical engineering for physical insight, electronic engineering for sensor and actuator design, and AI for perception, learning, and reasoning.

The training I am looking for is not a narrow specialization. It is a foundation that lets me see a robot as a whole system — from the encoder on a motor shaft to the learned policy deciding where the robot should go next. I want courses that force me to understand why a control loop is stable, why a sensor signal degrades in a noisy environment, why a learning algorithm converges or does not, and why a mechanical design survives a million cycles or fails at ten thousand. I want projects that make me build these things, not just analyze them. And I want an environment where the people around me are working on the same fundamental question I am: what does it take for a machine to understand what it is doing, and to act on that understanding?

## Epilogue

At the end of my internship, I stood on the Tianyi Welding floor and watched the FANUC robot finish a weld cycle. It moved through the crater-fill sequence — decelerating, the welding machine dipping its current, the wire feeder slowing, the shielding gas still flowing. It dwelled for half a second, then lifted and moved away. The weld, when I looked at it, was perfect. Consistent width, consistent height, consistent appearance. The kind of weld a skilled human would be proud of.

And yet the robot had no idea what it had just done. It had followed a path, executed a sequence, and produced a result that it could not see, could not judge, and could not explain. The intelligence in that weld did not belong to the robot. It belonged to the engineers who had taught the path, set the parameters, and tuned the sequence. The robot was their hands, not their mind.

I want to build the mind. Not to replace the hands — the hands are already extraordinary. But to give the hands something they do not have: an understanding of what they are doing, and the ability to decide what to do next.

That is what I want to study. That is what I want to build. And that is why I am applying to graduate school — not to become a specialist in any one technique, but to learn how to bring perception, learning, and control together into machines that finally begin to understand what they are doing.

I am ready to start.
