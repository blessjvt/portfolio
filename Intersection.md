***Refinements to the Crossroads Autonomous Intersection Management System: A Safety and Efficiency Evaluation***

Bless Jah

Bradley Department of Electrical and Computer Engineering  
*Virginia Tech*  
Blacksburg, VA, USA  
blessj@vt.edu

Elori Smile

Bradley Department of Electrical and Computer Engineering  
*Virginia Tech*  
Blacksburg, VA, USA  
esmile@vt.edu

***Abstract*****—This paper addresses critical limitations in the autonomous intersection management system known as Crossroads. Specifically, the scheduling algorithm was refined to incorporate realistic vehicle parameters, including vehicle length and turning maneuvers (left, right, straight), thereby improving operational efficiency and safety. Formal verification using the hybrid theorem prover KeYmaera X demonstrated correctness and collision avoidance of the improved system. Additionally, simulations conducted in a modified MATLAB intersection simulator confirmed that the enhanced scheduling algorithm efficiently manages intersection throughput with acceptable delays and ensures safety. Results from an experimental scenario involving eight vehicles showed an average delay of approximately 46 seconds and a throughput of 0.086 vehicles per second, affirming the practical feasibility and effectiveness of the refined approach.**

***Keywords—Autonomous vehicles, intersection management, traffic scheduling, hybrid systems, formal verification, KeYmaera X, vehicle-to-infrastructure communication, collision avoidance, simulation, connected vehicles.***

1. #  Introduction

Traffic intersections are among the most complex and accident-prone areas of road networks. Traditional traffic control systems, which rely on red-yellow-green signal cycles, are often suboptimal in handling variable traffic flows and lead to increased congestion, delays, and emissions. The rise of connected and autonomous vehicles (CAVs) opens new avenues for rethinking intersection management. By enabling real-time communication between vehicles and infrastructure, autonomous intersection management systems can dynamically schedule vehicle crossings with a focus on both safety and efficiency.

The Crossroads system \[1\], and its extension Crossroads+ \[2\], introduced a time-sensitive approach that replaces fixed signal phases with a scheduling algorithm. These systems use vehicle arrival predictions to compute safe actuation times, thereby eliminating unnecessary idling and improving throughput. However, the original work omits important real-world considerations, such as the physical length of vehicles and their directional intent (e.g., turning left or right), both of which affect space-time conflict zones within intersections.

In this paper, we address these limitations through a set of refinements to the Crossroads scheduling algorithm. Our contributions include:

* An updated formulation of the calculateTargetArrivalTime() and calculateActuationTime() functions to account for vehicle length.  
* Extension of the scheduling logic to support turning behaviors, which impact path overlap and timing constraints.  
* A formal model of our approach in KeYmaera X, demonstrating provable safety guarantees.  
* A modified version of the official Matlab simulator \[3\], used to validate our refinements through empirical testing.

Our findings show that the refined model preserves collision-free behavior and meets schedule-level safety standards, while achieving a reasonable level of throughput and delay in simulation tests.

2. # Related Work

Intersection management is a critical area in traffic engineering and intelligent transportation systems, particularly with the increasing adoption of connected and autonomous vehicles (CAVs). Several systems have been proposed in recent years to optimize intersection efficiency and safety beyond traditional fixed-signal mechanisms.

The Crossroads approach introduced by Andert et al. \[1\] utilizes vehicle-to-infrastructure (V2I) communication to dynamically schedule vehicles through intersections. Rather than employing static traffic lights, Crossroads calculates precise arrival and actuation times, significantly reducing idle times and theoretically increasing intersection throughput. Its successor, Crossroads+, proposed by Khayatian et al. \[2\], extended this method with additional considerations such as time-awareness and more advanced scheduling heuristics, improving intersection efficiency under varying traffic conditions.

However, these pioneering works exhibit critical limitations. Specifically, the original scheduling algorithm defined in Crossroads \[1\] (Algorithm 5\) and later employed by Crossroads+ \[2\] does not adequately address real-world constraints. Notably, it neglects the physical dimensions of vehicles, an essential factor determining safe time gaps and spatial arrangements in the intersection. Ignoring vehicle length can lead to underestimated spatial occupancy, thereby risking collisions or suboptimal scheduling.

Additionally, neither Crossroads nor Crossroads+ explicitly considers vehicle turning directions (e.g., left, right, straight). This omission simplifies the modeling but significantly impacts realism and practical applicability, as different vehicle trajectories substantially influence intersection dynamics and space-time conflict patterns.

Other approaches in the literature have begun exploring these dimensions. For example, some autonomous intersection models integrate explicit lane-based scheduling or platoon-based scheduling algorithms. However, they typically focus either solely on efficiency or only on simplified safety constraints, rarely combining rigorous formal verification with realistic simulation validation.

