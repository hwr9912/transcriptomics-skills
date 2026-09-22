---
name: annotate-cell-subclusters
description: "聚类、分群和细胞亚群注释"
---

# 聚类、分群和细胞亚群注释

## 输入与适用范围

- 输入中指定需要分离的主要细胞类型`chosen_celltype`，如 Microglia、 Astrocyte
- 输入文件:
  - 主目录下存在项目配置文件`config.yaml`
  - `data/`目录下存在测序样本对应的注释文件`targets.csv`
  - 包含`counts`和`lognorm`两个layer的`.h5ad`文件路径，且可以读取为`anndata`对象
    - `obs`插槽中包含`major_celltype`列（指示主要细胞类型，例如Microglia）
    - `obs`插槽中同步了`targets.csv`中的分组信息，如列`sample`
  - 指定待修改的`.ipynb`代码文件路径和代码框插入位置
- 所有数据和输出路径均相对于分析项目的工作目录，不是本 skill 的目录。
- 预处理 skill 应已创建 `data/`、`figures/`、`figures/dotplot_{chosen_celltype}_subcluster/`、`result/DEG_by_{chosen_celltype}_subcluster/` 和 `results/{chosen_celltype}/`。本 skill 不创建目录；缺失时指出预处理尚未完成并停止执行。
- 适用于人和小鼠高通量测序数据的下游处理
- 未加说明的情况下默认为疾病变量是缺血性脑卒中

## 工作流程

1. 根据上述要求检查数据，并在指定位置插入一个markdown代码框

```
# 聚类、分群和细胞亚群注释
```

2. 将本 skill 的 [scripts/analysis.ipynb](scripts/analysis.ipynb) 插入上述标题之后。根据输入替换 `:/` 占位参数，列表占位符应替换为实际列表而非嵌套列表；移除生成后不再需要的模板说明。
   - `resolutions` 使用数值列表，`map_list` 自动生成。查看聚类图后，再从 `map_list` 中选择 `cluster_key_chosen`，不预设固定分辨率。
   - 亚群编号默认以 `chosen_celltype + "_"` 为前缀，可按项目修改 `cluster_prefix`。
   - 默认依次计算 Harmony 和 scVI，保留两种整合结果。后续 `use_rep="X_pca_harmony"` 默认使用 Harmony；若效果不佳，只需改为 `use_rep="X_scVI"`，从邻接图计算开始重跑降维、聚类、差异分析、注释和可视化，无需重新计算整合。邻接图和 t-SNE 使用同一个 `use_rep`，UMAP 使用相应的邻接图。
3. 逐步执行首次差异分析；在合并／删除亚群前，读取本次 `tag` 对应的差异表并结合聚类图判断是否需要调整。按证据填写 `merge_map` 和 `exclude_clusters`，默认均为空。若有调整，对修改后的 `adata` 重新差异分析；代码只清理本次标签的旧差异表，不清空目录。继续执行到 `### 选取marker基因` 前的 checkpoint 停止。
4. 分析 `result/DEG_by_{chosen_celltype}_subcluster` 下本次 `tag` 对应、属于最终亚群的 top100 差异表，重点关注 names、logfoldchanges、pct_nz_group、pct_nz_reference 四列，推断细胞亚群，将推断过程输出到 `results/{chosen_celltype}/`。
5. 根据注释结果完整替换 `genes` 和 `cluster2annotation`，执行 checkpoint 后的 marker 展示及注释保存代码，生成 `data/{config['project_code']}_{chosen_celltype}_annot.h5ad`。
6. 将 [scripts/plot.ipynb](scripts/plot.ipynb) 添加到生物学注释代码之后，替换细胞类型、类别列表和 marker 列表。按顺序执行加载与绘图代码框，以刚生成的 `_annot.h5ad` 为输入完成可视化。

## 输出与验收

- 修改后`.ipynb`文件
- 两个`.h5ad`文件:`data/{config['project_code']}_{chosen_celltype}.h5ad`和`data/{config['project_code']}_{chosen_celltype}_annot.h5ad`
- 生物学注释保存完成后，绘图代码能独立加载 `_annot.h5ad` 并生成亚群 marker 展示图。
