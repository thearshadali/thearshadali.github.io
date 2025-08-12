---
layout: post
title: Concurrency in Golang
---
Let's learn concurrency from "Zero to Hero" in Go, covering key concepts with code examples and exercises. The course will progress from basic to advanced topics, ensuring a clear learning path. Each lesson includes explanations, code examples, and exercises to reinforce understanding. Let's dive in!

# Baby Step 

## Beginner-Friendly Zero to Hero: Concurrency in Go

Concurrency in Go lets you run multiple tasks at the same time, like cooking dinner while watching TV. Go makes this easy with **goroutines** (like mini-programs running together) and **channels** (a way for them to talk to each other). This course starts from scratch and builds step-by-step. Each lesson has a simple explanation, a code example, and an exercise to practice.

---

### Lesson 1: What Are Goroutines?

**Concept**: A goroutine is like a helper that does a task for you while you do something else. You start it with the `go` keyword, and it runs at the same time as your main program.

**Why It’s Cool**: Goroutines are super lightweight, so you can run thousands of them without slowing down your computer.

**Code Example**:
```go
package main

import (
	"fmt"
	"time"
)

func sayHello() {
	fmt.Println("Hello from the helper!")
}

func main() {
	// Start the helper
	go sayHello()
	
	// Main program says something
	fmt.Println("Hello from main!")
	
	// Wait a bit so the helper can finish
	time.Sleep(time.Second)
}
```

**What’s Happening**:
- `go sayHello()` starts the `sayHello` function as a goroutine, like asking a friend to say "Hello" while you do something else.
- The main program prints its message, and the goroutine prints its own.
- `time.Sleep` pauses the program to give the goroutine time to run (we’ll learn a better way to wait later).

**Output** (might vary slightly):
```
Hello from main!
Hello from the helper!
```

**Exercise**:
1. Write a program with two goroutines: one prints "I’m working!" and the other prints "I’m playing!" five times each.
2. Run the program and see what happens. Do the messages mix up? Why?
3. Add a third goroutine that prints "I’m sleeping!" three times.

**Sample Solution**:
```go
package main

import (
	"fmt"
	"time"
)

func work() {
	for i := 0; i < 5; i++ {
		fmt.Println("I’m working!")
	}
}

func play() {
	for i := 0; i < 5; i++ {
		fmt.Println("I’m playing!")
	}
}

func sleep() {
	for i := 0; i < 3; i++ {
		fmt.Println("I’m sleeping!")
	}
}

func main() {
	go work()
	go play()
	go sleep()
	
	time.Sleep(time.Second * 2)
	fmt.Println("Main done!")
}
```

**Exercise Notes**: The messages might appear in a random order because goroutines run independently, and the Go scheduler decides when each one gets to print. This shows how concurrency works—things happen at the same time!

---

### Lesson 2: Waiting for Goroutines with WaitGroup

**Concept**: Sometimes, you need to wait for all your helpers (goroutines) to finish before ending the program. The `sync.WaitGroup` is like a checklist: you add tasks, mark them done, and wait until everything’s checked off.

**Key Points**:
- `Add(n)`: Tell the checklist how many goroutines to wait for.
- `Done()`: Mark a goroutine as finished.
- `Wait()`: Pause until all goroutines are done.

**Code Example**:
```go
package main

import (
	"fmt"
	"sync"
)

func task(id int, wg *sync.WaitGroup) {
	fmt.Printf("Task %d is running\n", id)
	// Pretend to do some work
	time.Sleep(time.Second)
	fmt.Printf("Task %d is done\n", id)
	wg.Done() // Check this task off
}

func main() {
	var wg sync.WaitGroup
	
	// Start three tasks
	wg.Add(3) // Add 3 tasks to the checklist
	go task(1, &wg)
	go task(2, &wg)
	go task(3, &wg)
	
	wg.Wait() // Wait for all tasks to finish
	fmt.Println("All tasks are done!")
}
```

**What’s Happening**:
- We create a `WaitGroup` to track three goroutines.
- Each `task` prints a message, waits a second (like doing work), and calls `wg.Done()` to say it’s finished.
- `wg.Wait()` makes the main program wait until all tasks are done.

**Output**:
```
Task 1 is running
Task 2 is running
Task 3 is running
Task 1 is done
Task 2 is done
Task 3 is done
All tasks are done!
```

**Exercise**:
1. Write a program with 4 goroutines, each printing "Task X finished" (where X is 1 to 4) after waiting 1 second.
2. Use a `WaitGroup` to ensure "All done!" prints only after all tasks finish.
3. Change one task to wait 2 seconds. Does it affect when "All done!" prints?

**Sample Solution**:
```go
package main

import (
	"fmt"
	"sync"
	"time"
)

func task(id int, duration time.Duration, wg *sync.WaitGroup) {
	fmt.Printf("Task %d is running\n", id)
	time.Sleep(duration)
	fmt.Printf("Task %d finished\n", id)
	wg.Done()
}

func main() {
	var wg sync.WaitGroup
	
	wg.Add(4)
	go task(1, time.Second, &wg)
	go task(2, time.Second, &wg)
	go task(3, time.Second, &wg)
	go task(4, 2*time.Second, &wg)
	
	wg.Wait()
	fmt.Println("All done!")
}
```

**Exercise Notes**: The program waits for the slowest task (2 seconds) before printing "All done!" because `WaitGroup` ensures all goroutines finish.

---

### Lesson 3: Talking Between Goroutines with Channels

**Concept**: Channels are like a pipe that goroutines use to send and receive data, like passing notes between friends. They help goroutines work together safely.

**Key Points**:
- Create a channel with `make(chan Type)`.
- Send data with `ch <- value`.
- Receive data with `value := <-ch`.
- Channels make sure only one goroutine touches the data at a time.

**Code Example**:
```go
package main

import (
	"fmt"
)

func sendMessage(ch chan string) {
	ch <- "Hello from the sender!" // Send a message
}

func main() {
	ch := make(chan string) // Create a channel for strings
	
	go sendMessage(ch) // Start the sender
	
	message := <-ch // Receive the message
	fmt.Println(message)
}
```

**What’s Happening**:
- `make(chan string)` creates a channel for strings.
- The `sendMessage` goroutine sends a message through the channel.
- The main program waits to receive the message and prints it.