In this paper, we bridge these gaps by refining the original Crossroads scheduling algorithm to explicitly incorporate vehicle lengths and turning behaviors. We then perform formal verification of these refinements using the KeYmaera X verification framework and empirically validate our approach using a modified version of the Matlab simulator originally developed by Khayatian \[3\]. This dual verification—formal and empirical—ensures robust safety guarantees while offering realistic performance evaluations under practical scenarios.

3. # Technical Approach

This section describes the refinements made to the Crossroads scheduling algorithm to improve realism and correctness. Specifically, we address two major limitations of the original system: the omission of vehicle dimensions and the lack of support for turning maneuvers. Our refined scheduler incorporates these aspects while maintaining safety and improving efficiency.

## *A. Overview of the Original Scheduling Logic*

The original Crossroads system computes a Target Arrival Time (ToA) and an Actuation Time (TAc) for each vehicle based on its predicted arrival and desired velocity. These values ensure that vehicles are assigned non-conflicting times to enter and traverse the intersection. However, vehicles are modeled as point masses, and the scheduler assumes that all vehicles move straight through the intersection. As a result, space-time overlaps between real vehicles are not accurately handled.

## *B. Refinement 1: Incorporating Vehicle Length*

To account for physical vehicle dimensions, we revise the scheduling logic to maintain a minimum safety distance that includes both vehicles' lengths. Let `L1` and `L2` be the lengths of two consecutive vehicles, and let `safetyGap` be the required minimum clearance. The system ensures:
<img width="277" height="63" alt="Screenshot 2025-10-30 at 1 14 25 PM" src="https://github.com/user-attachments/assets/8bee532e-02fd-4926-ab22-05c35ff48fcd" />

The actuation time and ToA for a following vehicle (vehicle 2\) are computed as:

<img width="236" height="97" alt="Screenshot 2025-10-30 at 1 14 31 PM" src="https://github.com/user-attachments/assets/717f74bb-dde0-4436-a9a7-66b9aab4c308" />

Here, dpath2 is the distance vehicle 2 must travel through the intersection. This guarantees that the second vehicle enters the intersection only after the first has fully cleared it.

## *C. Refinement 2: Direction-Aware Scheduling*

The scheduler is extended to model each vehicle’s entry and exit lanes. Each origin-destination pair corresponds to a turning behavior (e.g., straight, left turn, or right turn), which defines the spatial path within the intersection. A conflict matrix is precomputed to determine whether two vehicle paths overlap.

When scheduling a new vehicle:

* All previously scheduled vehicles with overlapping paths are identified.

* The latest actuation time among these is determined.

* The new vehicle’s ToA is scheduled to occur only after all conflicting vehicles have exited safely.

This enhancement allows the scheduler to permit overlapping ToAs for non-conflicting paths (e.g., independent left turns) while enforcing separation where necessary.

## *D. Conflict-Aware Scheduling Algorithm*

The refined scheduling procedure is as follows:

1. Determine the (entry, exit) path for each incoming vehicle.

2. Identify scheduled vehicles with conflicting paths.

3. Set ToA to the earliest possible time after all conflicts have cleared.

4. Compute TAc and assign a velocity accordingly.

This approach enforces **mutual exclusion in space-time** for conflicting vehicles while improving overall throughput through smarter conflict resolution.

## *E. Implementation in Simulation*

We implemented the refined scheduling logic in the official Matlab-based simulator from \[3\]. The following enhancements were added:

* Dynamic conflict detection based on vehicle paths.

* Real-time calculation of ToA and TAc using length- and direction-aware logic.

* Support for heterogeneous vehicle characteristics such as length, velocity, and route.

These improvements enabled a more realistic simulation, which served as the basis for both the formal verification and the experimental evaluation that follow.

4. # Formal Modeling and Verification

To ensure that the refinements to the Crossroads scheduling system are **provably safe**, we developed a formal model of the intersection dynamics and scheduling logic using KeYmaera X, a theorem prover for hybrid systems based on differential dynamic logic (dL). explicitly capturing vehicle lengths (L1, L2), safety gaps (safetyGap), velocities (vmax1, vmax2), and calculated arrival times (ToA) and actuation times (TAct).

There were several important concerns to keep in mind: the initial condition, the Control Logic, the Continuous Dynamic Logic, the post condition.

## *A. Modeling Assumptions*

Our formal model captures both **discrete decision-making** (scheduling, actuation) and **continuous dynamics** (vehicle motion through the intersection). The key assumptions in our model include:

* Each vehicle has a known initial position, velocity, length, and destination (turning intent).

* Vehicles follow a preassigned, conflict-free schedule computed by the refined algorithm.

* The vehicle acceleration is bounded and deterministic, and the vehicles obey the assigned ToA and TAc precisely.

* The intersection geometry is modeled as a set of conflict zones based on origin-destination paths.

These assumptions align with those used in simulation and serve as the basis for our safety verification.

### *B. Safety Property*

The primary safety property we verify is **schedule-level collision freedom**, formalized as:

