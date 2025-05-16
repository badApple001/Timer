markdown
# Timer - Unity定时任务调度系统

![Unity](https://img.shields.io/badge/Unity-2021.3+-blue.svg)
![License](https://img.shields.io/badge/License-MIT-green.svg)

## 简介

Timer 是一个 Unity 环境下高效、灵活的定时任务调度系统，支持以下功能：

- 支持多种时间源（游戏时间 / 非缩放时间 / 真实时间）
- 支持一次性延迟执行和重复执行
- 提供 ID、回调、目标对象等多种查询和销毁方式
- 内建任务池（对象复用）避免频繁 GC
- 支持线程安全的任务添加/移除
- 提供完整生命周期管理

## 核心特性

| 特性 | 描述 |
|------|------|
| 多时间源支持 | GameTime、UnscaledTime 和 RealTime |
| 对象池优化 | 避免频繁分配，提升性能 |
| 安全多线程 | 添加/移除任务队列线程安全，主线程调度 |
| 支持任务查找 | 可通过 ID、Action、目标对象查找定时任务 |
| 一次性与循环任务 | 支持延迟一次性任务和循环执行任务 |
| 稳定运行 | 通过 MonoBehaviour 生命周期运行，自动初始化，独立运行场景生命周期外 |

## 时间源类型

```csharp
public enum TimerTimeSource {
    GameTime,      // Time.time
    UnscaledTime,  // Time.unscaledTime
    RealTime       // DateTimeOffset.UtcNow
}
快速开始
延迟执行一个函数
csharp
Timer.Delay(2.0f, () => Debug.Log("2秒后执行"));
循环执行（每1秒执行一次，无限循环）
csharp
Timer.Loop(1.0f, () => Debug.Log("每秒触发一次"));
循环执行（立即执行 + 执行5次）
csharp
Timer.Loop(1.0f, MyCallback, TimerTimeSource.UnscaledTime, immediate: true, times: 5);
API 说明
创建定时任务
方法	说明
Timer.Delay(float delay, Action callback, TimerTimeSource timeSource)	延迟执行一次回调
Timer.Loop(float interval, Action callback, TimerTimeSource timeSource, bool immediate, int times)	间隔时间循环执行回调，支持立即执行和限定次数
查找定时任务
方法	说明
Timer.Find(long id)	根据唯一 ID 查找任务
Timer.Find(Action func)	查找所有指定方法的任务
Timer.Find(object target)	查找指定目标对象绑定的方法任务
终止定时任务
方法	说明
Timer.Kill(long id)	根据 ID 终止任务
Timer.Kill(Action func)	终止所有指定方法的任务
Timer.Kill(object target)	终止指定对象上的所有任务
Timer.Kill<T>()	根据类名终止所有任务（包括 lambda）
Timer.KillAll()	清理所有任务
内部机制
对象池机制：任务对象使用 ConcurrentQueue<TimerTask> 循环复用，避免 GC

双缓冲快照列表：主线程调度时快照任务列表，避免遍历冲突

排序插入调度：内部任务按到期时间排序，保障调度精度与性能

自动初始化：通过 [RuntimeInitializeOnLoadMethod] 自动创建 Timer 挂载 GameObject

注意事项
使用成员方法而非 lambda 可提升可控性（利于 Kill 操作）

不支持精确毫秒调度，适合用于逻辑调度、UI、冷却、延迟等

RealTime 不受 Unity 时间系统影响，可跨暂停/切后台使用

示例场景
csharp
public class Example : MonoBehaviour
{
    void Start()
    {
        Timer.Delay(5f, OnTimeout); // 5秒后执行一次
    }

    void OnTimeout()
    {
        Debug.Log("延迟执行完毕");
    }

    void OnDestroy()
    {
        Timer.Kill(this); // 清理当前实例上所有定时器
    }
}
扩展建议
✅ 可拓展支持 Coroutine（协程回调）

✅ 可拓展带参数回调、异步支持（如返回 Task）

✅ 可集成 ECS 环境中运行

✅ 可接入编辑器模式（EditorApplication.update）

下载与安装
插件下载：百度云盘知识库分享

源码结构
Timer/
├── Runtime/
│   └── Timer.cs          # 核心实现
└── Editor/
    └── TimerInspector.cs # 编辑器扩展
贡献
欢迎提交 Issue 或 Pull Request。反馈请联系：Isysprey@foxmail.com
