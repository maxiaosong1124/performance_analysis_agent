# AI性能分析智能体 - C++项目架构设计

> 技术栈：C++11-C++20，优先使用C++17特性，避免C++23

## 🏗️ 整体架构

```
┌─────────────────────────────────────────────────────────────────┐
│                      Presentation Layer                          │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────────┐  │
│  │ CLI Tool │  │ TUI Tool │  │ Web UI   │  │ AI Chat Interface │  │
│  └──────────┘  └──────────┘  └──────────┘  └──────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────────┐
│                      Application Layer                           │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐   │
│  │ Use Cases    │  │ Orchestrator │  │ Report Generator     │   │
│  │ (性能分析场景) │  │ (工作流编排)  │  │ (报告生成)           │   │
│  └──────────────┘  └──────────────┘  └──────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────────┐
│                      Analysis Layer                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────────┐  │
│  │ Flame    │  │ Off-CPU  │  │ Bottleneck│  │ Anomaly         │  │
│  │ Graph    │  │ Analysis │  │ Detection │  │ Detection       │  │
│  └──────────┘  └──────────┘  └──────────┘  └──────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────────┐
│                      Data Collection Layer                       │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────────┐  │
│  │ ProcFS   │  │ Tracer   │  │ Sampler  │  │ eBPF Collector  │  │
│  │ Reader   │  │ Engine   │  │ Engine   │  │ (libbpf wrapper)│  │
│  └──────────┘  └──────────┘  └──────────┘  └──────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────────┐
│                      Infrastructure Layer                        │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────────┐  │
│  │ Ring     │  │ Thread   │  │ Event    │  │ Error Handling  │  │
│  │ Buffer   │  │ Pool     │  │ System   │  │ & Logging       │  │
│  └──────────┘  └──────────┘  └──────────┘  └──────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📁 目录结构

```
ai-perf-agent/
├── CMakeLists.txt                 # 根CMake配置
├── cmake/                         # CMake模块
│   ├── FindLibBpf.cmake
│   └── CompilerWarnings.cmake
│
├── src/                           # 源代码
│   ├── common/                    # 公共基础设施
│   │   ├── macros.hpp             # 宏定义
│   │   ├── types.hpp              # 类型别名
│   │   ├── noncopyable.hpp        # 不可复制基类
│   │   ├── result.hpp             # Result<T, E> 错误处理
│   │   └── expected.hpp           # std::expected 兼容实现(C++23前)
│   │
│   ├── infrastructure/            # 基础设施层
│   │   ├── ring_buffer/           # 无锁环形缓冲区
│   │   │   ├── ring_buffer.hpp
│   │   │   └── ring_buffer.cpp
│   │   ├── thread_pool/           # 线程池
│   │   │   ├── thread_pool.hpp
│   │   │   └── thread_pool.cpp
│   │   ├── event_bus/             # 事件总线（观察者模式）
│   │   │   ├── event_bus.hpp
│   │   │   └── event_bus.cpp
│   │   └── logger/                # 日志系统
│   │       ├── logger.hpp
│   │       └── logger.cpp
│   │
│   ├── collector/                 # 数据采集层
│   │   ├── base_collector.hpp     # 采集器接口基类
│   │   ├── procfs/                # /proc解析模块
│   │   │   ├── procfs_reader.hpp
│   │   │   ├── procfs_reader.cpp
│   │   │   ├── process_info.hpp   # 进程信息结构
│   │   │   ├── cpu_info.hpp       # CPU信息结构
│   │   │   └── memory_info.hpp    # 内存信息结构
│   │   ├── tracer/                # 追踪引擎(kprobe/uprobe)
│   │   │   ├── tracer.hpp
│   │   │   ├── tracer.cpp
│   │   │   ├── kprobe_tracer.hpp
│   │   │   └── uprobe_tracer.hpp
│   │   ├── sampler/               # 采样引擎
│   │   │   ├── sampler.hpp
│   │   │   ├── cpu_sampler.hpp    # CPU采样
│   │   │   └── stack_walker.hpp   # 栈回溯
│   │   └── ebpf/                  # eBPF封装层
│   │       ├── ebpf_loader.hpp    # eBPF程序加载器
│   │       ├── ebpf_loader.cpp
│   │       ├── ebpf_program.hpp   # eBPF程序封装
│   │       ├── bpf_helpers.h      # BPF辅助函数
│   │       └── programs/          # eBPF程序源码
│   │           ├── cpu_profile.bpf.c
│   │           └── off_cpu.bpf.c
│   │
│   ├── analyzer/                  # 分析引擎层
│   │   ├── base_analyzer.hpp      # 分析器接口基类
│   │   ├── flame_graph/           # 火焰图
│   │   │   ├── flame_graph.hpp
│   │   │   ├── flame_graph.cpp
│   │   │   └── svg_generator.hpp
│   │   ├── off_cpu/               # Off-CPU分析
│   │   │   ├── off_cpu_analyzer.hpp
│   │   │   └── off_cpu_analyzer.cpp
│   │   ├── bottleneck/            # 瓶颈检测
│   │   │   ├── bottleneck_analyzer.hpp
│   │   │   └── rules/             # 检测规则
│   │   └── anomaly/               # 异常检测
│   │       ├── anomaly_detector.hpp
│   │       └── statistical_model.hpp
│   │
│   ├── ai/                        # AI模块（可选）
│   │   ├── llm_client/            # LLM客户端
│   │   │   ├── llm_client.hpp
│   │   │   └── openai_client.hpp
│   │   ├── prompt_builder/        # Prompt构建
│   │   │   └── prompt_builder.hpp
│   │   └── diagnosis/             # 智能诊断
│   │       └── ai_diagnoser.hpp
│   │
│   ├── application/               # 应用层
│   │   ├── use_cases/             # 用例
│   │   │   ├── system_overview.hpp
│   │   │   ├── process_analysis.hpp
│   │   │   └── performance_report.hpp
│   │   └── orchestrator.hpp       # 工作流编排
│   │
│   └── presentation/              # 表示层
│       ├── cli/                   # 命令行界面
│       │   ├── main.cpp
│       │   ├── commands.hpp
│       │   └── formatters.hpp
│       └── tui/                   # 终端UI（可选）
│           └── tui_app.hpp
│
├── tests/                         # 测试代码
│   ├── unit/                      # 单元测试
│   ├── integration/               # 集成测试
│   └── benchmark/                 # 性能测试
│
├── docs/                          # 文档
│   ├── design/                    # 设计文档
│   └── api/                       # API文档
│
└── third_party/                   # 第三方库（可选submodule）
    ├── fmt/                       # 格式化库（C++20前替代std::format）
    ├── spdlog/                    # 日志库（可选，也可自实现）
    └── catch2/                    # 测试框架
