- DOING done me
  :LOGBOOK:
  CLOCK: [2024-09-11 Wed 16:31:41]
  CLOCK: [2024-09-11 Wed 16:31:45]
  CLOCK: [2024-09-11 Wed 19:47:43]
  CLOCK: [2024-09-11 Wed 19:47:46]
  :END:
- 问题：
	- ~~事件循环是什么，怎么用~~
	  logseq.order-list-type:: number
	- ~~理解Future，Coroutine，Task（和js对比）~~
	  logseq.order-list-type:: number
	- 执行昂贵阻塞任务
	  logseq.order-list-type:: number
	- 在其他线程中向事件循环提交任务
	  logseq.order-list-type:: number
	- 协程间同步操作（锁，信号量，超时，any，all）
	  logseq.order-list-type:: number
- ## 事件循环和异步函数
	- Python的事件循环和js的类似——事件，定时器，事件队列，回调……区别在于，js的事件循环更多是以回调函数为机制，而Python的事件循环则是基于协程，而且允许多线程，允许同时存在多个事件循环，但每个线程只能存在一个事件循环，代码逻辑均在该线程上执行，除了使用run_in_executor创建的任务，它们会在线程池或进程池上执行。因此，使用异步编程时大多数时候不需要担心线程安全问题，但仍可能需要做协程间的同步。
	- 启动事件循环的最简单的方法是`async.run`，它只能在主线程上执行，通过一个协程（async函数返回的就是协程Coroutine，行为类似生成器）启动，创建一个事件循环并阻塞直到所有协程执行完毕：
	- ```python
	  import asyncio
	  # 这么写其实是不优雅的——lock这样的对象会和特定的事件循环绑定
	  lock = asyncio.Lock()
	  async def go():
	      async with lock:
	          await asyncio.sleep(1)
	          print('Hello!') 
	      
	  async def main():
	      await asyncio.gather(go(), go(), go())
	  
	  print('before')
	  asyncio.run(main())
	  print('after')
	  ```
	- 每个线程都能通过`async.run`执行事件循环并阻塞，同一个线程在同一时刻只能运行一个事件循环。**`async.run`总是会创建和自动关闭新的事件循环**。如果要重用事件循环，需要手动创建和维持它，或者使用Runner，它是一个事件循环的上下文管理器：
	- ```python
	  loop = asyncio.new_event_loop()
	  asyncio.set_event_loop(loop) # 这行真的需要吗？
	  loop.run_until_complete(main())
	  loop.run_until_complete(main())
	  loop.close()
	  # or
	  with asyncio.Runner() as runner:
	      loop = runner.get_loop()
	      loop.run_until_complete(main())
	      loop.run_until_complete(main())
	  ```
- ## Future，Coroutine和Task
	- 在使用asyncio中，有三个概念会常常遇到，Future，Task和Coroutine。
	- **Future**，Future是一个底层的异步原语，如果拿Promise做比较的话，它就像一个不包含任何业务逻辑的Promise，但和Promise不同，Future通过调用方法`set_result`等去设定返回结果，并通知事件循环去调度所有await它的协程。**Future本质上是一种通知机制**。
	- 下面使用Future实现Lock，同时在js中通过promise去实现Lock，去体现它的性质，两边的区别在于Promise是直接把resolve函数放进等待队列中，而Future是把自己放进等待队列中，两种操作本质上都是相同的，都是为了在释放锁时能通知调度器再调一个。注意这里的Python代码处理了异常，调度器可能会在await处抛出异常，如超时等；出现异常时必须要把它从队列里拿出去：
	  collapsed:: true
		- ```python
		  # python
		  class Mutex:
		      def __init__(self) -> None:
		          self.__locked = False
		          self.__wait_queue: List[asyncio.Future] = []
		      async def acquire(self):
		          # 如果当前本就没有锁，直接获取锁
		          if not self.__locked:
		              self.__locked = True
		              return
		          # 否则，加入等待队列中，并等待它被“resolve”
		          future = asyncio.Future()
		          self.__wait_queue.append(future)
		          try:
		              await future
		          finally:
		              self.__wait_queue.remove(future)
		      def release(self):
		          if not self.__locked:
		              raise AssertionError("not locked")
		          if not self.__wait_queue:
		              self.__locked = False
		              return
		          # 通知等待队列
		          self.__wait_queue.pop(0).set_result(None)
		          self.__locked = True
		  ```
		- ```typescript
		  // typescript
		  export function mkLock() {
		      let locked = false
		      const waitQueue: ((...arg: any) => void)[] = []
		      async function acquire() {
		          if (!locked) {
		              locked = true
		              return
		          }
		          const future = new Promise(resolve => waitQueue.push(resolve))
		          await future
		      }
		      function release() {
		          if (!locked) {
		              throw 'not locked'
		          }
		          locked = false
		          if (waitQueue.length !== 0) {
		              waitQueue.pop()!()
		              locked = true
		          }
		      }
		      return {acquire, release}
		  }
		  ```
	- **Coroutine**，Coroutine就是协程，就行为上可以当成生成器去理解，**async函数返回的东西就是Coroutine**，直接执行async函数时，什么都不会发生，除了得到一个Coroutine对象（就像调用生成器函数得到一个生成器一样，需要东西去触发它），该对象只有使用await（类似yield from）才能使之真正执行，并得到它的返回值；未执行的Coroutine会得到警告。
	- **Task**，Task继承自Future，它负责运行Coroutine，在这些概念中，**Task和js的Promise最为接近**。
	- **Coroutine不会自动地被调度器去调度，而Task会**。
	  id:: 66e19162-ccb3-4889-a45f-0557c7417861
	- js和python有一个重大的不同——在js中，要另外开启一个和当前循环无关的任务时，直接执行相应异步函数，不管它的结果或后面再await它便可，而python中需要调用`asyncio.create_task`去把Coroutine转换成Task，原因上面也说了—— ((66e19162-ccb3-4889-a45f-0557c7417861))，如果不使用`asyncio.create_task`的话，该协程会在await时才**开始**执行。
	- ```javascript
	  // javascript
	  async function someFn() {
	    const task = someAsyncTaskIDontCare()
	    // my logic...
	    const res = await task // if necessary
	  }
	  ```
	- ```python
	  # python
	  async def some_fn():
	    task = asyncio.create_task(some_async_task_i_dont_care())
	    # my logic...
	    await task
	  ```
- ## 执行昂贵阻塞任务
	- 事件循环允许有配套的对于昂贵的IO任务，可以将它