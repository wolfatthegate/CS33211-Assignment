# CS33211-Assignment

Author - Waylon Luo (Kent State University)

Producer-Consumer Problem is a classical synchronization problem where the producer produces items and place them in the buffer - the consumer consumes the items from the buffer. In this assignment, the producer and the consumer run under two seperate processes while sharing the stack pointer, semaphore and mutex. 

This project contains four files. 
1) producer.c 
2) consumer.c 
3) protocol.h 
4) cleanup.c

To compile and run the program use the following commands. 
```
$ gcc producer.c -pthread -lrt -o producer
$ gcc consumer.c -pthread -lrt -o consumer
$ gcc cleanup.c -pthread -lrt -o cleanup
$ ./producer & ./consumer &
```

The output should look like: 

```
producer thread creaded..
consumer thread creaded..
Producer produces item 1
Producer produces item 2
Consumer consumes item 2
Producer produces item 2
Consumer consumes item 2
Consumer consumes item 1
```
If the program crashes in the middle of the process and does not have a chance to clean up the shared memory space, run 
``` 
$ ./cleanup
```

Note: The program does not run on macOS. It only runs on Linux server or Virtual box. 

Descriptions: 

`protocol.h` has the global variables for both `producer.c` and `consumer.c`. The number of threads, the buffer size and iterations can be adjusted in `protocol.h`. It has a data structure, `struct ShmData` which both `producer.c` and `consumer.c` share. 

```
struct ShmData {
	int x; 
	sem_t semEmpty;
	sem_t semFull;
	pthread_mutex_t mutexBuffer;
};
```
The usage of shared memory data, `ShmData`, is discussed in `producer()` and `consumer()`. 


#### `main()` method

`shm_open()` is used to create shared memory as followed. 
```
int fd = shm_open(NAME, O_CREAT | O_EXCL | O_RDWR, 0600); 
```

`consumer.c` also has to call `shm_open()` to access the share memory without passing `O_CREAT`, meaning that it is accessing the shared memory that `producer.c` created; it does not have to reserve share memory. 

`struct ShmData *shmData` is mapped in the shared memory. 
```
struct ShmData *shmData = mmap(0, sizeof(*shmData), PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0); 
```
As default, the number of threads is set to 1,which can be modified in `protocol.h`, `THREAD_NUM`. The buffer size is set to 2, meaning that there are two slots for items. At the end of the program, semaphores and mutex are destory, shared memory data are unmapped and shared memory is unlinked. 

#### `void* producer(struct ShmData *shmData);` and `void* consumer(struct ShmData *shmData);`

The iterations are limited to a certain number for a demonstration purpose. One can modify the iteration `ITR` in `protocol.h`. 

`shmData->x` is used as a stack pointer or the ID of the last item in the stack. For simplicity of the problem, we do not have an actual stack in our programs. 

Two semaphores, `shmData->semEmpty` and `shmData->semFull` are used so that the producer will not be skipping while the buffer is full, or the consumer will not be skipping while the buffer is empty. 

`shmData->mutexBuffer` is used to make sure that only one thread is accessing `shmData->x` at a time. 