```

---

## 🎯 核心设计原则

### 1. RAII与资源管理

```cpp
// 使用RAII管理文件描述符
class FileDescriptor {
public:
    explicit FileDescriptor(int fd) : fd_(fd) {}
    ~FileDescriptor() { if (fd_ >= 0) ::close(fd_); }
    
    // 禁止复制，允许移动
    FileDescriptor(const FileDescriptor&) = delete;
    FileDescriptor& operator=(const FileDescriptor&) = delete;
    
    FileDescriptor(FileDescriptor&& other) noexcept 
        : fd_(other.fd_) { other.fd_ = -1; }
    FileDescriptor& operator=(FileDescriptor&& other) noexcept {
        if (this != &other) {
            if (fd_ >= 0) ::close(fd_);
            fd_ = other.fd_;
            other.fd_ = -1;
        }
        return *this;
    }
    
    int get() const { return fd_; }
    bool valid() const { return fd_ >= 0; }
    
private:
    int fd_ = -1;
};
```

### 2. 错误处理：Result<T, E> 模式

```cpp
// common/result.hpp
#pragma once
#include <variant>
#include <utility>
#include <string>

namespace perf::common {

template<typename T, typename E = std::string>
class Result {
public:
    // 成功构造
    Result(T value) : data_(std::move(value)) {}
    
    // 错误构造
    static Result Err(E error) {
        Result result;
        result.data_ = std::move(error);
        return result;
    }
    
    bool ok() const { return std::holds_alternative<T>(data_); }
    bool hasError() const { return std::holds_alternative<E>(data_); }
    
    T& value() & { return std::get<T>(data_); }
    const T& value() const & { return std::get<T>(data_); }
    T&& value() && { return std::get<T>(std::move(data_)); }
    
