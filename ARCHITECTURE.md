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

---

## 🔌 扩展模块：外部工具集成 (External Tools)

> 本模块作为**核心学习功能**的补充，允许智能体无缝集成现有的专业性能分析工具。
> 设计目标：不替代核心学习，而是增强生产环境的实用性。

### 架构定位

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           外部工具集成层 (external/)                          │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │                      External Tool Interface                          │   │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────────────┐  │   │
│  │  │ PerfTool │  │ NcuTool  │  │ VtuneTool│  │ BpftraceTool         │  │   │
│  │  │ Adapter  │  │ Adapter  │  │ Adapter  │  │ Adapter              │  │   │
│  │  └────┬─────┘  └────┬─────┘  └────┬─────┘  └──────────┬───────────┘  │   │
│  │       │             │             │                    │              │   │
│  │       └─────────────┴─────────────┴────────────────────┘              │   │
│  │                              │                                         │   │
│  │                    ┌─────────┴─────────┐                               │   │
│  │                    │   Unified Model   │  ← 统一数据模型               │   │
│  │                    │   (跨工具通用)     │                               │   │
│  │                    └─────────┬─────────┘                               │   │
│  └──────────────────────────────┼────────────────────────────────────────┘   │
│                                 │                                            │
│                    ┌────────────┴────────────┐                               │
│                    ▼                         ▼                               │
│         ┌─────────────────┐      ┌─────────────────────┐                     │
│         │  ToolOrchestrator│      │  ExternalAIBridge   │                     │
│         │  (工具编排器)     │      │  (AI桥接)            │                     │
│         └─────────────────┘      └─────────────────────┘                     │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 目录结构

```
src/
├── ... (核心模块)
│
└── external/                          # 外部工具集成（可选编译）
    ├── base/                          # 基础接口
    │   ├── external_tool.hpp          # 外部工具基类
    │   ├── tool_adapter.hpp           # 适配器接口
    │   ├── tool_config.hpp            # 工具配置
    │   └── data_converter.hpp         # 数据转换接口
    │
    ├── adapters/                      # 具体工具适配器
    │   ├── perf/                      # Linux perf
    │   │   ├── perf_adapter.hpp
    │   │   ├── perf_adapter.cpp
    │   │   ├── perf_data_parser.hpp   # perf.data解析
    │   │   ├── perf_script_parser.hpp # perf script解析
    │   │   └── perf_events.hpp        # perf事件定义
    │   │
    │   ├── nvidia/                    # NVIDIA工具
    │   │   ├── ncu_adapter.hpp        # Nsight Compute
    │   │   ├── ncu_adapter.cpp
    │   │   ├── nsys_adapter.hpp       # Nsight Systems
    │   │   ├── nsys_adapter.cpp
    │   │   ├── ncu_csv_parser.hpp     # NCU CSV解析
    │   │   └── nsys_sqlite_parser.hpp # NSYS SQLite解析
    │   │
    │   ├── intel/                     # Intel工具
    │   │   ├── vtune_adapter.hpp
    │   │   └── vtune_adapter.cpp
    │   │
    │   ├── bpftrace/                  # bpftrace
    │   │   ├── bpftrace_adapter.hpp
    │   │   └── bpftrace_adapter.cpp
    │   │
    │   └── valgrind/                  # Valgrind
    │       ├── valgrind_adapter.hpp
    │       └── massif_parser.hpp
    │
    ├── unified/                       # 统一数据模型
    │   ├── unified_profile.hpp        # 统一Profile数据
    │   ├── unified_trace.hpp          # 统一Trace数据
    │   ├── unified_metrics.hpp        # 统一Metrics数据
    │   └── unified_report.hpp         # 统一报告格式
    │
    ├── orchestrator/                  # 工具编排
    │   ├── tool_orchestrator.hpp      # 多工具协调
    │   ├── pipeline_builder.hpp       # 分析流水线
    │   ├── dependency_graph.hpp       # 工具依赖管理
    │   └── result_merger.hpp          # 结果合并
    │
    └── ai_bridge/                     # AI模块桥接
        ├── external_data_bridge.hpp   # 数据转换桥
        └── tool_recommender.hpp       # 工具推荐
```

### 核心接口设计

#### 1. 外部工具基类

