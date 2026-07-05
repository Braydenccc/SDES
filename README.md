# SDES v1

通用的学生信息交换格式

SDES 即 Student Data Exchange Schema ，允许将学生信息及座位表编码为规范化、可扩展的单文件导出，方便在各个管理软件中交换数据。

SDES 维护单一的可拓展的 JSON 格式来满足各项需求，以期望最优的兼容性和效率。

尚未编写完备，暂且不要在项目中使用。之后使用 JSON 作为交换格式。

Todos：

- Schema 文件
- 代码实例

## 文件

- [`SPEC.md`](SPEC.md): SDES v1 详细规范草案。
- `example.yml`: 弃用。
- `example.json`: JSON 示例。

## 格式约定

- `format` 固定为 `student-data-exchange-schema`，用于识别文件类型。
- `version` 是格式版本，当前示例为 `1`。
- `manifest` （可选）保存导出软件、导出时间、语言等文件元信息。
- `classes` 保存一个或多个班级、名单或数据集，必须至少包含一个有效项。
- `classes[].metadata` （可选）保存班级、名单或数据集说明。
- `classes[].students` （可选）是学生实体列表，学生不直接内嵌座位。
- `classes[].tags` （可选）定义学生标签，`classes[].students[].tags` 只引用标签 ID。
- `classes[].attributeDefinitions` （可选）定义学生属性，`classes[].students[].attributes` 只保存属性值。
- `classes[].seatCharts` （可选）保存一个或多个座位表。
- `classes[].seatCharts[].assignments` （可选）保存学生与座位的绑定关系。
- `extensions` （可选）保存软件私有扩展。

除 `format`、`version`、`classes` 以外，大部分对象都是可选的。

## 通用约定

- 学生 ID 在当前 `class` 内唯一。
- 座位 ID 在当前 `seatChart` 内唯一。
- `assignments` 在当前 `seatChart` 内解析：`seatId` 引用当前座位表的 `seats[].id`，`studentId` 引用当前 `class` 的 `students[].id`。
- 颜色字符串使用 `#RRGGBB` 或 `#RRGGBBAA`。
- JSON 文件必须使用 UTF-8 编码，不应包含 BOM；对象键不得重复；对象字段顺序没有语义；数组顺序应保留；整数应保持在常见 JSON 实现可安全互操作的范围内。

## 座位表模型

`classes[].seatCharts[].layoutModel` 用于声明座位表结构：

- `groupedColumns`: 大组-列-行模型，适合座位表编辑器 v2 一类以大组、组内列、组内行为核心结构的软件。
- `grid`: 二维网格模型，适合 open_fuckseats 一类以 `x/y` 网格单元表示座位、走廊、讲台和空白区域的软件。

`classes[].seatCharts[].platformPosition` 用于声明讲台显示位置，取值为 `top` 或 `bottom`。
`classes[].seatCharts[].doorPosition` 用于声明门显示位置，取值为 `left` 或 `right`。

`classes[].seatCharts[].seats[]` 中可以存在多个 `kind: guard` 项。`kind` 为 `guard` 时，位置写入 `guardPos` 子对象：`guardPos.side` 取值为 `left` 或 `right`，`guardPos.index` 是同侧 guard 从讲台向后排序的非负整数，最靠近讲台的 guard 为 `0`。

## 扩展

在 `extensions` 中存放未列出的扩展数据。扩展数据应当按导出软件或组织的命名空间分组，如 `app.bsce`。导入时，对无法处理的数据可以忽略；若导入后还会再次导出，应尽量原样保留未知扩展。