    E& error() & { return std::get<E>(data_); }
    const E& error() const & { return std::get<E>(data_); }
    
    // 函数式操作 (C++17)
    template<typename F>
    auto map(F&& f) -> Result<std::invoke_result_t<F, T>, E> {
        if (ok()) {
            return Result<std::invoke_result_t<F, T>, E>(f(value()));
        }
        return Result<std::invoke_result_t<F, T>, E>::Err(error());
    }
    
    template<typename F>
    Result andThen(F&& f) {
        if (ok()) {
            return f(value());
        }
        return Result::Err(error());
    }
    
private:
    Result() = default;
    std::variant<T, E> data_;
};

} // namespace perf::common
```

### 3. 接口抽象：纯虚基类 + 工厂模式

```cpp
// collector/base_collector.hpp
#pragma once
#include <memory>
#include <vector>
#include <chrono>
#include "common/result.hpp"

namespace perf::collector {

// 采样数据基类
struct SampleData {
    virtual ~SampleData() = default;
    std::chrono::steady_clock::time_point timestamp;
};

// 采集器接口
class BaseCollector {
public:
    virtual ~BaseCollector() = default;
    
    // 核心接口
    virtual common::Result<void> initialize() = 0;
    virtual common::Result<void> start() = 0;
    virtual common::Result<void> stop() = 0;
    virtual common::Result<std::vector<std::unique_ptr<SampleData>>> collect() = 0;
    
    // 元信息
    virtual const char* name() const = 0;
    virtual bool isRunning() const = 0;
    
protected:
    BaseCollector() = default;
};

// 采集器工厂
class CollectorFactory {
public:
    template<typename T, typename... Args>
    static std::unique_ptr<BaseCollector> create(Args&&... args) {
        static_assert(std::is_base_of_v<BaseCollector, T>, 
                      "T must derive from BaseCollector");
        return std::make_unique<T>(std::forward<Args>(args)...);
    }
};

} // namespace perf::collector
```

### 4. 无锁环形缓冲区（核心基础设施）

```cpp
// infrastructure/ring_buffer/ring_buffer.hpp
#pragma once
#include <atomic>
#include <memory>
#include <vector>
#include <optional>
#include <type_traits>

namespace perf::infrastructure {

// 基于缓存行对齐的无锁环形缓冲区
template<typename T, size_t Capacity>
class RingBuffer {
    static_assert(std::is_trivially_copyable_v<T>, 
                  "T must be trivially copyable");
    static_assert((Capacity & (Capacity - 1)) == 0, 
                  "Capacity must be power of 2");
    
public:
    RingBuffer() : buffer_(Capacity) {}
    
    // 生产者：写入数据 (多生产者需外部同步，单生产者无锁)
    bool push(const T& item) {
        const size_t current_write = write_idx_.load(std::memory_order_relaxed);
        const size_t next_write = (current_write + 1) & (Capacity - 1);
        
        // 检查是否满
        if (next_write == read_idx_.load(std::memory_order_acquire)) {
            return false; // 缓冲区满
        }
        
        buffer_[current_write] = item;
        write_idx_.store(next_write, std::memory_order_release);
        return true;
    }
    
    // 消费者：读取数据 (支持多消费者)
    std::optional<T> pop() {
        const size_t current_read = read_idx_.load(std::memory_order_relaxed);
        
        // 检查是否空
        if (current_read == write_idx_.load(std::memory_order_acquire)) {
            return std::nullopt;
        }
        
        T item = buffer_[current_read];
        read_idx_.store((current_read + 1) & (Capacity - 1), 
                       std::memory_order_release);
        return item;
    }
    
    size_t size() const {
        const size_t write = write_idx_.load(std::memory_order_relaxed);
        const size_t read = read_idx_.load(std::memory_order_relaxed);
        return (write - read) & (Capacity - 1);
    }
    
    bool empty() const {
        return read_idx_.load(std::memory_order_acquire) == 
               write_idx_.load(std::memory_order_acquire);
    }
    
private:
    alignas(64) std::atomic<size_t> read_idx_{0};
    alignas(64) std::atomic<size_t> write_idx_{0};
    std::vector<T> buffer_;
};

} // namespace perf::infrastructure
```

### 5. 事件驱动架构

```cpp
// infrastructure/event_bus/event_bus.hpp
#pragma once
#include <functional>
#include <map>
#include <typeindex>
#include <vector>
#include <mutex>
#include <shared_mutex>

