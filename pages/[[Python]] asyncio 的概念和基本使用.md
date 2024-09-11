- DOING done me
  :LOGBOOK:
  CLOCK: [2024-09-11 Wed 16:31:41]
  CLOCK: [2024-09-11 Wed 16:31:45]
  CLOCK: [2024-09-11 Wed 19:47:43]
  CLOCK: [2024-09-11 Wed 19:47:46]
  :END:
- 问题：
	- 事件循环是什么，怎么用
	  logseq.order-list-type:: number
	- 理解Future，Coroutine，Task（和js对比）
	  logseq.order-list-type:: number
	- 创建和当前协程无关的异步任务，以及挪到其他线程/线程池去执行的任务
	  logseq.order-list-type:: number
	- 在同步代码中调用异步任务
	  logseq.order-list-type:: number
	- 协程间同步操作（锁，信号量，any，all）
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
	- Future是一个低级的异步原语，如果拿Promise做比较的话，它就像一个不包含任何业务逻辑的Promise，但和Promise不同，Future通过调用方法`set_result`等去设定