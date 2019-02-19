---
layout: post
title: Leetcode题目 CircleQueue
tags:
    - 算法
---

Leetcode题目 CircleQueue
===
昨天开始看leetcode上面的题目,彩笔的第二题就卡了两天.   
如题,环形队列  
先贴上最终实现的代码  (不给用swift.......)

```java
class MyCircularQueue {
    private int[] datas;
    private int q_head;
    private int q_tail;
    private int q_current;
    private int length;
    /** Initialize your data structure here. Set the size of the queue to be k. */
    public MyCircularQueue(int k) {
        datas = new int[k];
        length = k;
        q_head = 0;
        q_tail = -1;
        q_current = 0;
        for (int i = 0 ; i < length ; i ++ ) {
            datas[i] = -1;
        }


    }

    /** Insert an element into the circular queue. Return true if the operation is successful. */
    public boolean enQueue(int value) {
        if (isFull() == true) {
            return false;
        }

        q_tail = (q_tail + 1) % length;
        datas[q_tail] = value;
        q_current ++;
        return true;
    }

    /** Delete an element from the circular queue. Return true if the operation is successful. */
    public boolean deQueue() {
        if (isEmpty() == true) {
            return false;
        }

        datas[q_head] = -1;
        q_head = (q_head + 1) % length;
        q_current --;
        return true;
    }

    /** Get the front item from the queue. */
    public int Front() {

        if (isEmpty()) {
            return -1;
        }

        return datas[q_head];
    }

    /** Get the last item from the queue. */
    public int Rear() {
        if (isEmpty()) {
            return -1;
        }
        return datas[q_tail];
    }

    /** Checks whether the circular queue is empty or not. */
    public boolean isEmpty() {
        if (q_current == 0) {
            return true;
        }

        return false;
    }

    /** Checks whether the circular queue is full or not. */
    public boolean isFull() {
        if (q_current == length) {

            return true;
        }
        return false;
    }
}
```


其实最终实现起来并不难,但是我钻牛角尖了,以为必须用最少的属性来解决,所以在判断是否为空,是否为满上面卡了很久  
其实只需要定义一个size 属性就行了  
其次就是初始值的问题,两种思路  

* 初始head和tail都为0,这样在入队和出队的时候都需要判断一下是否是初始状态,因为第一次入队的时候tail指针不走
* 初始head为0, tail为-1, 这样,就不需要再入队的时候判断,但是要注意在 获取尾部元素的时候,要判断如果是空的时候,直接返回 -1, 不然,取值的时候会报错


另外,leetcode给的代码还加了一个判断,在出队判断为空的时候,直接将head 和 tail 置为初始值  

贴上官方的示例代码,这个代码是跟示例的幻灯片一样的  

```java
class MyCircularQueue {
    
    private int[] data;
    private int head;
    private int tail;
    private int size;

    /** Initialize your data structure here. Set the size of the queue to be k. */
    public MyCircularQueue(int k) {
        data = new int[k];
        head = -1;
        tail = -1;
        size = k;
    }
    
    /** Insert an element into the circular queue. Return true if the operation is successful. */
    public boolean enQueue(int value) {
        if (isFull() == true) {
            return false;
        }
        if (isEmpty() == true) {
            head = 0;
        }
        tail = (tail + 1) % size;
        data[tail] = value;
        return true;
    }
    
    /** Delete an element from the circular queue. Return true if the operation is successful. */
    public boolean deQueue() {
        if (isEmpty() == true) {
            return false;
        }
        if (head == tail) {
            head = -1;
            tail = -1;
            return true;
        }
        head = (head + 1) % size;
        return true;
    }
    
    /** Get the front item from the queue. */
    public int Front() {
        if (isEmpty() == true) {
            return -1;
        }
        return data[head];
    }
    
    /** Get the last item from the queue. */
    public int Rear() {
        if (isEmpty() == true) {
            return -1;
        }
        return data[tail];
    }
    
    /** Checks whether the circular queue is empty or not. */
    public boolean isEmpty() {
        return head == -1;
    }
    
    /** Checks whether the circular queue is full or not. */
    public boolean isFull() {
        return ((tail + 1) % size) == head;
    }
}
```

刚开始就卡了这么久,反而让我更有动力了,没有挑战性的东西是没有意义的,哈哈