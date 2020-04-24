# P2L3:Pthreads

## Pthreads (vs Birrell)

- Threads , Mutexes and Condition Variables
- Safety tips!
- Compilation and examples

## Pthread Creation

Type |Birrels’ Mechanims | Pthreads
--|-----------|--------
Data type| Thread | `pthread_t`
Thread creation |Fork (proc, args) | `pthread_create`
Thread completion|Join(thread) | `pthread_join`


## Pthread Attributes

- data type `pthread_attr_t`
- specified in `pthread_create`
- Defines features of the new thread (e.g. stack size, joinable, inheritance, scheduling policy, system/process thread scope, priority)
- has default behavior with `NULL` in `pthread_create`
- some usages
	```c
	int pthread_attr_init(phread_attr_t *attr)

	int pthread_attr_destory(phread_attr_t *attr)
		
	pthread_attr_(set/get) {attribute)
	```

## Detaching Pthreads

- Default: Joinable threads (as Birrell)
    - 问题是parent exists early，children threads can become zombies
- Detached threads
    - detach 之后就不能join
    - 但是可以在parent退出以后，继续运行

    int pthread_detach(pthread_t thread);
    
    //或者创建的时候
    pthread_attr_setdetachstate(&attr, PTHEAD_CREATE_DETACHED);
    pthread_create(…, attracted, …);

    #include <stdio.h>
    
    #include <pthread.h>
    
    void *foo (void *arg) { /* thread main*/
    	print(“Foobar!\n”);
    	pthread_exit(NULL);
    }
    
    int main (void) {
    
    	int i ;
    	pthread_t tid;
    	pthread_attr_t attractive;
    	pthread_attr_init(&attr); /* required*/
    	pthread_attr_setdetachstate(&attr, PTHEAD_CREATE_DETACHED);
    	pthread_attr_setscope(&attr, PTHEAD_SCOPE_SYSTEM);
    	pthread_create(NULL, &attr, foo, NULL);
    }

## Compile Pthreads

1. `#include <pthread.h>` in main file
2. Compile source with -lpthread or -pthread
    1. `gcc -o main main.c -lpthead`
    2. `gcc -o main main.c -pthread`
3. Check return values of common functions

## Pthread Creation Example 1

    #include <stdio.h>
    #include <pthread.h>
    #define NUM_THREADS 4
    
    void *hello (void *arg) { /* thread main*/
    	print("Hello Thread!\n");
    	return 0;
    }
    
    int main (void) {
    	int i ;
    	pthread_t tid[NUM_THREADS];
    
    	for ( int i = 0; I < NUM_THREADS; I++) {
    		pthread_create(tid, NULL, hello, NULL);
    	}
    }

## Pthread Creation Example 2

```c
#include <stdio.h>
#include <pthread.h>

#define NUM_THREADS 4

void* threadFunc (void *arg) { /* thread main*/
	int *p = (int*) arg
	int myNum = *p;
	print("Thread number %d\n");
	return 0;
}

Int main (void) {
	int i ;
	pthread_t tid[NUM_THREADS];
	for ( int i = 0; i < NUM_THREADS; i++) {
		pthread_create(&tid[i], NULL, &threadFunc, &i);
	}

	for ( int i = 0; i < NUM_THREADS; i++) {
		pthread_join(tid[i]);
}
	return 0;
}
```

问题: 输出
```
Thread number 0
Thread number 2
Thread number 2
Thread number 3
```

- `i` is defined in main => it’s globally visible variable!
- When it changes in one thread => all other threads see new values!
- data race of race condition => a thread tries to read a value, while another thread modifies it!
```c
    void* threadFunc (void *arg) { /* thread main*/
    	int *p = (int*) arg
    	int myNum = *p;
    
    	print("Thread number %d\n");
    	return 0;
    }
    
    int main (void) {
    	int i ;
    	pthread_t tid[NUM_THREADS];
    	int tNum[NUM_THREADS];
    	for ( int i = 0; i < NUM_THREADS; i++) {
    		tNum[i] = i;
    		pthread_create(&tid[i], NULL, threadFunc, &tNum[i]);
    	}
    
    	for ( int i = 0; i < NUM_THREADS; i++) {
    		pthread_join(tid[i]);
    	}
    	return 0;
    }
```
## Pthread Mutexs

“To solve mutual exclusion problem among concurrent threads”

Birrells’s Mechanisms

- Mutex
- Lock(mutex) {}