**Output**:
```
Hello from the sender!
```

**Exercise**:
1. Write a program where a goroutine sends your name through a channel, and the main function prints "Hello, [name]!".
2. Modify it to send three numbers (e.g., 1, 2, 3) through a channel and print them in the main function.
3. Make two goroutines send numbers (e.g., 1-3 and 4-6) to the same channel, and print all six numbers.

**Sample Solution** (for exercise 3):
```go
package main

import (
	"fmt"
	"sync"
)

func sendNumbers(start, end int, ch chan int, wg *sync.WaitGroup) {
	defer wg.Done()
	for i := start; i <= end; i++ {
		ch <- i
	}
}

func main() {
	ch := make(chan int)
	var wg sync.WaitGroup
	
	wg.Add(2)
	go sendNumbers(1, 3, ch, &wg)
	go sendNumbers(4, 6, ch, &wg)
	
	// Receive numbers in a separate goroutine
	go func() {
		wg.Wait()
		close(ch) // Close channel when all sends are done
	}()
	
	// Print all numbers
	for num := range ch {
		fmt.Println("Received:", num)
	}
}
```

**Exercise Notes**: We use a `WaitGroup` to know when to close the channel. The `for ... range` loop reads all numbers until the channel is closed. The order of numbers might vary because both goroutines send concurrently.

---

### Lesson 4: Buffered Channels for Flexibility

**Concept**: A buffered channel is like a pipe with extra space to hold data, so the sender doesn’t always wait for the receiver. It’s like leaving notes in a mailbox.

**Key Points**:
- Create with `make(chan Type, size)` where `size` is how many items it can hold.
- The sender can keep sending until the buffer is full.
- The receiver can take items later.

**Code Example**:
```go
package main

import (
	"fmt"
)

func main() {
	ch := make(chan string, 2) // Buffer can hold 2 strings
	
	// Send two messages
	ch <- "Message 1"
	ch <- "Message 2"
	
	// Receive and print
	fmt.Println(<-ch)
	fmt.Println(<-ch)
}
```

**What’s Happening**:
- The channel has a buffer of 2, so it can hold two messages without waiting.
- We send two messages, and they wait in the buffer.
- We receive and print them one by one.

**Output**:
```
Message 1
Message 2
```

**Exercise**:
1. Create a buffered channel with space for 3 integers. Send 3 numbers and print them.
2. Try sending 4 numbers to the same channel. What happens? (Hint: It might crash!)
3. Write a program where a goroutine sends 5 messages to a buffered channel (size 3), and the main function receives them all.

**Sample Solution** (for exercise 3):
```go
package main

import (
	"fmt"
	"sync"
)

func sendMessages(ch chan string, wg *sync.WaitGroup) {
	defer wg.Done()
	messages := []string{"Hi", "How", "Are", "You", "!"}
	for _, msg := range messages {
		ch <- msg
	}
}

func main() {
	ch := make(chan string, 3)
	var wg sync.WaitGroup
	
	wg.Add(1)
	go sendMessages(ch, &wg)
	
	go func() {
		wg.Wait()
		close(ch)
	}()
	
	for msg := range ch {
		fmt.Println("Received:", msg)
	}
}
```

**Exercise Notes**: The buffer holds up to 3 messages, so the sender might pause if it tries to send more before the receiver takes some. Closing the channel ensures the receiver stops when all messages are sent.

---

### Lesson 5: Choosing Between Channels with Select

**Concept**: The `select` statement is like waiting for multiple phones to ring and answering the first one. It lets a goroutine handle multiple channels at once.

**Key Points**:
- `select` picks the first channel that’s ready.
- If no channel is ready, it waits (unless there’s a `default` case).
- Great for handling different tasks at the same time.

**Code Example**:
```go
package main

import (
	"fmt"
	"time"
)

func sendSlow(ch chan string) {
	time.Sleep(time.Second)
	ch <- "Slow message"
}

func sendFast(ch chan string) {
	ch <- "Fast message"
}

func main() {
	ch1 := make(chan string)
	ch2 := make(chan string)
	
	go sendSlow(ch1)
	go sendFast(ch2)
	
	select {
	case msg1 := <-ch1:
		fmt.Println("Got:", msg1)
	case msg2 := <-ch2:
		fmt.Println("Got:", msg2)
	}
}
```

**What’s Happening**:
- Two goroutines send messages at different speeds.
- `select` waits for the first message to arrive and prints it.
- Since `sendFast` is faster, it’s usually picked first.

**Output** (likely):
```
Got: Fast message
```

**Exercise**:
1. Write a program with two goroutines: one sends "Apples" after 1 second, the other sends "Oranges" after 2 seconds. Use `select` to print the first message received.
2. Add a third goroutine that sends "Bananas" immediately. Which message is printed most often?
3. Add a `default` case that prints "Waiting..." if no channel is ready, and loop until a message is received.

**Sample Solution** (for exercise 3):
```go
package main

import (
	"fmt"
	"time"
)

func sendApples(ch chan string) {
	time.Sleep(time.Second)
	ch <- "Apples"
}

func sendOranges(ch chan string) {
	time.Sleep(2 * time.Second)
	ch <- "Oranges"
}

func sendBananas(ch chan string) {
	ch <- "Bananas"
}

func main() {
	ch1 := make(chan string)
	ch2 := make(chan string)
	ch3 := make(chan string)
	
	go sendApples(ch1)
	go sendOranges(ch2)
	go sendBananas(ch3)
	
	for {
		select {
		case msg1 := <-ch1:
			fmt.Println("Got:", msg1)
			return
		case msg2 := <-ch2:
			fmt.Println("Got:", msg2)
			return
		case msg3 := <-ch3:
			fmt.Println("Got:", msg3)
			return
		default:
			fmt.Println("Waiting...")
			time.Sleep(time.Millisecond * 100)
		}
	}
}
```

**Exercise Notes**: The `default` case keeps the program from getting stuck, and "Bananas" is usually received first because it’s sent immediately. The loop stops after the first message.

---

### Lesson 6: Protecting Shared Data with Mutex