```cpp
// external/base/external_tool.hpp
#pragma once
#include <string>
#include <vector>
#include <optional>
#include "common/result.hpp"
#include "external/unified/unified_profile.hpp"

namespace perf::external {

// 工具能力描述
struct ToolCapabilities {
    bool supports_cpu_profiling{false};
    bool supports_gpu_profiling{false};
    bool supports_memory_profiling{false};
    bool supports_io_profiling{false};
    bool supports_network_profiling{false};
    bool supports_system_wide{false};
    bool supports_attach_process{false};
    bool supports_attach_thread{false};
    std::vector<std::string> supported_architectures;
    std::vector<std::string> required_kernel_features;
};

// 工具执行配置
struct ToolExecutionConfig {
    std::string target_binary;           // 目标程序
    std::vector<std::string> target_args; // 目标程序参数
    std::optional<pid_t> attach_pid;     // 附加进程
    std::chrono::seconds duration{10};   // 采样时长
    std::string output_dir{"./perf_data"}; // 输出目录
    std::vector<std::string> extra_args; // 额外参数
};

// 外部工具接口
class ExternalTool {
public:
    virtual ~ExternalTool() = default;
    
    // 工具元信息
    virtual const char* name() const = 0;
    virtual const char* version() const = 0;
    virtual ToolCapabilities capabilities() const = 0;
    
    // 可用性检查
    virtual bool isAvailable() const = 0;
    virtual common::Result<void> checkRequirements() = 0;
    
    // 执行采集
    virtual common::Result<void> execute(const ToolExecutionConfig& config) = 0;
    
    // 停止采集
    virtual common::Result<void> stop() = 0;
    
    // 获取原始输出路径
    virtual std::vector<std::string> getOutputFiles() const = 0;
    
    // 转换为统一数据模型（核心方法）
    virtual common::Result<unified::UnifiedProfile> toUnifiedProfile() = 0;
    virtual common::Result<unified::UnifiedTrace> toUnifiedTrace() = 0;
    virtual common::Result<unified::UnifiedMetrics> toUnifiedMetrics() = 0;
    
protected:
    ExternalTool() = default;
};

// 工具工厂
class ExternalToolFactory {
public:
    using ToolCreator = std::function<std::unique_ptr<ExternalTool>()>;
    
    static ExternalToolFactory& instance();
    
    void registerTool(const std::string& name, ToolCreator creator);
    std::unique_ptr<ExternalTool> createTool(const std::string& name);
    std::vector<std::string> availableTools() const;
    
private:
    ExternalToolFactory() = default;
    std::unordered_map<std::string, ToolCreator> creators_;
};

} // namespace perf::external
```

#### 2. perf适配器实现

```cpp
// external/adapters/perf/perf_adapter.hpp
#pragma once
#include "external/base/external_tool.hpp"

namespace perf::external {

// perf特有配置
struct PerfConfig {
    enum class EventType {
        CPU_CYCLES,
        INSTRUCTIONS,
        CACHE_MISSES,
        BRANCH_MISSES,
        CONTEXT_SWITCHES,
        CUSTOM
    };
    
    std::vector<EventType> events{EventType::CPU_CYCLES, EventType::INSTRUCTIONS};
    std::string custom_event;    // 自定义perf事件
    uint32_t sample_freq{99};    // 采样频率(Hz)
    bool call_graph{true};       // 记录调用栈
    uint32_t stack_size{8192};   // 栈大小
    bool kernel_space{true};     // 包含内核态
    bool user_space{true};       // 包含用户态
};

class PerfAdapter : public ExternalTool {
public:
    explicit PerfAdapter(const PerfConfig& config = {});
    ~PerfAdapter() override;
    
    // ExternalTool接口实现
    const char* name() const override { return "perf"; }
    const char* version() const override;
    ToolCapabilities capabilities() const override;
    bool isAvailable() const override;
    common::Result<void> checkRequirements() override;
    common::Result<void> execute(const ToolExecutionConfig& config) override;
    common::Result<void> stop() override;
    std::vector<std::string> getOutputFiles() const override;
    
    // 数据转换
    common::Result<unified::UnifiedProfile> toUnifiedProfile() override;
    common::Result<unified::UnifiedTrace> toUnifiedTrace() override;
    common::Result<unified::UnifiedMetrics> toUnifiedMetrics() override;
    
    // perf特有功能
    common::Result<void> report(const std::string& perf_data_path);
    common::Result<std::string> annotate(const std::string& symbol);
    
private:
    class Impl;
    std::unique_ptr<Impl> pimpl_;
    PerfConfig perf_config_;
    ToolExecutionConfig exec_config_;
    pid_t perf_pid_{-1};
};

} // namespace perf::external
```

