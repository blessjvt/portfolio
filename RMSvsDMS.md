  
# RMS vs DMS

## **Flow Diagram of the Function Calls used:**

<img alt="RTOS-Project 2 drawio (2)" src="https://github.com/user-attachments/assets/958c1caf-b47c-4e0a-bf2e-497dcb63429d" />

# **Part 1 DMS Priority Assignment** 

The RMS implementation 

The two main functions of focus for the RMS implementation were vSchedulerPeriodicTaskCreate() and prvSetFixedPriorities().

vSchedulerPeriodicTaskCreate() handles the creation of a periodic task. This function is called in the main.ino file. After initializing all of the members, the important decision was determining what happens if schedSCHEDULING\_POLICY \== schedSCHEDULING\_POLICY\_RMS. The key implementation was to set the task's AbsoluteDeadline to be equal to the sum of the task's ReleaseTime and the task's RelativeDeadline. Within the same if statement, I set the task's LastWakeTime to the task's ReleaseTime.

prvSetFixedPriorities() handles the priority assignment of tasks. In this function, I made sure to check for the shortest period in order to assign it the highest priority. I used a nested if statement. The first set of conditions checks if a task is in use and if the priority has not been set. The second condition checks if the task’s period is greater than the previous shortest period and less than the current shortest period. If so, the task’s period is set as the xShortest, and its pxShortestTaskPointer is assigned the task’s period value. Then, the shortest task pointer is set to the highest priority, and the highest priority is decremented to look for the next shortest task’s period.

**Priority and Missed Deadline Check:**

 /\* Task Set 1 \*/ 

<img alt="Screenshot 2025-10-30 at 2 03 33 PM" src="https://github.com/user-attachments/assets/ab45c3d1-a249-4202-b05e-917aa37e8024" />

<img alt="Screenshot 2025-10-30 at 2 04 18 PM" src="https://github.com/user-attachments/assets/b074931a-e90b-407a-a62d-79aeb5f60660" />


The screenshot confirms a working script. The priority is set correctly, as Task 2 has the lowest period and therefore the highest priority. The list continues in this manner.

The missed deadline is handled correctly. Once detected, the task is recreated about one period after its original schedule, rather than starting immediately. This is achieved by:

   pxTCB\-\>xLastWakeTime \= xTaskGetTickCount();  
   pxTCB\-\>xReleaseTime \= pxTCB\-\>xLastWakeTime \+ pxTCB\-\>xPeriod;  
   pxTCB\-\>xAbsoluteDeadline \= pxTCB\-\>xRelativeDeadline \+ pxTCB\-\>xReleaseTime;

Inside the static void prvDeadlineMissedHook( SchedTCB\_t \*pxTCB, TickType\_t xTickCount )  
function.

# **Part 2 DMS Priority Assignment** 

The DMS implementation 

**Priority and Missed Deadline Check:**

The DMS implementation was simple once I got the RMS implementation working. The only key difference between the DMS and the RMS is what dictates the priority order. Instead of checking the period, the deadline is checked. The task with the lowest deadline is set as the highest priority. This was done by changing a few lines in prvSetFixedPriorities(), as well as altering the .h file for minor details, such as changing \_RMS to \_DMS.

In prvSetFixedPriorities(), I made sure to change the check from the shortest period to the shortest deadline. After each condition is met, the xShortest pointer is set to the RelativeDeadline. From that point on, the function follows the same logic as RMS.

<img alt="Screenshot 2025-10-30 at 2 04 45 PM" src="https://github.com/user-attachments/assets/3c9ccf18-9df5-436a-b602-b9337910a27d" />

<img alt="Screenshot 2025-10-30 at 2 05 14 PM" src="https://github.com/user-attachments/assets/a157eb74-85d3-4e98-9981-2f3b9ea583f7" />

# **Part 3 \- Periodic Tasks**

	  
**TASK SET**

<img alt="Screenshot 2025-10-30 at 2 05 53 PM" src="https://github.com/user-attachments/assets/b76d3d0e-63ff-4dbc-80a9-144e8384062a" />

<img alt="Screenshot 2025-10-30 at 2 06 09 PM" src="https://github.com/user-attachments/assets/34722e15-9bf7-4ae2-b388-3bf187a9a39f" />

**TASK SET 2**

<img alt="Screenshot 2025-10-30 at 2 06 29 PM" src="https://github.com/user-attachments/assets/b18e4088-07a3-404d-9f4b-1d414f932e30" />
<img alt="Screenshot 2025-10-30 at 2 06 41 PM" src="https://github.com/user-attachments/assets/6df0297c-d598-420c-889f-71e2c2f846d4" />

**Discussion:**

In DMS tasks with a shorter deadline will run at a higher priority. In RMS, tasks with a shorter period will always preempt those with longer periods despite deadline values. From my  
observation, the Deadline Monotonic Scheduler reduces the number of missed deadlines if tasks have deadlines that are not equal to their periods. In Rate Monotonic Scheduler, missed deadlines are more likely to happen if tasks have tight deadlines compared to their periods.

# **Part 4 Overhead**

My approach was to first determine what would be most affected by overhead. I concluded that release time would be the issue. Initially, I planned to track the latency or CPU performance each time the scheduler is called, using deadline misses as a measure. Overhead impacts release time, which in turn affects deadline misses. A task might miss its deadline due to excessive overhead, not necessarily because of programming errors.

To address this, I updated the absolute deadline based on the adjusted release time. This was done by checking if the release time exceeded the scheduler overhead and adjusting the absolute deadline accordingly. If the overhead is huge, the absolute deadline is set to the relative deadline to prevent underflow.  Based on the images below, there is less overhead under DMS than RMS. This makes sense since there are less deadline misses under DMS than RMS, since priorities are assigned based on the deadlines and not the periods. The more deadline misses there are, the more overheard there is because the scheduler has to complete more deadline checks, deleting tasks, and recreating tasks.

**TASK SET 1**

<img alt="Screenshot 2025-10-30 at 2 08 15 PM" src="https://github.com/user-attachments/assets/334499db-a57e-4019-ad63-19cdc90619d7" />

<img alt="Screenshot 2025-10-30 at 2 08 32 PM" src="https://github.com/user-attachments/assets/bbab7d70-3c6a-4810-b40f-d7c6afc65fe5" />

**TASK SET 2**  
<img alt="Screenshot 2025-10-30 at 2 08 46 PM" src="https://github.com/user-attachments/assets/d64729f3-6ed3-47fe-9ffd-3fb0b7b53497" />

<img alt="Screenshot 2025-10-30 at 2 09 05 PM" src="https://github.com/user-attachments/assets/c51aeb92-edd2-4155-9bc3-18cf3933f40d" />
