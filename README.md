# Lab09
Lab09 FSM implementation for Kasey Kemp and Adam Billings.

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
    * Due to troubles with interpreting an exact definition of "train_present," we will be ommitting this invariant in the table.
        * This is due to variations in definition for "train_present" and whether this will be true for "train_approaching" and "train_departing"
* Train cannot depart while arms become raised. 
    * This implies that the train was present (actively crossing intersection) while the arms were raised.
    * ~(~arms_down ^ (northbound_depart V southbound_depart))
* Alarm cannot be off while train is approaching or present.
    * Must provide an audio queue for potentially very dangerous moving train.
    * ~(~alarm_on ^ (nourthbound_present V southbound_present))
* Barrier cannot be down while there is no oncomming or departing train and no train present.
    * Disrupts traffic unneccessarily.
    * ~(barrier_down ^ ~((nourthbound_present V southbound_present)))
* Arms down while alarm off
    * The alarm should occur first and stop after
    * ~(~alarm_on ^ ~arms_down)



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
Below are the initial FSM graphs.
![Initial FSM Graph 1](/resources/images/InitialFSM(1).png "Initial FSM Graph")

## Prove it.
How would you go about proving that your model is correct?

1. Fill in this table (you can copy the markdown into your docs).
1. The first four variables (after `number`) are booleans, you should have 16 possible states.
1. Create your own list of invariants. Use the second table below; start the `number` column at number 16 (because the `number` column of the first table ends at `15`). Add more invariants as needed.
1. For each row, mark "safety_hazard" with the number of the invariant it violates or leave it blank.
1. For each row, write down the number of the state your system will transition to on that event.
   1. If an event violates an invariant (e.g. event not allowed in that state), write down the number of the invariant.
   1. If an event has no effect, you can stay in the same state.

| number | arms_down | alarm_on | northbound_present | southbound_present | north_approach | south_approach | north_depart | south_depart | time-elapsed | safety_hazard |
|--------|-----------|----------|--------------------|--------------------|----------------|----------------|--------------|--------------|--------------|---------------|
| 0      | 0         | 0        | 0                  | 0                  | 6              | 5              | 16           | 16           | 0            |               |
| 1      | 0         | 0        | 0                  | 1                  |                |                |              |              |              | 18            |
| 2      | 0         | 0        | 1                  | 0                  |                |                |              |              |              | 18            |
| 3      | 0         | 0        | 1                  | 1                  |                |                |              |              |              | 18            |
| 4      | 0         | 1        | 0                  | 0                  | 6              | 5              | 16,20        | 16,20        | 0            |               |
| 5      | 0         | 1        | 0                  | 1                  | 7              | 17             | 16,20        | 20           | 13           |               |
| 6      | 0         | 1        | 1                  | 0                  | 17             | 7              | 20           | 16,20        | 14           |               |
| 7      | 0         | 1        | 1                  | 1                  | 17             | 17             | 20           | 20           | 15           |               |
| 8      | 1         | 0        | 0                  | 0                  |                |                |              |              |              | (19), 21      |
| 9      | 1         | 0        | 0                  | 1                  |                |                |              |              |              | 18, 21        |
| 10     | 1         | 0        | 1                  | 0                  |                |                |              |              |              | 18, 21        |
| 11     | 1         | 0        | 1                  | 1                  |                |                |              |              |              | 18, 21        |
| 12     | 1         | 1        | 0                  | 0                  |                |                |              |              |              | (19)          |
| 13     | 1         | 1        | 0                  | 1                  | 15             | 17             | 16           | 4            | 13           |               |
| 14     | 1         | 1        | 1                  | 0                  | 17             | 15             | 4            | 16           | 14           |               |
| 15     | 1         | 1        | 1                  | 1                  | 17             | 17             | 13           | 14           | 15           |               |

| number | invariant                                                                              |
|--------|----------------------------------------------------------------------------------------|
| 16     | ~(northbound_depart ^ ~northbound_present V southbound_depart ^ ~southbound_present)   |
| 17     | ~(northbound_approach ^ northbound_present V southbound_approach ^ southbound_present) |
| 18     | ~(~alarm_on ^ (nourthbound_present V southbound_present))                              |
| 19     | ~(barrier_down ^ ~((nourthbound_present V southbound_present)))                        |
| 20     | ~(~arms_down ^ (northbound_depart V southbound_depart))                                |
| 21     | ~(~alarm_on ^ ~arms_down)                                                              |

## Specification vs. implementation
1. Start drawing an FSM using the table you just made.
1. Label each FSM state with the number in the table.
    1. Some FSM states may have multiple table numbers.
1. If an event violates an invariant that represents an impossible event or operating assumption, leave it off your machine.
    1. It's important to account for behavior that could occur outside your expectations, but we need to maintain a level of abstraction. Getting struck by lightning is possible, but not something you plan for.

Is your new FSM equivalent to the FSMs from the previously steps?