#### 3. NVIDIA Nsight Compute适配器

```cpp
// external/adapters/nvidia/ncu_adapter.hpp
#pragma once
#include "external/base/external_tool.hpp"

namespace perf::external {

// NCU特有配置
struct NcuConfig {
    std::vector<std::string> metrics;    // 采集的指标
    std::string kernel_regex;            // 内核名称过滤
    uint32_t kernel_count{10};           // 分析前N个内核
    bool replay_mode{true};              // 使用kernel replay
    bool import_source{true};            // 导入CUDA源码
    std::vector<std::string> sections;   // 采集的section
};

class NcuAdapter : public ExternalTool {
public:
    explicit NcuAdapter(const NcuConfig& config = {});
    ~NcuAdapter() override;
    
    // ExternalTool接口实现
    const char* name() const override { return "ncu"; }
    const char* version() const override;
    ToolCapabilities capabilities() const override;
    bool isAvailable() const override;
    common::Result<void> checkRequirements() override;
    common::Result<void> execute(const ToolExecutionConfig& config) override;
    common::Result<void> stop() override;
    std::vector<std::string> getOutputFiles() const override;
    
    // 数据转换
    common::Result<unified::UnifiedProfile> toUnifiedProfile() override;
    common::Result<unified::UnifiedTrace> toUnifiedTrace() override;
    common::Result<unified::UnifiedMetrics> toUnifiedMetrics() override;
    
    // NCU特有功能
    common::Result<void> exportToCSV(const std::string& output_path);
    common::Result<std::vector<NcuKernelMetrics>> parseKernelMetrics();
    
private:
    class Impl;
    std::unique_ptr<Impl> pimpl_;
    NcuConfig ncu_config_;
};

// NCU内核指标结构
struct NcuKernelMetrics {
    std::string kernel_name;
    uint32_t grid_size;
    uint32_t block_size;
    std::chrono::nanoseconds duration;
    std::unordered_map<std::string, double> metrics;  // 如: sm_efficiency, memory_throughput
};

} // namespace perf::external
```

#### 4. 统一数据模型

```cpp
// external/unified/unified_profile.hpp
#pragma once
#include <string>
#include <vector>
#include <map>
#include <chrono>

namespace perf::external::unified {

// 统一的采样数据
struct Sample {
    std::chrono::nanoseconds timestamp;
    uint64_t thread_id;
    uint64_t process_id;
    std::vector<std::string> stack;      // 调用栈（函数名）
    std::map<std::string, double> counters; // 计数器值
};

// 统一的函数统计
struct FunctionStat {
    std::string name;
    std::string module;
    uint64_t address;
    std::chrono::nanoseconds self_time;
    std::chrono::nanoseconds total_time;
    uint64_t sample_count{0};
    double percentage{0.0};
};

// 统一Profile数据结构
struct UnifiedProfile {
    std::string tool_name;               // 来源工具
    std::string tool_version;
    std::chrono::system_clock::time_point start_time;
    std::chrono::nanoseconds duration;
    
    std::vector<Sample> samples;
    std::vector<FunctionStat> functions;
    
    // 原始工具特定数据（保留用于高级分析）
    std::map<std::string, std::string> raw_metadata;
    std::vector<std::string> raw_files;
};

// 统一Trace数据结构
struct UnifiedTrace {
    struct Event {
        enum Type { FUNCTION_ENTRY, FUNCTION_EXIT, SCHEDULE, IO, CUSTOM };
        Type type;
        std::chrono::nanoseconds timestamp;
        uint64_t thread_id;
        std::string name;
        std::map<std::string, std::string> attributes;
    };
    
    std::vector<Event> events;
    std::string tool_name;
};

// 统一Metrics数据结构
struct UnifiedMetrics {
    struct Metric {
        std::string name;
        std::string unit;
        double value;
        std::optional<double> min;
        std::optional<double> max;
        std::optional<double> avg;
    };
    
    std::vector<Metric> metrics;
    std::chrono::system_clock::time_point timestamp;
    std::string tool_name;
};

} // namespace perf::external::unified
```

