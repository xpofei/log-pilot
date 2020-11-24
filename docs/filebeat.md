<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Filebeat 内部模块](#filebeat-%E5%86%85%E9%83%A8%E6%A8%A1%E5%9D%97)
  - [Concepts](#concepts)
  - [Crawler](#crawler)
    - [Input](#input)
  - [Pipeline](#pipeline)
    - [OutputController](#outputcontroller)
    - [Queue(mem)](#queuemem)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Filebeat 内部模块

## Concepts

Event: filebeat 中用来传输单行日志的数据结构

``` golang
// publisher.Event
// Event is used by the publisher pipeline and broker to pass additional
// meta-data to the consumers/outputs.
type Event struct {
	Content beat.Event
	Flags   EventFlags
}

// beat.Event
// Event is the common event format shared by all beats.
// Every event must have a timestamp and provide encodable Fields in `Fields`.
// The `Meta`-fields can be used to pass additional meta-data to the outputs.
// Output can optionally publish a subset of Meta, or ignore Meta.
type Event struct {
	Timestamp time.Time
	Meta      common.MapStr
	Fields    common.MapStr
	Private   interface{} // for beats private use
}
```

Batch: filebeat 中用来传输多行日志到 output 的数据结构。Batch 包含多个回调函数，用于异步处理发送结果。
当发送成功时调用 `ACK`，释放队列中的空间，更新 registar 状态。当发送失败时调用 `OnRetry` 重试。

``` golang
// Batch is used to pass a batch of events to the outputs and asynchronously listening
// for signals from these outpts. After a batch is processed (completed or
// errors), one of the signal methods must be called.
type Batch interface {
	Events() []Event

	// signals
	ACK()
	Drop()
	Retry()
	RetryEvents(events []Event)
	Cancelled()
	CancelledEvents(events []Event)
}
```

---------------------------

## Crawler

启动时加载配置文件中的 inputs。启动 reloader，定期加载动态配置文件目录中的 inputs。

### Input

Input 是一个用来包装 `harvester` 的数据结构，对外提供生命周期管理接口。Input 在建立起来时，会调用 pipeline 的 `ConnectWith` 方法获取一个 client，用于发送 events。

-------------------------------
## Pipeline

Pipeline 是一个大的功能模块，包含 `queue`, `outputController`, `consumer`, `output`

### OutputController

outputController 负责控制 queue 中的 events 发送到 output，包含 `retryer` 和 `consumer`。Output 中的 batch 发送失败时，会送到 retryer 重试。

### Queue(mem)

Queue 的默认参数

``` golang
var defaultConfig = config{
	Events:         4 * 1024,
	FlushMinEvents: 2 * 1024,
	FlushTimeout:   1 * time.Second,
}
```

工作机制:
- 队列容量为 `Events`
- 当队列中的 events 数量大于 `FlushMinEvents` 开始 flush
- 当队列中有 events 并且离上一次 flush 过了 `FlushTimeout` 时间，开始 flush

