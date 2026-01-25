---
title: 代码执行沙盒（
author: 一三
date: 2025-12-03 08:00:00 +0800
categories: [Learn, Work]
tags: [analysis, agent, AI]
---

## 为什么要重新写一个沙盒？
原有能力不足，缺少定制化 venv，缺少参数验证，缺少执行日志的持久化。<span style='color:red'>原有 Dify 代码节点虽然写了参数校验，但实际上不写也能跑，风险很高。</span>



## 额外实现了哪些功能？
1. `python` 虚拟环境和依赖实现动态管理，可以通过配置文件、接口、kafka三种方式进行虚拟环境管理
2. 更好的参数校验功能，支持输入和输出参数校验，如果有输出参数校验，可以输出格式化的json数据

## 实施细节
- 代码路径：`ubuntu/home/dify-sandbox`，基于 dify-sandbox 二次开发。
- 内部结构：核心服务（沙盒 API）、执行引擎（拉起隔离容器 / venv）、日志模块（持久化运行记录）。
- 运行模式：支持配置文件模式、REST API 模式、Kafka 事件驱动模式三种入口，默认启用安全参数校验。

## 快速上手（模板）
```bash
# 1) 切换到项目
cd ubuntu/home/dify-sandbox

# 2) 初始化虚拟环境 + 安装依赖
sh build/build_arm64.sh # 根据自己的环境选择

# 3) 启动服务
./main                  # 默认本地端口 8194，可在 config.yaml 中修改
```

### 配置示例（模板）
```yaml
sandbox:
  venvs:
    default:
      requirements: requirements.txt
      python: python3.11
  io:
    input_schema: schema/input.json
    output_schema: schema/output.json
  logging:
    sink: postgresql://user:pwd@host:5432/sandbox
    retain_days: 30
  runtime:
    mode: api      # api | file | kafka
    max_timeout: 60s
```

### API 调用示例（模板）
```bash
curl -X POST http://localhost:8080/run \
  -H "Content-Type: application/json" \
  -d '{
    "venv": "default",
    "entry": "scripts/hello.py",
    "inputs": {"name": "sandbox"},
    "validate_output": true
  }'
```

## 能力清单（模板）
- `隔离执行`：Python venv 隔离、资源配额（CPU/内存）、超时控制。
- `依赖治理`：多 venv 并行，动态创建/销毁，按需求安装依赖。
- `参数安全`：输入/输出双向校验，默认拒绝不合规请求。
- `审计溯源`：执行日志落库，含调用参数、运行耗时、出错堆栈。
- `事件驱动`：支持 Kafka 触发，方便接入异步链路。
- `可观测性`：指标暴露（Prometheus）、按任务级别记录。

## 未来功能？
- 支持流式输出（日志/标准输出实时推送）。
- 增强隔离（容器级 Cgroup、网络白名单）。
- 一键模板化生成“能力/项目”展示卡片。
