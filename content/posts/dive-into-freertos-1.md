---
title: Dive into FreeRTOS (1)
description: FreeRTOS notes on task management, kernel data types, and heap allocation strategies.
publishDate: 2024-08-28T16:44:20+08:00
updatedDate: "2026-07-19T00:00:00+08:00"
tags:
  - FreeRTOS
---

# FreeRTOS Memory, Task, and Interrupt Management

## Kernel source files

A minimal FreeRTOS kernel build normally includes `tasks.c`, `queue.c`, `list.c`, a platform-specific `portable/<compiler>/<architecture>/port.c`, and one memory manager such as `heap_4.c`. Include files such as `timers.c`, `event_groups.c`, and `stream_buffer.c` when their corresponding features are enabled.
![image.png](https://cdn.jsdelivr.net/gh/TANG617/images/202408281646751.png)

### Data types

`TickType_t` stores the kernel tick count used for timeouts and delays. Its width is controlled by the port and kernel configuration; a narrower type wraps sooner and is not a universal recommendation for smaller processors.

`BaseType_t` is defined as an efficient signed type for the target architecture. It is typically 32-bit on a 32-bit architecture, 16-bit on a 16-bit architecture, and 8-bit on an 8-bit architecture.

### Variable and function names

The kernel's naming convention uses prefixes such as `c` for `char`, `s` for `int16_t`, `l` for `int32_t`, and `x` for `BaseType_t` or other non-standard types. An unsigned variable also receives a `u` prefix, while a pointer receives a `p` prefix. For example, a `uint8_t` variable uses `uc`, a pointer to `char` uses `pc`, and `v` denotes `void`.

## Heap memory management

### Why not use the standard `malloc()` and `free()` directly?

Depending on the C library and target, they may be unavailable, relatively large, non-thread-safe, non-deterministic, or prone to fragmentation.

### Use `pvPortMalloc()` and `vPortFree()`

FreeRTOS provides five sample memory-manager implementations in `heap_1.c` through `heap_5.c`.
The kernel calls `pvPortMalloc()` when it needs dynamic memory and `vPortFree()` when the selected allocator supports deallocation. Their interfaces resemble the standard C library's `malloc()` and `free()` functions.

#### `heap_1`

![image.png](https://cdn.jsdelivr.net/gh/TANG617/images/202408281713141.png)

- supports allocation but not deallocation
- objects can be created after the scheduler starts, provided they are never deleted and enough heap remains

#### `heap_2`

![image.png](https://cdn.jsdelivr.net/gh/TANG617/images/202408281954329.png)

- retained for backward compatibility and not recommended for new designs
- uses a best-fit algorithm to select the smallest free block that satisfies an allocation
- suitable only for limited allocation patterns because it does not coalesce adjacent free blocks

#### `heap_3`

- wraps the standard library's `malloc()` and `free()`; the linker configuration defines the available heap, and `configTOTAL_HEAP_SIZE` has no effect

#### `heap_4`

![image.png](https://cdn.jsdelivr.net/gh/TANG617/images/202408281957948.png)
_C_ shows the state after a FreeRTOS queue has been created. _D_ shows the state after the application has called `pvPortMalloc()` directly.

- the array is statically declared, and dimensioned by configTOTAL_HEAP_SIZE
- `heap_4` coalesces adjacent free blocks into a larger block, which reduces external fragmentation.
- Note that, unlike when heap_2 was demonstrated, the memory freed when the TCB was deleted, and the memory freed when the stack was deleted, does not remain as two separate free blocks, but is instead combined to create a larger single free block.
- Its execution time depends on the state of the free list, so it should not be treated as strictly constant-time.

Start address for heap_4 can be set.

#### `heap_5`

`heap_5` is almost identical to `heap_4`, except that:

- it can allocate memory from multiple non-contiguous regions instead of one statically declared array;
- `vPortDefineHeapRegions()` must be called before creating any kernel object.

## Task management

![image.png](https://cdn.jsdelivr.net/gh/TANG617/images/202408291133656.png)

- FreeRTOS tasks must not be allowed to return from their implementing function in any way—they must not contain a ‘return’ statement and must not be allowed to execute past the end of the function.
- The FreeRTOS scheduler is the only entity that can switch a task in and out.
- with time slicing enabled, the scheduler can rotate between ready tasks of equal priority on tick interrupts

### States

![image.png](https://cdn.jsdelivr.net/gh/TANG617/images/202408291140840.png)

#### The Blocked State

Blocked tasks wait for a time-related event or a synchronization event from another task or interrupt.

#### The Suspended State

Suspended tasks are unavailable to the scheduler. `vTaskSuspend()` enters this state, while `vTaskResume()` or `xTaskResumeFromISR()` leaves it.

#### The Ready State

Ready tasks can run but are not currently in the Running state.

### Scheduling policies

#### Prioritized Pre-emptive Scheduling with Time Slicing

![image.png](https://cdn.jsdelivr.net/gh/TANG617/images/202408291152518.png)
![image.png](https://cdn.jsdelivr.net/gh/TANG617/images/202408291200867.png)
Time slicing lets equal-priority ready tasks take turns running.

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

## Interrupt management

Keep interrupt service routines (ISRs) short: acknowledge or capture the event, then defer longer processing to a task. FreeRTOS provides ISR-safe APIs such as `xTaskNotifyFromISR()`, `xSemaphoreGiveFromISR()`, and `xQueueSendFromISR()`.

### Deferred interrupt processing

Give the deferred-handler task a sufficiently high **task priority** so that it can run promptly after the ISR unblocks it. Task priority and hardware interrupt priority are separate concepts.

![Deferred interrupt processing](https://cdn.jsdelivr.net/gh/TANG617/images/202412251123793.png)

### Semaphores, queues, and task notifications

A binary semaphore records whether an event is available; a counting semaphore records how many events are pending. FreeRTOS implements semaphores using its queue infrastructure, but giving a semaphore does not copy an application payload.

Use a queue when the ISR must transfer fixed-size data to a task. Use a task notification when a single task is the recipient and the lighter-weight notification semantics are sufficient.

After an ISR unblocks a higher-priority task, request a context switch with the port-specific macro, commonly `portYIELD_FROM_ISR()`.

![Counting semaphore used from an ISR](https://cdn.jsdelivr.net/gh/TANG617/images/202412251130491.png)
