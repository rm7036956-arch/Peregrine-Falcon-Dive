What this project is about

A peregrine falcon can hit speeds up to 350 km/h when it dives for prey. While diving, only two forces act on it — gravity, which pulls it down, and air drag, which pushes back against its motion. This project models that dive in Simulink.

The project has two parts:


Model the equation of motion for a fixed drag area and drag coefficient (the falcon in a straight dive).
Model the falcon opening its wings partway through the dive, which increases drag area/coefficient and rapidly slows it down before landing.


This README covers Part 1 — the basic equation of motion block diagram — and how to read the resulting velocity graph.

PHYSICS BEHIND THE PROJECT:
m*(dv/dt) = (1/2)*ρ*Cd*A*v² − m*g

m = mass of the falcon
v = velocity (this is the state we're solving for)
ρ = air density
Cd = drag coefficient
A = frontal area of the falcon
g = acceleration due to gravity


The idea is the same as pushing a block on a surface — motion happens when there's a net difference between the forces acting on the object. Here, drag (1/2)*ρ*Cd*A*v² opposes the fall, and gravity m*g pulls it down. The difference between the two is the net force, and net force divided by mass gives acceleration (dv/dt).

The minus sign in front of m*g is important: drag is acting upward (opposing the dive) while gravity is acting downward, so they're subtracted, not added.

Rearranging for dv/dt (dividing everything by m):

dv/dt = (1/m) * [ (1/2)*ρ*Cd*A*v² − m*g ]

This is the form that actually gets built in Simulink, because dv/dt is what we integrate to get v.

Sign convention

Initial velocity is set to −10, not +10. Downward is taken as the negative direction here, so a falcon starting its dive at some downward speed has to start negative. As the dive continues, gravity keeps making the velocity more negative (falling faster) until drag builds up enough to balance it out.


How the block diagram is built

The whole model is really just the rearranged equation above, wired up block by block:


1.  Integrator block — This is the core of the loop. It takes dv/dt as input and outputs v. Since velocity is what we actually want to see (and use in the drag term), the integrator's output is fed back into the rest of the diagram — this is what closes the loop.
2. Feedback tap — Right after the integrator, the velocity signal is tapped off (branched) and sent both to a scope/output and into the drag calculation path.
3. Square block — Takes v and outputs v², since drag depends on velocity squared, not velocity directly.
4. Constant block for (1/2)*ρ*Cd*A — Air density, drag coefficient, and area are all fixed numbers for Part 1, so they're combined into a single constant (0.5 * ρ * Cd * A).
5. Product/Multiply block — Multiplies v² by the constant above, giving the full drag force term: (1/2)*ρ*Cd*A*v².
6. Constant block for m*g — Mass times gravity, also fixed.
7. Subtract block — Takes the drag term and subtracts m*g from it, giving the net force: (1/2)*ρ*Cd*A*v² − m*g.
8. Gain block for 1/m — Multiplies the net force by 1/m, which finally gives dv/dt.
9. This dv/dt signal is fed straight back into the integrator from step 1, closing the loop and repeating the cycle every time step.


So the signal path is:
v (from integrator) → square → v² → multiply by (0.5*ρ*Cd*A) → subtract m*g → gain (1/m) → dv/dt → back into integrator → v ...

Reading the graph:

The output graph plots velocity vs. time from 0 to 10 seconds. It starts at −10 (the initial dive speed) and drops to roughly −84 by the 10-second mark.

The curve is not a straight line — it flattens out gradually rather than dropping at a constant rate. That's because drag force depends on v², so as the falcon speeds up, drag grows faster than gravity's pull stays the same. If there were no drag at all, the line would be a straight diagonal (constant acceleration = just g). Here, the curve bends because the net downward force is being eaten into more and more by drag as speed increases — the falcon is heading toward (but hasn't yet reached) a terminal velocity where drag would fully balance gravity.

