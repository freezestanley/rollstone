首先检查同步任务有没有执行完(也就是调用栈是否为空)。
如果为空的话，就去检查micro task (微任务)队列是否有任务，如果有的话就一次性把micro task (微任务)执行完为止（包括micro task (微任务)执行过程中再次生成微任务），直到micro task (微任务)队列为空再去检查宏任务 macro task (宏任务)队列。每执行完一个 macro task (宏任务)还要去检查micro task (微任务)队列是否为空，不为空的话先去把micro task 队列清空。如此反复

上面说了micro task 与 macro task 就是异步任务的分类。创建它们的API也是不一样的比如：
创建micro task (微任务)的有Promise、MutationObserver 等
创建 macro task (宏任务) 的有：setTimeout、 setInterval、requestAnimationFrame MessageChannel还有一些IO事件等(这里rAF有争议，但它绝对不是微任务）
