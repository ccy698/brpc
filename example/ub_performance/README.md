# 简介
ub_performance是bRPC性能测试工具，用于测试bRPC在不同传输模式（TCP / RDMA / UBRING）下的性能。

本工具基于bRPC v1.15.0版本example/rdma_performance做修改，经历多次迭代，增加了request和response包大小（`req_size`/`rsp_size`）、QPS限速（`expected_qps`/`initial_tokens`）、连接重试（`max_retry`）、ignore_eovercrowded等配置，并已移植到bRPC v1.17.0，使用`options.socket_mode`选择传输模式。

# 使用方式
先编译bRPC源码（make或bazel均可），再编译本example。

## Bazel编译

```bash
bazel build -c opt //example:ub_performance_server
bazel build -c opt //example:ub_performance_client
```

启用RDMA传输时追加 `--define brpc_with_rdma=true`。

## CMake编译

```bash
cd example/ub_performance
mkdir build && cd build
cmake .. && make
```

## 运行

服务端：

```bash
./ub_performance_server --port=8002 --use_rdma=false --use_ubring=false \
    --rsp_size=1024 --max_concurrency=0 --server_ignore_oc=true
```

客户端：

```bash
./ub_performance_client --servers='0.0.0.0:8002' --use_rdma=false --use_ubring=false \
    --thread_num=1 --queue_depth=1 --expected_qps=500 --test_seconds=20 \
    --attachment_size=0 --req_size=1024 --echo_attachment=false \
    --client_ignore_oc=true --max_retry=3
```

> 说明：
> - 传输模式三选一：默认TCP；`--use_rdma=true` 走RDMA（需bRPC以RDMA编译且环境支持）；`--use_ubring=true` 走UBRING。
> - `attachment_size` 控制attachment大小，`req_size`/`rsp_size` 控制proto消息中name字段填充大小，二者可组合测试不同包大小场景。
> - 不同的测试用例，在执行client、server时需要指定不同的参数。