namespace perf::infrastructure {

// 事件基类
struct Event {
    virtual ~Event() = default;
};

// 类型安全的观察者模式
class EventBus {
public:
    using HandlerId = size_t;
    
    template<typename EventType>
    HandlerId subscribe(std::function<void(const EventType&)> handler) {
        static_assert(std::is_base_of_v<Event, EventType>,
                      "EventType must derive from Event");
        
        std::unique_lock lock(mutex_);
        HandlerId id = next_id_++;
        
        auto& handlers = subscribers_[typeid(EventType)];
        handlers.emplace_back(id, [handler](const Event& e) {
            handler(static_cast<const EventType&>(e));
        });
        
        return id;
    }
    
    template<typename EventType>
    void unsubscribe(HandlerId id) {
        std::unique_lock lock(mutex_);
        auto it = subscribers_.find(typeid(EventType));
        if (it != subscribers_.end()) {
            auto& handlers = it->second;
            handlers.erase(
                std::remove_if(handlers.begin(), handlers.end(),
                    [id](const auto& pair) { return pair.first == id; }),
                handlers.end()
            );
        }
    }
    
    template<typename EventType>
    void publish(const EventType& event) {
        static_assert(std::is_base_of_v<Event, EventType>,
                      "EventType must derive from Event");
        
        std::shared_lock lock(mutex_);
        auto it = subscribers_.find(typeid(EventType));
        if (it != subscribers_.end()) {
            for (const auto& [id, handler] : it->second) {
                handler(event);
            }
        }
    }
    
private:
    std::shared_mutex mutex_;
    std::map<std::type_index, std::vector<std::pair<HandlerId, 
             std::function<void(const Event&)>>>> subscribers_;
    std::atomic<HandlerId> next_id_{0};
};

} // namespace perf::infrastructure
```

---

## 📦 关键技术选型

### 语言特性使用建议

| 特性 | C++版本 | 使用场景 | 示例 |
|------|---------|---------|------|
| `auto` | C++11 | 类型推导 | `auto result = func();` |
| 智能指针 | C++11 | 资源管理 | `std::unique_ptr`, `std::shared_ptr` |
| 移动语义 | C++11 | 性能优化 | 实现移动构造函数 |
| `constexpr` | C++11/14/17 | 编译期计算 | 配置常量 |
| `lambda` | C++11/14/17/20 | 回调、算法 | 事件处理、STL算法 |
| `variant/optional` | C++17 | 多态返回值 | `Result<T,E>`实现 |
| `string_view` | C++17 | 字符串处理 | 解析/proc文件 |
| `if constexpr` | C++17 | 编译期分支 | 模板元编程 |
| `shared_mutex` | C++14/17 | 读写锁 | EventBus实现 |
| `concept` | C++20 | 模板约束 | RingBuffer类型检查 |
| `format` | C++20 | 格式化 | 日志输出（或fmt库） |
| `module` | C++20 | ❌暂不使用 | 编译器支持不完善 |
| `coroutine` | C++20 | ❌暂不使用 | 复杂度过高 |

### 推荐的第三方库

```cmake
# 必要的库
FetchContent_Declare(
    fmt
    GIT_REPOSITORY https://github.com/fmtlib/fmt.git
    GIT_TAG 10.1.1  # C++11兼容
)
# 用于：格式化输出（C++20前替代std::format）

FetchContent_Declare(
    spdlog
    GIT_REPOSITORY https://github.com/gabime/spdlog.git
    GIT_TAG v1.12.0
)
# 用于：高性能日志（可选，也可自实现）

FetchContent_Declare(
    catch2
    GIT_REPOSITORY https://github.com/catchorg/Catch2.git
    GIT_TAG v3.4.0
)
# 用于：单元测试

# eBPF相关
# libbpf: 系统包或静态链接
find_package(PkgConfig REQUIRED)
pkg_check_modules(LIBBPF REQUIRED libbpf)
```

---

## 🔧 模块详细设计

### 1. ProcFS解析模块

```cpp
// collector/procfs/process_info.hpp
#pragma once
#include <string>
#include <vector>
#include <chrono>

namespace perf::collector {

struct ProcessInfo {
    pid_t pid{0};
    pid_t ppid{0};
    std::string name;
    std::string state;
    
