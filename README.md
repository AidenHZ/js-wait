# Kotlin 模仿 JavaScript 的 `async/await` 协程实现

本项目使用 Kotlin 实现了类似 JavaScript 中 `async/await` 机制的协程处理模型。通过 `suspendCoroutine` 和 `Continuation` 等协程特性，实现了异步任务的管理与执行，并展示了如何与 `Retrofit` 网络请求库结合使用。

## 功能

- **异步处理**：模仿 JavaScript 的 `async/await` 机制，使用 Kotlin 协程管理异步任务。
- **`Retrofit` 集成**：结合 `Retrofit`，实现网络请求异步处理，通过协程等待 API 返回结果。
- **主线程调度**：使用 `Handler` 和 `Looper` 实现 Android 主线程中的异步任务调度。

## 核心功能

### 1. **`await` 函数**

该函数用于处理异步的 `Retrofit` 网络请求，并通过协程机制简化了回调的处理流程：

```kotlin
suspend fun <T> AsyncScope.await(block: () -> Call<T>) = suspendCoroutine<T> { continuation ->
    val call = block()
    call.enqueue(object : Callback<T> {
        override fun onFailure(call: Call<T>, t: Throwable) {
            continuation.resumeWithException(t)
        }

        override fun onResponse(call: Call<T>, response: Response<T>) {
            if (response.isSuccessful) {
                response.body()?.let(continuation::resume) ?: continuation.resumeWithException(NullPointerException())
            } else {
                continuation.resumeWithException(HttpException(response))
            }
        }
    })
}
```
## 功能

### 1. `await` 函数

**功能**：通过 `suspendCoroutine` 挂起协程，并将 `Retrofit` 的回调结果转换为协程返回值。成功时调用 `resume`，失败时调用 `resumeWithException`。

### 2. `async` 函数

```kotlin
fun async(context: CoroutineContext = EmptyCoroutineContext, block: suspend AsyncScope.() -> Unit) {
    val completion = AsyncCoroutine(context)
    block.startCoroutine(completion, completion)
}
```
**功能：启动一个协程执行块，类似于 JavaScript 的 async 函数。在给定的协程上下文中执行异步任务。**
```kotlin
val handlerDispatcher = DispatcherContext(object : Dispatcher {
    val handler = Handler()
    override fun dispatch(block: () -> Unit) {
        handler.post(block)
    }
})
```
**使用 Android 的 Handler 和 Looper 将异步任务调度回主线程，确保网络请求的结果在 UI 线程上处理。**
## 代码结构
- **go.kt**：包含 async 和 await 的核心实现，模仿了 JavaScript 的 async/await 异步机制。
- **DispatcherContext：** 提供了任务调度的上下文，将异步任务调度到 Android 主线程。
- **Retrofit:** 与 Retrofit 集成，使用协程替代回调方式进行网络请求。
### 3. 运行项目
- 确保已安装 JDK 1.8 及以上版本，并配置好 Android Studio 环境。
- 克隆项目到本地：
```bash
git clone https://github.com/你的用户名/项目名.git
```
- 在 Android Studio 中打开项目，并运行应用。
  ## 示例代码:
  ```kotlin
  fun main() {
    Looper.prepare()
    val handlerDispatcher = DispatcherContext(object : Dispatcher {
        val handler = Handler()
        override fun dispatch(block: () -> Unit) {
            handler.post(block)
        }
    })

    async(handlerDispatcher) {
        val user = await { githubApi.getUserCallback("bennyhuo") }
        log(user)
    }
    Looper.loop()
  }
