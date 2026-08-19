# Bug Reproduction

## 包的性质

当前 test_model_fix 保存的是被测模型修复后的结果源码，不是初始含 Bug 源码。要复现原始缺陷，必须检出下面固定的 parent SHA；不要在当前修复结果源码上期待重新出现修复前失败。生成系统使用的可信验证补丁和完整验证日志仅在本地留存，不提交到结果分支。

## 问题现象

按运输状态筛选分页时，items 只返回符合条件的记录，但 total 仍是所有状态的总数，导致客户端计算出不存在的后续页。请修复分页计数与明细查询的过滤一致性。 请只修改必要的生产代码，不得新增、删除或修改测试文件，不得跳过测试或放宽断言。

## 含 Bug 版本

- 仓库：zhanglei10281852-gif/coldchain-custody-task-24
- 仓库地址：https://github.com/zhanglei10281852-gif/coldchain-custody-task-24.git
- parent SHA：74148b0bcb0322698af5708ea6942812784d47a5

## 复现步骤

```bash
git clone -- https://github.com/zhanglei10281852-gif/coldchain-custody-task-24.git bug-repro
cd bug-repro
git checkout --detach 74148b0bcb0322698af5708ea6942812784d47a5
go test ./internal/storage/sqlite -run "^TestFilteredShipmentTotalMatchesItems$" -count=1
```

## 双架构完整错误信息

### linux/amd64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/storage/sqlite -run "^TestFilteredShipmentTotalMatchesItems$" -count=1
--- FAIL: TestFilteredShipmentTotalMatchesItems (0.08s)
    annotation_behavior_test.go:37: filtered page = {Items:[{ID:ship_filter_1 StudyID:study_1 OriginSiteID:site_1 DestinationSiteID:site_2 ContainerID:box_1 Reference:FILTER-1 State:planned PlannedDispatchAt:2026-08-18 09:00:00 +0000 UTC ExpectedArrivalAt:2026-08-18 10:00:00 +0000 UTC DispatchedAt:<nil> ArrivedAt:<nil> ClosedAt:<nil> TotalVolumeMilliLit:10 CreatedAt:2026-08-18 08:00:00 +0000 UTC UpdatedAt:2026-08-18 08:00:00 +0000 UTC Version:1}] Total:2}
FAIL
FAIL	github.com/zhanglei10281852-gif/coldchain-custody-base/internal/storage/sqlite	0.080s
FAIL

```

stderr：

```text
warning: internal/storage/sqlite/annotation_behavior_test.go has type 100755, expected 100644
warning: internal/storage/sqlite/store_test.go has type 100755, expected 100644
warning: internal/storage/sqlite/annotation_behavior_test.go has type 100755, expected 100644
warning: internal/storage/sqlite/store_test.go has type 100755, expected 100644

```

### linux/arm64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/storage/sqlite -run "^TestFilteredShipmentTotalMatchesItems$" -count=1
--- FAIL: TestFilteredShipmentTotalMatchesItems (0.33s)
    annotation_behavior_test.go:37: filtered page = {Items:[{ID:ship_filter_1 StudyID:study_1 OriginSiteID:site_1 DestinationSiteID:site_2 ContainerID:box_1 Reference:FILTER-1 State:planned PlannedDispatchAt:2026-08-18 09:00:00 +0000 UTC ExpectedArrivalAt:2026-08-18 10:00:00 +0000 UTC DispatchedAt:<nil> ArrivedAt:<nil> ClosedAt:<nil> TotalVolumeMilliLit:10 CreatedAt:2026-08-18 08:00:00 +0000 UTC UpdatedAt:2026-08-18 08:00:00 +0000 UTC Version:1}] Total:2}
FAIL
FAIL	github.com/zhanglei10281852-gif/coldchain-custody-base/internal/storage/sqlite	0.531s
FAIL

```

stderr：

```text
warning: internal/storage/sqlite/annotation_behavior_test.go has type 100755, expected 100644
warning: internal/storage/sqlite/store_test.go has type 100755, expected 100644
warning: internal/storage/sqlite/annotation_behavior_test.go has type 100755, expected 100644
warning: internal/storage/sqlite/store_test.go has type 100755, expected 100644

```

## 通过条件

定向公开行为验证通过，相关包和全量测试通过，go vet 及 linux/amd64 构建通过。 定向命令 go test ./internal/storage/sqlite -run ^TestFilteredShipmentTotalMatchesItems$ -count=1 必须由修复前失败变为修复后通过；相关包与 go test ./... -count=1 全量回归通过，回退 gold 关键修改后定向命令重新失败。