#### 5. 工具编排器

```cpp
// external/orchestrator/tool_orchestrator.hpp
#pragma once
#include "external/base/external_tool.hpp"
#include "external/unified/unified_report.hpp"

namespace perf::external {

// 分析场景定义
enum class AnalysisScenario {
    CPU_BOUND,           // CPU密集型
    MEMORY_BOUND,        // 内存密集型
    IO_BOUND,            // IO密集型
    GPU_CUDA,            // CUDA GPU分析
    GPU_ROCM,            // ROCm GPU分析
    SYSTEM_WIDE,         // 全系统分析
    NETWORK_LATENCY,     // 网络延迟
    CUSTOM
};

// 工具链配置
struct ToolChainConfig {
    AnalysisScenario scenario;
    bool parallel_execution{false};      // 并行执行多个工具
    std::chrono::seconds timeout{60};
    std::vector<std::string> tool_order; // 指定工具执行顺序
};

// 工具编排器
class ToolOrchestrator {
public:
    ToolOrchestrator();
    ~ToolOrchestrator();
    
    // 注册工具
    void registerTool(std::unique_ptr<ExternalTool> tool);
    
    // 基于场景自动选择工具
    std::vector<std::string> recommendTools(AnalysisScenario scenario);
    
    // 执行工具链
    common::Result<void> executeToolChain(
        const ToolChainConfig& config,
        const ToolExecutionConfig& exec_config
    );
    
    // 获取统一报告
    common::Result<unified::UnifiedReport> generateUnifiedReport();
    
    // 获取各工具结果
    std::map<std::string, unified::UnifiedProfile> getAllProfiles() const;
    std::map<std::string, unified::UnifiedMetrics> getAllMetrics() const;
    
private:
    class Impl;
    std::unique_ptr<Impl> pimpl_;
};

} // namespace perf::external
```

#### 6. AI桥接模块

```cpp
// external/ai_bridge/external_data_bridge.hpp
#pragma once
#include "external/unified/unified_report.hpp"
#include "ai/diagnosis/ai_diagnoser.hpp"

namespace perf::external {

// 外部数据AI桥接
class ExternalDataBridge {
public:
    ExternalDataBridge();
    ~ExternalDataBridge();
    
    // 将统一报告转换为AI可理解的提示
    std::string toAnalysisPrompt(const unified::UnifiedReport& report);
    
    // 增强AI诊断上下文
    void enrichDiagnosisContext(
        ai::DiagnosisContext& context,
        const unified::UnifiedReport& report
    );
    
    // 生成工具使用建议
    std::string generateToolRecommendationPrompt(AnalysisScenario scenario);
    
    // 关联多工具结果
    std::string correlateToolResults(
        const std::map<std::string, unified::UnifiedReport>& reports
    );
};

// 工具推荐器
class ToolRecommender {
public:
    struct Recommendation {
        std::string tool_name;
        std::string reason;
        int priority;  // 1-10
        std::vector<std::string> prerequisites;
    };
    
    // 基于场景推荐
    std::vector<Recommendation> recommend(AnalysisScenario scenario);
    
    // 基于目标推荐（如："查找内存泄漏"、"优化CUDA内核"）
    std::vector<Recommendation> recommendByGoal(const std::string& goal);
    
    // 推荐工具组合
    std::vector<std::vector<std::string>> recommendCombinations(
        AnalysisScenario scenario
    );
};

} // namespace perf::external
```

### 使用示例

