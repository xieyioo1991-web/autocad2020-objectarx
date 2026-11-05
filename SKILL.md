---
name: autocad2020-objectarx
description: "Use when developing, reviewing, migrating, building, or diagnosing native C++/ObjectARX plugins specifically for AutoCAD 2020 on Windows x64, including ARX/CRX loading, AcDb object lifetime, transactions, and proportionate validation. 适用于 AutoCAD 2020 原生插件开发与 C# 到 ObjectARX 的迁移。不适用于普通绘图、纯 C#/.NET、Python COM 或其他 AutoCAD 版本。"
---

# AutoCAD 2020 ObjectARX

帮助完成 Windows x64 原生插件的实际功能、故障修复和必要验证。使用用户的语言汇报；阅读和执行范围随本次任务决定。

## 开发范围与完成边界

- 默认完成开发或修复；用户要求审查时再以审查为目标。读取适用的 `AGENTS.md`、相关规则及调用路径，复用已确认上下文；不因使用本技能先审计全项目或通读全部参考。普通实现选择按已有授权推进，用户明确要求优先于技能建议。
- 检查、拒绝条件和确认步骤只来自明确需求或实际技术依赖。开发回读、故障注入和覆盖统计不自动进入产品运行链；正式发布要求不前移到局部修复。具体判断见[开发效率](references/development-efficiency.md#让检查服务于实际功能)。
- 修好同一机制实际影响的入口，范围外新能力另记。未读取、未适配、未关联与源数据缺失分开处理；不能以猜值、任意数量上限或额外用户凭据掩盖能力缺口。
- 选择能判定本次行为的必要检查，按实际依赖复用结果；通过且没有影响交付的未决问题即停止验证并交付。仅在相关输入变化、新失败或具体疑点出现时追加检查，不为凑数量、补报告或另做一轮审查重跑。见[验证与证据](references/verification.md#按影响选择验证与复用结果)。

## 按任务查阅

从已有工程配置定位 SDK、编译器和宿主；只在相关配置变化或故障需要时重新核查。关键接口以实际使用的 2020 SDK 为准，不凭目录名或新版示例猜签名。缺少资料时继续可独立完成的工作，只询问真正影响下一步的信息。

| 任务 | 参考 |
| --- | --- |
| 查接口、命令、图层、实体或 XData | [来源与本地检索索引](references/sources.md) |
| 环境、构建或加载失败 | [构建与加载](references/build-and-loading.md) |
| 对象管理、事务、C# 迁移、失败回滚 | [数据库生命周期](references/database-lifecycle.md) |
| Manifest/JSON 出图、跨库克隆、资源、Hatch | [原生出图模式](references/native-drawing-patterns.md) |
| 样式冲突、坐标、隐藏对象、换机或版本混用 | [图纸兼容与交付一致性](references/drawing-compatibility.md) |
| 过度验证、范围扩大、重复工作、性能或交付流程 | [开发效率与复用实践](references/development-efficiency.md) |
| 选择测试、判读日志、复用结果、判断完成范围 | [验证与证据](references/verification.md) |
| 等价几何、保存/复制、首次克隆、目标诊断、证据维护 | [可复现工作流](references/reproducible-workflows.md)，只选相关案例 |

## ObjectARX 技术边界

- 目标为 AutoCAD 2020 / R23.1 / x64，工具链以 VS2017/v141 为基线。保留已确认配置，区分官方支持与本机实测，不顺带升级 SDK、宿主或语言标准。
- 区分未入库、已入库、事务管理三种对象状态。默认 `unique_ptr` 删除器不适用于数据库已管理对象；明确打开、接管、关闭和失败清理责任。
- 原生事务接口依据 `dbtrans.h`：`startTransaction/endTransaction/abortTransaction/addNewlyCreatedDBRObject` 在事务管理器上，不机械翻译 .NET 的事务方法。
- 实体的图层、样式和关联 ID 来自正确的目标数据库；跨库通过映射重新取得 ID。事务结束后不复用对象指针，嵌套变换不重复应用。
- 几何、材料、尺寸和配筋值来自实际输入及已确认规则，不用包围盒替代真实造型，不用猜值补缺。按输出用途保留原生模板、样式、属性和 Hatch 关系。
- 按项目规则使用 DWG 副本，保护用户源图与手改结果；不结束用户 CAD，不覆盖正在使用的模块。写入与事务失败按提交边界回滚；可选诊断失败不撤回已成功提交的成果，必需依赖失败不能当成完整成功。
- `.arx` 改名 `.crx` 不改变依赖或接口。Core Console 同源宿主通过不证明生产 GUI ARX 已加载；编译、实际命令执行和 GUI 验收分别表述。

## 可选只读检查

需要核对已发现的路径或旧记录时，可用 [Inspect-Arx2020.ps1](scripts/Inspect-Arx2020.ps1)：

```powershell
& (Join-Path $skillRoot 'scripts/Inspect-Arx2020.ps1') -Workspace $projectRoot -ProjectFile $projectFile -BinaryPath $binaryPath
```

变量取自已定位的路径。仅 `Workspace` 必需，其余参数及输出说明见[验证参考](references/verification.md#使用只读检查器)。脚本不构建、启动 CAD 或写文件；退出码 0、空 `Findings` 和未知 `Match` 均不代表插件通过。无需为使用技能先运行检查器。

## 交付与技能边界

报告实际改动、已执行或复用的相关验证，以及影响本次结果的限制。优先复用项目现有入口；不要求每次新建工作区、完整快照、版本、回执体系或验收清单。正式发布、清理、提交、推送和技能副本同步按用户授权及项目适用约定执行，不因读取本技能扩大操作范围。

可按需配合 `cpp-coding-standards`、`cpp-testing`，无需因此加载整套通用流程。ObjectARX 2020 的工具链和对象管理约束用于裁剪示例。本技能提供方法、资料索引和只读工具；案例进度、业务准入和发布政策留在原项目。
