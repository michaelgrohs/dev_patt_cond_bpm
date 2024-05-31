# Usage of this repository
This repository is supposed to help to reproduce the work proposed in the paper "Beyond Log and Model Moves in Conformance Checking: Discovering Process-Level Deviation Patterns"

The file "DevPatterns.ipynb" contains all code used to implement and evaluate the proposed approach. The function "discover_patterns" executes the full approach whereas the function "discover_patterns_wo_cf" executes the approach without adjusting the cost function.

The folder "Evaluation" contains all evaluation results displayed in the paper. This folder contains all event logs, to-be models, and resources to reproduce the benchmark. 

# Details on function findEquivSeq
As mentioned in the paper, we give more details on the function findEquivSeq, which checks whether a projection of the inverse moves of a sequence seq<sub>1</sub> can be detected in a candidate sequence seq'<sub>3</sub>. If so, it returns this projection seq<sub>3</sub> and an empty sequence otherwise. The functions' logic is illustrated below. 

```
FUNCTION findEquivSeq(seq1,seq'3,M^A):
    seq3=⟨⟩
    FOR move IN seq1:
        IF inverse(move) IN seq'3:
            seq3 ← inverse(move)
            seq1.delete(move)
        ELIF activity(move) IN exclusive_relation(M^A):
            exclusives=get_exclusives(activity(move),M^A)
            IF len(moves_on_exclusives(exclusives, seq'3)) >0:
                seq3 ← moves_on_exclusives(exclusives, seq'3)
                seq1.delete(move)
        ENDIF;
    IF len(seq1)>0:
        move_combinations=get_all_combinations(seq1)
        FOR combination IN move_combinations:
            IF activity(combination) IN exclusive_relation(M^A):
                exclusives=get_exclusives(activity(combination),M^A)
                IF len(moves_on_exclusives(exclusives, seq'3)) >0:
                    seq3 ← moves_on_exclusives(exclusives, seq'3)
                    seq1.delete(combination)
            ENDIF;
    ENDIF;
    IF len(seq1)=0:
        RETURN seq3
    ELSE:
        RETURN ⟨⟩
    ENDIF;

END.
```

The function first initializes seq<sub>3</sub>. Then, it iterates over all moves in seq<sub>1</sub> and checks whether there is the inverse of each move in seq'<sub>3</sub>. If so, a direct projection of the inverse of the move is found in seq'<sub>3</sub>, which is then added to seq<sub>3</sub> whereas the move itself is deleted from seq<sub>1</sub>. 
As mentioned in the paper, the moves in seq'<sub>3</sub> do not have to equal the inverse of seq<sub>1</sub> but can also be behaviorally equivalent to them. Therefore, if no direct projection of a move is found, we check whether there is an indirect projection of the inverse move in seq'<sub>3</sub>. This indirect projection exists if, given M^A, there are moves on activities in an exclusive relationship to the activity of the move under consideration in seq'<sub>3</sub> such that they are behavioral equivalent. If this is the case, the behavioral equivalent moves are added to seq<sub>3</sub> whereas the move itself is deleted from seq<sub>1</sub>. 
Last, it can be the case that combinations of moves in seq<sub>1</sub> are behaviorally equivalent to (a combination of) moves in seq'<sub>3</sub>. Thus, we check whether any combination of moves in seq<sub>1</sub> is behaviorally equivalent to moves on activities in an exclusive relationship to the activity of the move combination under consideration in seq'<sub>3</sub>. If so, we found an indirect projection of the move combination, which is deleted from seq<sub>1</sub> whereas the indrect projection is added to seq<sub>3</sub>.
If all moves are deleted from seq<sub>1</sub> in these iterations, a direct or indirect projection of all moves in seq<sub>1</sub> exists in seq'<sub>3</sub>. In that case, we have found a behaviorally equivalent sequnce seq<sub>3</sub>, which is returned. Otherwise, we return the empyt set.

