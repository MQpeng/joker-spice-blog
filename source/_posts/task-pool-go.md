---
title: golang实现任务池管理器
date: 2023-08-16 11:29:29
categories: [Go,Golang,golang]
tags: [任务池,golang]
---

任务池管理器可以管理任务的调度和执行，控制并发执行的任务数量，支持动态任务添加，并在任务池为空时节省计算资源。这样可以有效地管理任务的执行，提高系统的性能和资源利用率。

项目中需要控制并发，因此需要实现一个任务池管理器，其必须具有以下作用：

1. 任务调度：任务池管理器可以接收并存储任务，并根据一定的调度策略从任务池中选择任务进行执行。在示例代码中，执行器每隔一秒从任务池中取出3个任务依次执行，保持了任务的顺序一致性。

2. 资源控制：任务池管理器可以限制并发执行的任务数量。在示例代码中，通过控制执行器每次从任务池中取出的任务数量，可以控制并发执行的任务数量。这样可以避免系统资源被过度占用，提高系统的稳定性。

3. 动态任务添加：任务池管理器支持动态添加任务。在示例代码中，通过调用 AddTask 方法可以将任务添加到任务池中。添加任务时，如果执行器没有启动，则会启动执行器。

4. 节省计算资源：当任务池为空时，执行器会节省计算资源。在示例代码中，当任务池为空时，执行器会暂停执行，并在有新任务添加到任务池时重新启动执行器。

## 定义任务池相关结构体

```go
// 任务结构体
type Task struct {
	ID   int
	Name string
}

// 任务池结构体
type TaskPool struct {
	mu             sync.Mutex
	tasks          []Task
	defaultExecutor *Executor
}
```

## 定义任务池相关方法

```go
// 初始化任务池
func NewTaskPool() *TaskPool {
	return &TaskPool{}
}
// 添加任务到任务池
func (tp *TaskPool) AddTask(task Task) {
	tp.mu.Lock()
	defer tp.mu.Unlock()

	tp.tasks = append(tp.tasks, task)

    // 如果没有定义执行器，则创建一个默认执行器
	if tp.defaultExecutor == nil {
		tp.defaultExecutor = NewExecutor()
	}
    // 如果执行器没有运行，则启动执行器
	if !tp.defaultExecutor.running {
		tp.defaultExecutor.Start(tp)
	}
}

// 获取任务池中的任务数量
func (tp *TaskPool) GetTaskCount() int {
	tp.mu.Lock()
	defer tp.mu.Unlock()
	return len(tp.tasks)
}

// 获取任务池中的指定数量任务
func (tp *TaskPool) GetTasks(count int) []Task {
	tp.mu.Lock()
	defer tp.mu.Unlock()

	if count > len(tp.tasks) {
		count = len(tp.tasks)
	}

	tasks := tp.tasks[:count]
	tp.tasks = tp.tasks[count:]

	return tasks
}
```

## 定义任务执行器结构体

```go
// 执行器结构体
type Executor struct {
	running bool
}
```

## 定义执行器相关方法

```go
// 初始化执行器
func NewExecutor() *Executor {
	return &Executor{
		running: false,
	}
}
// 启动执行器
func (e *Executor) Start(taskPool *TaskPool) {
	if e.running {
		return
	}

	e.running = true

	go func() {
		for {
			
			if len(taskPool.tasks) == 0 {
				e.running = false
				break
			}
            // 默认以3个任务分为一组
			tasks := taskPool.GetTasks(3)
			
			// taskPool.mu.Lock()
			for _, task := range tasks {
				e.executeTask(task)
			}
			// taskPool.mu.Unlock()
			time.Sleep(time.Second)
		}
	}()
}

// 执行任务
func (e *Executor) executeTask(task Task) {
	// 执行任务的逻辑
	fmt.Printf("Executing task ID: %d, Name: %s\n", task.ID, task.Name)
	time.Sleep(time.Second)
}
```

## 完整代码
```go
package main

import (
	"fmt"
	"sync"
	"time"
)

// 任务结构体
type Task struct {
	ID   int
	Name string
}

// 任务池结构体
type TaskPool struct {
	mu             sync.Mutex
	tasks          []Task
	defaultExecutor *Executor
}

// 执行器结构体
type Executor struct {
	running bool
}

// 初始化任务池
func NewTaskPool() *TaskPool {
	return &TaskPool{}
}

// 添加任务到任务池
func (tp *TaskPool) AddTask(task Task) {
	tp.mu.Lock()
	defer tp.mu.Unlock()

	tp.tasks = append(tp.tasks, task)

	if tp.defaultExecutor == nil {
		tp.defaultExecutor = NewExecutor()
	}
	if !tp.defaultExecutor.running {
		tp.defaultExecutor.Start(tp)
	}
}

// 获取任务池中的任务数量
func (tp *TaskPool) GetTaskCount() int {
	tp.mu.Lock()
	defer tp.mu.Unlock()
	return len(tp.tasks)
}

// 获取任务池中的指定数量任务
func (tp *TaskPool) GetTasks(count int) []Task {
	tp.mu.Lock()
	defer tp.mu.Unlock()

	if count > len(tp.tasks) {
		count = len(tp.tasks)
	}

	tasks := tp.tasks[:count]
	tp.tasks = tp.tasks[count:]

	return tasks
}

// 初始化执行器
func NewExecutor() *Executor {
	return &Executor{
		running: false,
	}
}

// 启动执行器
func (e *Executor) Start(taskPool *TaskPool) {
	if e.running {
		return
	}

	e.running = true

	go func() {
		for {
			
			if len(taskPool.tasks) == 0 {
				e.running = false
				break
			}

			tasks := taskPool.GetTasks(3)
			for _, task := range tasks {
				e.executeTask(task)
			}

			time.Sleep(time.Second)
		}
	}()
}

// 执行任务
func (e *Executor) executeTask(task Task) {
	fmt.Printf("Executing task ID: %d, Name: %s\n", task.ID, task.Name)
	time.Sleep(time.Second)
	// 执行任务的逻辑
}

func main() {
	taskPool := NewTaskPool()

	// 添加任务到任务池
	taskPool.AddTask(Task{ID: 1, Name: "Task 1"})
	taskPool.AddTask(Task{ID: 2, Name: "Task 2"})
	taskPool.AddTask(Task{ID: 3, Name: "Task 3"})
	taskPool.AddTask(Task{ID: 4, Name: "Task 4"})
	taskPool.AddTask(Task{ID: 5, Name: "Task 5"})

	// 添加更多任务到任务池
	taskPool.AddTask(Task{ID: 6, Name: "Task 6"})
	taskPool.AddTask(Task{ID: 7, Name: "Task 7"})
	taskPool.AddTask(Task{ID: 8, Name: "Task 8"})

	// 等待执行器完成任务
	time.Sleep(15 * time.Second)

	taskPool.AddTask(Task{ID: 6, Name: "Task 6"})
	taskPool.AddTask(Task{ID: 7, Name: "Task 7"})
	taskPool.AddTask(Task{ID: 8, Name: "Task 8"})

	time.Sleep(15 * time.Second)
}
// Output
// Executing task ID: 1, Name: Task 1
// Executing task ID: 2, Name: Task 2
// Executing task ID: 3, Name: Task 3
// Executing task ID: 4, Name: Task 4
// Executing task ID: 5, Name: Task 5
// Executing task ID: 6, Name: Task 6
// Executing task ID: 7, Name: Task 7
// Executing task ID: 8, Name: Task 8
// Executing task ID: 6, Name: Task 6
// Executing task ID: 7, Name: Task 7
// Executing task ID: 8, Name: Task 8
```