**Concept**: When multiple goroutines change the same data, they can mess it up (like two people editing the same document). A `sync.Mutex` is like a lock that lets only one goroutine touch the data at a time.

**Key Points**:
- `Lock()`: Grab the lock (only one goroutine at a time).
- `Unlock()`: Release the lock so others can use it.
- Always unlock after locking to avoid freezing the program.

**Code Example**:
```go
package main

import (
	"fmt"
	"sync"
)

type Counter struct {
	count int
	mu    sync.Mutex
}

func (c *Counter) Add() {
	c.mu.Lock()   // Lock before changing count
	c.count++
	c.mu.Unlock() // Unlock after
}

func (c *Counter) Get() int {
	c.mu.Lock()
	defer c.mu.Unlock() // Unlock automatically when done
	return c.count
}

func main() {
	var wg sync.WaitGroup
	counter := Counter{}
	
	// Start 10 goroutines that add 1
	wg.Add(10)
	for i := 0; i < 10; i++ {
		go func() {
			counter.Add()
			wg.Done()
		}()
	}
	
	wg.Wait()
	fmt.Println("Final count:", counter.Get())
}
```

**What’s Happening**:
- `Counter` holds a number (`count`) and a lock (`mu`).
- Each goroutine calls `Add()` to increase `count`, but the lock ensures only one changes it at a time.
- `Get()` safely reads the count with the lock.

**Output**:
```
Final count: 10
```

**Exercise**:
1. Write a program with a `Counter` that 5 goroutines increment 3 times each. Print the final count (should be 15).
2. Remove the mutex and run the program. Does the count change? Why?
3. Add a method to decrease the count safely and have 3 goroutines decrease it twice each. What’s the final count?

**Sample Solution** (for exercise 3):
```go
package main

import (
	"fmt"
	"sync"
)

type Counter struct {
	count int
	mu    sync.Mutex
}

func (c *Counter) Add() {
	c.mu.Lock()
	c.count++
	c.mu.Unlock()
}

func (c *Counter) Subtract() {
	c.mu.Lock()
	c.count--
	c.mu.Unlock()
}

func (c *Counter) Get() int {
	c.mu.Lock()
	defer c.mu.Unlock()
	return c.count
}

func main() {
	var wg sync.WaitGroup
	counter := Counter{}
	
	// 5 goroutines add 3 times
	wg.Add(5)
	for i := 0; i < 5; i++ {
		go func() {
			for j := 0; j < 3; j++ {
				counter.Add()
			}
			wg.Done()
		}()
	}
	
	// 3 goroutines subtract 2 times
	wg.Add(3)
	for i := 0; i < 3; i++ {
		go func() {
			for j := 0; j < 2; j++ {
				counter.Subtract()
			}
			wg.Done()
		}()
	}
	
	wg.Wait()
	fmt.Println("Final count:", counter.Get())
}
```

**Exercise Notes**: Without a mutex, the count might be wrong due to "race conditions" (goroutines stepping on each other). The final count is 15 (5 * 3) - (3 * 2) = 9.

---

### Lesson 7: Stopping Goroutines with Context

**Concept**: A `context` is like a signal to tell goroutines when to stop, like saying "Time’s up!" during a game. It’s useful for cancelling tasks if something goes wrong or takes too long.

**Key Points**:
- Create a context with `context.WithCancel()` or `context.WithTimeout()`.
- Pass it to goroutines, which check if it’s "cancelled."
- Call `cancel()` to stop everything.

**Code Example**:
```go
package main

import (
	"context"
	"fmt"
	"time"
)

func worker(ctx context.Context, id int) {
	for {
		select {
		case <-ctx.Done(): // Check if cancelled
			fmt.Printf("Worker %d stopped\n", id)
			return
		default:
			fmt.Printf("Worker %d working\n", id)
			time.Sleep(time.Millisecond * 500)
		}
	}
}

func main() {
	ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
	defer cancel() // Always clean up
	
	go worker(ctx, 1)
	go worker(ctx, 2)
	
	time.Sleep(3 * time.Second) // Let it run a bit
	fmt.Println("Main done")
}
```

**What’s Happening**:
- The context cancels after 2 seconds.
- Workers keep working until they see the cancel signal, then stop.
- `defer cancels()` ensures the context is cleaned up.

**Output**:
```
Worker 1 working
Worker 2 working
Worker 1 working
Worker 2 working
Worker 1 stopped
Worker 2 stopped
Main done
```

**Exercise**:
1. Write a program with one worker that prints "Working..." every 0.5 seconds until cancelled after 3 seconds.
2. Add a second worker that prints a different message (e.g., "Busy..."). Do both stop when cancelled?
3. Change the timeout to 1 second. How does it affect the output?

**Sample Solution** (for exercise 2):
```go
package main

import (
	"context"
	"fmt"
	"time"
)

func worker(ctx context.Context, id int, message string) {
	for {
		select {
		case <-ctx.Done():
			fmt.Printf("Worker %d stopped\n", id)
			return
		default:
			fmt.Println(message)
			time.Sleep(time.Millisecond * 500)
		}
	}
}

func main() {
	ctx, cancel := context.WithTimeout(context.Background(), 3*time.Second)
	defer cancel()
	
	go worker(ctx, 1, "Working...")
	go worker(ctx, 2, "Busy...")
	
	time.Sleep(4 * time.Second)
	fmt.Println("Main done")
}
```

**Exercise Notes**: Both workers stop when the context times out after 3 seconds. A shorter timeout (1 second) means fewer messages are printed before stopping.

---

### Lesson 8: Simple Worker Pool

**Concept**: A worker pool is like hiring a few helpers to do many tasks. You give them jobs through a channel, and they work together to finish everything.

**Key Points**:
- Use a channel to send tasks to workers.
- Limit the number of workers to control how many run at once.
- Use `WaitGroup` to wait for all tasks to finish.

**Code Example**:
```go
package main

import (
	"fmt"
	"sync"
)

func worker(id int, jobs <-chan int, wg *sync.WaitGroup) {
	defer wg.Done()
	for job := range jobs {
		fmt.Printf("Worker %d doing job %d\n", id, job)
	}
}

func main() {
	jobs := make(chan int, 5)
	var wg sync.WaitGroup
	
	// Start 2 workers
	wg.Add(2)
	go worker(1, jobs, &wg)
	go worker(2, jobs, &wg)
	
	// Send 5 jobs
	for i := 1; i <= 5; i++ {
		jobs <- i
	}
	close(jobs) // No more jobs
	
	wg.Wait()
	fmt.Println("All jobs done!")
}
```

