# EchoCache

一个用 Go 实现的分布式缓存示例项目，包含：

- 基于 LRU 的本地缓存淘汰
- 一致性哈希节点选择
- HTTP 节点间通信
- `singleflight` 并发请求合并
- `protobuf` 序列化的节点响应

项目当前仓库内容对应一个可运行版本，既包含缓存库 `geecache`，也包含一个简单的 demo 服务入口。

## 目录结构

```text
.
├── EchoCache/              # geecache 库实现
│   ├── consistenthash/     # 一致性哈希
│   ├── geecachepb/         # protobuf 定义与生成代码
│   ├── lru/                # LRU 缓存
│   ├── singleflight/       # 并发请求合并
│   ├── *.go
├── main.go                 # 示例服务入口
├── run.sh                  # 一键启动 3 个缓存节点 + 1 个 API 节点
├── go.mod
└── go.sum
```

## 环境要求

- Go 1.13+

## 快速启动

### 方式一：直接运行脚本

```bash
chmod +x run.sh
./run.sh
```

脚本会做这些事：

- 编译生成 `server`
- 启动 3 个缓存节点：`8001`、`8002`、`8003`
- 在 `8003` 进程上同时启动一个对外 API：`http://localhost:9999/api`
- 自动发起几次测试请求

### 方式二：手动启动

先启动三个缓存节点：

```bash
go run . -port=8001
go run . -port=8002
go run . -port=8003 -api=true
```

然后访问：

```bash
curl "http://localhost:9999/api?key=Tom"
```

预期返回：

```text
630
```

如果 key 不存在：

```bash
curl "http://localhost:9999/api?key=kkk"
```

会返回类似：

```text
kkk not exist
```

## 工作机制

1. `main.go` 创建名为 `scores` 的缓存组。
2. 本地模拟数据源 `db` 作为缓存未命中时的回源逻辑。
3. `HTTPPool` 维护所有节点地址，并通过一致性哈希决定某个 key 应由哪个节点负责。
4. 节点间通过 HTTP + protobuf 拉取缓存数据。
5. 同一个 key 的并发加载通过 `singleflight` 合并，避免击穿到底层数据源。

## 测试

运行全部测试：

```bash
go test ./...
```

如果只测缓存库：

```bash
go test ./EchoCache/...
```

## 说明

- 根目录模块通过 `replace geecache => ./EchoCache` 引用本地缓存库。
- 项目更偏向教学和原理演示，未包含生产环境下的鉴权、服务发现、监控和持久化能力。
