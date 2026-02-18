# AI Linux性能分析智能体 - 从零实现计划

> 目标：从零开始实现AI驱动的Linux性能分析与优化智能体，深入理解Linux系统和内核性能优化底层原理。
> 同时兼顾个人技术深度、项目展示价值与实际生产可用性。

## 📌 项目概述

本项目旨在通过**亲手实现**而非借用现有工具（如perf、valgrind等），从底层原理出发，构建一个完整的AI性能分析智能体。虽然会"重复造轮子"，但这将带来对Linux系统最深入的理解和个人成长。

### 项目三重目标

| 目标维度 | 具体内容 |
|---------|---------|
| **深度学习** | 从procfs到eBPF到栈回溯，每个模块都亲手实现 |
| **技术展示** | 完整的GitHub项目，展示系统编程+内核+AI的综合能力 |
| **生产可用** | AI自主分析、异常告警、多工具集成，可在真实环境运行 |

### 核心学习原则

1. **从零实现** - 不依赖现有性能工具，自己编写采集、分析引擎
2. **源码级理解** - 深入Linux内核源码，理解底层机制
3. **渐进式构建** - 从简单到复杂，每个阶段都有可运行的产出
4. **AI驱动** - AI智能体具备自主决策采集策略、根因定位、异常告警能力

---

## 📚 核心参考资料

### 必读书籍

| 书名 | 作者 | 用途 |
|------|------|------|
| 《Systems Performance: Enterprise and the Cloud, 2nd Edition》 | Brendan Gregg | **核心教材**，性能分析方法学 |
| 《BPF Performance Tools: Linux System and Application Observability》 | Brendan Gregg | eBPF深度理解 |
| 《Understanding the Linux Kernel》 | Daniel P. Bovet | 内核原理 |
| 《Linux Kernel Development》 | Robert Love | 内核开发实践 |

### 在线资源