```cpp
// 示例1：使用perf进行CPU分析
#include "external/adapters/perf/perf_adapter.hpp"

void analyze_cpu() {
    perf::external::PerfConfig config;
    config.events = {perf::external::PerfConfig::EventType::CPU_CYCLES,
                     perf::external::PerfConfig::EventType::CACHE_MISSES};
    config.call_graph = true;
    
    auto perf = std::make_unique<perf::external::PerfAdapter>(config);
    
    if (!perf->isAvailable()) {
        std::cerr << "perf not available" << std::endl;
        return;
    }
    
    perf::external::ToolExecutionConfig exec;
    exec.target_binary = "./my_app";
    exec.target_args = {"--input", "data.txt"};
    exec.duration = std::chrono::seconds(30);
    
    auto result = perf->execute(exec);
    if (result.ok()) {
        auto profile = perf->toUnifiedProfile();
        // 交给AI分析...
    }
}

// 示例2：工具编排器自动分析
#include "external/orchestrator/tool_orchestrator.hpp"

void auto_analyze() {
    perf::external::ToolOrchestrator orchestrator;
    
    // 注册可用工具
    orchestrator.registerTool(std::make_unique<perf::external::PerfAdapter>());
    orchestrator.registerTool(std::make_unique<perf::external::NcuAdapter>());
    
    // 自动推荐工具
    auto tools = orchestrator.recommendTools(
        perf::external::AnalysisScenario::GPU_CUDA
    );
    // tools: ["ncu", "perf"]
    
    // 执行工具链
    perf::external::ToolChainConfig chain;
    chain.scenario = perf::external::AnalysisScenario::GPU_CUDA;
    chain.parallel_execution = false;
    
    perf::external::ToolExecutionConfig exec;
    exec.target_binary = "./cuda_app";
    exec.duration = std::chrono::seconds(60);
    
    orchestrator.executeToolChain(chain, exec);
    
    // 生成统一报告
    auto report = orchestrator.generateUnifiedReport();
}
```

### 与核心模块集成点

```cpp
// application/use_cases/external_analysis.hpp
#pragma once
#include "external/orchestrator/tool_orchestrator.hpp"
#include "ai/diagnosis/ai_diagnoser.hpp"

namespace perf::application {

// 外部工具分析用例
class ExternalAnalysisUseCase {
public:
    explicit ExternalAnalysisUseCase(
        std::unique_ptr<external::ToolOrchestrator> orchestrator,
        std::unique_ptr<ai::AIDiagnoser> diagnoser
    );
    
    // 主入口：执行外部工具分析
    common::Result<AnalysisReport> analyze(
        external::AnalysisScenario scenario,
        const external::ToolExecutionConfig& config
    );
    
    // 智能推荐分析方案
    common::Result<AnalysisReport> smartAnalyze(
        const std::string& target_binary,
        const std::vector<std::string>& hints = {}
    );
    
private:
    std::unique_ptr<external::ToolOrchestrator> orchestrator_;
    std::unique_ptr<ai::AIDiagnoser> diagnoser_;
};

} // namespace perf::application
```

### CMake配置（可选编译）

```cmake
# external/CMakeLists.txt
option(ENABLE_EXTERNAL_TOOLS "Enable external tool integrations" ON)
option(ENABLE_PERF_ADAPTER "Enable Linux perf adapter" ON)
option(ENABLE_NVIDIA_ADAPTERS "Enable NVIDIA tool adapters" ON)
option(ENABLE_INTEL_ADAPTERS "Enable Intel tool adapters" OFF)

if(ENABLE_EXTERNAL_TOOLS)
    add_library(perf_agent_external STATIC)
    
    # 基础模块
    target_sources(perf_agent_external PRIVATE
        base/external_tool.cpp
        base/tool_adapter.cpp
        unified/unified_profile.cpp
        unified/unified_metrics.cpp
        orchestrator/tool_orchestrator.cpp
        ai_bridge/external_data_bridge.cpp
    )
    
    # 条件编译适配器
    if(ENABLE_PERF_ADAPTER)
        find_program(PERF_EXE perf)
        if(PERF_EXE)
            target_sources(perf_agent_external PRIVATE
                adapters/perf/perf_adapter.cpp
                adapters/perf/perf_data_parser.cpp
            )
            target_compile_definitions(perf_agent_external PRIVATE HAS_PERF=1)
        endif()
    endif()
    
    if(ENABLE_NVIDIA_ADAPTERS)
        find_program(NCU_EXE ncu)
        find_program(NSYS_EXE nsys)
        if(NCU_EXE OR NSYS_EXE)
            target_sources(perf_agent_external PRIVATE
                adapters/nvidia/ncu_adapter.cpp
                adapters/nvidia/nsys_adapter.cpp
            )
        endif()
        if(NCU_EXE)
            target_compile_definitions(perf_agent_external PRIVATE HAS_NCU=1)
        endif()
        if(NSYS_EXE)
            target_compile_definitions(perf_agent_external PRIVATE HAS_NSYS=1)
        endif()
    endif()
    
    target_link_libraries(perf_agent_external PUBLIC perf_agent_core)
endif()
```

---

*架构设计版本：v1.0*  
*外部工具集成模块：v0.1*  
*最后更新：2026-02-16*
