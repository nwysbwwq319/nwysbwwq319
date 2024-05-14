# 第二次上机实验报告

## 实验一
### pthread_create
```c 
#include <stdio.h>
#include <pthread.h>
int sum = 0;
void thread(void) {
int i;
for (i = 0; i< 1000000; i++)
    sum += 1;
}
```
### 建立一个无需同步的多线程程序：nosync-ex.c
```c
int main(void) {
pthread_t tid1, tid2;
pthread_create(&tid1, NULL, thread, NULL);
pthread_create(&tid2, NULL, thread, NULL);
pthread_join(tid1, NULL);
pthread_join(tid2, NULL);
printf (“1000000 + 1000000 = %d\n”, sum);
return (0);
}

```
### 实验结果

## 实验二
### Mutex的使用(mutex-ex.c)
```c 
#include <stdio.h>
#include <pthread.h>
int sum = 0;
pthread_mutex_t mutex;
void thread(void) {
int i;
for (i = 0; i< 1000000; i++) {
pthread_mutex_lock(&mutex);
sum += 1;
pthread_mutex_unlock(&mutex);
}
}
int main(void) {
pthread_t tid1, tid2;
pthread_mutex_init(&mutex, NULL);
pthread_create(&tid1, NULL, thread, NULL);
pthread_create(&tid2, NULL, thread, NULL);
pthread_join(tid1, NULL);
pthread_join(tid2, NULL);
printf (“1000000 + 1000000 = %d\n”, sum);
return (0);
}

```

### 实验结果

## 实验三
### Semaphore相关函数的说明
```c 
#include <semaphore.h>
int sem_init(sem_t * sem, int pshared, unsigned int value);
int sem_wait(sem_t * sem);
int sem_post(sem_t * sem);
```
### Semaphore的使用(sem-ex.c)
```c
#include <stdio.h>
#include <pthread.h>
#include <semaphore.h>
int sum = 0;
sem_t sem;
void thread(void) {
int i;
for (i = 0; i< 1000000; i++) {
sem_wait(&sem);
sum += 1;
sem_post(&sem);
}
}
int main(void) {
pthread_t tid1, tid2;
sem_init(&sem, 0, 1);
pthread_create(&tid1, NULL, thread, NULL);
pthread_create(&tid2, NULL, thread, NULL);
pthread_join(tid1, NULL);
pthread_join(tid2, NULL);
printf (“1000000 + 1000000 = %d\n”, sum);
return (0);
}

```
### 实验结果


## 实验四
### 代码实现
```c
#include <stdio.h>
#include <pthread.h>
#include <unistd.h>

#define BUFFER_SIZE 10

// 用数组模拟缓冲区
int buffer[BUFFER_SIZE];
int in = 0, out = 0;
int count = 0;

// 互斥锁和条件变量
pthread_mutex_t lock;
pthread_cond_t empty;
pthread_cond_t full;

// 初始化互斥锁和条件变量
void init() {
    pthread_mutex_init(&lock, NULL);
    pthread_cond_init(&empty, NULL);
    pthread_cond_init(&full, NULL);
}

// 销毁互斥锁和条件变量
void destroy() {
    pthread_mutex_destroy(&lock);
    pthread_cond_destroy(&empty);
    pthread_cond_destroy(&full);
}

// 生产者函数
void* producer(void* arg) {
    while (1) {
        pthread_mutex_lock(&lock);
        // 缓冲区满时等待
        while (count == BUFFER_SIZE) {
            pthread_cond_wait(&full, &lock);
        }
        // 向缓冲区添加产品
        buffer[(in + count) % BUFFER_SIZE] = rand() % 100;
        printf("Produced item: %d\n", buffer[(in + count) % BUFFER_SIZE]);
        count++;
        in = (in + 1) % BUFFER_SIZE;
        // 通知消费者缓冲区非空
        pthread_cond_signal(&empty);
        pthread_mutex_unlock(&lock);
    }
    return NULL;
}

// 消费者函数
void* consumer(void* arg) {
    while (1) {
        pthread_mutex_lock(&lock);
        // 缓冲区空时等待
        while (count == 0) {
            pthread_cond_wait(&empty, &lock);
        }
        // 从缓冲区取出产品
        int item = buffer[out++];
        out = out % BUFFER_SIZE;
        count--;
        printf("Consumed item: %d\n", item);
        // 通知生产者缓冲区非满
        pthread_cond_signal(&full);
        pthread_mutex_unlock(&lock);
    }
    return NULL;
}

int main() {
    pthread_t p1, p2;
    init();
    // 创建生产者和消费者线程
    pthread_create(&p1, NULL, producer, NULL);
    pthread_create(&p2, NULL, consumer, NULL);
    // 等待线程结束
    pthread_join(p1, NULL);
    pthread_join(p2, NULL);
    destroy();
    return 0;
}

```