*At no point in time do two vehicles occupy overlapping space within the intersection.*

This is modeled in KeYmaera X as an invariant over the combined continuous dynamics of all scheduled vehicles. Each vehicle's position is represented as a function of time, and mutual exclusion over shared spatial regions is enforced through schedule separation constraints.

## *C. Hybrid Program Structure*

There were several important phases to keep in mind: the initial condition, the Control Logic, the Continuous Dynamic Logic, the post condition. Our model encodes the system as a hybrid program with the following structure:

1) ### ***The initial Condition***

The initial condition defines the starting parameters for the system. It establishes a set of variables to ensure the system begins in a well-defined, safe, and realistic state. The vehicles are represented by x1 and x2, with corresponding velocities v1 and v2. Their times of actual arrival and target times of arrival are given by TAct1, ToA1, TAct2, and ToA2, ensuring the model accounts for timing consistency. Vehicle-specific parameters such as length (L1, L2), maximum speed (vmax1, vmax2), and path distance (dpath1, dpath2) are constrained to be greater than zero. A safetyGap is included to prevent collisions, and the global simulation time variable helps manage system progression. The approach1 and approach2 variables define each vehicle's approach direction (0 \= North, 1 \= East, 2 \= South, 3 \= West), while route1 and route2 describe their intended turn directions (0 \= left, 1 \= straight, 2 \= right). These constraints collectively ensure logical, bounded, and safe initialization of the model.

*ProgramVariables*  
  *Real TAct1;    Real ToA1;*  
  *Real TAct2;    Real ToA2;*  
  *Real L1;       Real vmax1;    Real dpath1;*  
  *Real L2;       Real vmax2;    Real dpath2;*  
  *Real safetyGap;*  
  *Real time;*  
  *Real x1;       Real v1;*  
  *Real x2;       Real v2;*

  *Real approach1;  /\* 0=N,1=E,2=S,3=W \*/*  
  *Real approach2;  /\* the other car's approach \*/*  
  *Real route1;     /\* 0=left,1=straight,2=right \*/*  
  *Real route2;*  
*End.*

*Problem*   
  *(    time \= 0*  
      *∧ x1 \= 0 ∧ x2 ≤ \-safetyGap*  
      *∧ ToA1 \= 0 ∧ ToA2 \= 0*  
      *∧ L1 \> 0 ∧ L2 \> 0*  
      *∧ vmax1 \> 0 ∧ vmax2 \> 0*  
      *∧ dpath1 \> 0 ∧ dpath2 \> 0*  
      *∧ safetyGap \> 0*  
      */\* approach2, route2 will be chosen at runtime \*/*  
  *)*

2) ### ***The Control Logic***

The control logic of a system schedules action to the variables in the system. The code logic in our project discreetly handles the scheduling of the arrival times of both cars and computes both vehicles *Time of Arrival* (ToA) and *Time Actually cleared* (TAct) based on the safety gaps. The safety gap is the minimum required distance and the minimum spacing required to maintain a safe following distance.

  *{  approach1 := \*;*   
      *?(0 ≤ approach1 ∧ approach1 ≤ 3);*  
      
      *route1 := \*;*  
      *?(0 ≤ route1 ∧ route1 ≤ 2);*  
          
   */\* choose vehicle 2s approach nondeterministically \*/*  
     *approach2 := \*;*

 *?(approach2 ≠ approach1 ∧ approach2 ≥ 0 ∧ approach2 ≤3);*

 */\* choose vehicle 2s turn nondeterministically \*/*

      *route2 := \*;*  
      *?(route2 ≥ 0 ∧ route2 ≤ 2);*

      */\* Schedule ToA1 after vehicle 2 clears \*/*  
      *{*  
        *?(ToA2 \+ (L2 \+ safetyGap) / vmax2 ≥ time);*   
        *TAct1 := ToA2 \+ (L2 \+ safetyGap) / vmax2; } \++*  
      *{*  
        *?(ToA2 \+ (L2 \+ safetyGap) / vmax2 ≤ time);*   
        *TAct1 := time;*  
      *};*  
      *ToA1 := TAct1 \+ (dpath1 \+ L1) / vmax1;*

      */\* Schedule ToA2 after vehicle 1 clears \*/*  
      *{*  
        *?(ToA1 \+ (L1 \+ safetyGap) / vmax1 ≥ time);*   
        *TAct2 := ToA1 \+ (L1 \+ safetyGap) / vmax1;*  
      *} \++ {  ?(ToA1 \+ (L1 \+ safetyGap) / vmax1 ≤ time); TAct2:= time;  };*  
      *ToA2 := TAct2 \+ (dpath2 \+ L2) / vmax2;*

3) ### ***Continuous Evolution***

The vehicles follow kinematic motion equations (e.g., constant velocity or acceleration) until they exit the intersection region. 

X1’ and X2 represent the positions of each vehicle change based on their velocities. The evolution domain of the system, *& (x1 ≤ dpath1 ∧ x2 ≤ dpath2),*  serves as a break for continuous evolution. It serves as the stop condition when either vehicle reaches the end of their path: *dpath\#* .

 */\* Continuous dynamics \*/*  
      *{*  
        *time' \= 1,*  
        *x1' \= v1,*  
        *x2' \= v2*  
        *& (x1 ≤ dpath1 ∧ x2 ≤ dpath2)*  
      *}*

      ***/\*\* Invariant only applies if routes conflict \*/***

  *@invariant*

        *( ((   /\* \=== Vehicle 1 from North (0) vs E,S,W \=== \*/*

   *(approach1 \= 0 ∧ route1 \= 0 ∧ approach2 \= 1 ∧ route2 \= 1\) ∨ (approach1 \= 0 ∧ route1 \= 0 ∧ approach2 \= 1 ∧ route2 \=2) ∨ (approach1 \= 0 ∧ route1 \= 1 ∧ approach2 \= 1 ∧ route2 \=0) ∨ (approach1 \= 0 ∧ route1 \= 1 ∧ approach2 \= 1 ∧ route2 \=1) ∨ (approach1 \= 0 ∧ route1 \= 1 ∧ approach2 \= 1 ∧ route2 \=2)  ∨ (approach1 \= 0 ∧ route1 \= 2 ∧ approach2 \= 1 ∧ route2 \=1) ∨ (approach1 \= 0 ∧ route1 \= 0 ∧ approach2 \= 2 ∧ route2 \=0) ∨ (approach1 \= 0 ∧ route1 \= 0 ∧ approach2 \= 2 ∧ route2 \=1)   ∨ (approach1 \= 0 ∧ route1 \= 1 ∧ approach2 \= 2 ∧ route2 \=0)   ∨ (approach1 \= 0 ∧ route1 \= 1 ∧ approach2 \= 2 ∧ route2 \=1) ∨ (approach1 \= 0 ∧ route1 \= 2 ∧ approach2 \= 2 ∧ route2 \=1) ∨ (approach1 \= 0 ∧ route1 \= 0 ∧ approach2 \= 3 ∧ route2 \=0) ∨ (approach1 \= 0 ∧ route1 \= 1 ∧ approach2 \= 3 ∧ route2 \=0) ∨ (approach1 \= 0 ∧ route1 \= 1 ∧ approach2 \= 3 ∧ route2 \=1) ∨ (approach1 \= 0 ∧ route1 \= 2 ∧ approach2 \= 3 ∧ route2 \=0) ∨ (approach1 \= 0 ∧ route1 \= 2 ∧ approach2 \= 3 ∧ route2 \=1)  )  ∨*  
