# AutoCAD 2020 ObjectARX Skill

供 Codex 使用的 AutoCAD 2020 原生 C++/ObjectARX 开发技能，包含开发规则、SDK 资料索引和只读工程检查脚本。

用于编写和审查插件命令、迁移 C# 数据库操作、排查加载失败，以及核对工程配置和测试记录。适用范围为 AutoCAD 2020 / Windows x64；普通绘图、纯 C# 插件、Python COM 自动化和其他 AutoCAD 版本不在默认范围内。

## 主要用途

| 任务 | 检查或处理的内容 |
|---|---|
| 开发与维护命令 | 按现有工程实现或审查命令，保留原有业务规则 |
| 从 C# 迁移到 C++ | 对照 ObjectARX 事务接口，保留目标数据库、绘图空间和命令行为 |
| 检查编译配置 | 核对 SDK、v141 工具链、x64 配置、C++ 运行库（CRT）和模块依赖 |
| 审查对象与事务 | 确认对象由谁管理、何时关闭或释放，以及失败后如何回滚 |
| 排查 ARX/CRX 加载失败 | 核对插件架构、依赖、导出和 AutoCAD 版本，分别检查图形界面与无界面的 Core Console |
| 核对测试记录 | 用文件哈希确认插件是否与日志对应，保留原始结果，说明还需要哪些测试 |

查阅 API 时，以项目实际使用的 2020 SDK 为准。资料索引覆盖命令、实体、图层、XData 和事务，可定位对应头文件、属性表及示例。

## 安装

需要 Node.js/npm；通过 Git 获取仓库时还需要 Git。当前仓库为私有仓库，安装前需在本机配置有读取权限的 Git/GitHub 身份验证。

使用 [Skills CLI](https://github.com/vercel-labs/skills) 安装到 Codex 用户目录：

```powershell
npx skills add xieyioo1991-web/autocad2020-objectarx --skill autocad2020-objectarx -a codex -g
```

已下载仓库时，在其父目录运行：

```powershell
npx skills add ./autocad2020-objectarx --skill autocad2020-objectarx -a codex -g
```

CLI 使用已有的身份验证配置。安装完成后，在 Codex 中使用 `$autocad2020-objectarx` 调用技能。AutoCAD、ObjectARX SDK 和编译器需要按项目要求另行准备。

## 使用

说明要处理的工程、具体问题和需要保留的行为。以下是三种常用请求。

### 检查工程配置和测试记录

```text
使用 $autocad2020-objectarx，只读检查当前 AutoCAD 2020 工程配置和已有验证记录，
列出配置差异、证据缺口和下一步需要验证的项目。
```

### 审查对象管理和事务回滚

```text
使用 $autocad2020-objectarx，审查这个命令的对象生命周期与失败回滚，
保持现有命令名、目标空间、实体格式和业务规则。
```

### 排查插件加载失败

```text
使用 $autocad2020-objectarx，排查这个插件在 AutoCAD 2020 中的加载错误，
核对实际插件路径、依赖与宿主版本，并说明需要怎样验证修复。
```

技能会先读取工作区的 `AGENTS.md`，确认允许修改的目录、必须保留的参考代码及 DWG 副本规则，再按任务查阅资料、检查或修改工程。业务验收标准由项目决定。

需要补充 C++ 代码审查或测试时，可配合 `cpp-coding-standards`、`cpp-testing` 使用。它们不是安装前提；其中的通用建议仍需符合当前工具链和 ObjectARX 对象管理规则。

## 目标环境

| 项目 | 要求 |
|---|---|
| 平台 | Windows x64 |
| 宿主 | AutoCAD 2020 / R23.1 |
| SDK | ObjectARX 2020，核对实际头文件与工程引用路径 |
| 工具链 | VS2017 / v141，具体补丁版本按项目核验 |
| C++ 语言级别 | 使用当前工具链已确认支持的级别，不自动升级到 C++20/23 |
| 脚本 | PowerShell；首版检查器行为测试使用 Windows PowerShell 5.1 |

官方支持的版本、本机曾经使用的配置和本次测试结果分别记录，不能互相替代。具体要求见 [构建与加载](references/build-and-loading.md)。

## 只读工程检查

`scripts/Inspect-Arx2020.ps1` 读取工程文件、SDK、插件和已有 JSON 日志，输出检查结果。它不构建工程、不启动 CAD，也不执行 MSBuild 导入。

在技能仓库目录运行，并将示例路径替换为实际项目路径：

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

输出为 JSON，主要字段如下：

| 字段 | 含义 |
|---|---|
| `Assessment` | 固定为 `InspectionOnly`，表示仅检查文件 |
| `CadExecuted` | 固定为 `false`，表示未运行 CAD |
| `Project.Declarations` | 从工程文件读到的设置；无法解析的复杂条件中，`Applies` 为 `null` |
| `Binary` | 插件文件的 PE 机器类型（如 x64）与 SHA256 |
| `Evidence` | 原始 JSON 记录，以及日志中的原生插件哈希与当前文件的比较结果 |
| `Findings` | 发现的差异或缺失信息；空列表不表示插件已通过运行验收 |

检查器不计算 MSBuild 导入后的最终配置，也不做完整 DLL 依赖或二进制接口（ABI）分析。参数、支持的日志字段及其他限制见 [验证与证据](references/verification.md)。

## 验证范围与限制

首版已完成技能结构校验、15 项检查器行为测试和 6 个技能使用场景检查。检查器测试覆盖配置条件、SDK 版本、PE 架构、缺失或异常文件、哈希比较、Native/Oracle 日志分类，以及检查前后输入文件是否保持不变。

这些结果只对应技能和检查器本身。具体插件仍需分别验证编译、Core Console 运行和 AutoCAD 图形界面操作，并将业务输出与基准结果比较；引用历史日志时，应确认记录对应当前插件文件。

CHM 帮助目前提供文件和主题索引，尚无全文索引。MFC/Jig/Reactors、自定义实体、Object Enabler 和 GUI 自动化需要另行查证接口并测试。

## 目录结构

仓库采用 [Agent Skills](https://agentskills.io/) 目录格式：

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

仓库保留完整的技能说明、参考资料和辅助脚本。Codex 专用显示配置、本地工作约定和对话附件不上传；AutoCAD、SDK、DWG、业务工程及本机测试日志由各自项目管理。

## 参考资料

| 资料 | 内容 |
|---|---|
| [技能入口](SKILL.md) | 适用范围、执行顺序和开发约束 |
| [来源与 SDK 索引](references/sources.md) | 2020 头文件、示例与 CHM 入口 |
| [构建与加载](references/build-and-loading.md) | 版本、编译配置、依赖和加载排查 |
| [数据库生命周期](references/database-lifecycle.md) | 对象由谁管理、何时释放，以及事务迁移 |
| [验证与证据](references/verification.md) | 测试安排、日志解释和结果报告 |
| [Autodesk ObjectARX 2020 Release Notes](https://help.autodesk.com/cloudhelp/2020/ENU/OARX-Readme/files/GUID-4DA730AB-7A47-4663-838B-21E71438D6C8.htm) | 官方版本说明 |

文档结构参考 [Vercel Agent Skills](https://github.com/vercel-labs/agent-skills/blob/main/README.md)。本项目独立维护，未复制其技能规则或代码，与 Autodesk、Vercel 无隶属关系。
