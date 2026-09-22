# 转录组学 Skills 开发工作区

用于开发、验证和维护转录组学分析技能。目前仅建立开发骨架，尚未实现或安装分析技能。

## 目录结构

```text
transcriptomics/
├── skills/                       # 独立技能；每个子目录可单独发布
│   ├── bulk-rnaseq/               # 群体 RNA-seq 分析
│   ├── single-cell-rnaseq/        # 单细胞 RNA-seq 分析
│   └── functional-enrichment/    # 基因集与通路富集分析
├── templates/                    # 技能编写模板
├── docs/                         # 开发约定与整体设计
├── tests/                        # 测试文件
└── scripts/                      # 工作区级开发和验证工具
```

## 开始开发

1. 选择 `skills/` 下的技能目录，或新增使用小写字母与连字符命名的目录。
2. 复制 `templates/SKILL.md.template` 到该目录，命名为 `SKILL.md`，完成其中的待填写内容。
3. 按实际需要增加 `scripts/`（执行脚本）、`references/`（按需读取的分析说明）、`assets/`（报告等输出模板）。技能内部资源应随技能一起分发。
4. 在 `tests/cases/` 编写输入、预期行为及验收条件，使用 `tests/fixtures/` 中的小型数据验证。

空目录中的 `.gitkeep` 用于保留目录结构，加入实际文件后可移除。当前工作区尚未初始化 Git。
