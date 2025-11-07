# hertz-gateway

[中文](#中文) | [English](#english)

---

## 中文

### 简介
hertz-gateway 是一个基于字节跳动 Hertz 框架的轻量级 API 网关，作为统一入口承接客户端请求并将请求分发给后端微服务。网关通过 Go 的反向代理转发请求，并结合服务发现（当前支持 Consul）获取后端微服务节点，实现上游的负载均衡。同时集成了 Prometheus 与 OpenTelemetry，用于指标监控与分布式追踪。

特色：
- 基于 Hertz 的高性能转发
- 支持 Consul 服务发现（可扩展其他发现组件）
- 集成 Prometheus（/metrics）与 OpenTelemetry（OTLP）
- JSON 字段 key 映射功能：可按 App 对 request/response 的 JSON key 做替换，便于多应用共用后端但字段名不同的场景（映射逻辑见 config/config.yaml 与 common/config.go）

参考配置：config/config.yaml  
映射实现：common/config.go -> LoadConfig 中构造 strings.Replacer

---

### 部署
参考仓库中的 Makefile（Makefile 提供了构建、运行、停止、查看日志、推镜像等常用命令）。

常用命令示例：
- 本地编译并运行（开发）：
    - go build -o hertz-gateway .
    - ./hertz-gateway -config ./config/config.yaml
- 使用 Makefile（构建镜像并运行）：
    - make build
    - make run
- 直接用 Docker（示例）：
    - docker build -t your-registry/hertz-gateway:latest .
    - docker run -d --name hertz-gateway -p 9000:9000 -p 9103:9103 -v $(pwd)/config:/app/config:ro your-registry/hertz-gateway:latest

注意：
- 若要 Prometheus 抓取，请确保将 metrics 端口（config 中 common.metrics_addr）映射到宿主机。
- 若使用 Consul/OTEL，确保配置中的地址（common.consul_addr、common.otel_endpoint）对运行环境可达。

---

### 配置与 JSON key 映射
示例（config/config.yaml）：
```yaml
common:
  otel_endpoint: "127.0.0.1:4317"
  consul_addr: http://127.0.0.1:8500
  metrics_addr: 0.0.0.0:9103
  metrics_path: /metrics

app_mapping:
  - app_id: ios_test_v1
    req_map_config: |
      {
        "base": "dog",
        "phone_area_code": "phone_area_code_1"
      }
    resp_map_config: |
      {
        "code": "coode",
        "msg": "message"
      }
```

说明：
- req_map_config：将上游（客户端）请求的 JSON key 替换为后端期望的 key（在入站前替换）。
- resp_map_config：将后端响应的 JSON key 替换为上游期望的 key（在出站前替换）。
- 目前实现基于 strings.Replacer 做简单字符串替换，匹配时需确保 JSON key 以双引号并包含冒号（例如 "key":）。详情请查看 common/config.go 的 LoadConfig 实现。

### 打赏
- 打开微信 → 点击“扫一扫” → 扫描上方二维码 → 选择任意金额并确认支付
- 推荐备注：“支持 hertz-gateway 项目” 或写下公司/项目名（可选）
- 小额也很欢迎（1~5 元也非常感谢）

  ![WeChat Donate](./assets/wechat-donate.png)
  
- 感谢支持！🙏
---

## English

### Overview
hertz-gateway is a lightweight API gateway built on the Bytedance Hertz framework. It accepts client requests and dispatches them to backend microservices using a Go reverse proxy. The gateway supports service discovery (Consul currently) to discover backend nodes and provides simple upstream load balancing. Prometheus and OpenTelemetry are integrated for metrics and tracing.

Key features:
- Hertz-based high-performance gateway
- Service discovery via Consul (extendable)
- Reverse proxy implemented in Go
- Prometheus metrics and OpenTelemetry tracing support
- JSON key mapping: rewrite request/response JSON keys per App so multiple apps can share the same backend while using different field names (mapping logic in config/config.yaml and common/config.go)

Config reference: config/config.yaml  
Mapping implementation: common/config.go -> LoadConfig (strings.Replacer)

---

### Build & Run
Prerequisites: Go (for local build), Docker (for container), Consul/OTEL/Prometheus as needed.

Common commands:
- Local build:
    - go build -o hertz-gateway .
    - ./hertz-gateway -config ./config/config.yaml
- Makefile:
    - make build   # build docker image
    - make run     # run docker container (Makefile variables control image name, tag, ports)
- Docker:
    - docker build -t your-registry/hertz-gateway:latest .
    - docker run -d --name hertz-gateway -p 9000:9000 -p 9103:9103 -v $(pwd)/config:/app/config:ro your-registry/hertz-gateway:latest

Notes:
- Ensure metrics port (common.metrics_addr) is reachable by Prometheus.
- Ensure common.consul_addr and common.otel_endpoint point to reachable services.

---

### Observability
- Metrics: exposed at common.metrics_addr + common.metrics_path (Prometheus).
- Tracing: configure common.otel_endpoint to your OTLP collector.

---

### Donation / WeChat
If you find this project useful, you can support its maintenance with a small donation. Please add the QR code image to the repo at assets/wechat-donate.png and it will be displayed here.

Suggested donation guidance (use in README near the image):

![WeChat Donate](./assets/wechat-donate.png)

---

License: MIT (或仓库中 LICENSE 所示)