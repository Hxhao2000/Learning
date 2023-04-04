## Goroutine

Goroutine是Golang中的coroutine，一种轻量级的用户线程，相较于系统线程，它的创建开销低，同时具备一些系统线程不具备的优点，比如说动态扩容的栈空间等等。

> 本文写自Golang1.20.1

### Goroutine是什么？

在Golang中，使用结构体g来代表goroutine，下面是Golang中关于结构体g的[源码](https://github.com/golang/go/blob/202a1a57064127c3f19d96df57b9f9586145e21c/src/runtime/runtime2.go#L407-L506)。

```go
type g struct {
	// Stack parameters.
	// stack describes the actual stack memory: [stack.lo, stack.hi).
	// stackguard0 is the stack pointer compared in the Go stack growth prologue.
	// It is stack.lo+StackGuard normally, but can be StackPreempt to trigger a preemption.
	// stackguard1 is the stack pointer compared in the C stack growth prologue.
	// It is stack.lo+StackGuard on g0 and gsignal stacks.
	// It is ~0 on other goroutine stacks, to trigger a call to morestackc (and crash).
	stack       stack   // offset known to runtime/cgo
	stackguard0 uintptr // offset known to liblink
	stackguard1 uintptr // offset known to liblink

	_panic    *_panic // innermost panic - offset known to liblink
	_defer    *_defer // innermost defer
	m         *m      // current m; offset known to arm liblink
	sched     gobuf
	syscallsp uintptr // if status==Gsyscall, syscallsp = sched.sp to use during gc
	syscallpc uintptr // if status==Gsyscall, syscallpc = sched.pc to use during gc
	stktopsp  uintptr // expected sp at top of stack, to check in traceback
	// param is a generic pointer parameter field used to pass
	// values in particular contexts where other storage for the
	// parameter would be difficult to find. It is currently used
	// in three ways:
	// 1. When a channel operation wakes up a blocked goroutine, it sets param to
	//    point to the sudog of the completed blocking operation.
	// 2. By gcAssistAlloc1 to signal back to its caller that the goroutine completed
	//    the GC cycle. It is unsafe to do so in any other way, because the goroutine's
	//    stack may have moved in the meantime.
	// 3. By debugCallWrap to pass parameters to a new goroutine because allocating a
	//    closure in the runtime is forbidden.
	param        unsafe.Pointer
	atomicstatus atomic.Uint32
	stackLock    uint32 // sigprof/scang lock; TODO: fold in to atomicstatus
	goid         uint64
	schedlink    guintptr
	waitsince    int64      // approx time when the g become blocked
	waitreason   waitReason // if status==Gwaiting

	preempt       bool // preemption signal, duplicates stackguard0 = stackpreempt
	preemptStop   bool // transition to _Gpreempted on preemption; otherwise, just deschedule
	preemptShrink bool // shrink stack at synchronous safe point

	// asyncSafePoint is set if g is stopped at an asynchronous
	// safe point. This means there are frames on the stack
	// without precise pointer information.
	asyncSafePoint bool

	paniconfault bool // panic (instead of crash) on unexpected fault address
	gcscandone   bool // g has scanned stack; protected by _Gscan bit in status
	throwsplit   bool // must not split stack
	// activeStackChans indicates that there are unlocked channels
	// pointing into this goroutine's stack. If true, stack
	// copying needs to acquire channel locks to protect these
	// areas of the stack.
	activeStackChans bool
	// parkingOnChan indicates that the goroutine is about to
	// park on a chansend or chanrecv. Used to signal an unsafe point
	// for stack shrinking.
	parkingOnChan atomic.Bool

	raceignore     int8     // ignore race detection events
	sysblocktraced bool     // StartTrace has emitted EvGoInSyscall about this goroutine
	tracking       bool     // whether we're tracking this G for sched latency statistics
	trackingSeq    uint8    // used to decide whether to track this G
	trackingStamp  int64    // timestamp of when the G last started being tracked
	runnableTime   int64    // the amount of time spent runnable, cleared when running, only used when tracking
	sysexitticks   int64    // cputicks when syscall has returned (for tracing)
	traceseq       uint64   // trace event sequencer
	tracelastp     puintptr // last P emitted an event for this goroutine
	lockedm        muintptr
	sig            uint32
	writebuf       []byte
	sigcode0       uintptr
	sigcode1       uintptr
	sigpc          uintptr
	gopc           uintptr         // pc of go statement that created this goroutine
	ancestors      *[]ancestorInfo // ancestor information goroutine(s) that created this goroutine (only used if debug.tracebackancestors)
	startpc        uintptr         // pc of goroutine function
	racectx        uintptr
	waiting        *sudog         // sudog structures this g is waiting on (that have a valid elem ptr); in lock order
	cgoCtxt        []uintptr      // cgo traceback context
	labels         unsafe.Pointer // profiler labels
	timer          *timer         // cached timer for time.Sleep
	selectDone     atomic.Uint32  // are we participating in a select and did someone win the race?

	// goroutineProfiled indicates the status of this goroutine's stack for the
	// current in-progress goroutine profile
	goroutineProfiled goroutineProfileStateHolder

	// Per-G GC state

	// gcAssistBytes is this G's GC assist credit in terms of
	// bytes allocated. If this is positive, then the G has credit
	// to allocate gcAssistBytes bytes without assisting. If this
	// is negative, then the G must correct this by performing
	// scan work. We track this in bytes to make it fast to update
	// and check for debt in the malloc hot path. The assist ratio
	// determines how this corresponds to scan work debt.
	gcAssistBytes int64
}
```

`_panic`用于记录当前goroutine最内层的panic，在recover操作时，可以从`_panic`中获取异常并进行后续的处理。

`_defer`用于记录_defer结构体链表，以便按照后进先出（LIFO）的顺序执行这些延迟语句。

`m`代表当前goroutine属于哪个m，即GMP模型中的machine。

`startpc`用于记录goroutine函数的入口地址。

`gopc`用于记录创建当前 Goroutine 的 Go 语句的起始地址（即创建该 Goroutine 的位置）。

...

### Goroutine如何被调度？

Golang中主要通过[goready()](https://github.com/golang/go/blob/202a1a57064127c3f19d96df57b9f9586145e21c/src/runtime/proc.go#L390-L394)和[gopark()](https://github.com/golang/go/blob/202a1a57064127c3f19d96df57b9f9586145e21c/src/runtime/proc.go#L364-L382)来控制Goroutine的运行状态，通过[schedule()](https://github.com/golang/go/blob/202a1a57064127c3f19d96df57b9f9586145e21c/src/runtime/proc.go#L3331-L3399)来进行调度具体的goroutine进行执行。

#### goready

通过乐观锁的方式将Goroutine从[_Gwaiting](https://github.com/golang/go/blob/202a1a57064127c3f19d96df57b9f9586145e21c/src/runtime/runtime2.go#L58)状态修改为[_Grunnable](https://github.com/golang/go/blob/202a1a57064127c3f19d96df57b9f9586145e21c/src/runtime/runtime2.go#L38)，如果失败，则会[根据失败原因决定继续循环重试还是抛出错误](https://github.com/golang/go/blob/202a1a57064127c3f19d96df57b9f9586145e21c/src/runtime/proc.go#L1013-L1028)，然后会通过[runqput()](https://github.com/golang/go/blob/202a1a57064127c3f19d96df57b9f9586145e21c/src/runtime/proc.go#L5962-L5993)将goruntine放到某个队列中。整个过程处于`systemstack()`的调用中，`systemstack()`将切换至系统堆栈，以避免抢占情况。

#### runqput



#### gopark

通过`mcall()`切换至系统堆栈，并运行`park_m()`将当前goroutine和m的连接断开，同时，通过乐观锁的方式将Goroutine从`_Gwaiting`状态修改为`_Grunnable`，处理方式同goready。在gopark的过程中，如果有锁需要释放，则会通过提供的unlockf来尝试释放，如果释放失败，则会将goroutine状态改回`_Grunnable`并继承剩余时间片继续运行该goroutine，如果释放成功，则会通过`schedule()`调度其他的goroutine进行运行。

#### schedule

1. 检查当前goroutine所在的m上持有的锁数量，如果锁数量大于0，则抛出错误；
2. 检查m是否处于锁阻塞状态，如果是，则触发handoff机制，当前m交出p，如果当前p中存在待执行的goroutine等情况，则会获取空闲的m进行执行，而原本的m将进入睡眠状态，等待被唤醒；
3. 检查m是否处于调用C代码的过程，如果是，抛出错误；
4. 通过[findRunnable()](https://github.com/golang/go/blob/202a1a57064127c3f19d96df57b9f9586145e21c/src/runtime/proc.go#L2672-L3009)查找下一个goroutine用于执行；
5. 如果m处于自旋状态，则重置自旋状态；
6. 再次检查m是否处于锁阻塞状态；
7. 运行通过`findRunnable()`找到的goroutine。

#### findRunnable
