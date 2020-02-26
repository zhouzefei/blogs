> 在平常工作或者面试过程中，经常会用或考 防抖`debounce` 相关知识。记录一下学习过程。

什么是防抖？即降低过程中的抖动。触发完事件的n秒的时间内不再触发。如果在此期间再次触发，则按新的触发时间为准重新计算。
在一些频繁的事件中例如：scroll, onSize, keyup等，可以利用防抖来避免这些事件被频繁的触发。那么就看下是如何做到的？

<br/>

+ 由浅入深，先看下简单点的实现

    ```javascript
        const debounce = (fn, delay) => {
            let timer = null;
            return () => {
                clearTimeout(timer)
                timer = setTimeout(fn,delay)
            }
        }

        window.onscroll = debounce(()=>{
            console.log(1)
        },60)
    ```

+ 立即执行
  ```javascript
    const debounce = (fn, delay, immediate) => {
        let timer = null;
        return () => {
           if(timer){
               clearTimeout(timer);
           }
           // 如果立即执行则
           if(immediate){
               // 如果是第一次 则立即执行
               const runNow = !timer;
               if(runNow){
                   fn();
               }
               // 此时赋值timer，到一定时间清空 进行下一次run
               timer = setTimeout(()=>{
                   timer = null
               },delay)
           }else{
               timer = setTimeout(fn,delay)
           }
        }
    }
  ```

+ 增加cancel功能
  ```javascript
    const debounce = (fn, delay, immediate) => {
        let timer = null;
        const debounced = () => {
           if(timer){
               clearTimeout(timer);
           }
           // 如果立即执行则
           if(immediate){
               // 如果是第一次 则立即执行
               const runNow = !timer;
               if(runNow){
                   fn();
               }
               // 此时赋值timer，到一定时间清空 进行下一次run
               timer = setTimeout(()=>{
                   timer = null
               },delay)
           }else{
               timer = setTimeout(fn,delay)
           }
        }
        debounced.cancel = () => {
            clearTimeout(timer);
            timer = null
        }
        return debounced;
    }
  ```

+ 一些动画场景中debounce的运用
   ```javascript
        const debounceAnimateFrame = () => {
            var t;
            return function () {
                cancelAnimationFrame(t)
                t = requestAnimationFrame(func);
            }
        }
   ```
