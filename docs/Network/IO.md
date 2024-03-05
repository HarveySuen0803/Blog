# Mode

为了避免 User App 破坏 Kernel, 需要分离 User 和 Kernel, 就分为了 User Space 和 Kernel Space. User Space 只能执行 Ring3, 不能直接调用系统资源, 需要通过 Kernel 提供的 System Call Interface 访问. Kernel Space 可以执行 Ring0, 调用一切系统资源

进程运行时, 既要执行 Ring3, 也要执行 Ring0, 所以需要频繁切换 User Mode 和 Kernel Mode

User 想要读取数据, 发送请求给 System Call Interface, 等待 Kernel 查询数据 (eg: 从 Disk 或 Network 中查询数据), 找到数据后, 先存储到 Kernel Space 的 Buffer 中, 再拷贝到 User Space 的 Buffer 中, 完成一次读取操作

User 想要写入数据时, 也需要先写入到 User Space 的 Buffer 中, 再复制到 Kernel Space 的 Buffer 中, Kernel 再从 Buffer 中读取数据, 写入到 Disk, 完成一次写入操作

![](https://note-sun.oss-cn-shanghai.aliyuncs.com/image/202401031139646.png)

# Synchronous, Asynchronous

IO 的 Synchronous 和 Asynchronous 要看两个阶段是否堵塞, 第一个阶段是 User 发送请求给 Kernel 这个过程, 第二个阶段是从 Kernel Space 复制数据到 User Space 这个过程, 所以 IO 真正的 Asynchronous 就只有 Asynchronous IO 这一种模式

![](https://note-sun.oss-cn-shanghai.aliyuncs.com/image/202401031139648.png)

# Blocking IO

Blocking IO 中, User 想要读取数据, 如果 Kernel 查找不到数据, 不会返回查找不到的信息给 User, 而是会一直等待, 直到数据就位后, 再完成后续操作, 返回结果给 User

Blocing IO 全程需要等待, 性能非常差

![](https://note-sun.oss-cn-shanghai.aliyuncs.com/image/202401031139649.png)

# Nonblocking IO

Nonblocking IO 中, User 想要读取数据, 如果 Kernel 查找不到数据, 会直接返回查找不到的信息给 User, User 过段时间再发送读取数据的请求, 循环往复, 直到读取到数据, 整个过程中, 只有数据拷贝会进入等待

Nonblocking IO 看似没有太多堵塞, 但是性能还是很差, User 读取不到数据, 没办法执行后续操作了, 还是需要反复请求 Kernel, 这个过程需要 CPU 执行各种命令, 反而性能不如 Blocking IO


![](https://note-sun.oss-cn-shanghai.aliyuncs.com/image/202401031139650.png)

# Multiplexing IO

Blocking IO 和 Non Blocing IO 都是第一时间调用 recvfrom() 获取数据, 数据不存在时, 要么等待, 要么空转, 都无法很好的利用 CPU, 还会导致其他 Socket 的等待, 非常糟糕

Client 连接 Server 后, 会建立一个关联的 Socket, 这个 Socket 有一个对应的 File Descriptor (FD), FD 是一个从 0 开始递增的 Unsigned Int, 用来关联一个文件, 在 Linux 中一切皆文件, 所以 FD 可以关联一些, 当然就可以用来关联 Socket

Multiplexing IO 中, User 想要读取数据, 会先调用 epoll(), 将所有 Socket 的 FD 传递给 Kernel, Kernel 只需要一个单线程时刻监听这些 FD, 哪个数据就绪了, 就告知 User 去获取数据, 这个时候 User 再调用 recvfrom() 去获取数据, 就不会有堵塞和空转的情况了, 非常好的利用了 CPU

![](https://note-sun.oss-cn-shanghai.aliyuncs.com/image/202401031139651.png)

---

select() 和 poll() 是早期用于 Multiplexing IO 的函数, 他们只会通知 User 有 FD 就绪了, 但是不确定是哪个 FD, 需要 User 遍历所有的 FD 来确定. epoll() 则会告知是哪些 FD 就绪了, 不需要 User 查找了, 非常高效

select() 将所有的 FD 存储到 fds_bits 中, 这是一个 1024b 的数组, 调用 select() 后, 需要先将 fds_bits 从 User Space 复制到 Kernel Space, 等 select() 执行完, 还需要将 fds_bits 从 Kernel Space 复制到 User Space. User 无法直接确定是哪个 FD 就绪了, 还需要再遍历 fds_bits 寻找就绪的 FD. fds_bits 的大小固定为了 1024b, 这个数量放在如今已经完全不够用了

poll() 通过一个 LinkedList 存储 FD, 所以可以存储的 FD 就没有上限了, 但是依旧无法避免两次复制和遍历寻找就绪的 FD. 如果存储的 FD 太多, 遍历的时间会变长, 性能就会下降

epoll() 通过一个 RedBlackTree 存储所有的 FD (rbr), 通过一个 LinkedList 存储就绪的 FD (rdlist). Server 启动时, 会调用 epoll_create() 创建一个 epoll 实例, 包含 rbr 和 rdlist. User 想要添加一个 FD 时, 会调用 epoll_ctl() 添加一个 FD 到 rbr 上, 并且绑定一个 ep_poll_callback, 一旦该 FD 就绪, 就会触发该 Callback, 将 FD 添加到 rdlist 中. 后续只需要循环调用 epoll_wait() 检查 rdlist 中是否有 FD, 如果没有, 就进入等待, 如果有, 就复制到 User Space 的 events 中, 实现事件通知, User 就知道哪些 FD 就绪了, 就可以针对性的发送请求进行读写操作了. epoll() 不需要来回的两次复制, 也不需要遍历寻找就绪的 FD, 性能极强, 而且通过 RedBlackTree 存储 FD, 既能存储大量的 FD, 也能保证性能的稳定

![](https://note-sun.oss-cn-shanghai.aliyuncs.com/image/202401031139652.png)

![](https://note-sun.oss-cn-shanghai.aliyuncs.com/image/202401031139653.png)

---

epoll_wait() 的事件通知有 LevelTriggered (LT, def) 和 EdgeTriggered (ET) 两种模式

LT 进行事件通知后, 会判断该 FD 是否已经全部处理完毕, 如果没有处理完, 就会反复通知, 直到处理完毕, 才会将该 FD 从 rdlist 中移除 (eg: A 需要处理 3KB 的数据, 调用 epoll_wait() 得到数据后, 只处理了 1KB 数据, 此时 LT 模式下, 不会移除该 FD, 而是再次通知 A 去处理数据)

ET 进行事件通知后, 会直接移除该 FD (eg: A 还剩 2KB 数据没有处理, 此时 ET 模式下, 不会管 A 是否处理完的, 直接移除 FD)

通过 LT 处理数据, 需要反复调用 epoll_wait() 进行处理, 这个过程是非常消耗资源的. 如果多个线程都在等待一个 FD, 通知一次给一个线程后, 还会再去通知其他线程, 这就造成了惊群

通过 ET 处理数据, 可以开启一个异步线程, 将 FD 中的所有数据全部循环处理完即可, 不需要反复调用 epoll_wait(), 性能极强, 而且一个线程被通知处理完数据后, 其他线程直接从 User Space 中读取数据即可, 不存在惊群

# Signal Driven IO

Signal Driven IO 中, User 先调用 sigaction() 告知 Kernel 去寻找数据, Kernel 直接返回结果给 User. Kernel 发现数据就绪后, 再调用 sigio() 发送信号给 User. 这个时候 User 再调用 recvfrom() 进行数据的复制, 所以 Signal Driven IO 的流程更类似于我们理解中的 Non Blocking IO

当 IO 数量增多时, 需要发送的信号就越多, sigio() 处理不及时的话, 就会导致 Signal Queue 溢出, 导致信号丢失, 而且这个信号的通知方式就是来回复制一份数据, 效率非常低

![](https://note-sun.oss-cn-shanghai.aliyuncs.com/image/202401031139654.png)

# Asynchronous IO

Asynchronous IO 中, User 调用 aio_read() 通知 Kernel 去查找数据后, 就可以干别的事去啦. Kernel 查找到数据后, 直接复制数据到 User Space 中, 再通知 User 去处理数据, 整个过程不需要 User 去等待, 非常的高效

Asynchronous IO 可以处理的任务数量是有限的, 不能积累太多, 尤其是在 Multi Thread Env 下, 需要做好并发访问的限流, 代码的实现难度就提高了很多, 而 Mutiplexing IO 实现的难度就要小很多, 所以 Multiplexing IO 使用的场景就更多了

![](https://note-sun.oss-cn-shanghai.aliyuncs.com/image/202401031139655.png)
