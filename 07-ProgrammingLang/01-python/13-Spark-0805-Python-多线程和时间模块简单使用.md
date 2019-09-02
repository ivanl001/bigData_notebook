## 1, 线程基础使用
```python
#
# author      : ivanl001
# creator     : 2018-12-06 16:21
# description : 
# 这个是高级接口，它的低级接口是_thread

import threading

# 这个是threading的低级接口
import _thread

# 定义一个函数，方便多线程演示
def hello():
    tname = threading.current_thread().getName()
    print(tname)
    print("ivanl001 is the king of world!")
    for i in [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]:
        print(tname + ":" + str(i))

# 注意一下：这里就算没有参数，也需要传递进去一个空的元祖
threading._start_new_thread(hello,())
threading._start_new_thread(hello,())
#_thread.start_new_thread(hello, ())

# 这里一定要注意：主线程不能死，祝线程一旦死掉，分线程就不能打印出内容了
# 如果这样的话，比较容易太耗资源,所以还是和java一样，用休眠的方法
# while True:
#     pass

import time

# 获取当前毫秒值
current = int(time.time()*1000)
print(current)

# 利用time模块让主进程休眠2秒钟
time.sleep(2)

```

## 2，多线程的卖票问题

```python
#
# author      : ivanl001
# creator     : 2018-12-08 12:16
# description : 
# 

import threading
import _thread

import time


class TicketSeller(threading.Thread):

    # 这种变量请使用类名进行调用TicketSeller.ticketCount
    ticketCount = 100

    def run(self):
        # 父类其实没什么实现，所以这里不需要继承
        # super().run()

        for i in range(TicketSeller.ticketCount):
            if TicketSeller.ticketCount <= 0:
                print("票已经卖完")
                break

            print("线程是：" + self.name + "-----" + str(TicketSeller.ticketCount))
            TicketSeller.ticketCount -= 1
            time.sleep(2)


seller01 = TicketSeller()
# 这里的名字是线程的方法哈
seller01.setName("ivanl001")


seller02 = TicketSeller()
# # 这里的名字是线程的方法哈
seller02.setName("ivanl002")

seller01.start()
seller02.start()

# time.sleep(3)
# print(TicketSeller.ticketCount)
```

## 3，多线程卖票问题解决：加锁

```python
#
# author      : ivanl001
# creator     : 2018-12-08 12:16
# description : 
# 

import threading
import _thread

import time


class TicketSeller(threading.Thread):

    # 这种变量请使用类名进行调用TicketSeller.ticketCount
    ticketCount = 100

    # 注意：这里要加括号哈
    lock = threading.Lock()


    def run(self):
        # 父类其实没什么实现，所以这里不需要继承
        # super().run()

        for i in range(TicketSeller.ticketCount):
            if TicketSeller.ticketCount <= 0:
                print("票已经卖完")
                break

            # 在这里加锁
            TicketSeller.lock.acquire()
            print("线程是：" + self.name + "-----" + str(TicketSeller.ticketCount))
            TicketSeller.ticketCount -= 1
            # 这里休眠一秒是为了显示出购票的问题
            # time.sleep(1)
            # 在这里释放锁
            TicketSeller.lock.release()



seller01 = TicketSeller()
# 这里的名字是线程的方法哈
seller01.setName("ivanl001")


seller02 = TicketSeller()
# # 这里的名字是线程的方法哈
seller02.setName("ivanl002")

seller01.start()
seller02.start()

# time.sleep(3)
# print(TicketSeller.ticketCount)
```

