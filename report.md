Name: John Le

1. First build main-race.c. Examine the code so you can see the (hopefully obvious) data race in the code. Now run helgrind (by typing valgrind --tool=helgrind ./main-race) to see how it reports the race.
    - Does it point to the right lines of code?  
    Yes, it points to line 16 and line 9 balance++;
    - What other information does it give to you?  
    Type of access (read or write), thread number responsible for conflicting access, stack trace, which locks were held.

2. What happens when you remove (e.g., comment out) one of the offending lines of code?  
    Helgrind does not report data race becauses there's no longer a race condition.

3. Add a lock around one of the updates to the shared variable. What does helgrind report?  
    Helgrind still reports a data race because one thread still writes without holding a lock. Partial locking doesn't fix the problem.

4. Now add locks around both. What does helgrind report?  
    Helgrind reports no errors.

5. Next examine main-deadlock.c. This code has a problem known as deadlock (which we discuss in much more depth in a forthcoming chapter). Based on this code,
    - Describe what a deadlock is.  
    Two or more threads waiting on a resource held by other thread, so neither make progress.
    - Why specifically does this code have a deadlock?  
    Thread 2 locks m1 and then tries to lock m2. Thread 3 locks m2 and then tries to lock m1. If the scheduler interleaves them so Thread 2 holds m1 and Thread 3 holds m2, each thread will block while waiting for the other lock.

6. Run helgrind on this code. What does it report?  
    Thread #3: lock order "0x10C040 before 0x10C080" violated  
    

7. Examine main-deadlock-global.c.
    - Does it have the same problem that main-deadlock.c has?  
    No
    - Why or why not?  
    Lock g prevents more than one thread being inside critical section at a time.
    - Should helgrind be reporting the same error?  
    No
    - What does this tell you about tools like helgrind?  
    helgrind just sees that locks can be acquired in different orders across threads, which sometimes results in false positives.

8. Look at main-signal.c. This code uses a variable (done) to signal that the child is done and that the parent can now continue. Why is this code inefficient? (what does the parent end up spending its time doing, particularly if the child thread takes a long time to complete?)  
    The parent wastes CPU cycles waiting and doing nothing.

9. Run helgrind on this program.
    - What does it report?  
    data race on done between worker writing it and main reading it and race between printf()s
    - Is the code correct?  
    It seems correct, I ran it a bunch a of times with no issues, but I guess the race on done could cause some issues I'm not aware of.

10. Now look at the slightly modified version of the code found in main-signal-cv.c. This version uses a condition variable to do the signaling (and associated lock).
    - Why is this code preferred to the previous version?  
    - Is it correctness, or performance, or both?  
    It better in both correctness and performace. Accesses to done are now protected by locks and parent calls Pthread_cond_wait which puts it to sleep until child signals it, freeing up resources for other processes.

11. Once again run helgrind on main-signal-cv. Does it report any errors?  
    No errors reported