*…. (see rest of code in Project4Proof.kyx file)*

4) ### *The Post Condition*

*Safety Checks:* Throughout execution, KeYmaera X verifies that no two vehicles' occupied space-time regions overlap.  

*(x1 ≥ x2 \+ safetyGap ∨ x2 ≥ x1 \+ safetyGap)*

That line of code ensures that the vehicles are always separated by at least safetyGap, enforcing the proof to hold true at termination.

The correctness proof confirms that under our refined algorithm, vehicles never collide. A screenshot of the successful verification, found in Figure 1, demonstrates this formal guarantee:

<img width="411" height="147" alt="Screenshot 2025-10-30 at 1 20 27 PM" src="https://github.com/user-attachments/assets/ea133c16-ac65-474f-b245-e98f5368acb0" />

Figure 1: Screenshot of formal verification in KeYmaera X

We used differential invariants and loop invariants to ensure that the safety conditions remain valid across all time steps and transitions.

## *D. Verification Results*

Our KeYmaera X model successfully verified the unsignalized four-way intersection scenario comprising a vehicle waiting on the south approach and a traversing vehicle approaching from the east, north, or west. All 9 maneuver combinations for each vehicle (left turn, right turn, straight) were systematically encoded, resulting in  27  possible turn scenarios. A safety-distance invariant was imposed to ensure that the waiting vehicle is released only after the traversing vehicle has cleared a predefined separation region.

Vehicle length and turning-direction dynamics were incorporated into a refined scheduling scheme to prevent space–time conflicts. All proof obligations associated with these conflict scenarios and the temporal-separation constraints were discharged successfully, demonstrating collision-free operation in every evaluated case. Under the specified modeling assumptions and constraints, the scheduler enforces sufficient temporal separation, thus providing a high-confidence guarantee that the system maintains collision-free operation for all scheduled vehicles.

5. # Experimental Evaluation

To evaluate the practical effectiveness of our refined intersection scheduling algorithm, we conducted simulation experiments using a modified version of the Matlab-based traffic simulator provided by Khayatian et al. \[3\]. The simulator was extended to support vehicle-specific dimensions, turning directions, and the updated scheduling logic described in Section 3\.

### *A. Methodology*