# Benchmark 
We benchmark our approach against García-Bañuelos et. al. [1]. The approach is implemented as a standalone Java tool ProConformance and an Apromore plugin Compare. The plugin is not available anymore which is why we use an updated version of the ProConformance implementation. We note that the execution of the original version of the standalone tool was not possible, a problem also found by another study [2].
The provided "ProConformance.jar" is supposed to return log-level feedback which is not comparable to our approach aming for trace-level feedback. To obtain trace-level feedback comparable to our approach, we check which traces deviate and store an event log consisting of one trace only per deviating variant. Then, we apply the implementation to this event log and map the detected deviation patterns to the traces without adjusting the approach. To reproduce the results, use the file "garica_benchmark.ipynb".

We want to stress conceptual differences:
1. Different pattern types used:
    We consider the five patterns inserted, skipped, repeated, replaced, and swapped. In contrast to that, [1] returns deviation patterns using 16 templates of natural language statements based on an event structure-based alignment of a complete event log and a process model. Of these 16 templates, four represent a skip of an activity in different forms, swaps and insertions of activities are verbalized by two statements each, whereas replacements, and repetitions of single activities are indicated by one statement each. In addition, [1] verbalizes other mismatches between the complete event log and a process model, e.g., (i) concurrency of two activities in the process model while they are sequential in the event log or (ii) vice versa. While these additional types of deviating behavior are interesting when comparing an entire event log to a process model, they can be no problem for a single trace (e.g., (i)  just indicates that an allowed order of two activities is recorded) or not observable in a single trace (e.g., (ii) indicates concurrency in recorded behavior which cannot occur in one trace). Thus, only the ten statements referring to our set of five deviation patterns are relevant for the benchmark as only those can be found in the labelled data.
2. Deviations of single activities rather than process fragments:
    [1] only considers process-level deviations of single activities rather than process fragments for all five pattern types. Thus, it will return multiple process-level deviations in case there is, for example, a repetition of multiple activities. Since the labels of all nine process-level deviations also consider process-level deviations on process fragments, the approach returns more deviations than existent according to the labels. 
    We note that the patterns related to cycles do include process fragments. However, these patterns do not refer to the set of patterns we used and are consequently not discovered in our labelled data.
3. Mutual exclusivity of deviation patterns:
    We consider deviation patterns to be mutually exclusive in our optimal interpretation of the process-level deviations. For example, if a replacement of activity A through activity B is detected, we do not report the skip of A and the insertion of B in addition. The approach [1], however, does not determine one interpretation but rather returns all of them. Thus, if activity A is replaced by B, the skip of A and the insertion of B are part of the output in addition to the replacement itself. 
4. The same deviation multiple times:
    [1] does not account for the same process-level deviation in one trace multiple times. Thus, if an insertion is present twice, it will only uncover one of these insertions.

These differences have an impact on the performance of the benchmark.


# Detailed information about the evaluation results

As mentioned in the paper, the evaluation results of the 8 BINet logs are aggregated as averages. Here, you can find detailed information on per event log:
![plot](./Evaluation/figures/full_binet.png)

# Time needed for deviation pattern discovery 

To illustrate the time needed for the discovery of deviation patterns in different processes, we show the time needed for data sets used in the paper.

| Log              |                            | Time in seconds |        |                  
|                  |                            | Our approach    | [1]    |
|------------------|----------------------------|-----------------|--------|
| Hosseinpour & Jans |                          | 11.80           |   2.20 |
| BINet            | Gigantic                   | 29.54           |  49.94 |
|                  | Huge                       | 34.30           |  74.95 |
|                  | Large                      | 47.30           | 112.83 |
|                  | Medium                     | 26.94           |  60.56 |
|                  | P2P                        | 21.64           |  78.66 |
|                  | Paper                      | 22.55           |  60.97 |
|                  | Small                      | 25.24           |  93.09 |
|                  | Wide                       | 20.22           |  45.05 |
| BPIC 12          | A_                         | 2.87            |        |


# References
[1] García-Bañuelos, L., Van Beest, N., Dumas, M., La Rosa, M., Mertens, W.: Complete and interpretable conformance checking of business processes. Trans Softw Eng 44(3), 262–290 (2017)

[2] Fahland, D., Reproduction of Experimental Evaluations in Process Mining, Seminar TUE