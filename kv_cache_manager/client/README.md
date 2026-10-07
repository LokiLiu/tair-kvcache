## 目录结构
* include: 头文件
* src: 源文件
## RTP remote-cache P1 接口

`MetaClient::StartWrite` 增加末尾参数 `min_replica_count`，默认 0，由服务端按 1 解释；负数会返回参数错误。
`MetaClient` 新增 `GetCacheLocationsByBackend` 和 `GetHostCacheState`，经 `GrpcStub` 调用真实 MetaService。
前者每次选择一个 backend，固定 `QT_BATCH_GET`，返回与输入 key 对齐的嵌套位置数组，保留 miss／mask 的空项及 storage type、spec size、URI；spec-name 筛选若非空，须每个 key 一个名称。
后者返回主机标识、本地可复用长度、P2P 拉取块数和最终命中长度；查询模式由服务端限制为 prefix／Mamba。

新增或变更虚接口必须重新产出匹配的头文件和动态库，不能将新头文件覆盖到旧 RPM。
RTP 使用 `sdk_config.drain_on_timeout=true`；此时外层预算停止排队任务，返回前等待已提交任务全部结束。等待没有额外时限，后端不结束会继续占用调用线程、buffer 引用，并影响 shutdown。
对本次真实 PACE 路径，它等待同步调用的取消排空；实际返回时间可能超过外层预算。其他客户端默认仍为 false，其他后端的底层取消能力仍受各自 SDK 契约限制。

接口和 I/O 生命周期说明见 [客户端 I/O 契约](../../docs/design/client_sdk_io_contract.md)。
