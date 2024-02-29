# Hash Hash Hash
The purpose of this lab is to fix a hash table implementation and make it safe to use concurrently (in a thread-safe manner). We do this through implementing mutexes and utilizing them to lock and unlock critical sections.

## Building
To build the module we can run the following command

```shell
make 	// build the module
```

## Running
After running the `make` command, the executable `hash-table-tester` is created. It takes two arguments: 

`-t`: number of threads to use (default is 4) \
`-s`: number of hash table entries to add per thread (default is 25000). 

To run this executable we can call the command
 
```shell
./hash-table-tester -t 4 -s 50000
```

## First Implementation
In the `hash_table_v1_add_entry` function, I first initialized `pthread_mutex_t mutex` in the `hash_table_v1` struct. This mutex object is used as a general lock for the entire hash table and is initialized at the end of the `*hash_table_v2_create()` function. 

I later call the `pthread_mutex_lock(&hash_table->mutex)` function at the beginning of the hash_table_v1_add_entry function, denoting the start of the critical section. This section is later unlocked by calling pthread_mutex_unlock(&hash_table->mutex) before we return on lines 93 and 106.

The code in between the lock and unlock commands is the critical section. Our locking system guarantees correctness as there is only one thread that can access the hash_table at a time to add or update an entry. This means that the output will always be deterministic and the function will always run properly. Once the thread is finishing adding the hash table entry, it will unlock the mutex allowing other threads to gain access to the critical section. 


### Performance
After running the test case, we get these results
```shell
bash-5.2$ ./hash-table-tester -t 4 -s 50000
Generation: 35,493 usec
Hash table base: 200,732 usec
  - 0 missing
Hash table v1: 470,094 usec
  - 0 missing
Hash table v2: 63,430 usec
  - 0 missing
```
Version 1 is a little slower than the base version. As we lock the entire data structure and make it mutually exclusive, only one thread can gain access to update the hash table at a time. Therefore, this severely limits ay attempts at parallelism in our program. Although this approach is correct, it lacks performance due to thread overhead. For instance, creating a new thread or terminating an existing one involves a certain amount of overhead, as does context switching, which is when the CPU switches from one thread to another. 

## Second Implementation
In the `hash_table_v2_add_entry` function, I chose to initialize `pthread_mutex_t mutex` in the `hash_table_entry` struct. Instead of declaring a mutex object for the entire hash table data structure, we declare many objects (one for each hash bucket). This mutex is initialized inside `hash_table_v2_create` function. Other than this difference, the locking and unlocking commands in the code are in the same place, and the mutex locks lines 89-107 (the critical section). Instead of one lock, there will be `HASH_TABLE_CAPACITY` number of locks.

### Performance
After running the test case, we get these results
```shell
bash-5.2$ ./hash-table-tester -t 4 -s 50000
Generation: 35,493 usec
Hash table base: 200,732 usec
  - 0 missing
Hash table v1: 470,094 usec
  - 0 missing
Hash table v2: 63,430 usec
  - 0 missing
```

This time the speed up v2 is faster by a factor of 3.16 which according to the formula given in the spec,`v2 ≤ base/(num cores - 1)`, meets the 55 point requirement. 
By increasing the granularity of our locks to each hash bucket instead of the entire hash table, the threads can access the hash table more often and in a more concurrent fashion, making it faster than the base. When one thread is operating on a particular hash bucket, say bucket 1, another thread can access a different bucket like bucket 3, performing the add entry operation in parallel. Since the mutexes are still on the individual hash buckets though, this still prevents concurrency issues as no two threads can access the same bucket and cause conflicts. I decided not to use more locks as having locks for each linked list node would allow for more parallelism but would also come with even greater overhead, making the program slower.

## Cleaning up
To clean up all binary files, we simply run the command defined in the Makefile
```shell
make clean	//remove the previous build information
```
which removes all generated files and directories created during the build process