Pthread

- `pthread_mutex_x`
- `pthread_mutex_lock` // explicit lock
- `int pthread_mutex_unlock` // explicit unlock

    int phread_mutex_init(pthread_mutex_t *mutex, const pthread_mutexattr_t *attr);

- `attr` NULL 是default
- `tryLock`
    - int pthread_mutex_trylock(pthread_mutex_t *mutex);

    //在mutex被lock的时候，不会阻塞线程

- `pthread_mutex_destory`

         int pthread_mutex_destory(pthread_mutex_t *mutex); //free up mutex

- many others

## Pthread Mutexes safety tips

- Shared data be accessed through a single mutex!
- Mutex scope must be visible to all!
- Globally order locks
- Always unlock a mutex
    - Always unlock the correct mutex

## Pthread Condition Variables

Birrell’s:

- Condition
- Wait
- Signal
- Broadcast

pThreads:

- `pthread_cond_t`
- `pthread_cond_wait`
- `pthread_cond_signal`
- `pthread_cond_broadcast`

## Other Condition Variable Operations

- attributes

        int pthread_cond_init(pthread_cond_t * cond, const pthread_conattr_t * attr);

    - e.g., if it’s shared
    - pass NULL will be default
- destroy

        int pthread_cond_destory(pthread_cond_t * cond);

## Condition Variable safety tips

- Do not forget to notify waiting threads~
    - predicate changes => signal / broadcast correct condition variable
- When in doubt broadcast
    - but performance loss
- You do not need a mutex to signal/broadcast

## Producer and Consumer Example
```c
   #include <stdio.h>
#include <stdlib.h>
#include <pthread.h>

#define BUF_SIZE 3		/* Size of shared buffer */

int buffer[BUF_SIZE];  	/* shared buffer */
int add = 0;  			/* place to add next element */
int rem = 0;  			/* place to remove next element */
int num = 0;  			/* number elements in buffer */

pthread_mutex_t m = PTHREAD_MUTEX_INITIALIZER;  	/* mutex lock for buffer */
pthread_cond_t c_cons = PTHREAD_COND_INITIALIZER; /* consumer waits on this cond var */
pthread_cond_t c_prod = PTHREAD_COND_INITIALIZER; /* producer waits on this cond var */

void *producer (void *param);
void *consumer (void *param);

int main(int argc, char *argv[]) {

	pthread_t tid1, tid2;  /* thread identifiers */
	int i;

	/* create the threads; may be any number, in general */
	if(pthread_create(&tid1, NULL, producer, NULL) != 0) {
		fprintf(stderr, "Unable to create producer thread\n");
		exit(1);
	}

	if(pthread_create(&tid2, NULL, consumer, NULL) != 0) {
		fprintf(stderr, "Unable to create consumer thread\n");
		exit(1);
	}

	/* wait for created thread to exit */
	pthread_join(tid1, NULL);
	pthread_join(tid2, NULL);
	printf("Parent quiting\n");

	return 0;
}

/* Produce value(s) */
void *producer(void *param) {

	int i;
	for (i=1; i<=20; i++) {
		
		/* Insert into buffer */
		pthread_mutex_lock (&m);	
			if (num > BUF_SIZE) {
				exit(1);  /* overflow */
			}

			while (num == BUF_SIZE) {  /* block if buffer is full */
				pthread_cond_wait (&c_prod, &m);
			}
			
			/* if executing here, buffer not full so add element */
			buffer[add] = i;
			add = (add+1) % BUF_SIZE;
			num++;
		pthread_mutex_unlock (&m);

		pthread_cond_signal (&c_cons);
		printf ("producer: inserted %d\n", i);
		fflush (stdout);
	}

	printf("producer quiting\n");
	fflush(stdout);
	return 0;
}

/* Consume value(s); Note the consumer never terminates */
void *consumer(void *param) {

	int i;

	while(1) {

		pthread_mutex_lock (&m);
			if (num < 0) {
				exit(1);
			} /* underflow */

			while (num == 0) {  /* block if buffer empty */
				pthread_cond_wait (&c_cons, &m);
			}

			/* if executing here, buffer not empty so remove element */
			i = buffer[rem];
			rem = (rem+1) % BUF_SIZE;
			num--;
		pthread_mutex_unlock (&m);

		pthread_cond_signal (&c_prod);
		printf ("Consume value %d\n", i);  fflush(stdout);

	}
	return 0;
}
```