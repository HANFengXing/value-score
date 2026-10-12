# value-score

问答价值评分 skill v1.1，以及可复核的回答—评分数据集 v0.1.0。

| 数据部分 | 数量 | 标签来源 | 用途 |
|---|---:|---|---|
| HelpSteer2 完整 validation 子集 | 1,038 个回答，519 组问题 | NVIDIA 原始人工标签 | 对照 helpfulness / correctness / coherence 等原始维度 |
| 中文构造案例 | 64 个回答，32 组问题 | Codex 编写并自评，尚未人工复核 | 演示个人量表、缺证据区间、严重错误上限和成对比较 |

**公开人工标签没有转换为个人价值总分。中文分数不是独立人工金标准。本仓库尚未测得评分准确率。**

## 内容

```text
skills/value-score/       原样附带的已安装 skill v1.1
data/public/             英文问题回答与五项原始人工标签，分开保存
data/chinese/            中文问题回答、分维评分依据、成对记录与 CSV
data/helpsteer2-validation.source.jsonl.gz  固定版本的原始数据
scripts/                 重建、完整性检查、盲评导出
tests/                   计分、区间、上限和导出验证
reports/validation.json  数据校验结果；不是模型效果报告
docs/                    数据卡和字段说明
dataset_manifest.json    数量、版本、划分方式
THIRD_PARTY_NOTICES.md    公开数据署名和许可证
```

## 使用

需要 Python 3.10+，仅使用标准库，没有 API 密钥或模型调用。

```bash
python scripts/validate_dataset.py
python -m unittest discover -s tests -v
```

浏览中文分数可直接打开 `data/chinese/scores.csv`。完整证据在 `data/chinese/labels.jsonl`，各维等级是0–4整数，按 skill 原有脚本计算成0–100加权指数。

重建固定版本数据：

```bash
python scripts/construct_chinese.py
python scripts/build_dataset.py
python scripts/validate_dataset.py
```

默认使用已附带的原始压缩文件。`--refresh-source` 只重新下载固定提交版本，不随上游 main 漂移。

## 做一次真正的复核

```bash
python scripts/make_review_batch.py --collection chinese --split holdout --out review/chinese
python scripts/make_review_batch.py --collection public --split holdout --out review/public
```

1. 先只提供 `questions.blind.jsonl`，完成问题评分后保存记录。
2. 再提供 `answers.blind.jsonl`，完成回答评分。评审者不要预先查看本仓库标签或参考答案。
3. 中文标注可以用 `skills/value-score/scripts/score.py` 检查结构与计算。人工复核保存为新版本，保留旧标签和分歧。
4. 英文 public 模板只使用原始对应维度。填好整数0–4预测后运行：

```bash
python skills/value-score/scripts/benchmark.py data/public/labels.jsonl review/public/annotations.template.jsonl --out review/public/comparison.json
```

若只预测 holdout，上述报告会把 calibration 记录显示为缺失；应按 `split` 对原始标签作同样筛选，再报告 holdout 覆盖率。

真实 A/B 与 B/A 顺序复核尚未执行。成对偏好由本次自评分数派生，并保留平局和无法判断；不能用于证明去偏或独立准确性。

## Skill

`skills/value-score` 包含 `SKILL.md`、量表、评测协议、来源说明、计分脚本、公开维度对照脚本和代理配置。该目录是当前已安装版本的文件副本，没有修改本机全局配置。复用时将整个目录放入目标环境的技能目录，先保留已有版本。

详细说明：[数据卡](docs/DATASET_CARD.md)、[字段与评分](docs/SCHEMA.md)、[第三方署名](THIRD_PARTY_NOTICES.md)。
