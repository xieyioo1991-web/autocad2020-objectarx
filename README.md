# AutoCAD 2020 ObjectARX 开发技能

供 Codex 使用的 AutoCAD 2020 原生 C++ / ObjectARX 开发技能，包含开发规则、SDK 资料索引和只读工程检查脚本。

它用于实现和审查插件命令、迁移 C# 数据库操作、排查 ARX/CRX 加载问题，以及核对工程配置和测试记录。处理结构出图等业务插件时，命令、绘图空间、比例、图元格式和配筋规则由具体项目定义，迁移过程中应保持这些约定。

## 目录

- [安装与使用](#安装与使用)
- [基本工作流程](#基本工作流程)
- [适用任务与目标环境](#适用任务与目标环境)
- [只读工程检查](#只读工程检查)
- [资料与目录](#资料与目录)
- [验证范围与限制](#验证范围与限制)
- [常见问题](#常见问题)
- [更新](#更新)

## 安装与使用

### 安装技能

需要 Node.js/npm，以及用于获取仓库的 Git。当前仓库为私有仓库，先在本机配置具有读取权限的 Git/GitHub 身份验证。

使用 [Skills CLI](https://github.com/vercel-labs/skills) 安装到 Codex 用户目录：

```powershell
npx skills add xieyioo1991-web/autocad2020-objectarx --skill autocad2020-objectarx -a codex -g
```

已有本地仓库时，在其父目录运行：

```powershell
npx skills add ./autocad2020-objectarx --skill autocad2020-objectarx -a codex -g
```

CLI 使用已有的仓库身份验证。技能安装完成后，AutoCAD、ObjectARX SDK 和编译器仍需按项目要求准备。

### 调用技能

在项目会话中使用 `$autocad2020-objectarx`，并说明工程位置、具体问题和需要保留的行为。下面的请求可以直接用于配置检查：

```text
使用 $autocad2020-objectarx，只读检查当前 AutoCAD 2020 工程配置和已有验证记录，
列出配置差异、证据缺口和下一步需要验证的项目。
```

审查数据库对象和事务时：

```text
使用 $autocad2020-objectarx，审查这个命令的对象生命周期与失败回滚，
保持现有命令名、目标空间、实体格式和业务规则。
```

排查加载问题时：

```text
使用 $autocad2020-objectarx，排查这个插件在 AutoCAD 2020 中的加载错误，
核对实际插件路径、依赖与宿主版本，并说明需要怎样验证修复。
```

涉及绘图结果的修改，应一并给出参考 DWG、预期输出及验收口径。例如哪些图层和实体允许改动，尺寸、配筋及文字应与哪份基线比较。

## 基本工作流程

### 1. 确认项目边界

先读取工作区的 `AGENTS.md` 和相关目录说明，确认可修改范围、冻结的参考代码、DWG 副本要求及本次任务。C# 迁移以原项目的命令和业务规则为依据，不附带改变数据模型。

### 2. 核对实际环境

从工程属性和已定位的文件确认 SDK、编译器、宿主及插件版本。文件夹名称、曾经成功的配置和本次实际使用的配置分别记录。

### 3. 查证接口并处理问题

按任务查阅构建、数据库生命周期或验证资料。关键 API 以工程实际使用的 2020 SDK 头文件和示例为准。

对象管理需区分未入库、已入库和事务管理三种状态，明确由谁接管、关闭或释放。迁移事务代码时，应核对原生事务管理器接口，不能逐字套用 .NET 的调用方式。

### 4. 按改动范围验证

配置检查、编译链接、Core Console、AutoCAD GUI 和业务差分各有独立的验证范围。报告说明本次执行了什么、结果对应哪个插件文件，以及哪些场景仍未验证。

文档和只读检查任务不启动 CAD。需要宿主测试时，使用项目允许的 DWG 副本和独立运行目录。

## 适用任务与目标环境

| 任务 | 主要处理内容 |
| --- | --- |
| 开发或维护命令 | 按现有工程实现功能，保持命令行为和业务规则 |
| 从 C# 迁移到 C++ | 对照原生事务与对象管理接口，保留目标数据库和绘图空间 |
| 检查编译配置 | 核对 SDK、v141、x64、C++ 运行库及模块依赖 |
| 审查对象与事务 | 检查所有权转移、关闭和释放时机、失败回滚 |
| 排查加载失败 | 核对插件架构、依赖、导出和宿主版本 |
| 核对测试记录 | 比较插件哈希与日志，保留原始结果及未验证项 |

目标环境如下：

| 项目 | 范围 |
| --- | --- |
| 平台 | Windows x64 |
| 宿主 | AutoCAD 2020 / R23.1 |
| SDK | ObjectARX 2020，按实际头文件和工程引用路径核对 |
| 工具链 | VS2017 / v141，具体补丁版本按项目核验 |
| C++ 语言级别 | 使用当前工具链已确认支持的级别，不自动升级到 C++20/23 |
| 脚本 | PowerShell；首版检查器行为测试使用 Windows PowerShell 5.1 |

普通绘图、纯 C#/.NET 插件、Python COM 自动化和其他 AutoCAD 版本不在默认范围内。工具链与加载要求见[构建与加载](references/build-and-loading.md)。

可配合 `cpp-coding-standards` 做一般 C++ 审查，配合 `cpp-testing` 设计测试；两者均非安装前提。通用建议需要按当前工具链和 ObjectARX 对象管理方式调整。

## 只读工程检查

`scripts/Inspect-Arx2020.ps1` 读取工程文件、SDK、插件和已有 JSON 日志，向标准输出返回 JSON。它不构建工程、不执行 MSBuild 导入、不启动 CAD，也不写文件或更改系统配置。

在技能仓库目录运行，将示例路径替换为实际项目路径：

```powershell
$inspection = @{
    Workspace = 'C:\work\MyArxProject'
    ProjectFile = 'MyPlugin.vcxproj'
    SdkRoot = 'C:\Autodesk\ObjectARX_for_AutoCAD_2020_Win_64_bit'
    BinaryPath = 'artifacts\Release\MyPlugin.arx'
    EvidencePaths = @('logs\verification.json')
    Configuration = 'Release'
    Platform = 'x64'
}
& .\scripts\Inspect-Arx2020.ps1 @inspection
```

`Workspace` 必填，其余参数可省略；相对路径以该工作区为基准。`BuildToolsRoot` 和 `AutoCadRoot` 可用于读取工具链与宿主的文件版本。

| 输出字段 | 如何理解 |
| --- | --- |
| `Assessment` | 固定为 `InspectionOnly`，表示本次仅检查文件 |
| `CadExecuted` | 固定为 `false`，表示未运行 CAD |
| `Project.Declarations` | 工程文件中的字面声明；复杂条件无法解析时，`Applies` 为 `null` |
| `Binary` | 插件的 PE 机器类型和 SHA256 |
| `Evidence` | 原始 JSON 记录，以及日志中原生插件哈希与当前文件的比较结果 |
| `Findings` | 检查发现的差异或缺失信息 |

退出码 0 表示检查完成；空 `Findings` 不能用作插件运行验收结论。`Match=null` 表示无法比较，并非哈希一致。

检查器不计算 MSBuild 导入后的最终配置，也不做完整 DLL 依赖或 ABI 分析。它只识别已支持的日志字段，未知格式仍需阅读原始记录。参数和字段说明见[验证与证据](references/verification.md)。

## 资料与目录

仓库采用 [Agent Skills](https://agentskills.io/) 目录格式：

```text
autocad2020-objectarx/
├── README.md                          # 安装、用法与资料导航
├── SKILL.md                           # 技能入口与开发规则
├── references/
│   ├── sources.md                     # 来源、版本与 SDK 主题索引
│   ├── build-and-loading.md           # 工具链、构建与加载诊断
│   ├── database-lifecycle.md          # 对象所有权、事务与迁移
│   ├── native-drawing-patterns.md     # Manifest 出图、跨图克隆、Hatch 与回读
│   └── verification.md                # 验证层级与证据判读
└── scripts/
    └── Inspect-Arx2020.ps1             # 只读检查器，输出 JSON
```

| 资料 | 查阅内容 |
| --- | --- |
| [技能入口](SKILL.md) | 适用范围、执行顺序和开发约束 |
| [来源与 SDK 索引](references/sources.md) | 命令、实体、图层、XData、事务相关头文件与示例，以及 CHM 入口 |
| [构建与加载](references/build-and-loading.md) | 版本、编译配置、模块依赖和加载排查 |
| [数据库生命周期](references/database-lifecycle.md) | 对象所有权、释放时机、事务和 C# 迁移 |
| [原生出图模式](references/native-drawing-patterns.md) | Manifest/JSON 出图、资源模板、跨图克隆、Hatch 和实体回读 |
| [验证与证据](references/verification.md) | 测试安排、日志判读和结果报告 |
| [ObjectARX 2020 Release Notes](https://help.autodesk.com/cloudhelp/2020/ENU/OARX-Readme/files/GUID-4DA730AB-7A47-4663-838B-21E71438D6C8.htm) | Autodesk 官方版本说明 |

仓库保留完整技能说明、参考资料和检查脚本。Codex 专用显示配置、本地工作约定和对话附件不上传；AutoCAD、SDK、业务工程、DWG 及本机测试日志由各自项目管理。

## 验证范围与限制

首版记录包括技能结构校验、15 项检查器行为测试和 6 个技能使用场景检查。检查器用例覆盖配置条件、SDK 版本、PE 架构、缺失或异常文件、哈希比较、Native/Oracle 日志分类，以及检查前后输入文件是否保持不变。

这些记录对应技能和检查器本身。具体插件的编译、Core Console 运行、GUI 操作及业务输出需要分别验证；引用历史日志时，应确认日志中的哈希与当前插件文件一致。

CHM 目前提供文件和主题索引，尚无全文索引。MFC、Jig、Reactors、自定义实体、Object Enabler 和 GUI 自动化需要针对具体任务另行查证接口并测试。

## 常见问题

| 情况 | 处理方式 |
| --- | --- |
| 私有仓库安装失败 | 先确认本机 Git 身份具有该仓库的读取权限 |
| 检查器退出正常，但插件仍无法加载 | 继续核对实际宿主、模块依赖和加载日志；只读检查未执行加载 |
| 日志通过，但哈希与当前插件不同 | 保留历史结论，对当前产物重新安排验证 |
| Core Console 通过，GUI 仍有问题 | 分别检查两个宿主下实际执行的命令和交互范围 |
| 想直接使用新版 SDK 示例 | 先在实际 2020 SDK 中核对接口和工具链支持情况 |

## 更新

通过 Skills CLI 安装的技能可使用其[更新命令](https://github.com/vercel-labs/skills#other-commands)：

```powershell
npx skills update autocad2020-objectarx
```

本地源码仓库使用 Git 同步。更新后检查安装位置中的 `SKILL.md` 和参考文件；技能规则更新不改变业务项目的验收基线。
