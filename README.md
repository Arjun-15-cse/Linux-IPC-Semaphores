# Linux-IPC-Semaphores
Ex05-Linux IPC-Semaphores

# AIM:
To Write a C program that implements a producer-consumer system with two processes using Semaphores.

# DESIGN STEPS:

### Step 1:

Navigate to any Linux environment installed on the system or installed inside a virtual environment like virtual box/vmware or online linux JSLinux (https://bellard.org/jslinux/vm.html?url=alpine-x86.cfg&mem=192) or docker.

### Step 2:

Write the C Program using Linux Process API - Sempahores

### Step 3:

Execute the C Program for the desired output. 

# PROGRAM:

## Write a C program that implements a producer-consumer system with two processes using Semaphores.
```
/*
 * sem.c - Producer-Consumer using Semaphores with ipcs -s visibility
 */
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/sem.h>
#include <sys/wait.h>
#include <time.h>

#define NUM_LOOPS 10  // Number of producer-consumer cycles

// Define union semun if not already available
union semun {
    int val;
    struct semid_ds *buf;
    unsigned short int *array;
    struct seminfo *__buf;
};

// Function to wait (P operation) on semaphore
void wait_semaphore(int sem_set_id) {
    struct sembuf sem_op;
    sem_op.sem_num = 0;
    sem_op.sem_op = -1;  // Wait
    sem_op.sem_flg = 0;
    if (semop(sem_set_id, &sem_op, 1) == -1) {
        perror("semop wait failed");
        exit(1);
    }
}

// Function to signal (V operation) on semaphore
void signal_semaphore(int sem_set_id) {
    struct sembuf sem_op;
    sem_op.sem_num = 0;
    sem_op.sem_op = 1;  // Signal
    sem_op.sem_flg = 0;
    if (semop(sem_set_id, &sem_op, 1) == -1) {
        perror("semop signal failed");
        exit(1);
    }
}

// Print current value of the semaphore
void print_semaphore_value(int sem_set_id) {
    int val = semctl(sem_set_id, 0, GETVAL);
    if (val == -1) {
        perror("semctl GETVAL failed");
    } else {
        printf("Current semaphore value: %d\n", val);
    }
}

int main() {
    int sem_set_id;
    union semun sem_val;
    int child_pid;

    // Generate a key using ftok
    key_t key = ftok(".", 'S');
    if (key == -1) {
        perror("ftok");
        exit(1);
    }

    // Create semaphore set with 1 semaphore
    sem_set_id = semget(key, 1, IPC_CREAT | 0666);
    if (sem_set_id == -1) {
        perror("semget");
        exit(1);
    }

    printf("Semaphore set created, ID = %d\n", sem_set_id);

    // Initialize to 0 (consumer waits initially)
    sem_val.val = 0;
    if (semctl(sem_set_id, 0, SETVAL, sem_val) == -1) {
        perror("semctl SETVAL");
        exit(1);
    }

    print_semaphore_value(sem_set_id);  // Initial value

    // Fork child
    child_pid = fork();

    if (child_pid < 0) {
        perror("fork");
        exit(1);
    }

    if (child_pid == 0) {
        // Child process - consumer
        for (int i = 0; i < NUM_LOOPS; i++) {
            wait_semaphore(sem_set_id);
            printf("consumer: '%d'\n", i);
            fflush(stdout);
            print_semaphore_value(sem_set_id);
        }
        exit(0);
    } else {
        // Parent process - producer
        for (int i = 0; i < NUM_LOOPS; i++) {
            printf("producer: '%d'\n", i);
            fflush(stdout);
            signal_semaphore(sem_set_id);
            print_semaphore_value(sem_set_id);
            usleep(500000);
        }

        wait(NULL);  // Wait for child

        // Allow user time to check `ipcs -s`
        printf("Producer done. Check semaphore with 'ipcs -s'. Sleeping for 10 seconds...\n");
        sleep(10);

        // Remove semaphore
        if (semctl(sem_set_id, 0, IPC_RMID, sem_val) == -1) {
            perror("semctl IPC_RMID");
        } else {
            printf("Semaphore removed.\n");
        }
    }

    return 0;
}

```
## OUTPUT
$ ./sem.o 

![Screenshot from 2025-05-23 16-40-52](https://github.com/user-attachments/assets/d9d5e707-ca94-4a79-aa06-ce4071ffde5e)


![Screenshot from 2025-05-23 16-41-12](https://github.com/user-attachments/assets/bf79eb09-cbe0-4505-8be5-5483b003ca7c)

$ ipcs

![Screenshot from 2025-05-23 16-41-42](https://github.com/user-attachments/assets/4557781e-7be5-4956-a1b8-5e1ad68a21e7)

# RESULT:
The program is executed successfully.
