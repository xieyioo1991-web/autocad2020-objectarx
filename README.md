# AutoCAD 2020 ObjectARX Skill

面向 **AutoCAD 2020 / Windows x64** 的开发技能，提供原生 C++/ObjectARX 开发指导、SDK 资料导航和只读工程检查。

围绕版本兼容、数据库对象生命周期和验证证据组织开发流程，帮助你在现有工程中实现命令、迁移事务逻辑、排查 ARX/CRX 加载问题，并明确每次修改实际验证到了哪里。

## 适用场景

- 为 AutoCAD 2020 原生插件新增、审查或维护命令。
- 将 C#/.NET 数据库操作迁移到 C++/ObjectARX，保留现有业务规则。
- 检查 SDK、v141 工具链、x64 配置、CRT 和模块依赖。
- 排查对象关闭、所有权转移、事务提交与失败回滚问题。
- 核对插件哈希、宿主版本和已有测试日志，判断还缺哪些验证。

普通绘图、纯 C# 插件、Python COM 自动化和其他 AutoCAD 版本不在本技能的默认范围内。

## 覆盖能力

| 能力 | 具体用途 |
|---|---|
| 版本与构建检查 | 核对 2020 SDK、工程声明、编译器与宿主信息，区分官方支持范围和本机实测配置 |
| SDK 资料导航 | 按命令、实体、图层、XData、事务等主题定位 2020 头文件、属性表与示例 |
| 对象生命周期 | 明确对象入库前、入库后和事务纳管后的责任，检查 close、delete 与失败路径 |
| C# 事务迁移 | 对照原生事务管理器接口，保留目标数据库、空间选择和原有命令语义 |
| 加载诊断 | 检查架构、依赖、导出和宿主差异，分别处理 AutoCAD GUI 与 Core Console |
| 验证证据检查 | 保留日志原始结果，对比当前插件哈希，区分编译、运行和业务验收 |

