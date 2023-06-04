---
title:       "互斥锁、条件变量和信号量"
subtitle:    ""
description: ""
date:        2023-05-31T21:19:55+08:00 
author:      ""
image:       ""
tags:        ["网络编程"]
categories:  ["Tech" ]
katex:       false
---

信号量和条件变量傻傻分不清楚，趁这个机会好好捋一捋，慢慢理解慢慢精进这篇博客

![可以看一看知乎博主BBAB这篇回答，讲的挺好<信号量和条件变量的关系是什么？ - BBAB的回答 - 知乎>](https://www.zhihu.com/question/481951579/answer/2291116376)

## 互斥锁
在多线程环境下，当多个线程共同使用同一块资源时，可能会由于各个线程使用资源的顺序不合理导致使用混乱的问题，这种情况叫做出现了「竞态条件」。  
如“生产者消费者模型”中：
- 当两个生产者线程可能由于同时写一块资源区，导致前一个线程写的信息被后一个资源写的信息覆盖，所以不能同时两个生产者使用一块资源
- 当一个生产者和一个消费者同时进入一块资源区，消费者可能读到了生产者刚生产的资源（消费者可能想读旧的资源），所以生产者和消费者不能同食进入资源区
- 当容器空时，消费者无法进行消费，所以只有生产者生产了一部分之后，消费者才能进入资源区
- 当容器满时，生产者无法进行生产，所以只有消费者消费了一部分之后，生产者才能进入资源区
我们把这种问题叫做「线程同步问题」，同步问题中涉及的共享资源叫做「临界区」

以下是加锁的代码
```c++
#include <stdio.h>
#include <pthread.h>
#include <stdlib.h>
#include <unistd.h>

// 链表作为容器
struct Node{
    int val;
    struct Node* next;
};

// 头结点
struct Node* head = NULL;

// 互斥量
pthread_mutex_t mutex;

// 头插法增加元素
void* producter(void* arg) {
    while (1) {
        pthread_mutex_lock(&mutex);
        struct Node* newNode = (struct Node*)malloc(sizeof(struct Node));
        newNode->val = rand() % 1000;
        newNode->next = head;
        head = newNode;
        printf("add node, num : %d, tid : %ld\n", newNode->val, pthread_self());
        pthread_mutex_unlock(&mutex);
        usleep(100);
    }
    return NULL;
}

// 头删法减少元素
void* consumer(void* arg) {
    while (1) {
        pthread_mutex_lock(&mutex);
        struct Node* tmp = head;
        // 当链表不为空时，才能删除
        if (head != NULL) {
            head = head->next;
            printf("del node, num : %d, tid : %ld\n", tmp->val, pthread_self());
            free(tmp);
            pthread_mutex_unlock(&mutex);
            usleep(100);
        } else {
            pthread_mutex_unlock(&mutex);
        }
    }
    return NULL;
}

int main()
{
    // 初始化互斥锁
    pthread_mutex_init(&mutex, NULL);
    // 创建5个生产者线程，和5个消费者线程
    pthread_t products[5], consumes[5];
    for (int i = 0; i < 5; i++) {
        pthread_create(&products[i], NULL, producter, NULL);
        pthread_create(&consumes[i], NULL, consumer, NULL);
    }

    // 分离，回收线程资源
    for (int i = 0; i < 5; i++) {
        pthread_detach(products[i]);
        pthread_detach(consumes[i]);
    }

    // 回收互斥锁
    pthread_mutex_destroy(&mutex);
    pthread_exit(NULL);     // 回收主线程
    return 0;
}
```

## 条件变量
上述代码存在一个问题：当临界区没有资源时，消费者仍然不断地在循环查看是否能加锁并进入临界区，这样十分消耗资源。  
造成这种现象的原因是：互斥锁只有 锁 和 没锁两种状态，并不会告诉你什么时候解锁，还有多少人需要用锁。  
此时就需要用到「条件变量」，条件变量是传递应用程序状态信息的一种通讯机制， 注意条件变量不是锁！  

### 条件变量相关操作
- 当满足条件时，才执行，不是锁，配合互斥量使用
- 条件变量的类型：`pthread_cond_t`
- 初始化：`int pthread_cond_init(pthread_cond_t *restrict cond, const pthread_condattr_t *restrict attr)`;
- 回收：`int pthread_cond_destroy(pthread_cond_t *cond)`;
- 等待，调用了该函数，线程会阻塞：`int pthread_cond_wait(pthread_cond_t *restrict cond, pthread_mutex_t *restrict mutex)`;
- 等待多长时间，调用了这个函数，线程会阻塞，直到指定的时间结束：`int pthread_cond_timedwait(pthread_cond_t *restrict cond,  pthread_mutex_t *restrict mutex, const struct timespec *restrict abstime)`;
- 唤醒一个或者多个等待的线程：`int pthread_cond_signal(pthread_cond_t *cond)`;
- 唤醒所有的等待的线程：`int pthread_cond_broadcast(pthread_cond_t *cond)`;

### 条件变量下的多生产者多消费者
```c++
#include <stdio.h>
#include <pthread.h>
#include <stdlib.h>
#include <unistd.h>

// 链表作为容器
struct Node{
    int val;
    struct Node* next;
};

// 头结点
struct Node* head = NULL;

// 互斥量
pthread_mutex_t mutex;
// 条件变量
pthread_cond_t cond;

// 头插法增加元素
void* producter(void* arg) {
    while (1) {
        pthread_mutex_lock(&mutex);
        struct Node* newNode = (struct Node*)malloc(sizeof(struct Node));
        newNode->val = rand() % 1000;
        newNode->next = head;
        head = newNode;
        printf("add node, num : %d, tid : %ld\n", newNode->val, pthread_self());
        
        // 只要生产了一个，就通知消费者消费
        pthread_cond_signal(&cond);

        pthread_mutex_unlock(&mutex);
        usleep(100);
    }
    return NULL;
}

// 头删法减少元素
void* consumer(void* arg) {
    while (1) {
        pthread_mutex_lock(&mutex);
        struct Node* tmp = head;
        // 当链表不为空时，才能删除
        if (head != NULL) {
            head = head->next;
            printf("del node, num : %d, tid : %ld\n", tmp->val, pthread_self());
            free(tmp);
            pthread_mutex_unlock(&mutex);
            usleep(100);
        } else {
            // 没有数据，需要等待
            // 当这个函数调用阻塞的时候，会对互斥锁进行解锁，当不阻塞的，继续向下执行，会重新加锁。
            pthread_cond_wait(&cond, &mutex);
            pthread_mutex_unlock(&mutex);
        }
    }
    return NULL;
}

int main()
{
    // 初始化互斥锁
    pthread_mutex_init(&mutex, NULL);
    // 初始化条件变量
    pthread_cond_init(&cond, NULL);
    // 创建5个生产者线程，和5个消费者线程
    pthread_t products[5], consumes[5];
    for (int i = 0; i < 5; i++) {
        pthread_create(&products[i], NULL, producter, NULL);
        pthread_create(&consumes[i], NULL, consumer, NULL);
    }

    // 分离，回收线程资源
    for (int i = 0; i < 5; i++) {
        pthread_detach(products[i]);
        pthread_detach(consumes[i]);
    }

    while (1) {
        sleep(10);
    }
    // 回收条件变量
    pthread_cond_destroy(&cond);
    // 回收互斥锁
    pthread_mutex_destroy(&mutex);
    pthread_exit(NULL);     // 回收主线程
    return 0;
}
```

我们能注意到一个现象：在调用pthread_cond_wait后要释放锁
调用pthread_cond_wait()这个函数发生了两件事情：
- 把调用线程放到条件等待队列上
- mutex会被自动释放，以供其他线程改变条件
这两个步骤必须保持原子性，否则：
1. 如果先释放mutex, 这时生产者线程向队列中添加数据，然后signal。因为消费者线程现在还没被放到条件等待队列上，所以singal后什么也不会干。
2. 如果先把线程放到条件等待队列上，这时候另外一个线程发送了pthread_cond_signal（这个函数的调用不需要mutex），然后调用线程立即获取mutex。
此时mutex还没被释放，两次获取mutex会产生死锁。
所以调用pthread_cond_wait肯定会释放锁，那么调用前需要加锁。

## 信号量  
信号量也叫信号灯，本质是一个计数器，描述临界资源的数目大小

### 信号量相关操作函数
- 信号量的类型：`sem_t`
- `int sem_init(sem_t *sem, int pshared, unsigned int value)`;
    -  功能：初始化信号量
    -  参数
        - `sem`：信号量变量的地址
        - `pshared`：0 用在线程间 ，非0 用在进程间
        - `value` ：信号量中的值，代表容器大小
- `int sem_destroy(sem_t *sem)`;
    - 功能：释放资源
- `int sem_wait(sem_t *sem)`;
    - 功能：对信号量加锁，调用一次对信号量的值-1，如果值为0，就阻塞
- `int sem_trywait(sem_t *sem)`;
- `int sem_timedwait(sem_t *sem, const struct timespec *abs_timeout)`;
- `int sem_post(sem_t *sem)`;
    - 功能：对信号量解锁，调用一次对信号量的值+1
- `int sem_getvalue(sem_t *sem, int *sval)`;

### 信号量下的多生产者多消费者
不需要单独判断容器为空的情况，因为条件变量只通知临界区是否可用，而信号量可以提供临界区内资源可用信息, 即还有多少可写，还有多少可读

```c++
#include <stdio.h>
#include <pthread.h>
#include <stdlib.h>
#include <unistd.h>
#include <semaphore.h>

// 链表作为容器
struct Node{
    int val;
    struct Node* next;
};

// 头结点
struct Node* head = NULL;

// 互斥量
pthread_mutex_t mutex;
// 信号量
sem_t psem;
sem_t csem;

// 头插法增加元素
void* producter(void* arg) {
    while (1) {
        sem_wait(&psem);
        pthread_mutex_lock(&mutex);
        struct Node* newNode = (struct Node*)malloc(sizeof(struct Node));
        newNode->val = rand() % 1000;
        newNode->next = head;
        head = newNode;
        printf("add node, num : %d, tid : %ld\n", newNode->val, pthread_self());
        pthread_mutex_unlock(&mutex);
        sem_post(&csem);
    }
    return NULL;
}

// 头删法减少元素
void* consumer(void* arg) {
    while (1) {
        sem_wait(&csem);
        pthread_mutex_lock(&mutex);
        struct Node* tmp = head;
        // 当链表不为空时，才能删除
        if (head != NULL) {
            head = head->next;
            printf("del node, num : %d, tid : %ld\n", tmp->val, pthread_self());
            free(tmp);
            pthread_mutex_unlock(&mutex);
            sem_post(&psem);
        }
    }
    return NULL;
}

int main()
{
    // 初始化互斥锁
    pthread_mutex_init(&mutex, NULL);
    // 初始化信号量
    // 最多生产8个
    sem_init(&psem, 0, 8);
    // 初始没有东西可以消费
    sem_init(&csem, 0, 0);

    // 创建5个生产者线程，和5个消费者线程
    pthread_t products[5], consumes[5];
    for (int i = 0; i < 5; i++) {
        pthread_create(&products[i], NULL, producter, NULL);
        pthread_create(&consumes[i], NULL, consumer, NULL);
    }

    // 分离，回收线程资源
    for (int i = 0; i < 5; i++) {
        pthread_detach(products[i]);
        pthread_detach(consumes[i]);
    }

    while (1) {
        sleep(10);
    }
    // 回收信号量
    sem_destroy(&csem);
    sem_destroy(&psem);
    // 回收互斥锁
    pthread_mutex_destroy(&mutex);
    pthread_exit(NULL);     // 回收主线程
    return 0;
}
```

## 条件变量与信号量的区别
1. 条件变量支持广播方式唤醒等待者；信号量不支持，只能一个个通知
2. 条件变量是无状态的，如果唤醒早于等待，则唤醒会丢失；信号机制是有状态的，可以记录唤醒的次数
3. 条件变量只能结合互斥量做同步使用（因为本身没有值，只能在线程中相互通知某种共享数据的可使用状态）；信号量除了可以做同步，还能用于共享资源并发访问加锁（二进制信号量）、进程并发数量控制