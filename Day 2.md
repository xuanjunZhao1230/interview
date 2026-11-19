# 测试开发实习生面试记录 — Day 2

## 一、Python 多线程与线程安全

### 1. Python 字典是线程安全的吗？如何保证线程安全？

**答案：**

Python 字典不能保证复合读写操作的线程安全。多个线程同时修改共享字典，可能产生数据竞争、更新丢失等问题。

可以使用 `threading.Lock()` 对共享字典的读写操作加锁，保证同一时间只有一个线程进入临界区。

```python
import threading

users = {}
lock = threading.Lock()

def save_user(user_id, token):
    with lock:
        users[user_id] = token

def get_user(user_id):
    with lock:
        return users.get(user_id)
```

### 2. 为什么推荐使用 with lock，而不是手动 acquire 和 release？

**答案：**

`with lock` 使用上下文管理器管理锁，进入代码块时自动获取锁，退出代码块时自动释放锁。

即使代码块内部发生异常，也能保证释放锁，避免因忘记释放锁导致其他线程一直等待。

### 3. 什么是死锁？如何避免？

**答案：**

死锁是多个线程相互持有对方需要的锁，导致所有线程都无法继续执行。

例如：

- 线程 A 获取 lock1，等待 lock2。
- 线程 B 获取 lock2，等待 lock1。
- 两个线程互相等待，形成死锁。

**解决方法：**

- 统一获取锁的顺序。
- 尽量避免嵌套锁。
- 必要时使用带超时的锁获取机制。

---

## 二、进程与线程

### 4. pytest-xdist 使用多线程还是多进程？为什么？

**答案：**

pytest-xdist 使用多进程执行测试。

主要原因：

1. 可以利用多核 CPU 并行执行测试。
2. 不同 worker 拥有独立的进程内存空间。
3. 降低测试之间共享状态产生的干扰。

**补充：**

使用 `pytest -n 4` 通常会启动 4 个 worker 进程。

每个 worker 都有独立的测试会话，因此 session 级 Fixture 通常会在每个 worker 中分别执行一次。

### 5. 多进程和多线程有什么区别？

| 对比   | 进程           | 线程         |
| ---- | ------------ | ---------- |
| 内存空间 | 相互独立         | 共享进程资源     |
| 资源开销 | 通常较大         | 通常较小       |
| 通信方式 | IPC 等        | 共享内存等      |
| 调度   | 操作系统调度进程中的线程 | CPU 调度基本单位 |

**记忆：**

- 进程是资源分配和保护的基本单位。
- 线程是 CPU 调度的基本单位。

---

## 三、Mock Server 项目

### 6. 为什么使用 ThreadingMixIn + HTTPServer，而不是直接使用 HTTPServer？

**答案：**

普通 HTTPServer 默认采用单线程处理请求。当一个请求处理时间较长时，其他请求需要等待。

ThreadingMixIn 可以让服务器为不同客户端请求创建独立线程，实现并发处理，提高 Mock Server 的并发响应能力。

```python
from socketserver import ThreadingMixIn
from http.server import HTTPServer

class ThreadingHTTPServer(ThreadingMixIn, HTTPServer):
    daemon_threads = True
```

**记忆：**

- HTTPServer：负责 HTTP 服务和请求监听。
- ThreadingMixIn：提供多线程请求处理能力。

### 7. Mock Server 如何处理并发请求和共享数据？

**答案：**

可以使用多线程处理请求，通过 `threading.Lock()` 保护共享字典的读写操作。

同时增加请求参数校验和用户合法性检查，避免无效请求进入业务逻辑。

需要注意：

- 加锁解决共享数据的同步问题。
- 前置过滤解决无效请求问题。
- 独立端口解决多个进程启动服务时的端口冲突问题。

---

## 四、Pytest 自动化测试

### 8. Pytest Fixture 不同作用域有什么区别？测试失败后 teardown 是否执行？

**答案：**

常见作用域：

| Scope    | 执行范围       |
| -------- | ---------- |
| function | 每个测试用例执行一次 |
| class    | 每个测试类执行一次  |
| module   | 每个测试模块执行一次 |
| session  | 每个测试会话执行一次 |

例如：

```python
@pytest.fixture(scope="session")
def login():
    print("登录")
    yield "token123"
    print("退出登录")


@pytest.fixture(scope="function")
def user(login):
    print("创建用户")
    yield login
    print("删除用户")
```

执行两个测试用例时：

```text
登录
创建用户
测试A
删除用户
创建用户
测试B
删除用户
退出登录
```

**重点：**

1. session 级 Fixture 整个会话执行一次。
2. function 级 Fixture 每个测试用例执行一次。
3. Fixture 正常执行到 yield 后，即使测试用例失败，teardown 仍会执行。
4. 清理顺序与依赖顺序相反，function 级先清理，session 级最后清理。

### 9. 测试用例之间应该存在依赖关系吗？如何处理 Token 获取失败？

**答案：**

测试用例应该尽量保持独立，避免一个用例失败导致其他用例连锁失败。

可以使用 Fixture 统一管理 Token。

```python
@pytest.fixture(scope="session")
def token():
    response = requests.post(login_url, json=login_data)
    assert response.status_code == 200
    return response.json()["token"]
```

如果 Token 获取失败，依赖该 Fixture 的测试会出现初始化错误，而不是继续使用无效 Token 请求接口。

**记忆：**

- 测试用例尽量独立。
- 公共前置条件使用 Fixture 管理。
- 前置条件失败时及时终止依赖操作。

---

## 五、接口测试与并发问题定位

### 10. 单线程创建订单正常，多线程并发时出现重复订单，如何定位和解决？

**答案：**

可能原因：

1. 接口缺少幂等性控制。
2. 数据库缺少唯一索引。
3. 查询和创建之间存在并发竞争。
4. 数据库事务和并发控制设计不合理。

**定位步骤：**

1. 检查请求日志，确认是否重复提交。
2. 检查接口订单创建逻辑。
3. 检查数据库事务和唯一约束。
4. 分析并发请求下的数据读写过程。

**解决方案：**

- 使用唯一请求 ID 实现重复请求识别。
- 使用 Redis 防重。
- 使用数据库唯一索引。
- 使用事务和适当的并发控制。

**验证方式：**

使用 JMeter 模拟多个线程同时提交相同订单请求，检查最终数据库中的订单数量，确保不会产生重复订单。

### 什么是接口幂等性？

同一个请求执行一次和执行多次，最终产生的业务结果应该一致。

例如：同一个订单创建请求重复提交 10 次，最终只能创建一个订单。

---

## 六、Day 2 复习重点

### 必须掌握

- [ ] threading.Lock() 的使用
- [ ] 线程安全与数据竞争
- [ ] 死锁的产生与解决
- [ ] 多进程和多线程的区别
- [ ] pytest-xdist 的工作原理
- [ ] ThreadingMixIn 的作用
- [ ] Fixture 的作用域和 teardown
- [ ] 测试用例独立性
- [ ] 接口幂等性与数据库并发控制

**Day 2 学习目标：**

能够结合自己的 Mock Server 和 Pytest 项目，清楚解释并发处理、线程安全、Fixture 生命周期，以及接口并发问题的定位和解决方法。