**What’s Happening**:
- Two workers take jobs from the `jobs` channel.
- We send 5 jobs (numbers 1 to 5), and the workers share them.
- Closing the channel tells workers to stop when done.

**Output**:
```
Worker 1 doing job 1
Worker 2 doing job 2
Worker 1 doing job 3
Worker 2 doing job 4
Worker 1 doing job 5
All jobs done!
```

**Exercise**:
1. Write a worker pool with 3 workers that process 6 tasks (print "Task X done" for each).
2. Change the tasks to strings (e.g., "Wash dishes", "Clean room"). Have workers print which task they’re doing.
3. Make each worker wait 1 second per task to simulate work. Does it take longer with 1 worker vs. 3?

**Sample Solution** (for exercise 2):
```go
package main

import (
	"fmt"
	"sync"
	"time"
)

func worker(id int, jobs <-chan string, wg *sync.WaitGroup) {
	defer wg.Done()
	for job := range jobs {
		fmt.Printf("Worker %d doing %s\n", id, job)
		time.Sleep(time.Second)
	}
}

func main() {
	jobs := make(chan string, 6)
	var wg sync.WaitGroup
	
	wg.Add(3)
	go worker(1, jobs, &wg)
	go worker(2, jobs, &wg)
	go worker(3, jobs, &wg)
	
	tasks := []string{"Wash dishes", "Clean room", "Do laundry", "Cook dinner", "Take out trash", "Water plants"}
	for _, task := range tasks {
		jobs <- task
	}
	close(jobs)
	
	wg.Wait()
	fmt.Println("All jobs done!")
}
```

**Exercise Notes**: With 3 workers, the 6 tasks finish faster than with 1 worker because they’re done in parallel. Each task takes 1 second, so 6 tasks with 3 workers take about 2 seconds total.

---

### Course Summary

You’ve learned the basics of concurrency in Go:
1. **Goroutines**: Run tasks at the same time, like helpers.
2. **WaitGroup**: Wait for all helpers to finish.
3. **Channels**: Pass messages safely between goroutines.
4. **Buffered Channels**: Store messages like a mailbox.
5. **Select**: Pick the first message from multiple channels.
6. **Mutex**: Protect shared data like a lock.
7. **Context**: Stop goroutines when needed.
8. **Worker Pool**: Share tasks among a few helpers.

**What’s Next**:
- Try combining these ideas in a small project, like a program that downloads multiple images at once.
- Read about Go’s concurrency in the official docs: https://golang.org/doc/effective_go#concurrency.
- Practice by modifying the exercises (e.g., add more workers or different tasks).




# Step 2 -

## Zero to Hero: Concurrency in Go

This course is designed for beginners to advanced learners, covering Go's concurrency model, which leverages goroutines, channels, and other primitives. Each lesson builds on the previous one, with practical examples and exercises to solidify your understanding.

---

### Lesson 1: Introduction to Goroutines

**Concept**: Goroutines are lightweight threads managed by the Go runtime, not OS threads. They allow functions to run concurrently, making it easy to perform tasks simultaneously.

**Key Points**:
- Use the `go` keyword to start a goroutine.
- Goroutines run in the background, and the main program may exit before they complete unless synchronized.
- Goroutines are cheap, with minimal memory overhead (~2KB per goroutine).

**Code Example**:
```go
package main

import (
	"fmt"
	"time"
)

func sayHello() {
	fmt.Println("Hello from goroutine!")
}

func main() {
	// Start a goroutine
	go sayHello()
	
	// Print from main
	fmt.Println("Hello from main!")
	
	// Sleep to allow goroutine to run (not ideal in practice)
	time.Sleep(time.Second)
}
```

