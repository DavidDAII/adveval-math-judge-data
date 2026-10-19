# 如何获取原始题目(MATH 数据集)

本仓库**不分发** MATH 数据集的题目原文与参考答案。请按下面任一方式自行获取,
再用 `data/problems_index.csv` 定位到本研究实际使用的那 300 道题。

---

## 方式一(推荐):HuggingFace

数据集仓库:`hendrycks/competition_math`(MATH 的官方镜像)

```python
# pip install datasets
from datasets import load_dataset
import pandas as pd, hashlib

ds = load_dataset("hendrycks/competition_math", split="test")
print(ds)                       # 查看各学科与条数
print(ds[0])                    # 字段: problem / solution / level / type

# 建成 {题面sha1: (subject, 行号)} 索引
idx = {}
for i, r in enumerate(ds):
    k = r["problem"].strip()
    idx[hashlib.sha1(k.encode()).hexdigest()[:16]] = (r.get("type", ""), i)
```

> HuggingFace 的 `competition_math` 与 Kaggle 镜像 `awsaf49/math-dataset` 的
> 文件命名不同。本研究的 `filename` 字段(如 `1003.json`)来自 **Kaggle 镜像**的文件名,
> 因此**校验请以题面 SHA-1 为准**(见下),不要依赖文件名。

---

## 方式二:Kaggle 镜像(与本项目 `filename` 一致)

```bash
pip install kagglehub
python - <<'PY'
import kagglehub
p = kagglehub.dataset_download("awsaf49/math-dataset")
print(p)          # .../MATH/test/{algebra,geometry,...}/*.json
PY
```

目录结构:

```
MATH/test/
├── algebra/1003.json, 1004.json, ...
├── geometry/...
├── intermediate_algebra/...
├── number_theory/...
├── prealgebra/...
└── precalculus/...
```

每个 JSON 的字段为 `problem` / `solution` / `level` / `type`。
**文件名(去掉 `.json`)即本研究 `filename` 字段的前缀**,二者可直接对应。

---

## 校验:确认你取到的题与本研究一致

```python
import csv, hashlib, json, os

def sha1_16(s):
    return hashlib.sha1(s.strip().encode("utf-8")).hexdigest()[:16]

# 1) 读映射表,只看评测集
rows = [r for r in csv.DictReader(open("data/problems_index.csv", encoding="utf-8-sig"))
        if r["in_eval300"] == "yes"]
print(f"评测集共 {len(rows)} 道,期望 300")

# 2) 从你下载的 MATH 目录取题,逐题校验
MATH = "/path/to/MATH/test"          # ← 改成你的路径
ok = miss = 0
for r in rows:
    p = os.path.join(MATH, r["subject"], r["filename"])
    if not os.path.exists(p):
        miss += 1
        continue
    if sha1_16(json.load(open(p, encoding="utf-8"))["problem"]) == r["problem_sha1"]:
        ok += 1
    else:
        miss += 1
print(f"校验通过 {ok}/{len(rows)},不匹配或缺失 {miss}")
```

**期望输出**:`校验通过 300/300,不匹配或缺失 0`。若出现不匹配,说明取到的数据集版本不同,
请改用 Kaggle 镜像的对应版本。

---

## 评测集的学科分布(用于交叉核对)

| 学科 | 题数 |
|---|---|
| geometry | 162 |
| algebra | 138 |
| **合计** | **300** |

---

## 本仓库如何处理第三方内容

| 内容 | 处理 |
|---|---|
| 题目原文 `problem` | **已从全部公开文件中删除** |
| 参考答案长文本 `correct_solution` | **已删除** |
| 判定记录中的 `question` / `reference_answer` | **已删除** |
| 正确答案短值 `correct_answer` / 错误答案 `wrong_answer` | **保留**(判定等价性所需,且为短值) |
| 题目定位 | 通过 `subject` + `filename` + `problem_sha1` 三元组 |

> 已校验:公开目录内 **无任何 `problem` / `correct_solution` / `question` / `reference_answer` 残留**。
