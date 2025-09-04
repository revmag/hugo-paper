+++
title = "Async Functions"
weight = 7
+++

So I had to calls a server over a lakh times to get some output.<br>
Then had to resort to async functions ( as I already has the list which I need to pass as input)

But some observations about how it works → 

```python
async def func()
{
}  
async def main():
{
tasks = [asyncio.create_task(func(i)) for i in range(10)]
await asyncio.gather(*tasks)
}
asyncio.run(main())
```

So what happens is → tasks makes list of all the numbers which need to be executed
and when asyncio.gather(*tasks) is called, it executes all of them in parallel.

>Pretty simple

But, if you have to set a limit of how many things can be concurrent at the same time, use semaphore, like this →

```python
async def func()
{
}  
async def main():
{
sem = asyncio.Semaphore(3)
tasks = [asyncio.create_task(func(sem, i)) for i in range(10)]
await asyncio.gather(*tasks)
}
asyncio.run(main())
```

But the questions arises, does the semaphore pass the next variables  after complete execution of the functions triggered by the first batch of numbers, or does it keep passing once it encounters a await function there ( and passes the missing space to semaphore)

So, yeah - it waits for the whole function to be executed, and then it passes the control to next numbers in the list, it holds the  semaphore until it completes the task and then releases it

Semaphore ensures at most N tasks( limit of semaphore) are actively executing at any given moment, and that’s the difference between let’s say batch processing, it will wait for the whole batch to be executed before moving on to the next batch.

So, if you have to use async for very large numbers:
Semaphore can be used, with let’s say a limit of 3-4
And if length of semaphore is bigger, let’s say around 10, and functions complete pretty fast, so sleep can be used, sleep timer of 1 will ensure that there is atleast a gap of 1 seconds between any N calls to the same server( given the server can handle N requests at a same time, and no more)

And more robust ways can be to induce callback, like if some errors come due to some functions, so just store it and make another call to it.

Updates : Async function doesn’t actually run multiple lines of code at the same time, async runs a block of code at a time, so it’s not entirely parallel programming( actual parallel programming is using multiprocess- which in reality runs multiple lines of code)
Single thread handles all tasks but switches if some tasks are waiting ( I/O or sleep timer)
But it can send multiple I/O requests at the same time.
**Reality**: From a CPU perspective, only one line of code is being executed at a time, but it feels like multiple tasks are progressing because the program "yields" control during I/O or waits.
It is has different meaning if run on Javascript vs Python ( difference as in - wait while I run this code vs aaah, I yield control ( and mostly functions in python are not designed to give control) )

There's another thing called event loop, which is inbuilt in javascript, but in python it needs to be created and then run.

I don't exactly understand running one block at a time, event loop, yield control but rough picture is : tasks are like things on queue, and when one gets executed other comes up ( vs just one entity in a loop - like in for loop), event loop is just which manages this whole infra, yield control I don't know.