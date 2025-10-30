  
# OCPP vs ICPP

## **OCPP Implementation**

<img width="553" height="160" alt="Screenshot 2025-10-30 at 2 29 33 PM" src="https://github.com/user-attachments/assets/543e2019-072c-42f8-9e2b-7d8e2760b44b" />

Figure 1: Task Set Implemented 

**Implementation:**

The approach of implementing this was keeping the OCPP rules in mind:

* OCPP only changes priority if an actual block has occurred.  
* A task is Not allowed to lock a resource, if there are locked semaphores that could later block it or higher priority tasks  
* Thus, a task can be blocked (even) when attempting to lock a Free resource  
* A job J is allowed to enter a critical section only if its priority is higher than ceilings of all semaphores locked by jobs other than J

I created three functions to handle those requirements:

**/\* Initialize one shared resource, giving it a name and a priority‐ceiling. \*/**  
**void Resource\_Init( SharedResource\_t\* SR, const char\* name, UBaseType\_t ceilingPriority );**

**/\* Try to lock.  Returns pdTRUE if acquired; pdFALSE if timeout → job drop. \*/**  
**BaseType\_t Resource\_Lock( SharedResource\_t\* SR);**

**/\* Unlock a previously-locked resource. \*/**  
**BaseType\_t Resource\_Unlock( SharedResource\_t\* SR );**

In Resource\_Lock I made sure to not boost a task’s priority up-front. Instead,  I made sure to block the task until it’s safe (resource is released), and then I pull the mutex for the correct task.  
In prvUpdateSystemCeiling I made sure to compare the task’s priority to the resource’s ceiling priority

To address “A job J is allowed to enter a critical section only if its priority is higher than ceilings of all semaphores locked by jobs other than J” , I created an if statement that checked if the current\_prioror is less than resource ceiling priority. If it is then it exits out the locking and waits.

   
       **/\* 1\) check we outrank every other resource we already own \*/**  
       **UBaseType\_t currPrio \= uxTaskPriorityGet( current );**  
       **for( int i \= 0; i \< numResources; i++ )**  
       **{**  
           **SharedResource\_t \*res \= &sharedResources\[ i \];**  
           **if( res\-\>lockFlag \== pdTRUE && res\-\>owner \== current )**  
           **{**  
               **if( currPrio \<= res\-\>ceilingPriority )**  
               **{**  
                   **return pdFAIL;**  
               **}**  
           **}**  
       **}**

This makes sure  J does not even attempt to take the mutex until its priority strictly becomes greater than the ceiling priority.  Once that condition breaks, I take the  mutex and immediately raise the ceiling priority to include the resources’s ceiling, so that a lower priority job can’t interrupt.

**How I tested for correctness:**  
I used the OCPP schedule in figure 2 to test for corrections. Keep in mind figure 2 had the critical section start one time unit after the beginning of the previous critical section.

<img width="470" height="351" alt="Screenshot 2025-10-30 at 2 29 56 PM" src="https://github.com/user-attachments/assets/68379604-e9f4-492e-8eb9-a36e3c7b0262" />

Figure 2: OCPP Trace

<img width="483" height="644" alt="Screenshot 2025-10-30 at 2 30 41 PM" src="https://github.com/user-attachments/assets/bc9a7e66-bc0b-4c31-ba14-ef12ef038083" />

Comparing the chart you can see the release and blocking time is somewhat off. The output seems incorrect, J3 grabs s2 but j5 never unlocks it. I'm not sure why a task is grabbing a resource that is locked but i made sure in the code it shouldn’t be able do that with this line in the locking:

    /\* if it’s free, see if we can take it \*/  
    if( SR-\>lockFlag \== pdFALSE )

unlock:  
    /\* ONLY the owner may unlock \*/  
    if( SR-\>lockFlag \!= pdTRUE || SR-\>owner \!= current ){  
        return pdFAIL;  
}

**ICPP Implementation**  
Used the same task set from Figure 1\.

The approach for ICPP was so much simpler than OCPP because I did not have to worry about blocking. I used the same functions from OCPP but tweak some of the actions 

The approach of implementing this was keeping the ICPP rules in mind:

* Once it enters the critical section of resource R, it immediately inherits Π  
* Thus, under ICPP, once a task starts executing, all resources it needs will be free  
* Same worst-case behavior as PCP

I implemented the priority change in **ICPP by**  **immediately raising** the task’s priority to the resource ceiling before grabbing the mutex. This is because in the ICPP the priority of the tasks is boosted under certain conditions:

I did that through this function in the lock function:

   **/\* Save base priority and boost to resource ceiling \*/**  
   **pxThis\-\>uxPriority \= currPriority;**  
   **if( SR\-\>ceilingPriority \> currPriority )**  
   **{**  
       **vTaskPrioritySet(current, SR\-\>ceilingPriority);**  
   **}**

**How I tested for correctness:**  
I used the ICPP schedule in figure 3 to test for corrections. Keep in mind the figure 3 had the critical section start one time unit after the beginning of the previous critical section.

<img width="515" height="385" alt="Screenshot 2025-10-30 at 2 31 20 PM" src="https://github.com/user-attachments/assets/f18fb51b-1dfd-4eaf-8128-62c88ef5d62b" />

Figure 3: ICPP Trace

<img width="595" height="522" alt="Screenshot 2025-10-30 at 2 31 36 PM" src="https://github.com/user-attachments/assets/c7e50f15-3098-4692-9f95-d637d8ffff62" />

Similar to the OCPP issue you can see the release and blocking time is somewhat off. The output seems correct in terms of how it handles the resources and their priorities under ICPP, but seems off in terms of the execution of each time. Each job locks a task and unlocks a task before it can be accessed by another  job. This could be due to not the proper scaling of my ticks to ms.  