The first step in extending the simulator was to identify the code that governs intersection behavior. All intersection scheduling and safety logic lives in I**ntersectionManagement.m**, under the section marked *IM management*. Due to the increasing complexity and execution overhead of this module, we refactored parts of the code to rely on helper functions within separate files.This restructuring promotes modularity, simplifies maintenance, and enables isolated verification of core algorithms.

Five helper functions now support the main scheduler. The function “makeIMPacket” constructs the network request by encapsulating each vehicle’s identifier, lane assignment, desired velocity, destination lane, and transmission timestamp into a unified message structure. The predicate “isConflict” evaluates whether two trajectories intersect within the intersection geometry, thereby filtering only those vehicle pairs that could potentially collide. The utility “pathLength” computes the center-line distance traversed through the junction, selecting between straight segments or quarter-circle arcs for turning maneuvers, based on the ingress and egress lanes and the standard lane width.

The functionality of the safety protocol implemented in KeYmaerax resides in the two timing functions. The first, “calculateActuationTime(nowTime, prevVeh, safetyGap)”, enforces sequential clearance by returning

<img width="353" height="66" alt="Screenshot 2025-10-30 at 1 22 47 PM" src="https://github.com/user-attachments/assets/6e966108-7637-4acc-86f3-1fe15fef5ce6" />

where *prevVeh* denotes the most recently scheduled vehicle. This evaluation guarantees that a vehicle’s front bumper enters the conflict zone only after its predecessor’s rear bumper has fully exited and the prescribed safetyGap has elapsed. Subsequently, “calculateTargetArrivalTime(TE, car, laneWidth)” assigns the vehicle its maximum cruising speed (vmax) and computes the time of arrival

<img width="215" height="58" alt="Screenshot 2025-10-30 at 1 23 12 PM" src="https://github.com/user-attachments/assets/dec7935c-bfd2-4fef-a810-b3abd92aa5e0" />

where dpath  is obtained from “pathLength”. This calculation returns the precise instant at which the vehicle’s rear bumper clears the intersection.

Within the refactored *IntersectionManagement.m*, each incoming request is transformed into a structured record capturing the vehicle’s state (ID, current lane, destination lane, pose, length, vmax, and current speed). The scheduler then invokes calculateActuationTime to determine the safe actuation instant, followed by calculateTargetArrivalTime to establish the clearance time. A response packet, containing the commanded speed and the original request timestamp, is returned to the originating vehicle via the network buffer, and the newly scheduled record (including its computed ToA) is appended to the active queue. Subsequent vehicles inherently respect the computed clearance intervals, thus maintaining collision-free separation.

To verify correctness, we implemented test cases: a unit test and a scheduler test. Unit tests verify that pathLength, calculateActuationTime, and calculateTargetArrivalTime produce expected outputs for canonical geometric and temporal scenarios. An integration test simulates vehicle arrivals and confirms that every pair of potentially conflicting vehicles satisfies the inequality

<img width="329" height="52" alt="Screenshot 2025-10-30 at 1 23 31 PM" src="https://github.com/user-attachments/assets/01f334e0-fe0b-4190-9437-01294ad7af5c" />

thereby ensuring the enforced temporal buffer (safetyGap) suffices to prevent overlap. These tests, in conjunction with our formal KeYmaeraX proof, provide rigorous assurance of the scheduler’s ability to uphold safety requirements under the Crossroads protocol.

### *B. Simulation Setup*

The experiment involved eight autonomous vehicles approaching a four-way intersection from various entry lanes and with varying intended destinations (e.g., straight, left, right turns). Each vehicle was assigned the following parameters:

* **Length:** Between 19.5 m and 28.7 m

* **Maximum Velocity (vmax):** Ranged from 3.86 m/s to 24.28 m/s

* **Arrival Time:** Staggered to simulate realistic arrival conditions

* **Lane to Destination Mapping:** Includes both straight and turning paths

The simulation processed all vehicles through the refined scheduler, which computed their Target Arrival Times (ToA), Actuation Times (TAc), and assigned velocities based on spatial conflicts and vehicle constraints. A unit test was designed and implemented to ensure the core functions and helper functions work as intended. Below in Figure 2 is the output of the Matlab command window when the unit test was ran.

<img width="282" height="111" alt="Screenshot 2025-10-30 at 1 23 59 PM" src="https://github.com/user-attachments/assets/7b645645-cdd5-42b1-ac3b-8beb1d974427" />
Figure 2: Screenshot of formal verification after running the Unit Test

### *C. Safety Results*

The simulation passed the **schedule-level safety test**, confirming that no two vehicles entered the intersection in overlapping space-time regions. This result aligns with our formal verification in KeYmaera X and demonstrates consistency between the theoretical and practical implementations of the scheduler.

**Safety Outcome:**

* 8 vehicles scheduled

* **Schedule-level safety: Passed**

* **No collisions or spatial violations detected**

### *D. Efficiency Metrics*

