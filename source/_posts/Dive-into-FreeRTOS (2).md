---
title: Dive into FreeRTOS (2)
date: 2024-12-25 12:44:20
tags:
---
# Interrupt Management 
For the principle of the ISR to be as quick as possible, the trigger of freeRTOS api and the implementation of the ISR functions are always separated. 
There are 3 mainstream usages:
### Deferred Interrupt Process
![image.png](https://cdn.jsdelivr.net/gh/TANG617/images/202412251123793.png)
The notion is to make the ISR related function higher priority.  Its advantage shows when the execution time is unknown, like for reading or storing data from peripheral.

### Binary / Counting Semaphores
For the Counting Semaphores, when ISR is triggered, a semaphore is pushed to a queue and the count is added. Then, the frozen task is unblocked and executing until the queue is cleared.
![image.png](https://cdn.jsdelivr.net/gh/TANG617/images/202412251130491.png)

Note that the datatype of the queue can be self-defined.
![image.png](https://cdn.jsdelivr.net/gh/TANG617/images/202412251139493.png)
When the ISR trigger, a semaphore is GIVEN through a ISR api and execute context switch. Then, a handler function is unblocked and TAKE the semaphore or decrease the semaphore before the ISR related process function.
Note that, the above procedure is pretty similar to the notifications management but in ISR.