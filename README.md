# Parallel-Notes
This Repository will provide all notes I have taken regarding the Parallel Course (CSC447) until now. The notes are of course flawed and may be incomplete but I am trying my best lol! Enjoy!

# Message Passing Computing:
Our main objective is to speed up computation. In order to speed up computation, we usually enhance and optimize our algorithms. But what if the over head is from the hardware we are using. We can achieve a speed up by dividing a task onto multiple processes. This can be achieved through message passing. So, what is message passing?

Think of a multiple processes. These processes do not have shared memory nor can variable in process Pi be accessed by process Pj. So Pi needs to send data to Pj in order for Pj to be able to access data in Pi.
We have multiple models:

# Multi-Program Multi-Data Model (MPMD):

![Screenshot (407)](https://user-images.githubusercontent.com/95071049/160109849-c3880ec2-51f8-42f4-bc93-d961a9c0c0d4.png)

We have multiple source files that compile with their own data. Each one is compiled and executed on a seperate processor

# Single Program Multi-Data (SPMD-Used in MPI):
![Screenshot (409)](https://user-images.githubusercontent.com/95071049/160110559-37b46f9a-99b3-4476-8249-bbd706e55240.png)

It takes one source file, produces one executable and the .exe file is executed on a seperate processor

We use condition statements (if-else) to control which process is executed on which processor

This model is applied in MPI.

# Master Worker Approach (Previously known as Master-Slave Approach but this term has become inappropriate and offensive):

One processor executes that master process then the other processors are spawned from within the master. So a processor can "spwan()" another process at run time to facilitate execution. It dunamically creates a processes. However, MPI statically creates processes.

# Point to Point Communication:

MPI uses "send" and receive or "recv" to transfer data. These are library routines:
![Screenshot (411)](https://user-images.githubusercontent.com/95071049/160112054-75a518f3-5bc1-4727-acc9-a710e7122107.png)

What if P1 calls send but P2 has not called receive yet? 

What can be done?

We have two message passing styles...

# Synchronous

Stops everything and waits for the message to be fully accepted -> synchronous send routine

Stops everything and waits for the message's arrival -> synchronous receive routine

# Asynchronous

Processes do not wait, if this data is unavailable the process continues regularly. This need precise timing of send and receive by the programmer and should be used with caution

# MPI Blocking and Non-Blocking:

Blocking: allows modification of sent variables. So if a variable is sent, memory is allocated for it and the variable is stored there, so if any subsequent statement where to modify it will wait until local operations are complete and then sent it to the next process. If a buffer is used, blocking will wait for available space in the buffer.

Non-Blocking: Sending occurs before the completion of local operations. So it will not take into account if any subsequent statement were to modify it and does not wait for the completion of local operations. A programmer must ensure that no local modification will occur after send is called. If a buffer is used, non-blocking will not wait for available space and fail if the buffer is full.

# MPI_BCAST (Broadcast):

Takes data and copies it into different processes.

Syntax: 

int MPI_Bcast( void *buffer, int count, MPI_Datatype datatype, int root, MPI_Comm comm )

Input/Output Parameters:

buffer: starting address of buffer (choice)

Input Parameters:
count: number of entries in buffer (integer)

datatype: data type of buffer (handle)

root: rank of broadcast root (integer)

comm: communicator (handle)

# MPI_Scatter:

Sends elements of to all processes. A certain count of elements of an array is scattered onto multiple process
 Syntax:
 
int MPI_Scatter(const void *sendbuf, int sendcount, MPI_Datatype sendtype, void *recvbuf, int recvcount, MPI_Datatype recvtype, int root, MPI_Comm comm)

Input Parameters:

sendbuf: address of send buffer (choice, significant only at root)

sendcount: number of elements sent to each process (integer, significant only at root)

sendtype: data type of send buffer elements (significant only at root) (handle)

recvcount: number of elements in receive buffer (integer)

recvtype: data type of receive buffer elements (handle)

root: rank of sending process (integer)

comm: communicator (handle)

Output Parameters:

recvbuf: address of receive buffer (choice)

# MPI_Gather:

The opposite of scatter, it will collect multiple elements into an array at a single process.

Syntax:

int MPI_Gather(const void *sendbuf, int sendcount, MPI_Datatype sendtype,
               void *recvbuf, int recvcount, MPI_Datatype recvtype, int root, MPI_Comm comm)
Input Parameters:

sendbuf: starting address of send buffer (choice)

sendcount: number of elements in send buffer (integer)

sendtype: data type of send buffer elements (handle)

recvcount: number of elements for any single receive (integer, significant only at root)

recvtype: data type of recv buffer elements (significant only at root) (handle)

root: rank of receiving process (integer)

comm: communicator (handle)

Output Parameters:

recvbuf: address of receive buffer (choice, significant only at root)

# Communicators:

Each process has a rank. Ranks are defined from 0 to p-1. A communicator defines the scope of processes. It is the median they communicate in. The standart MPI communicator is MPI_Comm_World. MPI is asynchronous.

# Embarrisingly Parallel:

A program is said to be embarrasingly parallel if the task can be equally divided onto the number of processes. So if we had an array of n elements and we wanted to compute the sum of the array. We can divide the array to n/p parts (number of elements over number of processes) and each process will compute the sum of it's poriton of the array. P0 will initially read the array and it will then collect the sum from each process and compute the final result.

# Divide and Conquer:

We will use the divide and conquer approach similar to merge sort. This uses the topology of a tree. So each process has a left child and right child process and the parent will send the data and assign the task to each process.

           P0
          /  \
         P1   P2
        / \   / \
       P3 P4 P5  P6

The index of the left child 2\*rank+1 and the index of the right child 2\*rank+2

The parent has an index of (rank-1)/2

The task continues to get divided until it becomes so trivial that a single process can solve it. For example, in the sum computation example, the array will continue to get divided until there are only two elements and those two elements are very easy to add up. So, P3 will add its two elements and send it to P1, so will P4.

P5 and P6 will do the same and send them to P2

P1 and P2 will compute their sum and sends them to P0 and P0 will compute all the final result.


# A Special Case  (Remainder):

So let's assume that we have an array and we wish to divide it using either the embarassingly parallel strategy or the divide and conquer strategy, that should be easy right? Well what if the array cannot be easily divided. So if I have an array of 125 elements and 4 processes, I can't really divide the array equally. I could even miss a part of the array and some of them might not be accessed by the processes. So 125/4=31.25 or 31 if we're dealing with integers. We might miss a part of the array due to this inaccuracy. So what we can do to fix this is we can compute the remainder. We can compute the number of elements n (size of array/nb of processes), then we compute the remainder of this division m (size of array % nb of processes), then we can send n+m elements. That way we can be inclusive of all elements and ensure all elements of the array are accessed.

