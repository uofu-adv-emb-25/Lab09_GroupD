# Lab09

### Railroad crossing problem


* There are two tracks, one going north and the other south. Trains on each track only run in one direction.
* At the crossing, a roadway (for cars, pedestrians, baby strollers, etc.) intersects both tracks. There is an arm which lowers to block crossing and an audible alarm which warns against crossing.
* Each track has two proximity sensors __approach__ and __depart__, placed on the tracks before and after the crossing respectively.
    * The two __approach__ sensors emit the events **northbound_approach** and **southbound_approach** when the first car of the train crosses the sensor (leading edge).
    * The two __depart__ sensors emit the events **northbound_depart** and **southbound_depart** when the last car of the train crosses the sensor (trailing edge).
* The system has an audible alarm, which can be in the state __on__ or __off__, and a barrier that is in either __lowered__ or __raised__ state to block the crossing.
* A timer emits an event **elasped** 10 seconds after the alarm starts ringing.
* When a train is approaching:
    1. The alarm begins clanging.
    1. After 10 seconds, the barrier lowers.
    1. The alarm continues to sound and the barrier remains lowered while a train is present.
    1. When no train is present the barrier raises.
    1. After 10 seconds the alarm stops.

## Invariants
1. With your partner, write down a set of invariants your system should have, with a specific emphasis on safety invariants.
1. You should be able to identify specific invariant conditions, that if violated, represent a unsafe condition or safety hazard.
1. Also consider invariants that are assumptions made about the environment or the scenario.
1. Write your invariants in the form a logical predicate, i.e. a statement that must be true.

Environment Invariants:
* Trains don't reverse (go in expected direction).
    * Potential for sensors to activate in unexpected order.
* Trains can't depart before approaching.
    * Expects that the arrive sensor works as intended and that there are initially no trains on the track. Messes with state transitions.
    * ~(northbound_depart ^ ~northbound_present V southbound_depart ^ ~southbound_present)
* Trains can't approach (in same direction) until after previous train has departed.
    * Trains can crash. This is an issue beyond the scope of this problem.
    * ~(northbound_approach ^ northbound_present V southbound_approach ^ southbound_present)
* System is in working condition.
    * This is an issue beyond the scope of the FSM.
* Nothing obstructs the tracks.
    * Obvious safetly concern outside of our interest.
* There are exactly two rails, northbound and southbound, and exactly one crossing.

System Invariants:
* Barrier cannot be up while trains are present.
    * Provides no visual queue and no physical barrier between objects and moving train.
    * ~(~arms_down ^ (nourthbound_present V southbound_present))
* Alarm cannot be off while train is approaching or present.
    * Must provide an audio queue for potentially very dangerous moving train.
    * ~(~alarm_on ^ (nourthbound_present V southbound_present))
* Barrier cannot be down while there is no oncomming or departing train and no train present.
    * Disrupts traffic unneccessarily.
    * ~(barrier_down ^ ~((nourthbound_present V southbound_present)))



## Varying invariants
Answer the question: does there exist a sequence of events, such that an invariant is not longer true?

1. Evaluate the [example FSM](example.pdf).
1. Find a counter-example sequence that makes an invariant false. Write it down.
* **Sequence**: 
    1. Start -> {idle}
    1. sb_approach -> {ringing, arms up}
    1. nb_approach -> {ringing, arms up}
    1. elapsed -> {ringing, arms down}
    1. sb_depart -> {ringing, arms up}
        * Invariant violated. (~arms_down ^ nb_present) violates ~(~arms_down ^ (nourthbound_present V southbound_present))

## Check your work
* We have checked ours and found problems.

## Prove it.
How would you go about proving that your model is correct?

1. Fill in this table (you can copy the markdown into your docs).
1. The first four variables (after `number`) are booleans, you should have 16 possible states.
1. Create your own list of invariants. Use the second table below; start the `number` column at number 16 (because the `number` column of the first table ends at `15`). Add more invariants as needed.
1. For each row, mark "safety_hazard" with the number of the invariant it violates or leave it blank.
1. For each row, write down the number of the state your system will transition to on that event.
   1. If an event violates an invariant (e.g. event not allowed in that state), write down the number of the invariant.
   1. If an event has no effect, you can stay in the same state.
  