    // CPU统计
    struct CpuStat {
        uint64_t utime{0};      // 用户态时间
        uint64_t stime{0};      // 内核态时间
        uint64_t cutime{0};     // 子进程用户态时间
        uint64_t cstime{0};     // 子进程内核态时间
    } cpu_stat;
    
    // 内存统计 (单位：KB)
    struct MemoryStat {
        uint64_t vsize{0};      // 虚拟内存
        uint64_t rss{0};        // 物理内存
        uint64_t shared{0};     // 共享内存
    } memory_stat;
    
    // IO统计
    struct IoStat {
        uint64_t read_bytes{0};
        uint64_t write_bytes{0};
        uint64_t cancelled_write_bytes{0};
    } io_stat;
    
    std::chrono::steady_clock::time_point timestamp;
};

class ProcFsReader {
public:
    // 获取所有进程列表
    common::Result<std::vector<ProcessInfo>> readAllProcesses();
    
    // 获取单个进程信息
    common::Result<ProcessInfo> readProcess(pid_t pid);
    
    // 获取系统整体信息
    common::Result<SystemInfo> readSystemInfo();
    
    // 计算进程CPU使用率（需要前后两次采样）
    double calculateCpuUsage(const ProcessInfo& prev, 
                            const ProcessInfo& curr) const;
};

} // namespace perf::collector
```

### 2. 追踪引擎模块

```cpp
// collector/tracer/tracer.hpp
#pragma once
#include <functional>
#include <string>
#include "collector/base_collector.hpp"

namespace perf::collector {

// 追踪事件数据
struct TraceEvent : public SampleData {
    enum Type { KPROBE, UPROBE, TRACEPOINT } type;
    std::string function;
    uint64_t timestamp_ns;
    std::vector<uint64_t> args;
    std::vector<uintptr_t> stack;
};

// 追踪配置
struct TracerConfig {
    std::string target;           // 目标函数名/地址
    bool recordStack{false};      // 是否记录栈
    uint32_t maxStackDepth{16};   // 最大栈深度
    std::string filter;           // 过滤条件（可选）
};

class Tracer : public BaseCollector {
public:
    explicit Tracer(TracerConfig config);
    ~Tracer() override;
    
    // BaseCollector接口实现
    common::Result<void> initialize() override;
    common::Result<void> start() override;
    common::Result<void> stop() override;
    common::Result<std::vector<std::unique_ptr<SampleData>>> collect() override;
    const char* name() const override { return "Tracer"; }
    bool isRunning() const override;
    
    // 设置回调（实时处理）
    void onEvent(std::function<void(const TraceEvent&)> callback);
    
private:
    class Impl; // PIMPL模式，隐藏平台相关实现
    std::unique_ptr<Impl> pimpl_;
    TracerConfig config_;
};

} // namespace perf::collector
```

### 3. eBPF封装层

```cpp
// collector/ebpf/ebpf_loader.hpp
#pragma once
#include <memory>
#include <string>
#include <vector>
#include <bpf/libbpf.h>
#include "common/result.hpp"

namespace perf::collector {

// eBPF程序加载结果
struct EbpfProgram {
    std::string name;
    enum bpf_prog_type type;
    struct bpf_program* prog{nullptr};
    struct bpf_link* link{nullptr};
};

// eBPF Maps封装
template<typename Key, typename Value>
class EbpfMap {
public:
    EbpfMap(struct bpf_map* map) : map_(map) {}
    
    common::Result<void> update(const Key& key, const Value& value);
    common::Result<Value> lookup(const Key& key);
    common::Result<void> deleteKey(const Key& key);
    common::Result<std::vector<std::pair<Key, Value>>> iterate();
    
private:
    struct bpf_map* map_;
};

// eBPF加载器
class EbpfLoader {
public:
    EbpfLoader();
    ~EbpfLoader();
    
    // 从文件加载eBPF程序
    common::Result<void> loadFromFile(const std::string& path);
    
    // 从内存加载（嵌入式字节码）
    common::Result<void> loadFromMemory(const std::vector<uint8_t>& bytecode);
    
