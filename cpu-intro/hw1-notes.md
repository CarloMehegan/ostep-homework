#  my notes on hw1
(should i call this hw 4 because it's chapter 4? probably)

9/4/2025

first of all, this is just the homework that he gave us for cloud computing at the start of the semester and it was hard! did reading the chapter equip me to work with this tool better? i don't think so. maybe looking at the full thing instead of just the parts he took from it will be better.

ok no he gave us the full readme too. i'm looking back at my notes now. he just linked to the same repo i'm in right now.

## part 1. actually understanding cloud computing hw 1

Download your own copy of [process-run.py](https://github.com/remzi-arpacidusseau/ostep-homework/blob/master/cpu-intro/process-run.py) . (also take a look at the README) Run process-run.py with these flags: 

```
python3 process-run.py -l 4:100,1:0
```

These flags specify one process with 4 instructions (all to use the CPU), and one that simply issues an I/O and waits for it to be done.

(a) How long does it take to complete both processes? Use -c and -p to find out if you are right.
(b) Can you change the input parameters so that the first process yields the CPU before it is done executing? State the parameters you used and include a screen shot of the results.
(c) Why do kernels context switch to another thread or process when the currently running thread or process issues a blocking I/O? (The I/O could be disk or network activity).

Ok for a. let's try running `process-run.py -l 4:100,1:0 -c -p`

```bash
carlo@pop-os:~/Documents/CSCI420 Assignments/hw1$ python3 process-run.py -l 4:100,1:0 -c -p
Time        PID: 0        PID: 1           CPU           IOs
  1        RUN:cpu         READY             1          
  2        RUN:cpu         READY             1          
  3        RUN:cpu         READY             1          
  4        RUN:cpu         READY             1          
  5           DONE        RUN:io             1          
  6           DONE       BLOCKED                           1
  7           DONE       BLOCKED                           1
  8           DONE       BLOCKED                           1
  9           DONE       BLOCKED                           1
 10           DONE       BLOCKED                           1
 11*          DONE   RUN:io_done             1          

Stats: Total Time 11
Stats: CPU Busy 6 (54.55%)
Stats: IO Busy  5 (45.45%)
```

Let's break down the command. -l is when we specify our list of processes, -c -p gives us more output data.

We are inputting 4:100 and 1:0 as our processes. 4 and 1 represent how many "instructions/tasks/idk what to call them" our process will take. the 100 and 0 represent what percent of those tasks will be CPU instructions. The rest will be I/O.

I didn't really get what I/O meant when I first did this, but the chapter actually did help. It means something like a user prompt, like `input()` in python where we are just waiting for input. Of course this is a simulation so we don't actually wait for a user, but it simulates this by waiting for 5 CPU cycles, which we know because the --iolength flag defaults to 5.

So, a CPU task takes 1 CPU cycle but needs the CPU to be free. An I/O task takes 5 (6?) CPU cycles worth of waiting but doesn't require any CPU processing.

We only have 1 CPU, so the simulation here follows this protocol (I think):
- Pick a process to run
- Keep running that process' CPU tasks unless it hits an I/O
- Switch to another process to avoid idling during I/O
- Continue until all processes are complete

b. All we have to do is change the CPU instruction chance from 100% to less than that. This gives us the chance to get the process blocked, moving to the next process.

```bash
(DATA303-dash) carlo@pop-os:~/Documents/GitHub/ostep-homework/cpu-intro$ python process-run.py -l 3:50,1:0 -c -p
Time        PID: 0        PID: 1           CPU           IOs
  1         RUN:io         READY             1          
  2        BLOCKED        RUN:io             1             1
  3        BLOCKED       BLOCKED                           2
  4        BLOCKED       BLOCKED                           2
  5        BLOCKED       BLOCKED                           2
  6        BLOCKED       BLOCKED                           2
  7*   RUN:io_done       BLOCKED             1             1
  8*        RUN:io         READY             1          
  9        BLOCKED   RUN:io_done             1             1
 10        BLOCKED          DONE                           1
 11        BLOCKED          DONE                           1
 12        BLOCKED          DONE                           1
 13        BLOCKED          DONE                           1
 14*   RUN:io_done          DONE             1          
 15        RUN:cpu          DONE             1          

Stats: Total Time 15
Stats: CPU Busy 7 (46.67%)
Stats: IO Busy  11 (73.33%)
```

In the above we see that PID (Process ID) 0 runs first, but the first task is an io. It switches in the next cycle to PID 1, unfortunately also an io. The first process' io finishes first so it runs the next task, another io. Now we get to switch back to PID 1. Notice on cycle 8 it says "READY", which means this process had to wait an extra cycle while we served PID 0. On cycle 9 we "process" the io I guess and now PID 1 is done. Then we wait until PID 0's io finishes, and it when it does we process that, then we finish PID 0's last task which luckily is a cpu instruction.

This makes more sense to me now than it did last semester. Is it because I read a chapter about it and I was ready for it? Maybe because it's not W&M's first snow day in three years and I'm distracted by how nice it is outside? It is pretty nice outside right now though. Also when I first did this assignment I definitely just ran all these files through ChatGPT to try and understand it and I guess it helped me get the assignment done but I didn't walk away with a full understanding.

Also an interesting note, everything I just explained is literally in the README almost exactly how I said it

Ok time to do the actual homework

## part 2. the textbook homework

### Q1

Q: Run process-run.py with the following flags: `-l 5:100, 5:100`. What should the CPU utilization be (eg the percent of time the CPU is in use?) Why do you know this? Use the -c and -p flags to see if you were right.

Since they're both 100% cpu instructions, it should be 100% use. Maybe slightly less if it needs cycles to switch processes but I don't think so.
Ok it was 100% awesome

### Q2

Now run with these flags: 4:100, 1:0. How long does it take to complete both processes?

Ok I already know this one. 4 cycles for the first and 7 cycles for the second (run io, 5 blocked, complete io), finishes in 11.

### Q3

Switch the order of the processes. What happens now?

It should finish in 7 now. Run io, switch to PID 1 and finish it, wait for io, complete io.

```bash
(DATA303-dash) carlo@pop-os:~/Documents/GitHub/ostep-homework/cpu-intro$ python process-run.py -l 1:0,4:100 -c -p
Time        PID: 0        PID: 1           CPU           IOs
  1         RUN:io         READY             1          
  2        BLOCKED       RUN:cpu             1             1
  3        BLOCKED       RUN:cpu             1             1
  4        BLOCKED       RUN:cpu             1             1
  5        BLOCKED       RUN:cpu             1             1
  6        BLOCKED          DONE                           1
  7*   RUN:io_done          DONE             1          

Stats: Total Time 7
Stats: CPU Busy 6 (85.71%)
Stats: IO Busy  5 (71.43%)
```

### Q4

One important flag is -S, which determines how the system reacts when a process issues an I/O. With the flag set to SWITCH_ON_END, the system will not switch to another process while one is doing IO. What happens when you run `-l 1:0,4:100 -c -S SWITCH_ON_END`?

Well now we're back to square one it'll take 11 cycles
And after testing it did take 11 cycles

### Q5

Now run the same but with the behavior set to switch to another process when one is WAITING on IO `-S SWITCH_ON_IO`. What happens now?

I mean isn't this the normal or is this different. yeah I ran it this is just Q3 again

### Q6

With `-I IO_RUN_LATER` when an IO completes the process that issued it is not necessarily run right away. Rather whatever was running at the time keeps running. What happens when you run this combination of processes? `python process-run.py -l 3:0,5:100,5:100,5:100 -S SWITCH_ON_IO -c -p -I IO_RUN_LATER`

Ok this is the default -I setting so it should run the PID 0's first IO, then switch to PID 1 and complete it, run PID 0's second IO, etc. switching back and forth

Oh I was wrong! It did PID 0's first IO, switched to PID 1 and completed it, BUT then it switched to PID 2 and finished it, then PID 3 and finished it, and only then went back to PID 0. So not efficient. I'm guessing the next question fixes this

### Q7

Run the same process with `-I IO_RUN_IMMEDIATE` set, which immediately runs the process that issued the IO. Why might running a process that just completed an IO again be a good idea?

And now we get the behavior we wanted. Immediate switching seems to be a good idea when certain processes have more IO than others. In this case one process had all the IO waiting, so we should prioritize getting those done as fast as possible.

### Q8

Now run with some randomly generated processes using flags `-s 1 -l 3:50,3:50` or `-s 2 -l 3:50,3:50`, etc. See if you can predict how the trace will turn out. What happens when you use the flag `-I IO_RUN_IMMEDIATE` versus the flag `-I IO_RUN_LATER`? What happens when you use the flag `-S SWITCH_ON_IO` versus `-S SWITCH_ON_END`?

The -s flag sets the random seed

Ok cool I think I've sucked all of the value out of this simulation now, it's nice to revisit this and get a better grasp on it

First homework DONE