**Update Oct 14** the `ringing` column should be something like "time elapsed" to reflect the 10-second delay. That way, it is an event related to the `alarm_on` state of the world (instead of being redundant).

| number | arms_down | alarm_on | northbound_present | southbound_present | north_approach | south_approach | north_depart | south_depart | ringing | safety_hazard |
|--------|-----------|----------|--------------------|--------------------|----------------|----------------|--------------|--------------|---------|---------------|
| 0      | 0         | 0        | 0                  | 0                  |                |                |              |              |         |               |
| 1      | 0         | 0        | 0                  | 1                  |                |                |              |              |         |               |
| 2      | 0         | 0        | 1                  | 0                  |                |                |              |              |         |               |
| 3      | 0         | 0        | 1                  | 1                  |                |                |              |              |         |               |
| 4      | 0         | 1        | 0                  | 0                  |                |                |              |              |         |               |
| 5      | 0         | 1        | 0                  | 1                  |                |                |              |              |         |               |
| 6      | 0         | 1        | 1                  | 0                  |                |                |              |              |         |               |
| 7      | 0         | 1        | 1                  | 1                  |                |                |              |              |         |               |
| 8      | 1         | 0        | 0                  | 0                  |                |                |              |              |         |               |
| 9      | 1         | 0        | 0                  | 1                  |                |                |              |              |         |               |
| 10     | 1         | 0        | 1                  | 0                  |                |                |              |              |         |               |
| 11     | 1         | 0        | 1                  | 1                  |                |                |              |              |         |               |
| 12     | 1         | 1        | 0                  | 0                  |                |                |              |              |         |               |
| 13     | 1         | 1        | 0                  | 1                  |                |                |              |              |         |               |
| 14     | 1         | 1        | 1                  | 0                  |                |                |              |              |         |               |
| 15     | 1         | 1        | 1                  | 1                  |                |                |              |              |         |               |

| number | invariant                                                                              |
|--------|----------------------------------------------------------------------------------------|
| 1      | ~(northbound_depart ^ ~northbound_present V southbound_depart ^ ~southbound_present)   |
| 2      | ~(northbound_approach ^ northbound_present V southbound_approach ^ southbound_present) |
| 3      | ~(~arms_down ^ (nourthbound_present V southbound_present))                             |
| 4      | ~(~alarm_on ^ (nourthbound_present V southbound_present))                              |
| 5      | ~(barrier_down ^ ~((nourthbound_present V southbound_present)))                        |

## Specification vs. implementation
1. Start drawing an FSM using the table you just made.
1. Label each FSM state with the number in the table.
    1. Some FSM states may have multiple table numbers.
1. If an event violates an invariant that represents an impossible event or operating assumption, leave it off your machine.
    1. It's important to account for behavior that could occur outside your expectations, but we need to maintain a level of abstraction. Getting struck by lightning is possible, but not something you plan for.

Is your new FSM equivalent to the FSMs from the previously steps?
# Model checking
In the lab, we tried several techniques by hand to verify a system model.
Even for a simple machine, the number of possible states grows exponentially.
This is called "state space explosion", and is the primary difficulty when checking models.
There are a wide variety of computer programs called "model checkers" that provide automation for searching state space.
You can also encode your model into a format that can be used with a tool like a SAT solver, which will prove the model is correct.
We started down that path with the state table, encoding each state into a binary number.
Often we will have a specification model, and the goal is to show equivalence with our implementation.

# Next steps
This lab is designed to give a taste for the topic of formal methods.
If you want more, there a courses offered that go into detail.

Priyank Kalla teaches a course on verification focusing on hardware logic.
ECE6715 Verification of Digital Circuits (Fall)

Ben Greenman teachs a course on formal verification of software.
CS6110 Software Verification (Spring)

Both classes are highly recommended.

# Reference implementation
No reference implementation is provided for this lab.