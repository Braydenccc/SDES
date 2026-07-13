# SDES v1 规范草案

SDES（Student Data Exchange Schema）是用于交换学生信息、班级/名单信息和座位表信息的单文件 JSON 格式。

以下“生产者”代表导出SDES的来源软件，“消费者”代表接受SDES的软件。

注：下方未标为“必填”的字段均可省略。

## 约定

- 文件内容必须是 UTF-8 编码的 JSON Object，不应包含 BOM。
- 字段名使用 camelCase。可选字段没有值时应省略，不应使用 `null` 表示缺失。
- 同一 JSON Object 内的字段名不得重复。对象字段顺序没有语义，数组顺序应保留。
- 整数应保持在常见 JSON 实现可安全互操作的范围内，避免超过 `-9007199254740991..9007199254740991`。
- 颜色字符串使用 `#RRGGBB` 或 `#RRGGBBAA`。
- 时间字符串建议使用带时区的 ISO 8601，例如 `2026-07-05T10:00:00+08:00`。
- 语言/地区字符串建议使用 BCP 47 风格标签，例如 `zh-CN`、`en-US`。
- 消费者遇到未知字段时可以忽略；私有数据应写入 `extensions`。

## ID 与引用

- 所有 ID 必须是非空字符串。
- `classes[].id` 若存在，必须在整个文件内唯一。
- 学生 ID 在当前 `class` 内唯一。
- 标签 ID、属性定义 ID、座位表 ID 在当前 `class` 内唯一。
- 座位/标志物 ID 在当前 `seatChart` 内唯一。
- `assignments` 在当前 `seatChart` 内解析：`seatId` 引用当前座位表的 `seats[].id`，`studentId` 引用当前 `class` 的 `students[].id`。
- 生产者应输出稳定 ID，避免每次导出都随机变化。

## 结构

```jsonc
{
  "format": "student-data-exchange-schema", // 必填，固定文件格式标识
  "version": 1, // 必填，SDES 格式版本；v1 固定为 1

  "manifest": {
    "producer": str, // 导出来源软件
    "producerVersion": str, // 导出来源软件版本
    "exportedAt": str, // 导出时间，建议 ISO 8601 且包含时区
    "locale": str // 默认语言/地区，如 "zh-CN"
  },

  "classes": [ // 必填，至少一个班级、名单或数据集
    {
      "id": str, // 班级/名单 ID；若存在，必须在整个文件内唯一

      "metadata": {
        "name": str, // 班级、名单或数据集名称
        "description": str, // 说明文字
        "class": int, // 班级编号或班级序号
        "grade": int // 年级编号
      },

      "floor": int, // 教室所在楼层
      "building": str, // 教学楼编号
      "room": str, // 教室号

      "students": [
        {
          "id": str, // 必填，学生 ID；必须在当前 class 内唯一
          "number": str, // 学号、座号或来源系统编号；建议用字符串保留前导零
          "gender": str, // 建议值："male"、"female"、"other"、"unknown" 等

          "name": { // 必填，姓名信息
            "display": str, // 必填，用于显示、导入匹配和人工阅读的姓名
            "family": str, // 姓
            "given": str, // 名
            "nickname": str // 昵称或常用名
          },

          "tags": [str, ...], // 标签 ID，引用当前 class 的 tags[].id
          "attributes": {
            "attr:id": any // 键引用 attributeDefinitions[].id，值类型由定义决定
          }
        },
        ...
      ],

      "tags": [
        {
          "id": str, // 必填，标签 ID；必须在当前 class 内唯一
          "name": str, // 必填，标签显示名
          "color": str // 颜色，"#RRGGBB" 或 "#RRGGBBAA"
        },
        ...
      ],

      "attributeDefinitions": [
        {
          "id": str, // 必填，属性 ID；必须在当前 class 内唯一
          "name": str, // 必填，属性显示名
          "type": "string" | "number" | "boolean" | "color", // 必填，属性值类型
          "unit": str // 单位，如 "分"、"cm"
        },
        ...
      ],

      "seatCharts": [
        {
          "id": str, // 必填，座位表 ID；必须在当前 class 内唯一
          "name": str, // 座位表名称
          "layoutModel": "grid" | "groupedColumns", // 必填，座位表布局模型

          "platformPosition": "top" | "bottom", // 讲台显示位置
          "doorPosition": "left" | "right", // 门显示位置

          "coordinateSystem": {
            "origin": "front-left" | "front-right" | "back-left" | "back-right",
            "xDirection": "left-to-right" | "right-to-left",
            "yDirection": "front-to-back" | "back-to-front"
          },

          "grid": { // layoutModel 为 "grid" 时必填
            "rows": int, // 必填，正整数，网格总行数
            "columns": int // 必填，正整数，网格总列数
          },

          "groupedColumns": { // layoutModel 为 "groupedColumns" 时必填
            "groups": [
              {
                "id": str, // 大组 ID；若存在，必须在当前 seatChart 内唯一
                "columns": int, // 必填，正整数，该大组列数
                "rows": int // 必填，正整数，该大组行数
              },
              ...
            ]
          },

          "seats": [
            {
              "id": str, // 必填，座位/标志物 ID；必须在当前 seatChart 内唯一
              "kind": "seat" | "guard" | "platform" | "door" | "aisle" | "empty", // 必填

              "x": int, // grid 坐标，0-based；seat 必须满足 0 <= x < grid.columns
              "y": int, // grid 坐标，0-based；seat 必须满足 0 <= y < grid.rows

              "group": int | str, // groupedColumns 中为大组引用，可为 groups[] 索引或 groups[].id；grid 中为物理/逻辑分组提示
              "column": int, // groupedColumns 组内列索引，0-based
              "row": int, // groupedColumns 组内行索引，0-based

              "capacity": int, // 不推荐，非负整数，默认 1，单个座位的容量

              "guardPos": { // kind 为 "guard" 时必填
                "side": "left" | "right", // guard 位于教室左侧或右侧
                "index": int // 非负整数，同侧从讲台向后排序，最靠近讲台为 0
              }
            },
            ...
          ],

          "assignments": [
            {
              "seatId": str, // 必填，引用当前 seatChart 的 seats[].id
              "studentId": str // 必填，引用当前 class 的 students[].id
            },
            ...
          ]
        },
        ...
      ]
    },
    ...
  ],

  "extensions": {
    "app.example": { // 推荐按软件或组织命名空间分组
      "customField": any
    }
  }
}
```

