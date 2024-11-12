---
title: Promise
featured_image: ./images/falcon.jpeg
---

#### Promise 的状态有哪些？它们如何转换？

    Pending（进行中）：初始状态，操作尚未完成。
    Fulfilled（已完成）：操作成功完成，Promise 有了结果。
    Rejected（已拒绝）：操作失败，Promise 有了失败原因。 状态一旦变为 Fulfilled 或 Rejected，就不可再改变。

---

#### Promise.resolve() 和 Promise.reject() 的用途

    Promise.resolve() 返回一个状态为 Fulfilled 的 Promise，并带有指定的值。
    Promise.reject() 返回一个状态为 Rejected 的 Promise，并带有指定的原因。

---

#### 解释 Promise.all()，以及它在什么情况下使用？它的缺点是什么？

    Promise.all() 会并行执行多个 Promise，并在所有 Promise 都 Fulfilled 时返回一个新 Promise，其中包含每个 Promise 的结果。缺点是如果任一 Promise 失败，整个 Promise.all() 会被拒绝。

---

#### Promise.race() 是如何工作的？它和 Promise.all() 有什么区别？

    Promise.race() 返回的 Promise 会在第一个输入的 Promise 解决或拒绝时解决或拒绝。与 Promise.all() 不同，它不等待其他 Promise 完成。

---

#### 如何处理 Promise 链中的错误？在链的中间某个 then() 抛出错误时会发生什么？

    如果 then() 抛出错误或返回一个被拒绝的 Promise，错误会传递给链中的下一个 catch()。

    ```js
        fetchData()
            .then(() => { throw new Error('Error in the chain'); })
            .catch(err => console.error('Caught:', err));

    ```

---

#### 什么是 Promise.finally()？它有什么作用？

    finally() 方法用于在 Promise 结束时（无论是 Fulfilled 还是 Rejected）执行清理操作。

    ```js
        fetchData()
            .finally(() => console.log('Cleanup done'));

    ```

---

#### 如果一个 Promise 在 then() 中返回一个新的 Promise，会发生什么？

    返回的新的 Promise 会接管链的控制，后续的 then() 会等待新 Promise 解决或拒绝。

---

#### 如何实现一个带有超时功能的 Promise？比如，一个 Promise 如果在 3 秒内没有完成，就自动失败。

    ```js
        function timeoutPromise(promise, ms) {
            let timeout = new Promise((_, reject) =>
                setTimeout(() => reject(new Error('Timed out')), ms)
            );
            return Promise.race([promise, timeout]);
        }

    ```

---

#### 如何创建一个自定义的 Promise 实现重试机制，最多重试 3 次？

    ```js
        function retryPromise(fn, retries = 3) {
            return fn().catch(err => {
                if (retries > 1) return retryPromise(fn, retries - 1);
                throw err;
            });
        }

    ```

---

#### 解释 async/await 是如何简化 Promise 链的，并描述其工作原理。 async/await 语法使异步代码看起来像同步代码。await 暂停 async 函数的执行，等待 Promise 解决，然后返回结果。

```js
    async function fetchDataAndProcess() {
    try {
        const data = await fetchData();
        const result = await processResult(data);
        return result;
    } catch (err) {
        console.error('Error:', err);
    }
    }

```

---

#### async/await 如何与 try/catch 配合使用来处理异步操作中的错误？ 可以在 async 函数中使用 try/catch 来捕获 await 抛出的错误：


    ```js
        async function fetchData() {
            try {
                const result = await fetch('https://api.example.com/data');
                return result.json();
            } catch (err) {
                console.error('Failed to fetch:', err);
            }
        }

    ```

---

#### await 阻塞线程吗？解释为什么或者为什么不阻塞。

    await 不会阻塞整个线程。它只暂停 async 函数的执行，释放线程以允许其他代码运行。

---

#### 在 async 函数中如果没有显式使用 await 会发生什么？

    async 函数返回的是一个 Promise，如果内部没有使用 await，它会立即返回一个已经被解决的 Promise。

---

#### Promise.allSettled() 与 Promise.all() 的区别是什么？什么场景下使用 allSettled()？

    Promise.allSettled() 等待所有 Promise 完成并返回每个 Promise 的状态，不论是否成功。适合用于不希望因为一个 Promise 失败而影响其他的场景。

---

#### 如何并行运行多个 Promise 并在每个 Promise 解决时立即处理结果？

    可以使用 Promise.all() 或者 Promise.allSettled() 并在 .then() 中逐一处理。

    ```js
        const promises = [fetchData1(), fetchData2(), fetchData3()];
        promises.forEach(p => p.then(console.log).catch(console.error));

    ```

---

#### 解释如何在 Node.js 的回调模式中使用 Promise。如何将一个异步的回调函数封装成 Promise？ 

    使用 util.promisify() 或者手动封装回调函数：

    ```js
        const fs = require('fs');
        const readFile = (path) => new Promise((resolve, reject) => {
        fs.readFile(path, 'utf-8', (err, data) => {
            if (err) reject(err);
            else resolve(data);
        });
        });

    ```

---

#### 你如何调试 Promise 链中的错误？是否有推荐的工具或方法？

    使用浏览器开发工具或 node --inspect 来调试。可以在 catch() 中添加日志、使用 console.trace() 或 debugger。

---

#### 写一个函数 delay(ms)，返回一个在指定毫秒后完成的 Promise。

    ```js
    function delay(ms) {
        return new Promise(resolve => setTimeout(resolve, ms));
    }

    ```

---

#### 实现一个简单的 Promise 池控制器，限制同时运行的 Promise 数量。

    ```js
    function promisePool(tasks, poolLimit) {
        let active = 0;
        const results = [];

        return new Promise(resolve => {
            function next() {
            if (tasks.length === 0 && active === 0) return resolve(results);
            if (tasks.length === 0 || active >= poolLimit) return;

            active++;
            const task = tasks.shift();
            task().then(result => {
                results.push(result);
                active--;
                next();
            }).catch(err => {
                results.push(err);
                active--;
                next();
            });

            next();
            }
            next();
        });
    }

    ```

---

#### 如何捕获并处理 Promise.all() 中单个 Promise 的错误，而不影响其他 Promise 的执行？

    ```js
        const promises = [p1.catch(err => err), p2.catch(err => err), p3.catch(err => err)];
        Promise.all(promises).then(results => console.log(results));
    ```

---

#### 写一个函数，使用 Promise 读取文件内容，如果读取失败则返回默认值。

    ```js
        const fs = require('fs').promises;

        async function readFileOrDefault(path, defaultValue) {
        try {
            return await fs.readFile(path, 'utf-8');
        } catch {
            return defaultValue;
        }
        }

    ```

---