技能内容采用 [Agent Skills](https://agentskills.io/) 的目录形式，包含 `SKILL.md`、按需参考资料和 PowerShell 辅助脚本。这些是项目主体，完整保留；Codex 专用显示配置、本地工作约定和对话附件不随仓库上传。

## 安装

使用 [Skills CLI](https://github.com/vercel-labs/skills) 安装到 Codex 用户目录：

```powershell
npx skills add xieyioo1991-web/autocad2020-objectarx --skill autocad2020-objectarx -a codex -g
```

这是私有仓库。执行安装的 Git/GitHub 环境需要具有仓库读取权限；CLI 会使用已有的身份验证配置。命令需要可用的 Node.js/npm 环境，Git 方式获取仓库还需要 Git。

已经下载到本地时，可在仓库的父目录安装：

```powershell
npx skills add ./autocad2020-objectarx --skill autocad2020-objectarx -a codex -g
```

安装完成后，在 Codex 中明确调用 `$autocad2020-objectarx`。安装技能不会安装 AutoCAD、ObjectARX SDK 或编译器；这些开发依赖按目标项目配置。

## 使用

描述目标、相关工程和需要保留的行为，即可让 Codex 按技能执行任务。

**检查工程与已有记录**

```text
使用 $autocad2020-objectarx，只读检查当前 AutoCAD 2020 工程配置和已有验证记录，
列出配置差异、证据缺口和下一步需要验证的项目。
```

**审查数据库对象与事务**

```text
使用 $autocad2020-objectarx，审查这个命令的对象生命周期与失败回滚，
保持现有命令名、目标空间、实体格式和业务规则。
```

**排查加载失败**

```text
使用 $autocad2020-objectarx，排查这个插件在 AutoCAD 2020 中的加载错误，
核对实际插件路径、依赖与宿主版本，并说明需要怎样验证修复。
```

它也可以与 `cpp-coding-standards`、`cpp-testing` 配合使用。通用 C++ 建议需服从当前工具链和 ObjectARX 对象所有权规则；这两个技能不是安装前提。

## 目标环境

| 项目 | 范围 |
|---|---|
| 平台 | Windows x64 |
| 宿主 | AutoCAD 2020 / R23.1 |
| SDK | ObjectARX 2020，以实际头文件及工程引用路径为准 |
| 工具链 | VS2017 / v141；具体补丁与兼容性按项目核验 |
| 语言级别 | 使用工具链已确认支持的级别，不自动升级到 C++20/23 |
| 脚本环境 | PowerShell；首版行为测试使用 Windows PowerShell 5.1 |

官方支持范围、本机历史配置与当前运行结果分别记录。详细约束见 [构建与加载](references/build-and-loading.md)。

## 目录结构

```text
autocad2020-objectarx/
├── README.md                          # 安装、用法与资料导航
├── SKILL.md                           # 技能入口与开发规则
├── references/
│   ├── sources.md                     # 来源、版本与 SDK 主题索引
│   ├── build-and-loading.md           # 工具链、构建与加载诊断
│   ├── database-lifecycle.md          # 对象所有权、事务与迁移
│   └── verification.md                # 验证层级与证据判读
└── scripts/
    └── Inspect-Arx2020.ps1             # 只读检查器，输出 JSON
```

## 只读检查器

在技能仓库目录中运行，按实际项目替换以下示例路径：

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

`Workspace` 必填，其余参数可省略；相对路径基于工作区。还可传入 `BuildToolsRoot` 和 `AutoCadRoot`，读取工具链与宿主的文件版本。

脚本输出 JSON，不构建工程、不启动 CAD，也不执行 MSBuild 导入。重要字段：

| 字段 | 含义 |
|---|---|
| `Assessment` | 固定为 `InspectionOnly`，表示这次只检查文件 |
| `CadExecuted` | 固定为 `false` |
| `Project.Declarations` | 工程字面声明；复杂条件的 `Applies` 保持 `null` |
| `Binary` | 当前插件的 PE 机器类型与 SHA256 |
| `Evidence` | 原始 JSON 记录，以及识别到的原生插件哈希比较结果 |
| `Findings` | 已检测的差异与缺失信息；空列表不代表运行验收通过 |

完整参数、支持的日志字段和限制见 [验证与证据](references/verification.md)。

## 验证与限制

首版在维护工作区完成技能结构校验、15 项检查器行为测试和 6 个技能使用场景检查。覆盖配置条件、SDK 版本、PE 架构、缺失输入、异常文件、哈希比较、Native/Oracle 日志分类及输入文件不变性。

这些检查验证的是技能和检查器。插件的编译、Core Console 运行、AutoCAD GUI 交互和业务差分仍需按具体修改分别执行；历史日志也需要与当前产物对应。

- CHM 目前提供文件与主题索引，尚未建立全文索引。
- 检查器不求值 MSBuild 导入，不做完整 DLL 依赖或 ABI 分析。
- MFC/Jig/Reactors、自定义实体、Object Enabler 和 GUI 自动化需要另行查证与验证。
- 工作区 `AGENTS.md` 决定可写目录、冻结基线和 DWG 副本规则；技能不会改变项目的业务验收标准。

仓库仅包含技能说明、参考索引和辅助脚本；AutoCAD、SDK、DWG、业务工程及本机测试日志由各自项目管理。

## 参考资料

| 资料 | 用途 |
|---|---|
| [技能入口](SKILL.md) | 查看触发范围、工作顺序与关键约束 |
| [来源与 SDK 索引](references/sources.md) | 查找 2020 头文件、示例与 CHM 入口 |
| [构建与加载](references/build-and-loading.md) | 核对版本、配置、依赖和加载问题 |
| [数据库生命周期](references/database-lifecycle.md) | 处理对象所有权、事务与迁移 |
| [验证与证据](references/verification.md) | 设计检查、解释日志与报告验证范围 |
| [Autodesk ObjectARX 2020 Release Notes](https://help.autodesk.com/cloudhelp/2020/ENU/OARX-Readme/files/GUID-4DA730AB-7A47-4663-838B-21E71438D6C8.htm) | 核对官方版本说明 |

README 的组织参考 [Vercel Agent Skills](https://github.com/vercel-labs/agent-skills/blob/main/README.md)：先说明适用场景与能力，再给出安装、使用和文件结构。本文按本技能重新撰写，未复制其技能规则或代码。本项目为独立维护的技能，与 Autodesk、Vercel 无隶属关系。