Two primary metrics were used to assess efficiency:

* **Average Delay:** Defined as the difference between the vehicle's ToA and its original arrival time.

* **Throughput:** Defined as the number of vehicles successfully scheduled per second.

The results were as follows:

* **Average Delay:** 46.267 seconds

* **Throughput:** 0.086 vehicles/second

While the average delay is relatively high, primarily due to complex turning interactions and large vehicle lengths, the throughput is consistent with safety-first scheduling policies that prioritize conflict avoidance.

### *E. Vehicle-Level Results*

A summary of individual vehicle data is presented in Table 1.**<img width="342" height="62" alt="Screenshot 2025-10-30 at 1 24 52 PM" src="https://github.com/user-attachments/assets/aef9b4b4-b74e-4fed-9a4e-cf96f5103b16" />**Table 1\. Vehicle scheduling results showing individual performance metrics.


### *F. Analysis*

The high delays observed for vehicles 3–8 can be attributed to compounded conflicts and limited maneuvering options at the intersection. Longer vehicles and those executing turns required significantly more space-time clearance, resulting in sequential scheduling and delayed ToAs.

Despite these challenges, the simulation validates that the refined scheduler can **safely and consistently manage complex vehicle flows**, while supporting diverse vehicle profiles and directional intentions.

6. # Conclusion

In this paper, we presented a set of refinements to the Crossroads autonomous intersection management system, addressing critical limitations in its original design. Specifically, we enhanced the scheduling algorithm to account for vehicle length and turning direction—factors that are essential for ensuring realistic, safe, and efficient intersection control.

We formally modeled the refined system using KeYmaera X, and proved that it satisfies a schedule-level safety property: no two vehicles occupy conflicting space-time regions within the intersection. This formal guarantee provides strong confidence in the safety of our approach.

In addition, we extended the official Matlab-based traffic simulator to support vehicle-specific dimensions and directional behaviors. Simulation results on a scenario involving eight vehicles confirmed that our scheduling approach preserves safety while managing complex vehicle flows. Although the average delay was significant due to conservative scheduling in conflict-heavy scenarios, throughput remained stable, and no safety violations occurred.

These results demonstrate the viability of direction- and dimension-aware intersection management in fully autonomous traffic environments. Our contributions bring the Crossroads framework closer to real-world applicability by addressing important physical and logical constraints that were previously abstracted away.

Future research will focus on further improving efficiency while maintaining safety. Potential directions include:

* Introducing dynamic re-scheduling or optimization heuristics to reduce unnecessary delays.

* Supporting mixed autonomy environments with both autonomous and human-driven vehicles.

* Scaling the system to support multi-intersection coordination and larger traffic networks.

* Incorporating more complex vehicle dynamics and acceleration profiles into both modeling and scheduling.

By combining formal verification with simulation-based validation, we provide a robust foundation for next-generation autonomous intersection control systems.

##### References

\[1\] E. Andert et al., "Crossroads: Time-sensitive autonomous intersection management technique," *Proceedings of DAC*, 2017\.

\[2\] M. Khayatian et al., "Crossroads+ a time-aware approach for intersection management of connected autonomous vehicles," *ACM Transactions on Cyber-Physical Systems*, vol. 4, no. 2, pp. 1-28, 2019\.

