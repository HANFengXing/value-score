# value-score

问答价值评分技能1.2、回答—评分数据集0.1.0，以及已训练的本地模型1.0.0，合并发布为分发包0.2.0。

[下载最新完整包](https://github.com/HANFengXing/value-score/blob/master/value-score-v0.2.0.zip) · [模型与评测说明](MODEL_CARD.md) · [实际训练报告](training_report.json) · [中文标签预览](scores.csv) · [数据卡](DATASET_CARD.md)

## 已交付

- 1038条HelpSteer2原始人工评分回答，519个问题；固定来源版本与CC-BY-4.0署名。
- 64条中文构造回答、32个问题，附量表分维等级、证据、缺证据区间和改进方向；这些是作者自评，未人工复核。
- 已训练TF-IDF + Ridge模型权重：英文三个原始人工维度、中文问答八个个人量表维度；严重错误复核信号另用逻辑回归。
- 技能1.2接入本地模型入口，原量表1.1、权重、未知区间和错误上限保留。
- 训练代码、模型文件、逐维留出评测、按组划分审计和15项测试。

英文有用性和正确性没有优于常数基线，中文问题预测也未改善。中文回答四维平均MAE 0.897，基线1.195，但只对14条构造自评回答评测。模型当前用于实验性辅助复核，不宣称独立人工准确率或自动事实核验。详见模型卡。

## 下载运行

仓库通过完整ZIP保留目录结构，先下载再解压。需要Python3.11；原有计分和数据检查只用标准库，模型需要requirements-model.txt中的依赖。

```powershell
python -m zipfile -e value-score-v0.2.0.zip value-score-package
cd value-score-package
python -m pip install -r requirements-model.txt
python skills/value-score/scripts/score.py examples/predict-input.json --predict --kind answer
python skills/value-score/scripts/score.py examples/predict-input.json --predict --kind question
```

输出提供`dimension_predictions`和中文`provisional_uncapped_index`；它们未经内容核验。`verified_result`初始为待评范围0–100，正式等级由证据记录决定。检查证据后，原命令`python skills/value-score/scripts/score.py evaluation.json`照常计算。

## 重训与检查

```powershell
python scripts/train_model.py
python scripts/validate_dataset.py
python -m unittest discover -s tests -v
```

训练仅使用calibration，三折按问题/相关目标家族隔离，留出预测先保存后评测。输入只有context/question/answer，参考答案与标签不进入特征。未知维度不作为0分训练，不将公共标签映射为个人价值总分，也不根据模型信号确认严重错误。

## 包内结构

```text
skills/value-score/          技能1.2、量表1.1、计分和本地预测入口
  models/baseline-v1/        已训练的JSON词表、元数据、NPZ数值权重
data/public/                英文问题回答与原始五项人工标签
data/chinese/               中文问答、自评分维依据与成对记录
scripts/train_model.py      真正训练与保存模型；无外部API
reports/model/              盲留出预测、训练报告与划分审计
examples/                   可运行的预测输入
tests/                      计分、数据隔离、未知标签、权重复载与证据边界检查
docs/                       数据卡和字段说明
requirements-model.txt      此次训练的精确依赖版本
```

本地技能接入时复制包内整个`skills/value-score`目录，包括models，先备份已有版本。技能辅助评分只使用当前授权材料，不读取其他聊天，不自动记录长期日志。

旧版[value-score-v0.1.0.zip](https://github.com/HANFengXing/value-score/blob/master/value-score-v0.1.0.zip)保留数据与原skill1.1。数据本体仍是0.1.0；新增的是训练代码、模型与技能接入。
