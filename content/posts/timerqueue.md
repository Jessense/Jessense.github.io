---
title: "手写定时器"
date: 2022-04-08T10:29:00+08:00
draft: false
---

```cpp
#include<iostream>
#include<vector>
#include<map>
#include<queue>
#include<time.h>
#include<pthread.h>
#include<unistd.h>
using namespace std;

typedef void*(*callback_t) (void*);
struct timer {
    timespec expire;
    callback_t callback;
    timer(timespec expire_, callback_t callback_)
        : expire(expire_), callback(callback_) 
    {
    }
};

// 主线程添加的 timer，子线程需要处理
vector<timer> pending_timers;
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t cond = PTHREAD_COND_INITIALIZER;

timespec get_epoch_now() {
    timespec now;
    clock_gettime(CLOCK_REALTIME, &now);
    return now;
}

bool earlier(const timespec& a, const timespec& b) {
    return a.tv_sec < b.tv_sec 
        || (a.tv_sec == b.tv_sec && a.tv_nsec < b.tv_nsec);
}

void* eventloop(void* data) {
    auto cmp = [](timer a, timer b) {
         return earlier(b.expire, a.expire);
    };
    priority_queue<timer, vector<timer>, decltype(cmp)> q(cmp);
    while (true) {
        if (q.empty()) {
            pthread_mutex_lock(&mutex);
            while (pending_timers.empty()) {
                pthread_cond_wait(&cond, &mutex);
            }
            for (auto t : pending_timers) {
                q.push(t);
            }
            pending_timers.clear();
            pthread_mutex_unlock(&mutex);  
        } else {
            timer next_timer = q.top();
            pthread_mutex_lock(&mutex);
            int err = 0;
            // 等待 next_timer expire 或主线程添加新的 timer
            while (pending_timers.empty() && err != ETIMEDOUT) {
                // timewait 接受的是绝对时间
                err = pthread_cond_timedwait(&cond, &mutex, &next_timer.expire);
            }
            if (err == ETIMEDOUT) { //如果是 next_timer expire
                pthread_t tid;
                pthread_create(&tid, NULL, next_timer.callback, NULL);
                pthread_detach(tid);
                q.pop();
            }
            if (!pending_timers.empty()) { // 如果是主线程添加了 timer
                for (auto t : pending_timers) {
                    q.push(t);
                }
                pending_timers.clear();                
            }
            
            pthread_mutex_unlock(&mutex);
        }
    }
}

void* func1(void* data) {
    timespec now = get_epoch_now();
    cout << now.tv_sec << " : " << "fun1" << endl;
    return NULL;
}

void* func2(void* data) {
    timespec now = get_epoch_now();
    cout << now.tv_sec << " : " << "fun2" << endl;
    return NULL;
}

void* func3(void* data) {
    timespec now = get_epoch_now();
    cout << now.tv_sec << " : " << "fun3" << endl;
    return NULL;
}

// seconds 和 nanoseconds 是相对该函数调用时的时间
void register_timer(time_t seconds, long nanoseconds, callback_t callback) {
    timespec now = get_epoch_now();
    // 绝对时间 target = now + 相对时间
    timespec target = {
        now.tv_sec + seconds,
        now.tv_nsec + nanoseconds
    };
    if (target.tv_nsec > 1e9) {
        target.tv_sec += 1;
        target.tv_nsec -= 1e9;
    }
    pthread_mutex_lock(&mutex);
    pending_timers.emplace_back(target, callback);
    pthread_cond_signal(&cond);
    pthread_mutex_unlock(&mutex);

}

int main()
{
    timespec now = get_epoch_now();
    cout << now.tv_sec << endl;    
    pthread_t t;
    pthread_create(&t, NULL, eventloop, NULL);
    register_timer(4, 0, func1); // now+4s 时执行 fun1
    register_timer(6, 0, func3); // now+6s 时执行 fun3
    register_timer(2, 0, func2); // now+2s 时执行 fun1

    pthread_join(t, NULL);
    return 0;
}
```