\[3\] Traffic Intersection Simulator for Autonomous Vehicles, [GitHub Repository](https://github.com/mkhayatian/Traffic-Intersection-Simulator-for-Autonomous-Vehicles).

7. # Appendix: Added files
List files that that were added or modified:  
pathLength.m  
IntersectionManagement.m  
unit\_test.m  
test.m  
isConflict.m  
calculateTargetArrivalTime.m  
calculateActuationTime.m

[image1]: <data:image/png;base64,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>

[image2]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAANcAAABaCAYAAAArU8OzAAAPsElEQVR4Xu2diY8UxRfH/QeMQYJRkRgQUANeoGAEoigCKlEwCHhgBBUVUKKiaIz3BYKggIiCCooiKIoKeETigRceHF7xBsULDy4BD8T65fPI61910bM7OzO907PzPklnu6tqZmp2+tv16lXVq12cYRipsEuYYBhGaTBxGUZKmLgMIyVMXIaREiauBkKXLl3crrvuGiZnlj/++MO1bdvWjRo1Sup9/vnnh0VifPPNN2FSXjz22GNuzz33dOecc47bsGGDpC1evNgdf/zx7oILLghKlxYTVwPgzz//dI0aNXJt2rQJszLLrFmzRFS///67u/vuu93y5cvDIjGmT58eJuUFn3HrrbeGye7NN980cRm18/rrr8tNNGTIkDArs4wfP17q/Pfff4dZiQwYMCBMqpVly5a5Qw45xP31119hlmDiMmrl1FNPlRv1ww8/DLPqhb59+7qePXu6ww8/PCbwk08+2XXv3t316tXLnXbaad4rdrQoenzyySeStmXLFjETe/fuLWbcd999584777xYWY5vv/3WnXXWWdE1rTbweVxPnjzZ/ffff65FixZuwYIF/sfGWLRoUXT+9ttvR/Wlro8//niUd/bZZ4tpyXtfeuml8tl77723++WXX6IySZi4Kpzt27fLD7/ffvuFWfXG3Llz5S916dOnT5TuC+fVV191L774YpQ3btw4yf/tt9/k+p9//nHHHnusGzlypLzP0KFDXfv27SUPMVE2hM8l/c4775RrzGNuevjyyy8lb+XKlf5LctKvX7+ovtSVc63ve++95y6//HJJ27Rpk6R9+umn7qijjnJbt2713yaGiavCoa/Cj3766aeHWfXCr7/+6p5++uno2m+hJk2aJEJRfNMuFNfUqVPlmveDFStWRILKJa5t27a5Aw44QFobwDxGnKDiWrVqlfeK3Lz77rux+iJsv76PPvroTnXgesKECbE0HxNXhYP5xI+8ZMmSMEue5MCTnTI///xzUKI0nHTSSfL+HGPHjo3SucZcfPLJJ93q1avdiSeeGOWF4mrZsqXbZ599onyfXOICzcOkbNeunZiDyoUXXuhmzJjhlf4/iBlzWlue66+/Pqovde3QoUOsvnPmzNmpDlx36tQpluZj4qpwWrdu7XbffXe5uXzeeeed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>

[image3]: <data:image/png;base64,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>

[image4]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAVAAAAAwCAYAAABHayI3AAAN1ElEQVR4Xu2dh68VRRSH/QcIEYIRkYggakDU0AMCCoJIVUMVAZGiVAVsiB1RpBkLRelSpKoQQwcLvVkQK4Id7GBFsIx+E85m7rC7b+++9+CV8yU3d3a23N19u785c86ZeacZRVEUJRWn+RWKoihKMlRAFUVRUqICqiiKkhIVUEVRlJSogCqKoqREBVRRFCUlKqCKoigpUQFVFEVJiQqooihKSlRAFUVRUqICGsKWLVvM1Vdf7VcrilJMmDZtml8Vigqox48//mhWrFjhVyvFhDfeeMOUKFHCr07M888/71cFDB8+3Jx33nnmhx9+8FflCZy78PPPP5vOnTubw4cPO1ucGnbt2mVefvlle19nzpzprw74/PPP/apI/vrrL3P33Xebxx57zLz33nvm2LFjplWrVqZkyZL+pqnYt2+fOffcc/3qE1ABdRg7dqxp0qSJX60UM9IKKEKR076PP/54vgnoqFGjMpYffvjhrAV00aJFflWuadeunV91Ar///ntGA5AT3Oc+ffpk1HGMChUqZNTllk6dOvlVGaiAOpQqVcq88847frVSzOA5SMNVV12Vo4DSSOeHgC5duvQEAR05cmTWArpw4UK/Ktdce+21ftUJ3HXXXVkL6FtvveVXmxtvvNGvyhX0GF566SW/OkAF9DiffvqpefHFF/1qpRjiCigv6tSpU823335rKleuHNSXKVPGfPHFF9bq6dWrl63r379/VgLK8/bhhx/aMsebM2eOFRGsni5dupj9+/ebuXPnBsdEJOXc3nzzTSt2Q4YMsct0f30BZblDhw722Z49e3aO5wYLFizwq06AcwWunesBjn3TTTeZf/75x7Rp08bdPENAw64ZuE5XQP/44w/TqFEjc+TIEbs8YsSI4FpxAySxNMuWLWvOP/98W3700UfNN998Y8uvvvqqKV26tBVGznfo0KFm3rx57q4B7HP66af71QEqoMfp3bu3vZmK4gooIvTdd9/Z8qBBg8y///5ry2eccYb58ssvbVl85tkIKMJQpUqVoP6GG24wl112mS3T9cZnCO+//35wzK5du2Yc3xWqKAENO04cSQSUaxeeeOIJ+/3UU0+ZnTt32vL8+fOtO0MQAY27Zl9AYcqUKWbNmjW2zLWKACKgF198sbtpKIsX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>

[image5]: <data:image/png;base64,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>

[image6]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAUAAAAAxCAYAAACrmXB6AAAPuElEQVR4Xu2dh68UVRSH/QeMAaNB0IiAhSoqxdgLIliwIoIUxYYaCyggKgioCGqMBRSVYlQIoFKUCGJBimDvIjYUlGIFLCgiXPPd5Ax3z5vZnd23+2TZ8yWbN23vnN0385tzzzn37i7OMAyjQtlFbzAMw6gUTAANw6hYTAANw6hYTAANw6hYTAANw6hYTAANw6hYTAANw6hYTAANw6hYTACNvOjSpYvbsGGD3myUkAkTJuhNRpEwATRSs3XrVrd06VK//N1337mhQ4e6AQMGqKNKy6677upfxx57rN6VGmwfOXKkt78mee211wqy/eyzz3azZs3Sm40iYAJopGLt2rVut912y9h22mmnuSOPPDJjW7Hp3bu3W7hwYca2k08+OW8RiQP7S422HQqx/corr3Q33nij3mxUExNAIxWjRo1yZ555Zsa2Tp06lVQA//zzT3fAAQdUEZEOHToUJCIa7C8l2K9th0Jsf/HFF92ee+6pNxvVxASwjOnbt6+rW7eu71aNHz/eXXfdde6mm25y//77r+vcubM75phj3HnnnRcdT/eVm759+/Z+/5QpU/z2Dz/8MOpa8rrkkkui5c8//9wfwzJduJCzzjqrigByznPOOcctWbLEr0vbnLd///7ujDPO8AIWMn36dL+9R48e7rHHHvOi9/bbb2fY1LJly+h42kJEaI9lxKEQsD/koYce8vafeuqp3n5s57yc/9tvv/Xnw/bnnnsu433dunXz9mP7iBEjvP0HHXRQov3Yvnr16rztb9eund5kVBMTwDLmgw8+cLfffru/webNm+e3sdyrVy/3zz//+PWOHTtGxyOG7P/000+jeFSIiN8777zj6tWrl3Gjs2/FihXB0VUFkHigJEhq1arlvvrqK7d+/XrXtGlT17x5c/f777/7fVdddZUXabj55pvd/vvv75cffPBBt8cee7i2bdu6+fPnu0mTJvnzai8K0eA9f/zxh19HXIhP5ksogNhO1xr7OS/2Yzvnxn6+F+zHdj6LgP2bNm3yy9jOd4T9QDvadsD2Sy+91Nt/xRVXZIhjNvr06aM3GdXEBLDMeeWVVzKEjJtLBAWGDBkSidJbb73lRUbE4rDDDouOg+uvv97fxMSbRFAFzvHrr79mbAsFUDw9Yb/99nMDBw70y3hN4T48pffee88v77333pFIf/zxxxnHZRPA8DiWpb18CAWQNmbMmOGXEWfsF7D/xx9/9MvYHp4b+wW233nnndF6kgByXFJ72ZDv0ygeJoBlzoIFCzJuoGbNmmXc2MOGDXM//fSTX7711lv9sdzQdOnatGkTHSfg9Q0fPlxv9u/TXlYogBdeeGHijYxghV7OE0884d58802/fN9993nR3bZtm/dmQ+9KBJDPGKLb4xjEXaBLnQYtgO+//36wdzthrBDbw8+J/YQJsB/b161bF+3Dfm076O8i7nvTYQIgc20UFxPAMgcPI5cA/vDDD36Zbh0lFZs3b/brdNU2btyY0bWlq7f77ru7L774ItoGnOPnn3/O2BYK4A033BB7I4MWLG56KaeZOHGi91IbNmzou5daQEQA6aKKQJG91QIo7cE333wTLWdDC+Crr74a7N1ONgHEfjxY7A9th1AAw3Ih/V3EfW9s1wwaNEhvMqqJCWCZg8cW3kDciKH3wI0nAlenTh1/09PFo+tFAuX55593Y8eOjd6LyM2ePdu3ee+990btsP7SSy9F63DSSSdldKOvvfZa9+STT3pviO60CNbRRx/tEwPCww8/7LvuQAyN7PI999zjJk+e7N59993ouM8++yyy46ijjopibbR34IEHRsdxjLSHF3nKKadE+6655hq/P86rxX7xarGd47CfBwT2C5xPYpbYHn7f2I9nhv2h7YD92P733397+wVsT2pv2rRp/txx3d3wcxnFwQSwjMHrovvIDUQX9IUXXvDLvOhinnjiiX553333dQ888IB74403fKYVD0pECI8vfB8CQjJA1rmx4YgjjsiIb4WZY8kOb9myxe2zzz6+K/j666/7bYigHPfyyy+7888/33uiiDF1bYiG7JdX2BXFg6X+EGGA1q1bV2mPZWmPxAKfW8B+9odeHLbj/bL9kEMO8fZjO+/Hfr4j7McOOV+LFi38+bCddanJ0/YjdKH92I7Hh/1p2qMrT5IqjEEC9lkZTPGJFUD+MUbxkBhcOcMNeu655+rN1QYBHzduXLS+atUqH5vkhi8E4nEIfgieI13/YkOWG/uxGfiL0MbFVvOB7q/+rhFG/bmM6hMrgOHT8oQTToj+weIZCMSf5EmflgsuuMC3EQatS8WcOXPcXnvtFZWEIOzy2fAU8Jp0cW8ueIpTQ5cPcZlA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>

[image7]: <data:image/png;base64,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>

[image8]: <data:image/png;base64,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>
