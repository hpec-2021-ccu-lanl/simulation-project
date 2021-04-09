# simulation-project

make sure git is installed as well as docker and that they are working correctly<br>
then type<br>
`git clone https://github.com/sc-2021-ccu-lanl-statistics/simulation-project.git`<br>
`cd simulation-project`<br>



readme.txt

```
To run a single simulation and see it work, one only needs to run the following commands:
    docker build . -t simulation
    docker create --name test_suite -t simulation
    docker start -i test_suite
    python3 run_simulation.py -i 1_simulation.config -o /home/sim/experiments/1_sim
    
This runs a simulation based on Workload2, WL2.csv.  It runs 2 simulations; both with
1490 nodes and 30,000 simulated jobs.  The first simulation is a baseline simulation
with a system mean time between failure of 24 hours (i.e. a failure rate lambda_good = 1/24).
The second simulation is 5x worse (lambda_bad = 5*lambda_good). This would correspond to
Table 4 of the paper, rwo 2, column 3.

After the simulation, at the very bottom is the normalized makespan.  It should be close to 1.31.
This is saying that at 5x worse failures than baseline, the makespan is around 31% worse.

You may get some numbers that are not that close to 1.31 since the failures are random across
nodes and time, as described in the paper. Due to this, we treated this 
as a Monte Carlo simulation.  The results presented in the paper (Table 4) 
are averaged across 1500 runs to determine the average makespan for the baseline system, and
also an additional 1500 runs to determine the average makespan for the less reliable system for a given workload. 
Using these two averages we compute the average normalized makespans for each of the scenarios
presented in Table 4 of the paper, as the ratios of the average makespans for the less reliable 
systems to the average makespans for the given baseline systems.

If one wants to run the simulation again you must specify a different output folder.
    python3 run_simulation --output ~/experiments/___name___

If one wants to use different parameters you must specify a different config file
    python3 run_simulation --config sample.config --output ~/experiments/___name___
    
The config file has several moving parts, but for our purposes here, we will highlight a couple things you can do.
We will start by addressing the config file that is in ~/basefiles/
Notice it starts with the name of the overall experiment, "test". Most sweeps over parameters have 3 ways of
being performed:

as values in a list:             "range":[1,2,3]                       This chooses one value in the list after the other 

as a range with min,max,step:    "min":1,"max":3,"step":1              This starts at min, ends at max (inclusive)
                                                                      and increases or decreases by step each time

as the "formula"                 "formula":"1 * i","range":[500,600]  trival example as i is only multiplied by one.
combined with both of the                                             But the idea is that you can use a variable "i" 
above methods                                                         that the range or min/max/step replaces it with
 

1_simulation.config 
 
{"test":{
             "input": {
                                                            
                    "node-sweep":{
                              "range":[1490]
                    },
                    "SMTBF-sweep":{
                                   "compute-SMTBF-from-NMTBF":true, 
                                   "formula":"128736000 * (1/i)", "range":[1,5]
                      },
                      "checkpoint-sweep":{
                                    "range":["optimal"]
                       },
                      "performance-sweep":{
                                    "range":[1.0]
                       },
                      "checkpointing-on":true, "seed-failures":true, 
                      "synthetic-workload":{
                        "number-of-jobs":30000,
                                                               
                        "number-of-resources":"/home/sim/basefiles/workload_types/wl2.csv:0:csv", 
                        "duration-time":"/home/sim/basefiles/workload_types/wl2.csv:1:h:csv", 
                        "submission-time":"0:fixed", 
                        "wallclock-limit":-1, "dump-time":"3%", 
                        "read-time":"2%"
                      }
                },
              "output": {
                        "AAE":true, "avg-makespan":1
              }
    }
 }


We do not cover every option here, but three typical parameters include the SMTBF-sweep, the workload, as
well as the output.

SMTBF
-----
It starts out saying our formula is based on a node mean time between failure, not a system mean time
between failure.
So it must compute the system mean time between failure. Next, the node mean time between failure is in
seconds, so this is
1490 nodes * 24 hr.*3600seconds/hr  And we want the baseline(1) and 5x worse failures (5).
If you want to find out the same information as before but with, say 2x worse failures just change the
"range":[1,5] to "range":[1,2]  .  run_simulation does some stuff automatically, so it is expecting
baseline ("range":[1] )
to be first in the list.

Workload
---------
If you want to compare and contrast the impact of using different workloads, edit the 
"number-of-resources" and "duration-time" file to use, say wl3.csv instead.  It is currently at wl2.csv.
You may want to change the number of jobs used (pulled) from each workload to accomodate the different types of workloads.
For example: a workload that is full of primarily small, short jobs will need more jobs in a given simulation.
These are the numbers we use for amount of jobs:
wl1 50,000
wl2 30,000
wl3  5,000
wl4  3,000
wl5  1,000
wl6    500


Output
--------
AAE is the Average Application Efficiency.  This metric is not used/addressed in this paper submission but was
of interest to us for studying the impact of checkpointing intervals, dump times and restart times. It is also produced after every simulation.
The avg-makespan, on the other hand, controls how many runs of each to do.  We would recommend at least 20, but keep in
mind we performed 1500 runs in order to ensure that our averages converged.


Running With New Parameters
-----------------------------

After making the above changes to the included 1_simulation.config file, make sure you have
the file in correct json (commas,colons,braces, etc...) and save as another file.  So for example,
to keep everything the same except the avg-makespan (lets bump it to 20) then save as
20_simulation.config and run it using:
    python3 run_simulation.py --config 20_simulation.config --output ~/experiments/20_simulation



```