**Explanation**:
- `go sayHello()` launches `sayHello` as a goroutine, running concurrently with `main`.
- `time.Sleep` is used to prevent the main program from exiting immediately (we'll replace this with better synchronization later).

**Exercise**:
1. Write a program that launches three goroutines, each printing a different message (e.g., "Goroutine 1", "Goroutine 2", "Goroutine 3").
2. Modify the program to print the messages 10 times in a loop within each goroutine.
3. Observe the output order. Is it consistent? Why or why not?

**Solution** (for reference):
```go
package main

import (
	"fmt"
	"time"
)

func printMessage(msg string) {
	for i := 0; i < 10; i++ {
		fmt.Println(msg)
	}
}

func main() {
	go printMessage("Goroutine 1")
	go printMessage("Goroutine 2")
	go printMessage("Goroutine 3")
	
	time.Sleep(time.Second * 2) // Temporary hack to wait
	fmt.Println("Main done!")
}
```

**Exercise Notes**: The output order is non-deterministic because goroutines are scheduled by the Go runtime, and their execution order depends on CPU availability and scheduling.

---

### Lesson 2: Synchronization with WaitGroup

**Concept**: Using `sync.WaitGroup` to synchronize goroutines, ensuring the main program waits for all goroutines to complete before exiting.

**Key Points**:
- `WaitGroup` tracks a set of goroutines.
- Methods: `Add(n)` to set the number of goroutines, `Done()` to signal completion, and `Wait()` to block until all are done.
- Avoids reliance on `time.Sleep`.

**Code Example**:
```go
package main

import (
	"fmt"
	"sync"
)

func worker(id int, wg *sync.WaitGroup) {
	defer wg.Done() // Signal completion
	fmt.Printf("Worker %d starting\n", id)
	time.Sleep(time.Second) // Simulate work
	fmt.Printf("Worker %d done\n", id)
}

func main() {
	var wg sync.WaitGroup
	
	// Launch 3 workers
	for i := 1; i <= 3; i++ {
		wg.Add(1) // Increment counter for each goroutine
		go worker(i, &wg)
	}
	
	wg.Wait() // Wait for all workers to finish
	fmt.Println("All workers done!")
}
```

**Explanation**:
- `wg.Add(1)` increments the WaitGroup counter for each goroutine.
- `wg.Done()` decrements the counter when a goroutine finishes.
- `wg.Wait()` blocks until the counter reaches zero.

**Exercise**:
1. Modify the example to launch 5 workers, each printing its ID and a random duration of work (use `time.Sleep(time.Millisecond * time.Duration(rand.Intn(1000)))`).
2. Add a counter in each worker to print how many iterations it completes (e.g., loop 5 times).
3. Ensure the main function prints "All done!" only after all workers complete.

**Solution** (for reference):
```go
package main

import (
	"fmt"
	"math/rand"
	"sync"
	"time"
)

func worker(id int, wg *sync.WaitGroup) {
	defer wg.Done()
	for i := 1; i <= 5; i++ {
		fmt.Printf("Worker %d: iteration %d\n", id, i)
		time.Sleep(time.Millisecond * time.Duration(rand.Intn(1000)))
	}
	fmt.Printf("Worker %d done\n", id)
}

func main() {
	rand.Seed(time.Now().UnixNano())
	var wg sync.WaitGroup
	
	for i := 1; i <= 5; i++ {
		wg.Add(1)
		go worker(i, &wg)
	}
	
	wg.Wait()
	fmt.Println("All done!")
}
```

**Exercise Notes**: Random sleep durations simulate varying workloads, and `WaitGroup` ensures proper synchronization.

---

### Lesson 3: Channels for Communication

**Concept**: Channels are Go's mechanism for communication and synchronization between goroutines. They allow safe data exchange without race conditions.

**Key Points**:
- Channels are typed (e.g., `chan int` for integers).
- Use `<-` to send (`ch <- value`) or receive (`value := <-ch`) data.
- Unbuffered channels block until both sender and receiver are ready.

**Code Example**:
```go
package main

import (
	"fmt"
)

func sender(ch chan string) {
	ch <- "Message from sender" // Send message
}

func main() {
	ch := make(chan string) // Create an unbuffered channel
	
	go sender(ch) // Start sender goroutine
	
	msg := <-ch // Receive message
	fmt.Println(msg)
}
```

**Explanation**:
- `make(chan string)` creates an unbuffered channel.
- The sender sends a string, and the main function receives it.
- The program blocks until the sender and receiver synchronize.

**Exercise**:
1. Write a program where one goroutine sends 5 integers (1 to 5) to a channel, and the main function receives and prints them.
2. Modify the program to use a loop in the sender to send the numbers.
3. Add a second goroutine that sends 5 different integers (6 to 10) to the same channel, and ensure all 10 numbers are printed in the main function.

**Solution** (for reference):
```go
package main

import (
	"fmt"
	"sync"
)

func sender(start, end int, ch chan int, wg *sync.WaitGroup) {
	defer wg.Done()
	for i := start; i <= end; i++ {
		ch <- i
	}
}

func main() {
	ch := make(chan int)
	var wg sync.WaitGroup
	
	// Launch two senders
	wg.Add(2)
	go sender(1, 5, ch, &wg)
	go sender(6, 10, ch, &wg)
	
	// Receive and print in a separate goroutine
	go func() {
		for i := 1; i <= 10; i++ {
			fmt.Println(<-ch)
		}
	}()
	
	wg.Wait()
}
```

**Exercise Notes**: The order of received numbers may vary due to goroutine scheduling. Use `WaitGroup` to ensure senders complete.

---

### Lesson 4: Buffered Channels

**Concept**: Buffered channels allow sending multiple values without immediate receipt, up to a specified capacity. They reduce blocking but require careful management to avoid deadlocks.

**Key Points**:
- Create with `make(chan T, capacity)`.
- Sends block only when the buffer is full; receives block when the buffer is empty.
- Useful for batch processing or when sender and receiver operate at different speeds.

**Code Example**:
```go
package main

import (
	"fmt"
)

func main() {
	ch := make(chan string, 2) // Buffered channel with capacity 2
	
	// Send two messages without blocking
	ch <- "Message 1"
	ch <- "Message 2"
	
	// Receive and print
	fmt.Println(<-ch)
	fmt.Println(<-ch)
}
```

**Explanation**:
- The channel can hold two messages before blocking.
- Sends succeed immediately until the buffer is full.
- Receives retrieve messages in FIFO order.

**Exercise**:
1. Create a buffered channel with capacity 3 and send 5 strings to it from a goroutine. What happens? Why?
2. Modify the program to receive all messages in the main function.
3. Add a loop to send 10 messages, but ensure the buffer doesn’t overflow by receiving messages concurrently.

**Solution** (for reference):
```go
package main

import (
	"fmt"
	"sync"
)

func sender(ch chan string, wg *sync.WaitGroup) {
	defer wg.Done()
	for i := 1; i <= 10; i++ {
		ch <- fmt.Sprintf("Message %d", i)
	}
}

func main() {
	ch := make(chan string, 3)
	var wg sync.WaitGroup
	
	wg.Add(1)
	go sender(ch, &wg)
	
	// Receive concurrently
	go func() {
		for i := 1; i <= 10; i++ {
			fmt.Println(<-ch)
		}
	}()
	
	wg.Wait()
}
```

**Exercise Notes**: Without concurrent receiving, sending more than the buffer capacity (3) would cause a deadlock. The receiver goroutine prevents buffer overflow.

---

### Lesson 5: Select Statement

**Concept**: The `select` statement allows a goroutine to wait on multiple channel operations, choosing the first one that’s ready. It’s like a switch for channels.

**Key Points**:
- `select` picks the first non-blocking case or blocks until one is ready.
- A `default` case runs if no other case is ready, preventing blocking.
- Useful for multiplexing channels or implementing timeouts.

**Code Example**:
```go
package main

import (
	"fmt"
	"time"
)

func main() {
	ch1 := make(chan string)
	ch2 := make(chan string)
	
	go func() {
		time.Sleep(time.Second)
		ch1 <- "From channel 1"
	}()
	
	go func() {
		time.Sleep(2 * time.Second)
		ch2 <- "From channel 2"
	}()
	
	for i := 0; i < 2; i++ {
		select {
		case msg1 := <-ch1:
			fmt.Println(msg1)
		case msg2 := <-ch2:
			fmt.Println(msg2)
		}
	}
}
```

**Explanation**:
- `select` waits for either `ch1` or `ch2` to send a message.
- Messages are processed as they arrive, based on the timing of the sends.

**Exercise**:
1. Write a program with two channels, one sending even numbers and the other odd numbers. Use `select` to receive and print them.
2. Add a timeout using `time.After` to stop waiting after 3 seconds.
3. Modify the program to include a `default` case that prints "No data ready" if no channel is ready.

**Solution** (for reference):
```go
package main

import (
	"fmt"
	"sync"
	"time"
)

func sendEvens(ch chan int, wg *sync.WaitGroup) {
	defer wg.Done()
	for i := 2; i <= 10; i += 2 {
		time.Sleep(time.Millisecond * 500)
		ch <- i
	}
}

func sendOdds(ch chan int, wg *sync.WaitGroup) {
	defer wg.Done()
	for i := 1; i <= 9; i += 2 {
		time.Sleep(time.Millisecond * 700)
		ch <- i
	}
}

func main() {
	evens := make(chan int)
	odds := make(chan int)
	var wg sync.WaitGroup
	
	wg.Add(2)
	go sendEvens(evens, &wg)
	go sendOdds(odds, &wg)
	
	for {
		select {
		case num := <-evens:
			fmt.Println("Received even:", num)
		case num := <-odds:
			fmt.Println("Received odd:", num)
		case <-time.After(3 * time.Second):
			fmt.Println("Timeout!")
			return
		default:
			fmt.Println("No data ready")
			time.Sleep(time.Millisecond * 100)
		}
	}
}
```

**Exercise Notes**: The `default` case prevents blocking when no channel is ready, and `time.After` ensures the program doesn’t run indefinitely.

---

### Lesson 6: Mutex for Shared Resources

**Concept**: A `sync.Mutex` ensures safe access to shared resources by preventing concurrent modifications, avoiding race conditions.

**Key Points**:
- Use `Lock()` to acquire the mutex and `Unlock()` to release it.
- Protects critical sections where shared data is accessed.
- Use `sync.RWMutex` for read-heavy scenarios (allows multiple readers, one writer).

**Code Example**:
```go
package main

import (
	"fmt"
	"sync"
)

type Counter struct {
	mu    sync.Mutex
	count int
}

func (c *Counter) Increment() {
	c.mu.Lock()
	defer c.mu.Unlock()
	c.count++
}

func (c *Counter) Value() int {
	c.mu.Lock()
	defer c.mu.Unlock()
	return c.count
}

func main() {
	var wg sync.WaitGroup
	counter := Counter{}
	
	for i := 0; i < 100; i++ {
		wg.Add(1)
		go func() {
			defer wg.Done()
			counter.Increment()
		}()
	}
	
	wg.Wait()
	fmt.Println("Final count:", counter.Value())
}
```

**Explanation**:
- `Counter` uses a mutex to protect `count`.
- `Increment` and `Value` lock the mutex to ensure safe access.
- Without the mutex, concurrent increments could lead to race conditions.

**Exercise**:
1. Write a program with a shared map where multiple goroutines add key-value pairs concurrently. Use a mutex to protect the map.
2. Add a method to read from the map safely and print all key-value pairs.
3. Use `sync.RWMutex` to allow concurrent reads but exclusive writes.

**Solution** (for reference):
```go
package main

import (
	"fmt"
	"sync"
)

type SafeMap struct {
	mu   sync.RWMutex
	data map[int]string
}

func (sm *SafeMap) Put(key int, value string) {
	sm.mu.Lock()
	defer sm.mu.Unlock()
	sm.data[key] = value
}

func (sm *SafeMap) GetAll() map[int]string {
	sm.mu.RLock()
	defer sm.mu.RUnlock()
	result := make(map[int]string)
	for k, v := range sm.data {
		result[k] = v
	}
	return result
}

func main() {
	var wg sync.WaitGroup
	sm := SafeMap{data: make(map[int]string)}
	
	for i := 0; i < 10; i++ {
		wg.Add(1)
		go func(id int) {
			defer wg.Done()
			sm.Put(id, fmt.Sprintf("Value %d", id))
		}(i)
	}
	
	wg.Wait()
	fmt.Println(sm.GetAll())
}
```

**Exercise Notes**: `RWMutex` allows multiple goroutines to read the map concurrently, improving performance for read-heavy workloads.

---

### Lesson 7: Patterns - Worker Pool

**Concept**: A worker pool is a common concurrency pattern where a fixed number of goroutines process tasks from a shared channel, balancing load and resource usage.

**Key Points**:
- Distribute work across multiple goroutines.
- Use a channel to dispatch tasks and collect results.
- Limits resource consumption compared to spawning a goroutine per task.

**Code Example**:
```go
package main

import (
	"fmt"
	"sync"
)

func worker(id int, tasks <-chan int, results chan<- int) {
	for task := range tasks {
		fmt.Printf("Worker %d processing task %d\n", id, task)
		results <- task * 2 // Simulate work
	}
}

func main() {
	tasks := make(chan int, 10)
	results := make(chan int, 10)
	var wg sync.WaitGroup
	
	// Start 3 workers
	for i := 1; i <= 3; i++ {
		wg.Add(1)
		go func(id int) {
			defer wg.Done()
			worker(id, tasks, results)
		}(i)
	}
	
	// Send tasks
	for i := 1; i <= 10; i++ {
		tasks <- i
	}
	close(tasks)
	
	// Collect results
	go func() {
		wg.Wait()
		close(results)
	}()
	
	for result := range results {
		fmt.Println("Result:", result)
	}
}
```

**Explanation**:
- Workers read tasks from a buffered channel and send results to another.
- Closing the tasks channel signals workers to stop.
- `WaitGroup` ensures all workers finish before closing the results channel.

**Exercise**:
1. Modify the worker pool to process strings, converting each to uppercase.
2. Add a timeout for each worker’s task (e.g., 1 second) using `select` and `time.After`.
3. Collect results in a slice and print them sorted by task ID.

**Solution** (for reference):
```go
package main

import (
	"fmt"
	"sort"
	"strings"
	"sync"
	"time"
)

type Result struct {
	id    int
	value string
}

func worker(id int, tasks <-chan struct {
	id   int
	data string
}, results chan<- Result) {
	for task := range tasks {
		select {
		case <-time.After(time.Second):
			fmt.Printf("Worker %d: task %d timed out\n", id, task.id)
		default:
			results <- Result{task.id, strings.ToUpper(task.data)}
		}
	}
}

func main() {
	tasks := make(chan struct {
		id   int
		data string
	}, 10)
	results := make(chan Result, 10)
	var wg sync.WaitGroup
	
	for i := 1; i <= 3; i++ {
		wg.Add(1)
		go func(id int) {
			defer wg.Done()
			worker(id, tasks, results)
		}(i)
	}
	
	// Send tasks
	for i := 1; i <= 10; i++ {
		tasks <- struct {
			id   int
			data string
		}{i, fmt.Sprintf("task %d", i)}
	}
	close(tasks)
	
	// Collect results
	var resultList []Result
	go func() {
		wg.Wait()
		close(results)
	}()
	
	for result := range results {
		resultList = append(resultList, result)
	}
	
	// Sort and print
	sort.Slice(resultList, func(i, j int) bool {
		return resultList[i].id < resultList[j].id
	})
	for _, r := range resultList {
		fmt.Printf("Task %d: %s\n", r.id, r.value)
	}
}
```

**Exercise Notes**: The timeout ensures tasks don’t hang, and sorting results provides predictable output.

---

### Lesson 8: Advanced Pattern - Fan-Out/Fan-In

**Concept**: Fan-out distributes tasks across multiple workers; fan-in collects their results into a single channel. This pattern maximizes parallelism and aggregates output.

**Key Points**:
- Fan-out: Send tasks to multiple workers for parallel processing.
- Fan-in: Merge results from multiple channels into one.
- Often used with worker pools for scalability.

**Code Example**:
```go
package main

import (
	"fmt"
	"sync"
)

func producer(tasks chan<- int, n int) {
	for i := 1; i <= n; i++ {
		tasks <- i
	}
	close(tasks)
}

func worker(id int, tasks <-chan int, results chan<- int) {
	for task := range tasks {
		results <- task * task // Square the task
	}
}

func merge(cs ...<-chan int) <-chan int {
	var wg sync.WaitGroup
	out := make(chan int)
	
	// Fan-in: collect results from each channel
	for _, c := range cs {
		wg.Add(1)
		go func(ch <-chan int) {
			defer wg.Done()
			for n := range ch {
				out <- n
			}
		}(c)
	}
	
	go func() {
		wg.Wait()
		close(out)
	}()
	return out
}

func main() {
	tasks := make(chan int, 10)
	results := make([]chan int, 3)
	
	// Initialize result channels
	for i := range results {
		results[i] = make(chan int, 10)
	}
	
	// Start producer
	go producer(tasks, 10)
	
	// Fan-out: start 3 workers
	var wg sync.WaitGroup
	for i := 0; i < 3; i++ {
		wg.Add(1)
		go func(id int) {
			defer wg.Done()
			worker(id, tasks, results[id])
		}(i)
	}
	
	// Fan-in: merge results
	merged := merge(results...)
	
	// Collect and print
	for result := range merged {
		fmt.Println("Result:", result)
	}
}
```

**Explanation**:
- `producer` sends tasks to a channel.
- Three workers (fan-out) process tasks in parallel, sending results to separate channels.
- `merge` (fan-in) combines results into a single channel.

**Exercise**:
1. Modify the program to process strings, converting each to a reversed string.
2. Add a worker that fails randomly (use `rand.Intn(2)`), sending an error to a separate error channel.
3. Use `select` to handle both results and errors in the main function.

**Solution** (for reference):
```go
package main

import (
	"fmt"
	"math/rand"
	"sync"
	"time"
)

func producer(tasks chan<- string, n int) {
	for i := 1; i <= n; i++ {
		tasks <- fmt.Sprintf("task %d", i)
	}
	close(tasks)
}

func worker(id int, tasks <-chan string, results chan<- string, errors chan<- string) {
	for task := range tasks {
		if rand.Intn(2) == 0 {
			errors <- fmt.Sprintf("Worker %d failed on %s", id, task)
			continue
		}
		// Reverse string
		runes := []rune(task)
		for i, j := 0, len(runes)-1; i < j; i, j = i+1, j-1 {
			runes[i], runes[j] = runes[j], runes[i]
		}
		results <- string(runes)
	}
}

func mergeResults(cs ...<-chan string) <-chan string {
	var wg sync.WaitGroup
	out := make(chan string)
	
	for _, c := range cs {
		wg.Add(1)
		go func(ch <-chan string) {
			defer wg.Done()
			for n := range ch {
				out <- n
			}
		}(ch)
	}
	
	go func() {
		wg.Wait()
		close(out)
	}()
	return out
}

func main() {
	rand.Seed(time.Now().UnixNano())
	tasks := make(chan string, 10)
	results := make([]chan string, 3)
	errors := make(chan string, 10)
	
	for i := range results {
		results[i] = make(chan string, 10)
	}
	
	go producer(tasks, 10)
	
	var wg sync.WaitGroup
	for i := 0; i < 3; i++ {
		wg.Add(1)
		go func(id int) {
			defer wg.Done()
			worker(id, tasks, results[id], errors)
		}(i)
	}
	
	merged := mergeResults(results...)
	
	for {
		select {
		case result, ok := <-merged:
			if !ok {
				return
			}
			fmt.Println("Result:", result)
		case err := <-errors:
			fmt.Println("Error:", err)
		}
	}
}
```

**Exercise Notes**: The error channel allows handling failures gracefully, and `select` ensures non-blocking processing of results and errors.

---

### Lesson 9: Context for Cancellation

**Concept**: The `context` package provides a way to manage cancellation, deadlines, and timeouts in concurrent programs, ensuring resources are cleaned up properly.

**Key Points**:
- `context.Background()` or `context.TODO()` as the root context.
- `context.WithCancel`, `context.WithTimeout`, or `context.WithDeadline` to create cancellable contexts.
- Pass contexts to goroutines to signal cancellation.

**Code Example**:
```go
package main

import (
	"context"
	"fmt"
	"time"
)

func worker(ctx context.Context, id int) {
	for {
		select {
		case <-ctx.Done():
			fmt.Printf("Worker %d cancelled\n", id)
			return
		default:
			fmt.Printf("Worker %d working\n", id)
			time.Sleep(time.Millisecond * 500)
		}
	}
}

func main() {
	ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
	defer cancel()
	
	go worker(ctx, 1)
	go worker(ctx, 2)
	
	time.Sleep(3 * time.Second) // Allow time to observe cancellation
	fmt.Println("Main done")
}
```

**Explanation**:
- `context.WithTimeout` creates a context that cancels after 2 seconds.
- Workers check `ctx.Done()` to exit when cancelled.
- `defer cancel()` ensures cleanup.

**Exercise**:
1. Modify the worker to process tasks from a channel until the context is cancelled.
2. Add a deadline to the context instead of a timeout.
3. Create a program where a parent goroutine cancels child goroutines when it receives a specific task (e.g., task ID 5).

**Solution** (for reference):
```go
package main

import (
	"context"
	"fmt"
	"sync"
	"time"
)

func worker(id int, tasks <-chan int, ctx context.Context, wg *sync.WaitGroup) {
	defer wg.Done()
	for {
		select {
		case <-ctx.Done():
			fmt.Printf("Worker %d cancelled\n", id)
			return
		case task, ok := <-tasks:
			if !ok {
				return
			}
			fmt.Printf("Worker %d processing task %d\n", id, task)
		}
	}
}

func main() {
	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()
	
	tasks := make(chan int, 10)
	var wg sync.WaitGroup
	
	// Start workers
	for i := 1; i <= 3; i++ {
		wg.Add(1)
		go worker(i, tasks, ctx, &wg)
	}
	
	// Send tasks
	for i := 1; i <= 10; i++ {
		tasks <- i
		if i == 5 {
			cancel() // Cancel when task 5 is sent
		}
	}
	close(tasks)
	
	wg.Wait()
	fmt.Println("Main done")
}
```

**Exercise Notes**: Cancellation propagates to all workers, ensuring clean shutdown when the condition (task ID 5) is met.

---

### Lesson 10: Real-World Example - Concurrent Web Crawler

**Concept**: Combine goroutines, channels, mutexes, and context to build a concurrent web crawler that fetches URLs in parallel, respecting a maximum concurrency limit.

**Key Points**:
- Use a worker pool to limit concurrent HTTP requests.
- Use a mutex to track visited URLs.
- Use context for cancellation and timeouts.
- Handle errors gracefully.

**Code Example**:
```go
package main

import (
	"context"
	"fmt"
	"net/http"
	"sync"
	"time"
)

type Crawler struct {
	visited map[string]bool
	mu      sync.Mutex
}

func (c *Crawler) crawl(url string, ctx context.Context, results chan<- string, wg *sync.WaitGroup) {
	defer wg.Done()
	
	// Check if URL was visited
	c.mu.Lock()
	if c.visited[url] {
		c.mu.Unlock()
		return
	}
	c.visited[url] = true
	c.mu.Unlock()
	
	// Simulate fetching URL
	client := &http.Client{Timeout: 5 * time.Second}
	req, err := http.NewRequestWithContext(ctx, "GET", url, nil)
	if err != nil {
		results <- fmt.Sprintf("Error creating request for %s: %v", url, err)
		return
	}
	
	resp, err := client.Do(req)
	if err != nil {
		results <- fmt.Sprintf("Error fetching %s: %v", url, err)
		return
	}
	defer resp.Body.Close()
	
	results <- fmt.Sprintf("Fetched %s with status %d", url, resp.StatusCode)
}

func main() {
	ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
	defer cancel()
	
	crawler := &Crawler{visited: make(map[string]bool)}
	urls := []string{
		"https://example.com",
		"https://golang.org",
		"https://x.ai",
		"https://example.com", // Duplicate
	}
	tasks := make(chan string, len(urls))
	results := make(chan string, len(urls))
	var wg sync.WaitGroup
	
	// Start worker pool
	for i := 0; i < 2; i++ {
		wg.Add(1)
		go func() {
			for url := range tasks {
				crawler.crawl(url, ctx, results, &wg)
			}
			wg.Done()
		}()
	}
	
	// Send URLs to tasks
	for _, url := range urls {
		tasks <- url
	}
	close(tasks)
	
	// Collect results
	go func() {
		wg.Wait()
		close(results)
	}()
	
	for result := range results {
		fmt.Println(result)
	}
}
```

**Explanation**:
- A worker pool with 2 workers fetches URLs concurrently.
- A mutex ensures duplicate URLs are skipped.
- Context enforces a 10-second timeout.
- Results are collected and printed.

**Exercise**:
1. Modify the crawler to extract and follow links from the fetched pages (use a simple regex or HTML parser like `goquery`).
2. Add a maximum depth limit to prevent infinite crawling.
3. Implement a rate limiter to avoid overwhelming servers (e.g., max 1 request per second per worker).

**Solution Notes**: For brevity, I’ll skip the full solution due to complexity (parsing HTML and rate limiting require additional libraries). Instead, here’s a starting point for rate limiting:

```go
func (c *Crawler) crawl(url string, ctx context.Context, results chan<- string, wg *sync.WaitGroup, limiter *time.Ticker) {
	defer wg.Done()
	
	<-limiter.C // Wait for rate limiter tick
	// Rest of crawl logic...
}
```

Add `limiter := time.NewTicker(time.Second)` in `main` and pass it to workers. For link extraction, use `goquery` or similar libraries.

**Exercise Notes**: This exercise ties together all concepts, encouraging you to handle real-world challenges like rate limiting and state management.

---

### Course Summary

This "Zero to Hero" course covered:
1. **Goroutines**: Lightweight concurrent execution.
2. **WaitGroup**: Synchronizing goroutines.
3. **Channels**: Safe communication between goroutines.
4. **Buffered Channels**: Handling asynchronous data.
5. **Select**: Multiplexing channel operations.
6. **Mutex**: Protecting shared resources.
7. **Worker Pool**: Efficient task distribution.
8. **Fan-Out/Fan-In**: Parallel processing and result aggregation.
9. **Context**: Managing cancellation and timeouts.
10. **Real-World Example**: Building a concurrent web crawler.






