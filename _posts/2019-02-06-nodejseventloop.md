---
layout: post
title: NodeJS and Event Loop
categories: [blogs]
tags: [nodejs, event loop]
key: nodejseventloop
author: Shiro
isShowDate: true
---

This is a blog to summarize my understanding about Node JS and its Event Loop. I will explain some terms (which used to confuse me) and a figure (which I think how Node JS works)

#### Node JS
Node JS provides a JS runtime enviroment which helps us to run our JS codes outside of web browser (such as Chrome, Firefox, Safari, and so on). NodeJS can do these things because it has a very important dependancy named V8, Google Chrome's JS engine. V8 helps to interpret and execute JS codes. This is one of advantages of NodeJS, our codes now is JS including client and server side, so convenient for JS developer!

_NodeJS is written in JS and C++._

NodeJS is single-threaded, non-blocking I/O, event-driven asynchronous to build scalable, realtime, intensive I/O applications.

Yeah, I saw these things everywhere when people talked about NodeJS. So what are they?

1. Single-threaded: this means at a time, our JS application only executes one operation one by one in order, top-bottom.
2. Non-blocking I/O: I/O operations are expensive tasks of any application, for example, read a file, get data from other server, hash a key... It takes time to get the results. On the other hand, we have blocking I/O terms as well, it means our application will be blocked when entering to codes pertaining to these I/O operations, the app has no ability to do anything else. So non-blocking means the app will not be blocked, and this is an advantage of NodeJS. It helps single-threaded NodeJS application will not be stuck with any exprensive I/O task.
3. Event-driven asynchronous: JS is event-based language. Everytime user interacts with UI, like click a button, make a request, an event and event listener will be registered (to one of eventloop's queue); whenever an event is finished, a group of tasks will be executed which is called 'callback'. To allow JS to execute I/O tasks without blocking them, NodeJS uses event-driven asynchronous mechanism to handle those finishing I/O tasks. It means whenever I/O tasks is finished, the registered callback will be invoked and return result to NodeJS (so, those tasks are actually run outside of NodeJS).

Until here, there are alot of stuff maybe hard to understand fully, let dive deeper.

![Concurrency Model and Event Loop on MDN web docs](https://developer.mozilla.org/files/4617/default.svg)

As you can see, at runtime this is a visual representation of JS. So JS has a heap to allocate variables, a call stack to execute JS codes and event loop.

#### Event Loop
After reading alot of articles, the next figure is my summary of Node JS and EventLoop.
[![Understanding of Event Loop]({{ "/assets/img/nodejseventloop.png" | relative_url }})]({{ "/assets/img/nodejseventloop.png" | relative_url }}){:target="_blank"}  
Explaination for these things then will be:
1. **_Heap_** is the place will allocate our variables inside an application
2. **_Call Stack_**: our JS codes will executed here sequentially. However, JS operations will be devided in 1 types which are sync & async operations. Sync operations will be run right inside of call stack. In constrast, async operations will not be executed inside call stack, insteads, it will register an event & an event listener into its appropriate queue of event loop, then hand the task over to a suitable external handler and pop the async operation out of call stack. It will run sequentially, top-bottom until pop everything out of stack. And now for the event loop's performance.
3. **_Event Loop_**: is a loop of handling all async callbacks. One loop will be run within one 'tick'. A loop is a set of phases which has 4 main phases: timer, poll, check, close callbacks (Refer to [[1.]](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/)). Each phase has its own queue which is recorded/ added by Call Stack above. When timer comes or I/O operation has got its result, event loop will add its callback to call stack and execute it right away. A phase will continue running until the queue is exaushted or reached the hard maximum then move to the next phase.
  - **Timer phase**: handles queue of callbacks which are registered by setTimeout, set Interval. When event loop enters to this phase, it checks whether callback can be executed; if yes, it shifts the callback to call stack then run; if no, check the next iterator until done the queue or reached the maximum and move to the next phase.
  - **Poll phase**: almost I/O operations' callbacks are handled here, also is the place to receive incoming connections or results of async tasks. If there is a new result/ connection from external handler, poll phase will add to its queue. If poll's queue is not empty, then get callback from queue and hand over call stack to execute. If poll's queue is empty, it checks setTimeOut & setInterval registration whether the callback can be executed or not then wrap back to timer phase to execute it or move to check phase.
  - **Check phase**: handles queue of callbacks which are registered by setImmediate  
  - **Close phase**: handles queue of callbacks which are registered by close callbacks such as connection close  
4. **_Especial queues_**
  - **Next Tick queue**: queue of callbacks are registered by invoking process.nextTick(). Process.nextTick() is not a part of event loop and is used to tell JS that let behave these codes as async codes while it is actually synchronous. These callbacks will be handled when the call stack is empty and before the event loop continues. It does not depend on phases; if it is called in any specific phase, it will be processed after the current phase completes. It is used to handle errors, make request again before event loop continues, clean up unneeded data. We should not try to process something heavy in here because it will block the event loop, makes the application run slowly or freeze.
  - **Micro Tasks queue**: where callbacks of resolved Promise is handled.
  - Next tick queue has higher priory than micro tasks queue.
5. The event loop will run until all of callback queues are empty and exit/ end the application.
6. **_External Handlers_**: NodeJS provides a dependency to handle I/O operations named libuv. And the other thing is OS scheduler which has no relation to NodeJS, but belongs to OS. NodeJS' Event Loop is single-threaded but libuv dependency is not, by default, libuv will create 4 threads; and we can change the number of threads by set process.UV_THREADPOOL_SIZE.

Because NodeJS and its dependencies are good with I/O intensive application by running asynchronously but actually it uses a small number of threads to handle.
Therefore, it is not suitable for CPU intensive applications which demands a lot of computation to calculate/ process complex jobs/ tasks such as image manipulation or sorting large data sets.

In summary, when we clearly know about the concept of JS, it will be easier to predict how our codes will be executed and we have ability to improve the codes for running effectively.

### References
> [1.] [NodeJS Homepage](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/)  
> [2.] [JavaScript Documentation](https://developer.mozilla.org/en-US/docs/Web/JavaScript/EventLoop)  
> [3.] [Article of Mr. Deepal Jayasekara](https://jsblog.insiderattack.net/event-loop-and-the-big-picture-nodejs-event-loop-part-1-1cb67a182810)