- **[Brendan Gregg's Blog](https://www.brendangregg.com/)** - Linux性能分析权威资料
- **[Linux Kernel Documentation - ftrace](https://docs.kernel.org/trace/ftrace.html)** - ftrace官方文档
- **[eBPF & BPF Documentation](https://www.brendangregg.com/ebpf.html)** - eBPF系列文章
- **[perf wiki](https://perfwiki.github.io/)** - perf工具文档

### 源码参考

```
Linux内核源码关键路径：
├── tools/perf/           # perf工具实现
├── kernel/trace/         # ftrace实现
├── kernel/bpf/           # eBPF实现
├── kernel/events/        # perf_event实现
├── kernel/ptrace.c       # ptrace实现
└── fs/proc/              # procfs实现
```

---

## 🗺️ 学习路线图

> 时间估计基于有Linux系统开发和内核模块经验的中高级工程师。
> 纯学习路线（每周20+小时），如时间有限可按比例延长。

### 第一阶段：Linux性能分析基础 (3-4周)

> 有系统开发经验，此阶段可快速推进，重点在于建立性能分析方法学框架。

| 周次 | 主题 | 关键知识点 | 实践任务 |
|------|------|------------|----------|
| 1 | **性能分析方法学** | USE方法、RED方法、Off-CPU分析、火焰图概念 | 阅读《性能之巅》第1-4章，整理方法论笔记 |
| 1-2 | **Linux procfs & sysfs** | `/proc/[pid]/stat`, `/proc/meminfo`, `/proc/cpuinfo` 解析 | 自己解析/proc文件，实现简单的top工具 |
| 2-3 | **System Call追踪** | ptrace系统调用原理，PTRACE_SYSCALL/PTRACE_SINGLESTEP | 实现一个极简的strace |
| 3-4 | **CPU性能监控** | PMU(Performance Monitoring Unit), rdpmc指令, `perf_event_open` | 实现CPU周期计数器读取工具 |

**阶段目标:** 能独立实现一个简易的`mytop`工具，显示进程CPU/内存使用情况

---

### 第二阶段：内核追踪机制原理 (5-7周)

| 周次 | 主题 | 关键知识点 | 实践任务 |
|------|------|------------|----------|
| 4-5 | **Tracepoints原理** | 内核静态插桩机制，tracepoint定义与注册 | 编写内核模块添加自定义tracepoint |
| 5-6 | **Kprobes/Kretprobes** | 动态内核插桩，breakpoint机制，kretprobe返回值捕获 | 实现内核函数调用追踪工具 |
| 6-8 | **Uprobes原理** | 用户态动态插桩，uprobe_register，内核如何处理用户态断点 | 实现用户态函数调用追踪 |
| 8-9 | **Ring Buffer实现** | 无锁环形缓冲区，per-CPU缓冲区设计，内存屏障 | 自己实现一个高性能Ring Buffer |
| 9-11 | **Ftrace原理** | function tracer实现，mcount插桩，function_graph_tracer | 实现一个简化的function tracer |

**阶段目标:** 实现一个完整的内核/用户态函数调用追踪工具，能显示函数调用耗时

---

### 第三阶段：eBPF编程与应用 (5-7周)

> 聚焦于**使用eBPF进行性能分析**，采用成熟库进行开发，重点理解CO-RE和内核版本兼容。

#### 推荐的eBPF开发库

| 库/工具 | 语言 | 适用场景 | 学习曲线 |
|---------|------|---------|---------|
| **libbpf** | C | 生产环境，内核源码配套 | 较陡 |
| **bpftrace** | DSL | 快速原型，one-liner脚本 | 平缓 |
| **BCC** | Python/C++ | 快速验证想法 | 中等 |

#### eBPF学习路线

| 周次 | 主题 | 关键知识点 | 实践任务 |
|------|------|------------|----------|
| 11-12 | **eBPF基础与验证器** | 程序类型(kprobe/tracepoint/XDP)、Maps、验证器规则 | 阅读《BPF Performance Tools》 |
| 12-13 | **CO-RE与BTF** | Compile Once Run Everywhere原理，BTF类型信息，bpf_core_read | 编写CO-RE兼容的eBPF程序 |
| 13-14 | **libbpf开发实践** | Skeleton生成、BPF程序加载、perf_buffer/ring_buffer通信 | 实现CPU采集eBPF工具 |
| 14-16 | **高级eBPF应用** | 火焰图eBPF采集、Off-CPU分析、BPF map聚合 | 实现基于eBPF的完整性能采集管道 |
| 16-18 | **内核版本兼容** | 运行时内核版本检测、特性探测、优雅降级策略 | 让eBPF程序在5.4/5.15/6.x上均可运行 |

**阶段目标:** 熟练使用eBPF进行性能数据采集，能编写CO-RE兼容的自定义工具

#### eBPF内核版本兼容策略

```
内核版本检测流程：
┌─────────────────────────────────────────────────────────┐
│ 运行时检测                                               │
│  1. 读取 /proc/version 或 uname() 获取内核版本           │
│  2. 检测 /sys/kernel/btf/vmlinux 是否存在（BTF支持）     │
│  3. 通过 BPF_PROG_LOAD 测试特性可用性                   │
│  4. 根据结果选择 CO-RE / BCC / 降级方案                 │
└─────────────────────────────────────────────────────────┘

版本特性矩阵：
┌──────────────┬───────┬────────────┬──────────┬──────────┐
│ 特性          │ 5.4   │ 5.10 (LTS) │ 5.15 LTS │ 6.x      │
├──────────────┼───────┼────────────┼──────────┼──────────┤
│ BPF ring buf │  ❌   │    ✅       │   ✅     │   ✅     │
│ CO-RE        │  ⚠️   │    ✅       │   ✅     │   ✅     │
│ BTF          │  ✅   │    ✅       │   ✅     │   ✅     │
│ fentry/fexit │  ❌   │    ✅       │   ✅     │   ✅     │
│ bpf_iter     │  ❌   │    ✅       │   ✅     │   ✅     │
└──────────────┴───────┴────────────┴──────────┴──────────┘
降级策略：ring_buffer → perf_buffer → 用户态轮询
```

#### eBPF学习资源

- **《BPF Performance Tools》** - Brendan Gregg (必读)
- **[libbpf-bootstrap](https://github.com/libbpf/libbpf-bootstrap)** - 官方入门项目
- **[BCC示例](https://github.com/iovisor/bcc)** - 丰富的示例代码
- **[BTF Hub](https://github.com/aquasecurity/btfhub)** - 跨发行版BTF支持

---

### 第四阶段：采样与Profiling (4-6周)

| 周次 | 主题 | 关键知识点 | 实践任务 |
|------|------|------------|----------|
| 18-20 | **CPU采样原理** | 定时中断采样(SIGPROF/setitimer)，PMC溢出采样(perf_event_open + PERF_SAMPLE_STACK_USER) | 实现基于信号的采样器 |
| 20-21 | **栈回溯实现** | frame pointer展开，ORC（Oops Rewind Capability），DWARF解析，LBR（Last Branch Record） | 实现多种栈回溯方法 |
| 21-23 | **Off-CPU分析** | 调度事件追踪（sched_switch），睡眠原因分析，Off-CPU时间计算 | 实现Off-CPU时间分析工具 |
| 23-26 | **火焰图生成** | 折叠栈格式（folded stacks），Brendan Gregg算法，SVG生成，交互式火焰图 | 实现完整火焰图生成器 |

**阶段目标:** 实现一个完整的CPU Profiler，能生成交互式SVG火焰图

---

### 第五阶段：AI智能体集成 (8-10周)

> 这是项目最具差异化的部分。核心能力：自主决策采集策略、自动根因定位、异常检测与主动告警。

#### 5.1 AI智能体整体设计

AI智能体采用 **ReAct (Reason + Act)** 模式：

```
┌────────────────────────────────────────────────────────────────────┐
│                        ReAct Agent Loop                             │
│                                                                     │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌─────────────┐  │
│  │ Observe  │───▶│  Think   │───▶│   Act    │───▶│   Observe   │  │
│  │ (感知)   │    │ (推理)   │    │ (执行)   │    │ (结果感知)  │  │
│  └──────────┘    └──────────┘    └──────────┘    └──────┬──────┘  │
│       ▲                                                   │        │
│       │                  ┌──────────┐                     │        │
│       │                  │  Report  │◀────────────────────┘        │
│       │                  │ (诊断报告)│  (当置信度足够时)            │
│       │                  └──────────┘                              │
│       │                       │                                    │
│  系统状态快照               输出给用户                              │
└────────────────────────────────────────────────────────────────────┘

Loop终止条件：
  - 找到根因（置信度 > 阈值）
  - 达到最大迭代次数
  - 用户中断
  - Token预算耗尽
```

#### 5.2 AI工具调用设计（Tool Calling Schema）

AI智能体通过 **Function Calling** 与性能采集系统交互。以下是完整的工具清单：

##### 系统感知工具

```json
// 工具1：列出进程
{
  "name": "list_processes",
  "description": "列出当前运行的进程及其CPU/内存使用情况，按指定字段排序",
  "parameters": {
    "sort_by": {"type": "string", "enum": ["cpu", "memory", "pid", "name"], "default": "cpu"},
    "top_n": {"type": "integer", "default": 20, "description": "返回前N个进程"},
    "filter_pid": {"type": "integer", "description": "过滤特定PID（可选）"}
  }
}

// 工具2：系统全局指标
{
  "name": "get_system_metrics",
  "description": "获取系统整体性能快照：CPU负载、内存使用、I/O吞吐、网络统计",
  "parameters": {
    "duration_secs": {"type": "integer", "default": 5},
    "include_numa": {"type": "boolean", "default": false},
    "include_irq": {"type": "boolean", "default": false}
  }
}

// 工具3：系统拓扑
{
  "name": "get_system_topology",
  "description": "获取硬件拓扑：CPU核数/NUMA节点/L1-L3缓存大小/内存控制器",
  "parameters": {}
}
```

##### 性能采集工具

```json
// 工具4：CPU Profiling
{
  "name": "cpu_profile",
  "description": "对指定进程或系统进行CPU采样，返回火焰图数据",
  "parameters": {
    "pid": {"type": "integer", "description": "目标PID，0表示系统全局"},
    "duration_secs": {"type": "integer", "default": 10},
    "frequency_hz": {"type": "integer", "default": 99},
    "include_kernel": {"type": "boolean", "default": true},
    "output_format": {"type": "string", "enum": ["flamegraph_svg", "folded_stacks", "top_functions"], "default": "top_functions"}
  }
}

// 工具5：Off-CPU分析
{
  "name": "off_cpu_analysis",
  "description": "分析进程Off-CPU时间（阻塞在I/O、锁、睡眠的时间），识别调度延迟",
  "parameters": {
    "pid": {"type": "integer"},
    "duration_secs": {"type": "integer", "default": 10},
    "min_block_us": {"type": "integer", "default": 1000, "description": "最小阻塞时间(微秒)，过滤噪声"}
  }
}

// 工具6：硬件计数器
{
  "name": "get_hw_counters",
  "description": "读取PMU硬件计数器：IPC, cache miss率, branch miss率, 内存带宽",
  "parameters": {
    "pid": {"type": "integer", "description": "0表示系统级"},
    "duration_secs": {"type": "integer", "default": 5},
    "events": {
      "type": "array",
      "items": {"type": "string", "enum": ["cycles", "instructions", "cache-misses", "cache-references", "branch-misses", "branches", "mem-bandwidth-local", "mem-bandwidth-remote"]},
      "default": ["cycles", "instructions", "cache-misses"]
    }
  }
}

// 工具7：内核函数追踪
{
  "name": "trace_kernel_function",
  "description": "追踪特定内核函数的调用频率、延迟分布（直方图）",
  "parameters": {
    "function": {"type": "string", "description": "内核函数名，支持通配符"},
    "duration_secs": {"type": "integer", "default": 5},
    "output_latency_hist": {"type": "boolean", "default": true},
    "pid_filter": {"type": "integer", "description": "仅追踪特定进程（可选）"}
  }
}

// 工具8：运行eBPF分析程序
{
  "name": "run_ebpf_analyzer",
  "description": "从内置库中加载并运行特定的eBPF分析程序",
  "parameters": {
    "program": {
      "type": "string",
      "enum": [
        "tcp_latency",          // TCP连接延迟
        "disk_io_latency",      // 磁盘I/O延迟
        "memory_alloc_trace",   // 内存分配追踪
        "lock_contention",      // 锁竞争分析
        "syscall_count",        // 系统调用统计
        "page_fault_trace",     // 缺页异常追踪
        "sched_latency",        // 调度延迟
        "futex_contention"      // futex竞争
      ]
    },
    "pid": {"type": "integer", "description": "目标PID，0表示全局"},
    "duration_secs": {"type": "integer", "default": 10}
  }
}
```

##### 外部工具调用

```json
// 工具9：运行外部性能工具
{
  "name": "run_external_tool",
  "description": "调用系统已安装的外部性能分析工具（perf/ncu/nsys/vtune）",
  "parameters": {
    "tool": {"type": "string", "enum": ["perf", "ncu", "nsys", "vtune", "bpftrace", "valgrind"]},
    "target": {"type": "string", "description": "目标程序路径或PID"},
    "args": {"type": "array", "items": {"type": "string"}, "description": "工具额外参数"},
    "duration_secs": {"type": "integer", "default": 30}
  }
}
```

##### 分析与告警工具

```json
// 工具10：查询历史指标时序
{
  "name": "query_metrics_history",
  "description": "查询过去N分钟的性能指标时序数据，用于趋势分析",
  "parameters": {
    "metric": {"type": "string", "enum": ["cpu_usage", "memory_usage", "io_read_mbps", "io_write_mbps", "network_rx", "network_tx", "page_faults", "context_switches"]},
    "pid": {"type": "integer", "description": "0表示系统级"},
    "window_minutes": {"type": "integer", "default": 10}
  }
}

// 工具11：获取当前异常报告
{
  "name": "get_anomaly_report",
  "description": "获取统计模型检测到的当前异常指标列表及严重程度",
  "parameters": {
    "min_severity": {"type": "string", "enum": ["low", "medium", "high", "critical"], "default": "medium"}
  }
}

// 工具12：设置告警阈值
{
  "name": "set_alert_threshold",
  "description": "为特定指标设置告警阈值，触发时自动启动AI分析",
  "parameters": {
    "metric": {"type": "string"},
    "threshold": {"type": "number"},
    "condition": {"type": "string", "enum": ["greater_than", "less_than", "deviation_pct"]},
    "auto_analyze": {"type": "boolean", "default": true, "description": "触发时自动启动AI分析"}
  }
}

// 工具13：搜索性能知识库
{
  "name": "search_knowledge",
  "description": "在性能优化知识库中搜索特定症状的解决方案和已知模式",
  "parameters": {
    "symptom": {"type": "string", "description": "症状描述，如 'high cache miss rate' 或 '内存带宽瓶颈'"},
    "top_k": {"type": "integer", "default": 3}
  }
}

// 工具14：分析容器性能
{
  "name": "analyze_container",
  "description": "分析特定容器的性能，基于cgroup v2读取受限资源使用情况",
  "parameters": {
    "container_id": {"type": "string"},
    "runtime": {"type": "string", "enum": ["docker", "containerd", "podman"], "default": "docker"},
    "include_throttling": {"type": "boolean", "default": true}
  }
}

// 工具15：生成诊断报告
{
  "name": "generate_diagnosis_report",
  "description": "基于当前分析结果生成结构化诊断报告（用于完成本次分析会话）",
  "parameters": {
    "severity": {"type": "string", "enum": ["info", "warning", "critical"]},
    "bottleneck_type": {"type": "string", "enum": ["cpu_bound", "memory_bound", "io_bound", "network_bound", "gpu_bound", "lock_contention", "scheduler", "unknown"]},
    "root_cause": {"type": "string", "description": "根因描述"},
    "evidence": {"type": "array", "items": {"type": "string"}, "description": "支持结论的证据列表"},
    "recommendations": {"type": "array", "items": {"type": "string"}, "description": "可执行的优化建议"}
  }
}
```

#### 5.3 Prompt工程设计

##### System Prompt模板

```
你是一个专业的Linux性能分析智能体，具备以下能力：
- 通过工具调用主动采集系统性能数据
- 使用USE方法论（Utilization/Saturation/Errors）系统分析瓶颈
- 定位性能问题根因并给出可执行的优化建议
- 识别异常行为模式，主动告警

分析原则：
1. 先建立全局视图（系统整体健康状态），再聚焦局部
2. 数据驱动：每个结论必须有具体数据支持，不做主观猜测
3. 递进追踪：发现异常指标后，进一步下钻（如发现cache miss高，则追踪哪个函数导致）
4. 明确区分：CPU-bound / Memory-bound / IO-bound / Lock-contention
5. 优化建议必须具体可执行（不是"优化内存使用"，而是"将热点函数X中第Y行的随机内存访问改为顺序访问"）

可用工具：[工具清单由系统自动注入]

当前分析目标：{user_query}
```

##### 数据格式化策略（避免超出Context Window）

```
输入给LLM的数据压缩层次：

Level 1 - 摘要（默认）：
  - 前10个热点函数（函数名 + CPU%）
  - 关键指标一行概览（IPC, cache-miss%, off-cpu-ms）

Level 2 - 中等详情（发现异常时触发）：
  - 前30个热点函数 + 调用链
  - 指标时序数据（最近5分钟，1秒粒度）

Level 3 - 完整数据（深度诊断时触发）：
  - 完整火焰图路径（折叠栈格式，限制前1000行）
  - 完整直方图分布
```

#### 5.4 上下文管理策略

```
Context Window管理：
┌──────────────────────────────────────────────────────────┐
│  可用 Token 预算（如 128K token）                         │
│  ┌────────────┬────────────┬────────────┬──────────────┐ │
│  │ System     │ Tool       │ Current    │  Response    │ │
│  │ Prompt     │ Results    │ History    │  Budget      │ │
│  │ (~2K)      │ Cache(压缩)│ (滑窗N轮)  │  (~4K)       │ │
│  └────────────┴────────────┴────────────┴──────────────┘ │
└──────────────────────────────────────────────────────────┘

策略：
1. 工具结果缓存：相同工具+相同参数的结果缓存60秒，避免重复调用
2. 历史滑窗：保留最近10轮对话，更早的轮次进行摘要压缩
3. 大数据截断：超过2K token的工具结果自动压缩（保留统计摘要）
4. 关键证据固定：确认为根因的证据固定在上下文中不滚出
```

#### 5.5 异常检测与主动告警流程

```
告警驱动的AI分析流水线：

┌──────────┐    ┌──────────────┐    ┌──────────────┐
│  指标采集 │───▶│  统计异常检测 │───▶│  告警去重/聚合│
│  (1秒周期)│    │(Z-score/IQR) │    │  (5秒窗口)   │
└──────────┘    └──────────────┘    └──────────────┘
                                           │
                                    ┌──────▼──────┐
                                    │ 严重程度评估  │
                                    │ LOW/MED/HIGH │
                                    └──────┬──────┘
                                           │
                              ┌────────────┼────────────┐
                              ▼            ▼            ▼
                          LOW告警      MED告警      HIGH告警
                           (记录)    (通知+等待)   (立即触发)
                                                       │
                                               ┌───────▼──────┐
                                               │  自动启动      │
                                               │  ReAct Agent  │
                                               │  进行根因分析  │
                                               └───────┬──────┘
                                                       │
                                               ┌───────▼──────┐
                                               │  诊断报告      │
                                               │  + 优化建议   │
                                               └───────┬──────┘
                                                       │
                                               ┌───────▼──────┐
                                               │  Webhook/     │
                                               │  Slack/Email  │
                                               └──────────────┘

异常检测算法：
- 短期异常：Z-score (均值±3σ，基于最近5分钟)
- 周期性基线：指数移动平均 (EMA, α=0.1，检测周期性规律)
- 突变检测：CUSUM（累积和控制图，检测均值漂移）
- 多变量关联：同时检测CPU+内存+I/O的联合异常
```

#### 5.6 实践任务（逐周推进）

| 周次 | 主题 | 实践任务 |
|------|------|---------|
| 26-27 | **性能数据建模** | 实现时序数据存储（SQLite/in-memory），Z-score异常检测 |
| 27-28 | **工具调用框架** | 实现Tool Registry，将性能采集函数封装为工具 |
| 28-29 | **LLM客户端** | 实现OpenAI/Claude API客户端，支持Function Calling |
| 29-30 | **ReAct Agent** | 实现Agent Loop，集成工具调用，测试基本诊断流程 |
| 30-31 | **Prompt优化** | 编写System Prompt，测试各类性能问题的诊断准确率 |
| 31-32 | **Context管理** | 实现滑动窗口、数据压缩、工具结果缓存 |
| 32-33 | **告警流水线** | 实现AnomalyDetector → AlertManager → AgentTrigger |
| 33-34 | **知识库** | 建立性能模式知识库（JSON规则 + 向量检索增强） |
| 34-36 | **集成测试** | 端到端测试，真实负载场景下的诊断准确率评估 |

**阶段目标:** 能对给定进程/系统运行AI自主诊断，在5分钟内输出包含根因和优化建议的诊断报告

---

### 第六阶段：生产化与工程化 (3-4周)

> 将原型变成可在真实环境部署的工具。

| 周次 | 主题 | 关键任务 |
|------|------|---------|
| 36-37 | **容器与云原生支持** | cgroup v2指标采集，容器感知的进程归属，Kubernetes Pod关联 |
| 37-38 | **配置与部署** | YAML配置文件系统，配置热重载，单二进制部署，systemd集成 |
| 38-39 | **可观测性** | 为智能体自身增加性能指标（分析延迟、工具调用次数），Prometheus导出 |
| 39-40 | **文档与Demo** | 编写README/使用文档，录制Demo视频，准备GitHub展示 |

**阶段目标:** 有README、CI/CD、Docker支持的生产级项目

---

## 🏗️ 项目架构设计

```
ai-perf-agent/
├── core/                          # 核心性能采集引擎（从零实现）
│   ├── procfs/                    # /proc解析模块
│   ├── tracer/                    # 追踪引擎(kprobe/uprobe)
│   ├── sampler/                   # 采样引擎
│   ├── ringbuf/                   # 环形缓冲区实现
│   └── ebpf/                      # eBPF应用层封装
│       ├── loader/                # eBPF程序加载器(基于libbpf)
│       ├── programs/              # 自定义eBPF程序集合
│       └── maps/                  # Maps读写封装
│
├── analysis/                      # 分析引擎
│   ├── flamegraph/                # 火焰图生成
│   ├── offcpu/                    # Off-CPU分析
│   ├── metrics/                   # 时序指标存储与查询
│   └── anomaly/                   # 异常检测
│
├── ai/                            # AI智能体（核心差异化模块）
│   ├── agent/                     # ReAct Agent Loop
│   ├── tools/                     # Tool Calling定义与实现
│   ├── llm_client/                # LLM客户端(OpenAI/Claude)
│   ├── prompt_builder/            # Prompt工程
│   ├── context/                   # Context Window管理
│   ├── diagnosis/                 # 诊断报告生成
│   └── alert/                     # 异常检测与告警
│
├── external/                      # 外部工具集成（生产环境增强）
│   ├── adapters/                  # 工具适配器(perf/ncu/nsys/vtune)
│   ├── unified/                   # 统一数据模型
│   ├── orchestrator/              # 工具编排器
│   └── ai_bridge/                 # AI模块桥接
│
├── config/                        # 配置管理
│   ├── config_manager/            # YAML配置加载与热重载
│   └── schemas/                   # 配置Schema验证
│
├── cli/                           # 命令行界面
└── docs/                          # 学习笔记和技术文档
```

---

## 💡 学习建议

### 1. 精读源码策略

对于每个工具，按以下顺序阅读：

1. **了解用户接口** - 先熟悉该工具的使用方法和输出格式
2. **找到系统调用入口** - 如 `perf_event_open`, `ptrace`, `bpf`
3. **跟踪内核实现路径** - 从系统调用到核心逻辑
4. **理解数据结构** - 如 `perf_event`, `bpf_prog`, `task_struct`
5. **看工具层封装** - 理解如何组织和使用内核接口

### 2. 动手实践原则

- ✅ **先实现MVP** - 不要追求完美，先让代码跑起来
- ✅ **对比验证** - 你的实现结果要和现有工具（perf/top）对比验证
- ✅ **编写测试** - 为每个模块编写单元测试
- ✅ **记录问题** - 把遇到的问题和解决方案记录下来

### 3. AI模块特别建议

- **先用bpftrace脚本验证分析思路**，再用AI工具调用封装
- **收集10-20个真实性能问题案例**作为测试集，评估AI诊断准确率
- **Prompt要迭代**：把每次诊断错误的案例加入Few-shot示例
- **从GPT-4o开始**，验证功能后再考虑替换为更便宜的模型

---

## 🔗 关键技术资料索引

| 技术 | 内核源码路径 | 推荐学习资源 |
|------|-------------|-------------|
| procfs | `fs/proc/` | `/proc`文件系统详解 |
| ptrace | `kernel/ptrace.c` | "How Does a C Debugger Work?" |
| perf_event | `kernel/events/` | Brendan Gregg的perf系列博客 |
| ftrace | `kernel/trace/` | "Ftrace: The Hidden Light Switch" |
| eBPF | `kernel/bpf/` | "BPF: A New Type of Software" |
| Ring Buffer | `kernel/trace/ring_buffer.c` | LWN Ring Buffer系列文章 |
| PMU | `arch/x86/events/` | Intel SDM Volume 3B |
| CO-RE/BTF | `tools/lib/bpf/` | libbpf官方文档 |
| cgroup v2 | `kernel/cgroup/` | Linux cgroup v2文档 |

---

## 📝 进度追踪

| 阶段 | 计划周数 | 实际开始 | 实际完成 | 状态 |
|------|---------|---------|---------|------|
| 阶段一：基础 | 3-4周 | - | - | ⬜ 未开始 |
| 阶段二：内核追踪 | 5-7周 | - | - | ⬜ 未开始 |
| 阶段三：eBPF应用 | 5-7周 | - | - | ⬜ 未开始 |
| 阶段四：Profiling | 4-6周 | - | - | ⬜ 未开始 |
| 阶段五：AI集成 | 8-10周 | - | - | ⬜ 未开始 |
| 阶段六：生产化 | 3-4周 | - | - | ⬜ 未开始 |

> **预计总时长**: 28-38周（约7-9个月）

---

## 🎯 里程碑检查点

- [ ] **M1**: 完成简易mytop工具（阶段一结束）
- [ ] **M2**: 完成内核/用户态函数追踪工具（阶段二结束）
- [ ] **M3**: 运行第一个CO-RE兼容的eBPF程序（阶段三结束）
- [ ] **M4**: 生成第一张交互式SVG火焰图（阶段四结束）
- [ ] **M5**: AI Agent能自主完成一次完整的性能诊断（阶段五结束）
- [ ] **M6**: 项目可通过Docker一键部署，有完整README（阶段六结束）

---

## 🔌 扩展功能：外部性能工具集成

> 作为核心学习功能的补充，提供与现有专业性能工具的无缝集成，使智能体能够：
> 1. 直接利用现有工具的强大数据采集能力
> 2. 统一不同工具的数据格式
> 3. 让AI智能体能分析和优化任何生产环境

### 支持的外部工具矩阵

| 工具类别 | 工具名称 | 适用场景 | 数据类型 | 优先级 |
|---------|---------|---------|---------|--------|
| **Linux perf** | perf | CPU Profiling、系统级分析 | perf.data、折叠栈 | P0 |
| **NVIDIA** | Nsight Compute | CUDA内核分析 | .ncu-rep、CSV | P0 |
| | Nsight Systems | 系统级GPU/CPU分析 | .nsys-rep、SQLite | P0 |
| | nvprof | 传统CUDA分析 | 文本、CSV | P1 |
| **Intel** | VTune Profiler | CPU/GPU深度分析 | .vtune、CSV | P1 |
| | Intel Advisor | 向量化/并行分析 | 文本、JSON | P1 |
| **eBPF生态** | bpftrace | 动态追踪 | 文本、JSON | P0 |
| | BCC | 高级eBPF分析 | 文本、Python对象 | P1 |
| **内存分析** | Valgrind/Massif | 内存分析 | 文本、XML | P1 |
| | heaptrack | 堆内存追踪 | .gz、JSON | P1 |
| **GPU通用** | rocprof | AMD GPU分析 | 文本、JSON | P2 |

### 外部工具与AI集成点

```
外部工具结果 → UnifiedProfile/Metrics → ExternalDataBridge → AI Context
                                                            ↓
                                            AI可直接问："ncu显示的
                                            sm_efficiency低是什么原因？"
```

---

*文档版本：v2.0*
*最后更新：2026-02-18*

*祝学习愉快！记住：Slow is smooth, smooth is fast.*
