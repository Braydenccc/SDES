# SDES

>通用的学生信息交换格式
>
>v1.0

SDES 即 Student Data Exchange Schema ，是一个基于 JSON 的 DSL ，为学生信息及座位表编码提供规范化、可扩展的单文件交换格式，方便在各个管理软件中交换数据。

SDES 维护单一的可拓展的 JSON 格式来满足各项需求，以期望最优的兼容性和效率。

征集意见中，欢迎拷打。

Todos：

- 代码实例

## 文件

- [`SPEC.md`](SPEC.md): SDES v1 详细规范。
- [`example.sdes.json`](example.seds.json): 示例。

## 座位表模型

`classes[].seatCharts[].layoutModel` 用于声明座位表结构：

- `groupedColumns`: 大组-列-行模型，适合座位表编辑器 v2 一类以大组、组内列、组内行为核心结构的软件。
- `grid`: 二维网格模型，适合 open_fuckseats 一类以 `x/y` 网格单元表示座位、走廊、讲台和空白区域的软件。

`classes[].seatCharts[].platformPosition` 用于声明讲台显示位置，取值为 `top` 或 `bottom`。
`classes[].seatCharts[].doorPosition` 用于声明门显示位置，取值为 `left` 或 `right`。

`classes[].seatCharts[].seats[]` 中可以存在多个 `kind: guard` 项。`kind` 为 `guard` 时，位置写入 `guardPos` 子对象：`guardPos.side` 取值为 `left` 或 `right`，`guardPos.index` 是同侧 guard 从讲台向后排序的非负整数，最靠近讲台的 guard 为 `0`。

## 扩展

在 `extensions` 中存放未列出的扩展数据。扩展数据应当按导出软件或组织的命名空间分组，如 `app.bsce`。导入时，对无法处理的数据可以忽略；若导入后还会再次导出，应尽量原样保留未知扩展。

## 致谢

- [CSIS](https://github.com/unnuTechnology/CSIS)
- [CSES](https://github.com/SmartTeachCN/CSES)
- [UAF](https://github.com/wwiinnddyy/UnifiedAssignmentFormat)