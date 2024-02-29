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

I later call the `pthread_mutex_lock(&hash_table->mutex)` function at the beginning of the hash_table_v1_add_entry function, denoting the start of the critical section. This section is later unlocked by calling pthread_mutex_unlock(&hash_table->mutex) before we return on lines 96 and 109.

The code in between the lock and unlock commands is the critical section. Our locking system guarantees correctness as there is only one thread that can access the hash_table at a time to add or update an entry. This means that the output will always be deterministic and the function will always run properly. Once the thread is finishing adding the hash table entry, it will unlock the mutex allowing other threads to gain access to the critical section. 


### Performance
```shell
TODO how to run and results
```
Version 1 is a little slower/faster than the base version. As TODO

## Second Implementation
In the `hash_table_v2_add_entry` function, I TODO

### Performance
```shell
TODO how to run and results
```

TODO more results, speedup measurement, and analysis on v2

## Cleaning up
```shell
TODO how to clean
```
