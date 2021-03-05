---
title: "LeetCode 232:用栈实现队列"
date: 2021-03-05T14:58:29+08:00
draft: true
toc: false
images:
tags: 
  - LeetCode
---

### 题目

请你仅使用两个栈实现先入先出队列。队列应当支持一般队列的支持的所有操作（push、pop、peek、empty）：

实现 MyQueue 类：

* void push(int x) 将元素 x 推到队列的末尾
* int pop() 从队列的开头移除并返回元素
* int peek() 返回队列开头的元素
* boolean empty() 如果队列为空，返回 true ；否则，返回 false


说明：

* 你只能使用标准的栈操作 —— 也就是只有 push to top, peek/pop from top, size, 和 is empty 操作是合法的。
* 你所使用的语言也许不支持栈。你可以使用 list 或者 deque（双端队列）来模拟一个栈，只要是标准的栈操作即可。


进阶：

* 你能否实现每个操作均摊时间复杂度为 O(1) 的队列？换句话说，执行 n 个操作的总时间复杂度为 O(n) ，即使其中一个操作可能花费较长时间。


示例：

```
输入：
["MyQueue", "push", "push", "peek", "pop", "empty"]
[[], [1], [2], [], [], []]
输出：
[null, null, null, 1, 1, false]
```

解释：
```
MyQueue myQueue = new MyQueue();
myQueue.push(1); // queue is: [1]
myQueue.push(2); // queue is: [1, 2] (leftmost is front of the queue)
myQueue.peek(); // return 1
myQueue.pop(); // return 1, queue is [2]
myQueue.empty(); // return false
```

提示：

1 <= x <= 9
最多调用 100 次 push、pop、peek 和 empty
假设所有操作都是有效的 （例如，一个空的队列不会调用 pop 或者 peek 操作）

>  来源：力扣（LeetCode）
> 链接：https://leetcode-cn.com/problems/implement-queue-using-stacks
> 著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

### 思路

这题比较简单，主要是实现队列的基本功能，即先进先出。而我们知道栈是先进后出的，这就需要我们额外的操作了。

题目提示的比较明显，使用两个栈来实现。我们可以做出如下模型：

* 两个栈分别为输入栈和输出栈
* 输入栈负责接收 push 的内容
* 输出栈负责 pop 和 peek 的内容
* 当执行 pop 或者 peek 时，当输出栈为空时，将输入栈的内容输出到输入栈中

```
in : []
out: []

--push(1) 
in : [1]
out: []

--push(2)
in : [2,1]
out: []

--peek
in : []
out: [1,2]
peek = 1

--pop
in : []
out: [2]
pop = 1

--isEmpty
isEmpty(in) && isEmpty(out)

```

### 代码

```go

// MyQueue 队列
type MyQueue struct {
	in  Stack
	out Stack
}

// Constructor MyQueue构造方法
func Constructor() MyQueue {
	in := new(Stack)
	out := new(Stack)
	return MyQueue{
		in:  *in,
		out: *out,
	}
}
func (mq *MyQueue) in2out() {
	for len(mq.in) > 0 {
		mq.out.Push(mq.in.Pop())
	}
}

// Push Push
func (mq *MyQueue) Push(x int) {
	mq.in.Push(x)
}

// Pop Pop
func (mq *MyQueue) Pop() int {
	if mq.out.Len() == 0 {
		mq.in2out()
	}
	return mq.out.Pop()
}

// Peek Peek
func (mq *MyQueue) Peek() int {
	if mq.out.Len() == 0 {
		mq.in2out()
	}
	return mq.out.Top()
}

// Empty Empty
func (mq *MyQueue) Empty() bool {
	return mq.in.IsEmpty() && mq.out.IsEmpty()
}

// -------------------------------------------------------

// Stack 栈
type Stack []int

// Len 获取栈的长度
func (stack *Stack) Len() int {
	return len(*stack)
}

// Push push
func (stack *Stack) Push(value int) {
	*stack = append(*stack, value)
}

// Top 获取栈的第一个
func (stack *Stack) Top() int {
	return (*stack)[len(*stack)-1]
}

// Pop 弹出最后一个
func (stack *Stack) Pop() int {
	value := (*stack)[len(*stack)-1]
	*stack = (*stack)[:len(*stack)-1]
	return value
}

// IsEmpty 判断是否为空
func (stack *Stack) IsEmpty() bool {
	return len(*stack) == 0
}

```

