# SING-SQL: 领域内Text-to-SQL数据生成框架

基于SING-SQL论文的实现，支持高质量、高覆盖率的领域内Text-to-SQL数据生成。数据集可用与金盘项目的评测，也可以用于in-domain Text-to-SQL任务的训练和评估。

## 功能特性

- **多表支持**：从单表场景扩展到多表场景
- **表级子模式**：基于外键图生成可连通的表组合
- **列级子模式**：保留PK/FK连接列，对非连接列进行滑窗采样
- **SQL生成**：支持4个复杂度级别（simple, moderate, challenging, window）
- **质量验证**：LLM-as-a-judge验证、SQL可执行性检查、自动修复
- **推理轨迹**：divide-and-conquer策略生成step-by-step推理轨迹
- **列聚焦生成**：自动识别低频率列并进行针对性生成
- **模块化设计**：清晰的模块划分，易于扩展和维护

## 系统架构

```
dataset_generation/
├── schema/                        # Schema相关模块
│   ├── data_ingestion.py         # 数据加载（SQLite + Matrix One接口）
│   ├── table_level_subschema.py  # 表级子模式生成
│   └── column_level_subschema.py # 列级子模式生成
├── generation/                    # 生成模块
│   ├── sql_generator.py          # SQL生成（4个复杂度级别）
│   ├── nl_translator.py          # SQL转自然语言
│   ├── reasoning_generator.py    # 推理轨迹生成
│   └── focus_generator.py        # 列聚焦生成
└── validation/                    # 验证模块
    ├── llm_judge.py              # LLM-as-a-judge验证
    ├── sql_executor.py           # SQL可执行性检查
    └── sql_repair.py             # SQL自动修复
```

## 核心模块

### Schema模块

- **data_ingestion.py**: 数据库schema加载，支持SQLite，预留Matrix One接口
- **table_level_subschema.py**: 表级子模式生成，基于外键图分析表之间的连接关系，生成可连通的表组合
- **column_level_subschema.py**: 列级子模式生成，采用滑动窗口策略对非连接列进行采样

### Generation模块

- **sql_generator.py**: SQL生成核心模块，支持4个复杂度级别（simple/moderate/challenging/window）和列聚焦生成
- **nl_translator.py**: 将SQL语句转换为自然语言问题
- **reasoning_generator.py**: 采用divide-and-conquer策略生成推理轨迹
- **focus_generator.py**: 列聚焦生成，包含列频率统计和针对性生成，提升数据覆盖率

### Validation模块

- **llm_judge.py**: 使用LLM-as-a-judge方法验证SQL-Text对的质量
- **sql_executor.py**: SQL可执行性检查，确保生成的SQL语法正确
- **sql_repair.py**: SQL自动修复，对存在问题的SQL进行修正

## 关键参数

| 参数 | 说明 |
|------|------|
| `TABLE_COUNTS` | 表级子模式的表数限制列表，如 `[3, 2, 1]` 表示生成3表、2表、1表组合 |
| `WINDOW_SIZE` | 列级滑窗窗口大小（默认15列） |
| `STRIDE` | 列级滑窗步长（默认10列） |
| `EXAMPLES_PER_LEVEL` | 每个复杂度级别生成的SQL条数（默认2条） |
| `COLUMN_FREQUENCY_THRESHOLD` | 列聚焦生成的频率阈值（默认5） |
| `GENERATE_REASONING_TRACE` | 是否生成推理轨迹（用于RL数据集） |

## 输出格式

```json
[
  {
    "question": "查询所有作者的姓名和主页",
    "sql": "SELECT name, homepage FROM author",
    "level": "simple",
    "subschema": {
      "author": ["aid", "name", "homepage"]
    },
    "reasoning_trace": "步骤1: 识别需要查询的表...\n步骤2: 选择需要的列..."
  }
]
```

字段说明：
- `question`: 自然语言问题
- `sql`: SQL查询语句
- `level`: 复杂度级别（simple/moderate/challenging/window）
- `subschema`: 使用的子模式（表名到列列表的映射）
- `reasoning_trace`: 推理轨迹（可选）

## 生成效果示例

以academic数据库（15个表）为例：
- 表级子模式：68个组合（3表: 36个, 2表: 17个, 1表: 15个）
- 所有表都被覆盖：15/15 ✓
- 列级子模式：基于表组合和列滑窗生成
- 质量验证：自动过滤无效SQL-Text对
- 列聚焦生成：自动补充低频率列的数据

## 参考

- **SING-SQL论文**: A Synthetic Data Generation Framework for In-Domain Text-to-SQL Translation
