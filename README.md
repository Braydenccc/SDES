# 学生信息交换格式（SDES）

SDES 即 Student Data Exchange Schema ，允许将学生信息及座位表编码为规范化、可拓展的单文件导出，方便在各个管理软件中交换数据。

## 文件

- `example.yml`: 带行内注释的 YAML 示例。
- `example.json`: 与 `example.yml` 等价的 JSON 示例。

## 格式约定

- `format` 固定为 `student-data-exchange-schema`，用于识别文件类型。
- `version` 是格式版本，当前示例为 `1`。
- `manifest` 保存导出软件、导出时间、语言等文件元信息。
- `metadata` 保存班级、名单或数据集说明。
- `students` 是学生实体列表，学生不直接内嵌座位。
- `tags` 定义学生标签，`students[].tags` 只引用标签 ID。
- `attributeDefinitions` 定义学生属性，`students[].attributes` 只保存属性值。
- `seatCharts` 保存一个或多个座位表。
- `seatCharts[].assignments` 保存学生与座位的绑定关系。
- `extensions` 保存软件私有扩展。

## 座位表模型

`seatCharts[].layoutModel` 用于声明座位表结构：

- `grid`: 二维网格模型，适合 open_fuckseats 一类以 `x/y` 网格单元表示座位、走廊、讲台和空白区域的软件。
- `groupedColumns`: 大组-列-行模型，适合座位表编辑器 v2 一类以大组、组内列、组内行为核心结构的软件。

## 拓展

在 extensions 中存放未列出的拓展数据。所有拓展数据的键前应当加上导出软件的命名空间，如 'sce:autoSaveTime'。对无法处理的数据应当丢弃。
