# Project-5-Concurrent-Heaps-solution

Download Here: [Project 5, Concurrent Heaps solution](https://jarviscodinghub.com/assignment/project-5-concurrent-heaps-solution-2/)

For Custom/Original Work email jarviscodinghub@gmail.com/whatsapp +1(541)423-7793

Objectives
The objective of this programming assignment is for you to gain some experience writing threaded code in Java.
Introduction
In this project, you will implement the concurrent heap data structure described in this paper:

Hunt, Michael, Parthasarathy and Scott. “An efficient algorithm for concurrent priority queue heaps.” Information Processing Letters, 69:151-157, 1996.
(The paper contains pseudocode in a Pascal-like language which you will translate into Java.)

The algorithm presented in this paper uses locks on each node of the heap to allow multiple threads to insert and delete data from the heap. Insertion and deletion pretty much work the same as an ordinary binary heap. The locks and tags make sure that when threads try to access the same node in the heap that this is done in a way that does not result in deadlock or in a corrupted heap.

One change from ordinary heaps is that the bottom-most level of the concurrent heap is not filled in from left-to-right. The concurrent heap is still filled in level by level, but the pattern for filling in the bottom level makes consecutive insertions happen in separate subtrees of the heap. (In an ordinary heap, consecutive insertions happen right next to each other. This makes it much more likely that the threads doing the consecutive insertion would try to access the same nodes.) For example, when the fourth level of the heap is filled, instead of inserting into nodes 8, 9, 10, 11, 12, 13, 14 and 15 (in that order), the concurrent heap will insert into nodes 8, 12, 10, 14, 9, 13, 11 and 15. Note that this order still maintains the property that when a node has a right child then it must also have a left child. This pattern is called the “bit reversed” pattern. We can obtain the ordering described above by counting in binary and reversing the last three bits:

8 = 1 0 0 0 -> 1 0 0 0 = 8
9 = 1 0 0 1 -> 1 1 0 0 = 12
10 = 1 0 1 0 -> 1 0 1 0 = 10
11 = 1 0 1 1 -> 1 1 1 0 = 14
12 = 1 1 0 0 -> 1 0 0 1 = 9
13 = 1 1 0 1 -> 1 1 0 1 = 13
14 = 1 1 1 0 -> 1 0 1 1 = 11
15 = 1 1 1 1 -> 1 1 1 1 = 15

This pattern generalizes for every level of the heap, of course.

Assignment
Your assignment is to translate the pseudocode in the paper into Java. The pseudocode was transcribed on a University of Rochester website. The code has been adapted for our needs:

BRCounter.pseudocode
ConcHeapClass.pseudocode
Insert.pseudocode
Delete.pseudocode
Note that in Pascal, assignment is := (instead of =) and comparison is = (instead of ==). Also, you must follow the logic of the pseudocode exactly. The algorithm is rather delicate, especially with respect to the order of locking and unlocking. So, do not change the order of the code or leave out pieces that might look useless. Specifically, your heap must have the root at index 1 of the array, and not at index 0.

Since the locking and unlocking in this algorithm are not nested in some cases, we cannot use the synchronize facility in Java. (Sometimes the code locks A, locks B and then unlocks A before unlocking B. We cannot use synchronize to accomplish this ordering.) Fortunately, Java does provide the ReentrantLock class, which you can access by including

import java.util.concurrent.locks.ReentrantLock ;
Finally, since this is a one-week project, some explicit instructions are provided for you below. (This is much more a lab assignment than a design project.)

Instructions
Step 1: Skim Ahead
Skim through all of the instructions so you can see where we are headed. (No, the last step does not say “Ignore all the previous steps.”)

Step 2: Bit-Reversed Counter
The pseudocode for the bit-reversed counter is a bit confusing, so the Java equivalent has been provided for you:

BRCounter.java
Also, this gives you one example of Java code translated line-by-line from the pseudo-code.

Write a main program that uses the BRCounter class. Your program should call increment() 31 times and print out the values returned by increment() each time. Then your program should call decrement() 31 times. Notice how this counter can be used to keep track of the location of the “last element” of the heap.

Put your main program in a class called Test1 in a file called Test1.java in a package called driver. Save a copy of this file, you will submit it.

Step 3: the ConcHeap class
Create a ConcHeap.java file in a package called concheap. (BRCounter is also in this concheap package.) The ConcHeap class will hold most of your code. You will need a private inner class for each item of the heap. Let’s all use the following definition, so we can have common names:

// Each item in the heap has these fields.
// Use the names in the paper to avoid confusion.
//
private class HeapItem {
int priority ; // heap is ordered on this key.
long tag ; // Thread.getId() returns long
ReentrantLock itemLock ;

HeapItem() {
priority = 0 ;
tag = EMPTY ;

// *** true = we need fairness or algorithm could
// *** livelock and starve.
itemLock = new ReentrantLock(true) ;
}
}
Also, to copy the pseudocode, we need tag values AVAILABLE and EMPTY. Instead of using an enumeration, we can just define these as long constants since the tag member sometimes holds a thread’s id number. Java guarantees that a thread id is not a negative number, so we can use:

public static final long EMPTY = -1 ;
public static final long AVAILABLE = -2 ;
Your actual heap should be an array of HeapItem (not an ArrayList). This should be a data member in the ConcHeap class. It’s best if you call it H[], then it will be compatible with some later code.

Although each HeapItem has a lock, the algorithm also requires a lock for the entire heap. This is for when a thread uses the bit reversed counter to determine index of the last item in the heap.

Your ConcHeap class needs a constructor with the following signature:

public ConcHeap(int maximumSize) ;
Here maximumSize is the largest number of items that the heap will be asked to hold. (Remember that you are not using H[0].) Your constructor should initialize each entry of H[] so it points to an allocated HeapItem. (There should be no null references in H[]).

It is also convenient for testing and debugging to have a constructor that will copy the data from an integer array:

public ConcHeap(int [] A, int maximumSize) ;
This constructor should just assume that the items in the array A[] are already heap ordered. Furthermore, we will assume that the array A[] fills the last level of the heap. This requires that the size of the heap be one less than a power of 2 (e.g., 15, 31, 255). This avoid issues with the bit-reversed counter at the last level of the heap. (Note: since we are not using A[0], the A.length would be a power of 2.)

The HeapItems with data initialized by this constructor should have their tag fields set to AVAILABLE. Also, make sure that the bit-reversed counter is updated. So, your code should have something like this in the middle of a loop:

H[i].priority = A[i] ;
H[i].tag = AVAILABLE ;
count.increment() ; // update bit-reversed counter
You can use this sanityCheck() method to make sure that the heap is correct: sanityCheck.java. You should make sanityCheck() a method in your ConcHeap class. It assumes that you used maxSize for the name of the data member that holds the maximum size of the heap.

Step 4: insert
Now you are ready to implement insert. We want each insertion to be done by a separate thread, but we also want to be able to call insert in the usual way. So you should have a method:

public void insert(int x) ;
In order to fork off a thread that has access to the heap stored in H[], you need to create another private inner class. Let’s call this one InsertObject:

private class InsertObject implements Runnable ;
The “implements Runnable” modifier guarantees that the class has a method called run with the signature:

public void run() ;
It is this run() method that contains the code for inserting into the heap. The insert() method must create an InsertObject, pass it to the constructor for a Thread object and tell that Thread to start(). That will fork off a thread to do the insertion into the heap. The wrapper method insert() will return to the main program, which can call insert() again.

Note that the pseudocode provided is for a max heap, where the largest value is stored at the root of the heap. Let’s keep things consistent and implement a max heap.

One problem: the run() method in InsertObject is not allowed to have any parameters. So, how does it know what value to insert into the heap? When the wrapper method insert() creates the InsertObject, it must give x to the InsertObject constructor which in turn must store x in an instance member. Then, run() can refer to this instance member as the key to be inserted.

Now you can go ahead and translate the insertion pseudocode into Java: Insert.pseudocode. Note that Pascal is not object oriented. So, when the pseudocode says something like:

LOCK(i)
to lock the i-th item of the heap, you should write:

H[i].itemLock.lock() ;
Be sure to print out the diagnostic messages (including the Thread id) in the pseudocode as indicated so you can track the progress of your thread. You can obtain the ID number of the current thread by calling:

Thread.currentThread().getId()
Step 5: Test2
Write a main program to test your insertion method. This time put the main() method in a class called Test2 in a package called driver. You can initialize the heap with an array like this one: InitialHeap.java. It has ten full levels (1023 items). You should make the maximum size of the heap something much bigger than 1023, so you have room to insert. Start by inserting just one number, say 77132. Then your output should look something like:

Thread 8: inserting 77132.
Thread 8 (insert 77132): getting heap lock.
Thread 8 (insert 77132): initial index = 1024.
Thread 8 (insert 77132): at index 1024 getting itemLocks.
Thread 8 (insert 77132): at index 1024 parent available.
Thread 8 (insert 77132): at index 512 released itemLocks.
Thread 8 (insert 77132): at index 512 getting itemLocks.
Thread 8 (insert 77132): at index 512 parent available.
Thread 8 (insert 77132): at index 256 released itemLocks.
Thread 8 (insert 77132): at index 256 getting itemLocks.
Thread 8 (insert 77132): at index 256 parent available.
Thread 8 (insert 77132): at index 0 released itemLocks.
Now, modify the main program so it calls heap.insert() again, right after the first call to heap.insert(). You should be able to see some interleaving of the threads:

Thread 9: inserting 69145.
Thread 9 (insert 69145): getting heap lock.
Thread 8: inserting 77132.
Thread 9 (insert 69145): initial index = 1024.
Thread 8 (insert 77132): getting heap lock.
Thread 9 (insert 69145): at index 1024 getting itemLocks.
Thread 8 (insert 77132): initial index = 1536.
Thread 9 (insert 69145): at index 1024 parent available.
Thread 8 (insert 77132): at index 1536 getting itemLocks.
Thread 9 (insert 69145): at index 512 released itemLocks.
Thread 8 (insert 77132): at index 1536 parent available.
Thread 9 (insert 69145): at index 512 getting itemLocks.
Thread 8 (insert 77132): at index 768 released itemLocks.
Thread 9 (insert 69145): at index 512 parent available.
Thread 8 (insert 77132): at index 768 getting itemLocks.
Thread 9 (insert 69145): at index 0 released itemLocks.
Thread 8 (insert 77132): at index 768 parent available.
Thread 8 (insert 77132): at index 384 released itemLocks.
Thread 8 (insert 77132): at index 384 getting itemLocks.
Thread 8 (insert 77132): at index 384 parent available.
Thread 8 (insert 77132): at index 0 released itemLocks.
If you do not see any interleaving, it might be that your system is too fast and the insertion thread was done before your main program can call insert() again. In that case, make a call to Thread.sleep() as indicated in the pseudocode. (You’ll have to play with the number of milliseconds to sleep.)

Finally, modify the main program so that it calls insert 5 times, one right after the other. When the threads are done, make a call to sanityCheck() and see if your heap is still heap-ordered. You can wait for all the insertion threads to finish by polling:

int tCount ;
do {
Thread.sleep(300) ;
tCount = Thread.activeCount() ;
} while (tCount > 1) ;

heap.sanityCheck() ;
If your main program in Test2 works correctly, save a copy of it. This is one of the programs that you have to submit.

Step 6: deleteMax
Implement deleteMax(). The situation is analogous to that for insert(). You will need to write a wrapper method with the signature:

public void deleteMax() ;
and you will need to define a private inner class that implements Runnable as before. Notice that this version of deleteMax() does not return any values. This is because the main program would have to wait for the method to finish before it can get a return value. For this project, we want our main program to turn around and call another method, so we will forgo the return value. (In any case, the semantics of deleteMax() is a bit vague when there could be a concurrent thread that is in the middle of inserting a really large value.)

Now make a copy of Test2.java, call it Test3.java. Modify Test3.java so it inserts twice followed by a call to deleteMax(). If the timing is right (try adjusting the Thread.sleep() parameter in insert), you might see a situation like this one:

Thread 8: inserting 77132.
Thread 8 (insert 77132): getting heap lock.
Thread 8 (insert 77132): initial index = 1024.
Thread 8 (insert 77132): at index 1024 getting itemLocks.
Thread 9: inserting 69145.
Thread 9 (insert 69145): getting heap lock.
Thread 9 (insert 69145): initial index = 1536.
Thread 9 (insert 69145): at index 1536 getting itemLocks.
Thread 10 (deleteMax): getting heapLock.
Thread 8 (insert 77132): at index 1024 parent available.
Thread 8 (insert 77132): at index 512 released itemLocks.
Thread 8 (insert 77132): at index 512 getting itemLocks.
Thread 9 (insert 69145): at index 1536 parent available.
Thread 9 (insert 69145): at index 768 released itemLocks.
Thread 9 (insert 69145): at index 768 getting itemLocks.
Thread 10 (deleteMax): trickling down 27044 at index 1. Getting itemLocks.
Thread 10 (deleteMax): trickling down 27044 at index 2. Getting itemLocks.
Thread 10 (deleteMax): trickling down 27044 at index 4. Getting itemLocks.
Thread 10 (deleteMax): trickling down 27044 at index 8. Getting itemLocks.
Thread 10 (deleteMax): trickling down 27044 at index 16. Getting itemLocks.
Thread 10 (deleteMax): trickling down 27044 at index 32. Getting itemLocks.
Thread 10 (deleteMax): trickling down 27044 at index 64. Getting itemLocks.
Thread 10 (deleteMax): trickling down 27044 at index 128. Getting itemLocks.
Thread 8 (insert 77132): at index 512 parent available.
Thread 8 (insert 77132): at index 256 released itemLocks.
Thread 8 (insert 77132): at index 256 getting itemLocks.
Thread 10 (deleteMax): trickling down 27044 at index 256. Getting itemLocks.
Thread 10 (deleteMax): trickling down 27044 at index 512. Getting itemLocks.
Thread 9 (insert 69145): at index 768 parent available.
Thread 9 (insert 69145): at index 384 released itemLocks.
Thread 9 (insert 69145): at index 384 getting itemLocks.
Thread 8 (insert 77132): at index 256 not my ID.
Thread 8 (insert 77132): at index 128 released itemLocks.
Thread 8 (insert 77132): at index 128 getting itemLocks.
Thread 9 (insert 69145): at index 384 parent available.
Thread 9 (insert 69145): at index 0 released itemLocks.
Thread 8 (insert 77132): at index 128 parent available.
Thread 8 (insert 77132): at index 0 released itemLocks.
Heap passes sanity check!!!
Notice the “not my ID” reported by Thread 8:

Thread 8 (insert 77132): at index 256 not my ID.
This is because Thread 10, which is cleaning up after the deleteMax(), had previously obtained the lock on node 256, when it was examining nodes 128, 256 and 257:

Thread 10 (deleteMax): trickling down 27044 at index 128. Getting itemLocks.
Thread 10 noticed that 27044 is smaller than 77132 and swapped those two, placing 77132 in node 128. Now when Thread 8 got its turn for the lock on node 256, it noticed, by looking at the tag (which should have its Thread id), that node 256 no longer held its number. So, Thread 8 “chases” its number “up” the tree (towards the root) and eventually catches up to it. At the end, the call to sanityCheck() indicates that the heap is still properly ordered.

Now, modify Test3.java to do 5 repetitions of 2 insertions followed by 1 deleteMax. Save a copy of this main program. It is one of the programs that you will submit.

Step 7: Stress Test
Write a main program (call it Test4.java) that does a stress test on the concurrent heap. First, initialize the heap with some random data. One way to do this is to first put the data in the A[] array. You can pick a large value for the root and for each non-root node, pick a random value to subtract from its parent. (This guarantees heap ordering.) Make an array this way with 32768 items and use it to initialize a heap with maximum size 65536. Call sanityCheck() to make sure that your initial heap is good.

Now, repeatedly call either insert() or deleteMax() (randomly choose one of them) several hundred times. If you are calling insert, supply it with a random number as a parameter. Make sure that the parameters chosen have a chance of being sufficiently large to be the root (compared to the random numbers initially placed in the heap). After all the calls are done, use sanityCheck() to confirm that you still have a heap.

What to Submit
Copy over your BRCounter.java, ConcHeap.java, Test1.java, Test2.java, Test3.java and Test4.java files to the GL system and place them in the appropriate directories.

Make sure that ANT will compile the programs. You can run the main programs on GL using:

java -cp bin driver.Test1
java -cp bin driver.Test2
java -cp bin driver.Test3
java -cp bin driver.Test4
Don’t worry if the output on GL is not the same as the output on your own machine. The implementations of the Java Virtual Machines are not identical. Also the behavior of threaded programs will differ depending on how many CPU cores the operating system decides to give to your program.