## 座位表补充规则

- `front` 表示教室前方，通常是讲台所在方向。
- `left` / `right` 以坐在教室后方向讲台看时的左右为准。
- `platformPosition` 是显示层面的讲台位置，不改变 `front` 的语义。
- `kind: seat` 和 `kind: guard` 通常可分配学生；`platform`、`door`、`aisle`、`empty` 通常不可分配。
- 同一学生在同一个座位表中应最多出现一次。
- 同一座位的分配数量不应超过该座位的 `capacity`；未声明 `capacity` 时按 `1` 处理。

## 布局模型转换建议

本节是兼容导入建议，不是 SDES v1 的强制语义。生产者应优先按来源软件的原始结构输出 `layoutModel`；消费者无法原生表示某种模型时，可以按下列策略降级。

### grid 转 groupedColumns

- 先将 `grid` 归一化为 `grid.rows * grid.columns` 的二维单元；未出现在 `seats[]` 的坐标按缺失或空白处理。
- 若 `kind: seat` 单元提供 `group`，且同一 `group` 形成不重叠的连续物理列区间，消费者应优先按这些 `group` 划分 `groupedColumns.groups[]`。
- 否则，可将没有真实座位、且左右两侧都存在真实座位的整列视为组间隔列，并折叠为相邻大组之间的间隔。
- 每个剩余连续列区间转换为一个大组；座位坐标转换为 `column = x - groupStartX`，`row = y`。
- 大组内部的 `aisle`、`empty`、`platform`、`door` 或缺失坐标可降级为空座位或不可用座位；无法表达的标志物应保留在原始 `seats[]` 或 `extensions`，或报告为信息损失。
- 只有 `kind: seat` 的分配建议迁移；分配到非座位单元的记录应跳过并报告。

### groupedColumns 转 grid

- 按 `groupedColumns.groups[]` 顺序从左到右展开大组；每个 group 占用自身的 `columns` 列，`grid.rows` 取所有 group 的 `rows` 最大值。
- 相邻 group 之间可以插入 `kind: aisle` 的间隔列；默认可插入 1 列以保留大组分隔语义，不需要显式间隔的消费者也可以不插入。
- 插入间隔列时，`grid.columns = sum(groups[].columns) + insertedAisleColumns`。默认每两个相邻 group 间插入 1 列时，`insertedAisleColumns = groups.length - 1`。
- 对每个 `kind: seat`，解析其 `group`（`groups[]` 索引或 `groups[].id`），并校验 `column` / `row` 是否在该 group 范围内；网格坐标为 `x = groupStartX + column`，`y = row`。
- 转换后的 seat 应保持原 `id`；建议保留原始 `group` / `column` / `row` 作为回转提示。`assignments` 通过 `seatId` 引用座位，seat ID 不变时无需重写。
- group 内未出现真实座位的位置可以省略；若目标消费者需要完整矩阵，可以生成 `kind: empty`。
- `kind: guard` 没有天然 `x/y`，可按 `guardPos` 放入额外辅助列；无法表达时保留原记录但不参与 grid 坐标。`platformPosition` / `doorPosition` 应继续作为座位表字段保留，不应无条件塞入座位区。

## 扩展与兼容

- `extensions` 的顶层键应是能标识来源软件或组织的命名空间，例如 `app.bsce`、`org.open-fuckseats`。
- 扩展数据不得改变规范字段的基础语义。
- 消费者不得执行扩展数据中的代码、命令、路径或网络地址。
- 只读取数据的消费者可以忽略未知扩展；会再次导出 SDES 的消费者应尽量原样保留未知扩展。
- `format` 不匹配时，应拒绝作为 SDES 文件导入。
- `version` 高于当前支持版本时，可以拒绝，也可以进入兼容导入模式。
