# AI4Science: 材料带隙预测项目

本项目使用机器学习方法，基于材料的化学式和费米能级（`efermi`）预测材料的能隙（`band_gap`）。项目使用 **Materials Project** 数据库和 **Matminer** 特征化工具，结合 **随机森林（Random Forest）回归模型**，完成从数据下载、特征提取、数据集划分、模型训练到预测的完整流程。

---

## 🔹 项目结构
```
AI4Science/
│
├─ data/
│ ├─ download.py # 下载材料数据
│ ├─ feature.py # 特征工程
│ ├─ split.py # 数据集划分
│ └─ raw_materials.pkl # 下载的原始数据
│
├─ models/
│ ├─ test.py # 模型训练脚本
│ ├─ baseline_rf.pkl # 训练好的随机森林模型
│ └─ feature_columns.pkl # 训练特征列顺序
│
├─ predict/
│ └─ predict.py # 输入化学式+efermi预测带隙
│
└─ README.md
```

---

## 🔹 工作流程说明

### 1. 数据下载（`download_materials_data.py`）
- 使用 **Materials Project API** 下载材料数据，包括：
  - `material_id`、`formula`、`structure`、`band_gap`、`efermi`  
- 保存为二进制文件 `raw_materials.pkl`。

### 2. 特征工程（`feature_engineering.py`）
- 将化学式转换为 `Composition` 对象  
- 使用 **Matminer 的 Magpie 特征** 提取材料属性，包括：
  - 元素平均属性（如原子量、电子亲和能等）  
  - 这些数值化特征构成模型的输入  
- 生成 `features.pkl` 保存所有特征

> 💡 提示：目前特征仅从化学式提取，后续可结合结构、电子态密度、声子谱等提取更多前沿特征。

### 3. 数据集划分（`split_dataset.py`）
- 目标变量：`band_gap`  
- 输入特征：Magpie 特征 + `efermi`  
- 划分训练集 / 验证集 / 测试集（比例 70% / 15% / 15%）  
- 保存为：
  - `X_train.pkl`, `y_train.pkl`  
  - `X_val.pkl`, `y_val.pkl`  
  - `X_test.pkl`, `y_test.pkl`  
- 同时保存训练特征列顺序 `feature_columns.pkl`（保证预测时列顺序一致）

### 4. 模型训练（`train_baseline.py`）
- 使用 **随机森林回归模型**  
- 输入训练特征（Magpie + efermi）和目标 `band_gap`  
- 训练完成后：
  - 保存模型为 `baseline_rf.pkl`  
  - 保存训练特征列顺序为 `feature_columns.pkl`

### 5. 预测（`predict/predict.py`）
- 输入：
  - 材料化学式（如 `LiFePO4`）  
  - 对应的 `efermi`  
- 生成 Magpie 特征 + `efermi`  
- 按训练时特征列顺序排列 → 传入模型预测 `band_gap`  
- 输出预测结果（eV）

---

## 🔹 特征处理流程图
```
┌─────────────┬─────────────┬───────┬────────────┐
│ Magpie_x1   │ Magpie_x2   │ ...   │ efermi     │
├─────────────┼─────────────┼───────┼────────────┤
│ 2.31        │ 1.83        │ ...   │ 5.0        │  ← 材料1
│ 3.14        │ 2.01        │ ...   │ 4.8        │  ← 材料2
│ 1.97        │ 1.66        │ ...   │ 5.3        │  ← 材料3
│ ...         │ ...         │ ...   │ ...        │  ← ...
└─────────────┴─────────────┴───────┴────────────┘
这里的Magpie特征包括原子量,电负性等特征。
```
```text
 化学式 + efermi
        │
        ▼
  Composition 对象
        │
        ▼
   Magpie 特征提取
        │
        ▼
  特征矩阵（数值化）
        │
        ▼
  补齐 efermi 列（预测时使用）
        │
        ▼
  按训练顺序排列列
        │
        ▼
    模型预测 band_gap
```