    // Attach程序
    common::Result<void> attachKprobe(const std::string& prog_name, 
                                       const std::string& kernel_func);
    common::Result<void> attachTracepoint(const std::string& prog_name,
                                          const std::string& category,
                                          const std::string& name);
    common::Result<void> attachUprobe(const std::string& prog_name,
                                       const std::string& binary,
                                       uint64_t offset);
    
    // 获取Map
    template<typename Key, typename Value>
    common::Result<EbpfMap<Key, Value>> getMap(const std::string& name);
    
    // 开始/停止轮询Maps
    void startPolling(std::chrono::milliseconds interval);
    void stopPolling();
    
private:
    class Impl;
    std::unique_ptr<Impl> pimpl_;
};

} // namespace perf::collector
```

---

## 🏃 构建配置

### CMakeLists.txt 示例

```cmake
cmake_minimum_required(VERSION 3.16)
project(ai-perf-agent VERSION 0.1.0 LANGUAGES CXX)

# C++标准
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# 编译选项
add_compile_options(
    -Wall -Wextra -Wpedantic
    -Werror=return-type
    -fno-omit-frame-pointer  # 保留栈帧指针，便于栈回溯
)

# 调试/Release配置
set(CMAKE_CXX_FLAGS_DEBUG "-g -O0 -fsanitize=address,undefined")
set(CMAKE_CXX_FLAGS_RELEASE "-O3 -DNDEBUG")

# 查找依赖
find_package(PkgConfig REQUIRED)
pkg_check_modules(LIBBPF REQUIRED libbpf)
find_package(Threads REQUIRED)

# FetchContent for third_party
include(FetchContent)
FetchContent_Declare(
    fmt
    GIT_REPOSITORY https://github.com/fmtlib/fmt.git
    GIT_TAG 10.1.1
)
FetchContent_MakeAvailable(fmt)

# 源文件
file(GLOB_RECURSE SOURCES src/*.cpp)
file(GLOB_RECURSE HEADERS src/*.hpp)

# 主库
add_library(perf_agent_core STATIC ${SOURCES})
target_include_directories(perf_agent_core PUBLIC 
    ${CMAKE_SOURCE_DIR}/src
    ${LIBBPF_INCLUDE_DIRS}
)
target_link_libraries(perf_agent_core PUBLIC
    fmt::fmt
    ${LIBBPF_LIBRARIES}
    Threads::Threads
    elf
    z
)

# CLI可执行文件
add_executable(perf_agent_cli src/presentation/cli/main.cpp)
target_link_libraries(perf_agent_cli PRIVATE perf_agent_core)

# 测试
enable_testing()
add_subdirectory(tests)
```

---

## 🧪 测试策略

```cpp
// tests/unit/test_ring_buffer.cpp
#include <catch2/catch_test_macros.hpp>
#include "infrastructure/ring_buffer/ring_buffer.hpp"

using namespace perf::infrastructure;

TEST_CASE("RingBuffer basic operations", "[ring_buffer]") {
    RingBuffer<int, 16> rb;
    
    SECTION("push and pop single element") {
        REQUIRE(rb.push(42));
        auto val = rb.pop();
        REQUIRE(val.has_value());
        REQUIRE(*val == 42);
    }
    
    SECTION("buffer full behavior") {
        for (int i = 0; i < 15; ++i) {
            REQUIRE(rb.push(i));
        }
        REQUIRE_FALSE(rb.push(100)); // 应该失败
    }
}

TEST_CASE("RingBuffer concurrent access", "[ring_buffer][concurrent]") {
    // 多线程测试...
}
```

---

## 📈 性能优化要点

1. **内存管理**
   - 使用对象池管理频繁创建/销毁的对象
   - 预分配环形缓冲区，避免运行时分配
   - 使用`std::string_view`避免字符串拷贝

2. **并发设计**
   - 单生产者单消费者场景使用无锁队列
   - 多生产者/多消费者使用细粒度锁或无锁算法
   - 避免伪共享（cache line对齐）

3. **系统调用优化**
   - 批量读取/proc信息
   - 使用`readv`/`writev`减少系统调用次数
   - eBPF maps使用批量读取接口

4. **编译优化**
   - 关键路径使用`[[likely]]`/`[[unlikely]]` (C++20)
   - 热点函数使用`inline`或`__attribute__((always_inline))`
   - PGO（Profile Guided Optimization）

---

*架构设计版本：v1.0*
*最后更新：2026-02-15*
