# HighPerformance-CommScaffold

欢迎来到 **HighPerformance-CommScaffold** 项目！这是一个高性能的通用服务器脚手架，设计用于处理高并发、低延迟的网络服务。该项目借鉴了 Nginx 的 Master-Worker 多进程模型，结合事件驱动机制、线程池和连接池，旨在为开发者提供一个高效、稳定的网络通讯基础框架。

---

## 项目介绍

该项目旨在实现一个高性能的通用服务器脚手架。借鉴了 Nginx 的 Master-Worker 多进程模型，采用事件驱动的方式，结合高效的线程池和连接池，实现了高并发、低延迟的网络服务处理框架。

项目的框架主要包括四个部分：

1. **业务处理线程池**：用于执行用户请求的具体业务逻辑。
2. **数据发送线程**：负责将处理后的结果数据发送回客户端。
3. **连接回收线程**：定期检查并回收闲置的连接，确保资源可以被下一个用户使用。
4. **心跳检测线程**：用于监测用户是否在线，确保连接的有效性。

在处理网络连接时，项目预先创建了连接池，采用 Master-Worker 进程模型，并通过互斥锁机制解决了惊群问题。我们使用 `epoll` 处理大量并发请求，通过线程池优化业务逻辑的处理，避免频繁创建和销毁线程带来的开销。

在数据收发方面，项目定义了包头+包体的格式，解决了 TCP 通信中的粘包问题。包头中的消息 ID 用于确定执行不同的业务处理函数。用户只需专注于实现具体的业务逻辑，而框架负责处理连接管理和调度。

---

## 项目亮点

1. **高效的线程池和连接池管理**：
   - 线程池避免了频繁创建和销毁线程的开销，显著提升了服务器的性能和稳定性。
   - 连接池的设计保证了连接的高效管理，支持高并发场景下的大量连接。

2. **事件驱动模型与 `epoll` 的应用**：
   - 使用 `epoll` 进行事件驱动处理，支持高并发请求的高效处理。
   - 采用水平触发模式（LT），结合连接池减少了事件的重复处理，提升了整体性能。

3. **安全性与稳定性**：
   - 引入了 `SO_REUSEADDR` 选项，解决了端口在 `TIME_WAIT` 状态下无法绑定的问题，提升了服务的可用性。
   - 通过心跳包机制和 Flood 攻击检测，增强了服务器的安全性。

4. **日志系统的优化**：
   - 使用 `O_APPEND` 选项和原子操作，保证了多进程环境下日志的安全写入，避免了日志内容的混乱。

5. **守护进程**：
   - 实现了程序在后台运行，不依赖终端。为了防止守护进程随终端关闭而退出，屏蔽了 `SIGHUP` 信号。

---

## 项目难点

1. **惊群效应处理**：
   - 在多 Worker 进程模式下，多个进程同时抢占同一连接可能导致惊群效应。通过引入 `accept_mutex` 互斥锁，有效地解决了这个问题，确保系统的稳定性和高效性。

2. **粘包问题**：
   - 由于 TCP 协议的特性，可能出现粘包问题。项目通过在数据包中引入包头，包含包体长度和消息 ID 等信息，确保数据的完整性和正确解析。

3. **高并发下的资源管理**：
   - 如何在高并发场景下管理有限的资源（如连接、线程、文件描述符等）是一个难点。项目通过设计高效的线程池、连接池以及延迟回收机制，确保了资源的合理利用和系统的稳定运行。

---

## 项目流程

1. **启动与初始化**：
   - Master 进程读取配置文件，初始化日志系统等；`fork` 出多个 Worker 进程，继承 Master 进程的环境和描述符。

2. **事件循环处理请求**：
   - Worker 进程进入事件循环，使用 `epoll` 进行事件驱动处理，通过互斥锁抢占连接，避免惊群效应；连接建立后，Worker 进程负责读取数据，根据包头信息解析出完整数据包，放入线程池中处理；业务处理完成，数据被封装并放入发送队列，待 `epoll` 检测到可写事件后发给客户端。

3. **连接管理与回收**：
   - 使用动态增长的连接池高效管理大量的连接，并通过延迟回收机制防止连接在处理中途被回收；定期的心跳机制用于监测连接的存活状态，确保连接的及时回收和有效利用。

---

## 环境搭建

要运行此项目，请按照以下步骤操作：

1. **克隆仓库**：
   ```bash
   git clone https://github.com/zzcodd/HighPerformance-CommScaffold.git
   cd HighPerformance-CommScaffold
   ```

2. **构建项目**：
   ```bash
   make
   ```


---

## 贡献

欢迎任何形式的贡献！如果你想改进项目或修复 Bug，可以 Fork 本仓库，进行修改，然后提交 Pull Request。请确保你的代码遵循项目的编码风格并通过所有测试。

---

## 许可

本项目基于 MIT 许可协议，详情请参阅 [LICENSE](LICENSE) 文件。

---

## 作者

本项目由 **zzcodd** 实现并维护。如果你有任何问题或建议，欢迎随时联系！

[![GitHub](https://img.shields.io/badge/GitHub-zzcodd-blue?style=flat-square&logo=github)](https://github.com/zzcodd)

---

感谢你查看本项目！祝你编程愉快！🚀
