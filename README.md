# CS33211-Assignment
Producer-Consumer Problem

To compile and run the program use the following commands. 
```
$ gcc producer.c -pthread -lrt -o producer
$ gcc consumer.c -pthread -lrt -o consumer
$ ./producer & ./consumer &
```
Description: 

This project contains three files. 
1) producer.c 
2) consumer.c - The process for consumer
3) protocol.h - The header file for shared variables and required libraries. 

`protocol.h` has the global variables for both `producer.c` and `consumer.c`. The number of threads or the buffer size can be adjusted in `protocol.h`. It has a data structure, `struct ShmData` which both `producer.c` and `consumer.c` share. 

```
struct ShmData {
	int x; 
	sem_t semEmpty;
	sem_t semFull;
	pthread_mutex_t mutexBuffer;
};
```

`int x` is use as a stack pointer or the ID of the last item in the stack. For simplicity of the problem, we do not have an actual stack in our programs. 

Two semaphores, `sem_t semEmpty` and `sem_t semFull` are used so that the producer will not be skipping while the buffer is full, or the consumer will not be skipping while the buffer is empty. 

`pthread_mutex_t mutexBuffer` is used to make sure that only one thread is accessing `int x` at a time. 


In `producer.c`, `shm_open()` is used to create shared memory. 
```
int fd = shm_open(NAME, O_CREAT | O_EXCL | O_RDWR, 0600); 
```

then `struct ShmData *shmData` is declared in shared memory. 
```
struct ShmData *shmData = mmap(0, sizeof(*shmData), PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0); 

```
