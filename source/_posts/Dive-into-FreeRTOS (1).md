---
title: Dive into FreeRTOS (1)
date: 2024-08-28 16:44:20
tags:
---
# Memory and Task Management
## Chapter 1
### File
Only 3 files are necessary for the most basic FreeRTOS: `task.c`  `list.c`  `queue.c`. As for porting, additional `timer.c` `heap_n.c`  and `prtable/XX` are necessary. 
![image.png](https://cdn.jsdelivr.net/gh/TANG617/images/202408281646751.png)

###  Data Types
`TickType_t` is used for storing tick count to measure time. 16bits is recommended to use in 16bit and 8bit system. There is no reason to use a 16-bit type on a 32-bit architecture.
`BaseType_t` is always defined as the most efficient data type for the architecture.Typically, this is a 32-bit type on a 32-bit architecture, a 16-bit type on a 16-bit architecture, and an 8-bit type on an 8-bit architecture.

###  Variable Names and Function Names
‘c’ for char, ‘s’ for int16_t (short), ‘l’ int32_t (long), and ‘x’ for BaseType_t and any other non-standard types (structures, task handles, queue handles, etc.).  If a variable is unsigned, it is also prefixed with a ‘u’. If a variable is a pointer, it is also prefixed with a ‘p’. For example, a variable of type uint8_t will be prefixed with ‘uc’, and a variable of type pointer to char will be prefixed with ‘pc’. 'v' stands for void.

## Chapter 2: Heap Memory Management
###  malloc and free are not that suitable
1. not always available on mcu
2. large implementation
3. rarely thread -safe
4. not deterministic in time
5. fragmentation

### Use pvPortMalloc() and vPortFree() instead
5 examples of them are defined in `heap_x.c`.
When FreeRTOS requires RAM, instead of calling malloc(), it calls pvPortMalloc(). When RAM is being freed, instead of calling free(), the kernel calls vPortFree(). pvPortMalloc() has the same prototype as the standard C library malloc() function, and vPortFree() has the same prototype as the standard C library free() function.

#### Heap1
![image.png](https://cdn.jsdelivr.net/gh/TANG617/images/202408281713141.png)

- only create tasks and other kernel objects before the scheduler has been started
- the task or kernel object should never be deleted

#### Heap2
![image.png](https://cdn.jsdelivr.net/gh/TANG617/images/202408281954329.png)
- be used for backward compatibility, but its use is not recommended for new designs.
- The best fit algorithm ensures that pvPortMalloc() uses the free block of memory that is closest in size to the number of bytes requested.
- Heap_2 is suitable for an application that creates and deletes tasks repeatedly, provided the size of the stack allocated to the created tasks does not change.
#### Heap3
- uses the standard library malloc() and free() functions, so the size of the heap is defined by the linker configuration, and the configTOTAL_HEAP_SIZE setting has no affect.

#### Heap4
![image.png](https://cdn.jsdelivr.net/gh/TANG617/images/202408281957948.png)
*C* shows the situation after a FreeRTOS queue has been created.
*D* shows the situation after pvPortMalloc() has been called directly from application

code, rather than indirectly by calling a FreeRTOS API function.
- the array is statically declared, and dimensioned by configTOTAL_HEAP_SIZE
- heap_4 combines (coalescences) adjacent free blocks of memory into a single larger block, which minimizes the risk of memory fragmentation.
- Note that, unlike when heap_2 was demonstrated, the memory freed when the TCB was deleted, and the memory freed when the stack was deleted, does not remain as two separate free blocks, but is instead combined to create a larger single free block.
- Heap_4 is not deterministic, but is faster than most standard library implementations of malloc() and free().

Start address for heap_4 can be set.

#### Heap5
Almost identical as heap_4 but:
- heap_5 can allocate memory from multiple and separated memory spaces, instead of single statically declared array.
- When heap_5 is used, vPortDefineHeapRegions() must be called before any kernel objects (tasks, queues, semaphores, etc.) can be created.

## Chapter 3: Task Management
![image.png](https://cdn.jsdelivr.net/gh/TANG617/images/202408291133656.png)

- FreeRTOS tasks must not be allowed to return from their implementing function in any way—they must not contain a ‘return’ statement and must not be allowed to execute past the end of the function.
- The FreeRTOS scheduler is the only entity that can switch a task in and out.
- time slice: the scheduler itself must execute at the end of each time slice


### State

![image.png](https://cdn.jsdelivr.net/gh/TANG617/images/202408291140840.png)

#### The Blocked State
a sub-state of not-running
- time-related events, at delay moment
- synchronization events, waiting for another task or interrupt

#### The Suspended State
a sub-state of not-running
Tasks in the Suspended state are not available to the scheduler. The only way into the Suspended state is through a call to the vTaskSuspend() API function, the only way out being through a call to the vTaskResume() or xTaskResumeFromISR() API functions. Most applications do not use the Suspended state.

#### The Ready State
They are able to run, and therefore ‘ready’ to run, but are not currently in the Running state.

### Scheduling Policy
#### Prioritized Pre-emptive Scheduling with Time Slicing
![image.png](https://cdn.jsdelivr.net/gh/TANG617/images/202408291152518.png)
![image.png](https://cdn.jsdelivr.net/gh/TANG617/images/202408291200867.png)
Time slicing make Task2 and Idle task switch in turns at the time slice entry.

![image.png](https://cdn.jsdelivr.net/gh/TANG617/images/202408291203716.png)
When configIDLE_SHOULD_YIELD is set to 1, the task selected to enter the Running state after the Idle task does not execute for an entire time slice, but instead executes for whatever remains of the time slice during which the Idle task yielded.
#### Prioritized Pre-emptive Scheduling (without Time Slicing)
![image.png](https://cdn.jsdelivr.net/gh/TANG617/images/202408291205077.png)
- fewer task context switches
- tasks of equal priority receiving greatly different amounts of processing time

#### Co-operative Scheduling
![image.png](https://cdn.jsdelivr.net/gh/TANG617/images/202408291209751.png)
- Tasks are never pre-empted
- Time slicing cannot be used




