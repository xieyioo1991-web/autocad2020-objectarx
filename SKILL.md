---
name: autocad2020-objectarx
description: "Use when developing, reviewing, migrating, building, or diagnosing native C++/ObjectARX plugins specifically for AutoCAD 2020 on Windows x64, including ARX/CRX loading, AcDb object lifetime, transactions, and validation evidence. 适用于 AutoCAD 2020 原生插件开发与 C# 到 ObjectARX 的迁移。不适用于普通绘图、纯 C#/.NET、Python COM 或其他 AutoCAD 版本。"
---

# AutoCAD 2020 ObjectARX

面向 Windows x64 的版本限定开发指南。先核对项目事实，再修改和验证；使用用户的语言汇报。

## 开始工作

1. 读取工作区 `AGENTS.md` 及相关目录规则，确定可写目录、冻结基线、DWG 副本规则和本次任务。迁移时从项目规则读取命令、出图、比例与数据格式；不自行引入自定义实体或重构数据模型。
2. 从工程属性、已有说明和工具安装目录定位 SDK、编译器、宿主与插件；不要凭文件夹名认定版本。缺少资料时标为未知，继续可独立完成的工作。
3. 按任务读取下表中的参考；关键 API 要在**实际使用的 2020 SDK** 中查证。网上新版教程只提供检索线索，不作为 2020 签名或兼容性的证明。

| 任务 | 读取 |
|---|---|
| 查接口、命令入口、图层/实体/XData 示例 | [来源与本地检索索引](references/sources.md) |
| 环境、工程、编译、加载失败 | [构建与加载](references/build-and-loading.md) |
| 数据库对象、事务、C# 迁移、失败回滚 | [对象生命周期](references/database-lifecycle.md) |
| Manifest/JSON 到 DWG、跨图克隆、资源模板、Hatch、实体回读 | [原生出图模式](references/native-drawing-patterns.md) |
| 设计测试、解释日志、判断完成范围 | [验证与证据](references/verification.md) |

## 必须保持的判断

- 目标为 AutoCAD 2020 / R23.1 / x64。工具链以 VS2017/v141 为基线；官方补丁范围与本机有限实测分开记录。保留已确认配置；不因通用 C++ 示例自动升级 C++20/23、SDK 或宿主。
- 数据库对象区分未入库、已入库、事务管理三种状态。默认 `unique_ptr` 删除器不适用于数据库已管理对象。谁打开、谁接管、失败时谁清理要明确，避免重复关闭或删除。
- 原生事务接口依据 `dbtrans.h`：`startTransaction/endTransaction/abortTransaction/addNewlyCreatedDBRObject` 在事务管理器上；不要把 .NET 的 `Transaction.AddNewlyCreatedDBObject` 机械翻译到 `AcTransaction`。
- `.arx` 改名 `.crx` 不改变依赖或接口。Core Console 和 AutoCAD GUI 分别验证；无 UI 探针的历史成功不覆盖 UI 插件。
- 配置符合、编译成功、历史记录通过、本次运行通过、业务验收是不同结论。每个结论附对应范围和证据，哈希不符的旧记录不算当前二进制通过。
- DWG 只按项目规则使用测试副本。不要结束用户的 CAD、覆盖占用中的模块或把输出写入受保护源目录。测试工具自身也先检查输出路径。
- JSON/Manifest 到 DWG 时，先冻结版本化契约，再让 ARX 做语法/语义校验；ARX 不应猜测上游业务规则或依赖旧的绝对路径、句柄和隐含坐标。
- 图框、尺寸、轴号块、带属性块和专业标注优先从经过哈希校验的原生模板对象克隆，再只修改契约允许的几何或文字字段；克隆数量正确不等于对象关系、属性坐标和显示样式正确。
- Hatch、线型、标注样式等宿主对象要在目标数据库内解析并回读；不要把当前工作数据库中的 `ObjectId` 写入独立数据库，也不要用一次 Core Console 统计替代实体级或 GUI 验证。

## 只读检查

可用 [Inspect-Arx2020.ps1](scripts/Inspect-Arx2020.ps1) 读取已发现的路径：

```powershell
# 从工作区执行；$skillRoot、$sdkRoot 等须来自已定位的真实路径。
$argsForInspection = @{
    Workspace = (Get-Location).Path
    ProjectFile = $projectPath
    SdkRoot = $sdkRoot
    BuildToolsRoot = $buildToolsRoot
    AutoCadRoot = $autoCadRoot
    BinaryPath = $pluginPath
    EvidencePaths = @($evidencePath)
    Configuration = 'Release'
    Platform = 'x64'
}
& (Join-Path $skillRoot 'scripts/Inspect-Arx2020.ps1') @argsForInspection
```

除 `Workspace` 外均可省略；相对路径相对于工作区。脚本只输出 JSON，不构建、启动 CAD、安装软件、执行 MSBuild 导入或写文件。需要保存时由调用方写到工作区内的新报告。`Applies=null` 表示条件未解析，`Match=null` 表示无法比较；退出码 0 和空 `Findings` 都不代表通过验收。完整限制见验证参考。

## 修改与交付

遵循现有工程结构实现最小改动，核对 SDK 返回值和失败路径，再按任务执行相关验证。只有文档或检查任务时，不附带运行 CAD。报告：实际改动、查证来源、此次执行的验证、历史证据及剩余未知项。

涉及从结构数据或其他 JSON 契约生成 DWG 时，按需读取[原生出图模式](references/native-drawing-patterns.md)，其中的资源来源、跨图克隆、Hatch 和回读规则属于 ObjectARX 2020 的高风险路径。

可配合 `cpp-coding-standards` 做一般 C++ 设计，配合 `cpp-testing` 做适当单元测试；它们不是必需依赖。工具链、ObjectARX 所有权和宿主约束优先用于裁剪通用建议。不要直接复制 C++20 示例、默认删除器或不适用 v141 的测试工具参数。

首版提供开发指导、资料索引和只读检查。MFC/Jig/Reactors、自定义实体、Object Enabler、GUI 自动化与安装器需另行查证和验证；技能安装不代表插件已经迁移或通过运行验收。
