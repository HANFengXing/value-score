# value-score 本地模型 v1.0.0

这是已实际训练并导出权重的CPU文本模型，采用TF-IDF + Ridge分维回归；严重错误复核信号用逻辑回归。没有微调大语言模型。

技能版本1.2，个人量表仍为1.1；模型版本1.0.0，分发包0.2.0。模型只能提供实验性预评分，不能核验事实、确认严重错误或证明用户收益。

## 训练和评测

| 部分 | 训练 | 留出 | 标签 |
|---|---:|---:|---|
| 英文回答 | 832回答 / 416问题 | 206回答 / 103问题 | HelpSteer2原始人工标签 |
| 中文回答 | 50回答 / 25问题 | 14回答 / 7问题 | 同作者构造并自评，未人工复核 |
| 中文问题 | 25个独立问题 | 7个独立问题 | 同作者自评，未知标签剔除 |

相关中文目标家族不跨集合，问题训练不重复使用两个回答。固定原数据划分、随机种子20261012，三折按组交叉验证仅在训练部分选择正则化参数，词表与IDF在各折重新学习。保存留出预测后，再取留出标签评测。参考答案、标签、作者、ID、来源和回答顺序不作为特征。

以下是0–4级的MAE，越低越好；cap_risk为0/1错误复核信号。常数基线只使用训练标签中位数，没有看留出标签。

| 模型 / 维度 | 留出有效标签 | 模型MAE | 常数基线MAE |
|---|---:|---:|---:|
| public_answer / helpfulness | 206 | 1.058 | 1.015 |
| public_answer / correctness | 206 | 1.024 | 0.990 |
| public_answer / coherence | 206 | 0.345 | 0.350 |
| chinese_question / importance | 6 | 0.167 | 0.167 |
| chinese_question / unknown | 6 | 0.333 | 0.333 |
| chinese_question / leverage | 6 | 0.167 | 0.167 |
| chinese_question / researchability | 7 | 0.571 | 0.571 |
| chinese_answer / alignment | 14 | 0.929 | 1.071 |
| chinese_answer / reliability | 13 | 1.231 | 1.923 |
| chinese_answer / gain | 14 | 1.214 | 1.571 |
| chinese_answer / efficiency | 14 | 0.214 | 0.214 |
| chinese_answer / cap_risk | 14 | 0.143 | 0.214 |

英文有用性、正确性未优于常数基线，连贯性只有极小差异。中文问题预测与基线相同，关键未知训练标签只有一个等级，不能用低误差宣称学到了问题价值。中文回答四维平均MAE为0.897，基线1.195；这是14条作者自评标签的一致性，不能称为独立人工准确率。

完整逐维精确一致率、相差不超过一级、二次加权κ、分布、偏差、按独立问题/目标家族bootstrap区间见[training_report.json](training_report.json)，包内也保留盲预测与划分审计。不把接近常数的高一致率当成理解能力。没有真实AB/BA或长期用户结果记录。

## 运行与接入

完整分发包的`skills/value-score/models/baseline-v1/`包含model.json、vocabulary.json和weights.npz。NPZ使用allow_pickle=False，入口检查SHA256；无外部API调用。

```powershell
python skills/value-score/scripts/score.py examples/predict-input.json --predict --kind answer
python skills/value-score/scripts/score.py examples/predict-input.json --predict --kind question
python scripts/train_model.py
python -m unittest discover -s tests -v
```

预测输出含分维建议、中文未应用错误上限的预评分、复核信号。正式等级初始全部unknown，verified_result为待评范围0–100；先核验材料，再用原计分器计算正式分数。严重错误信号不是校准概率，也不自动触发39/49分上限。

中文优先用于提示复核；英文模型仅用于明确请求的预测/对照。正式评分遵循原证据与区间规则。技能仅使用当前授权的问答，不读取其他聊天，不保存长期日志。

## 适用边界

词法模型易学习风格、措辞和主题，无法验证复杂推理或事实。英文832条训练样本很少，中文仅50条作者自评回答，缺独立中文人工标签。对新领域、新语言、长文本及事实细节的外推不可靠。现有留出结果已公开，后续根据它调参时必须用新增独立标签验证，不能反复刷这一组分数。

复用应保留模型建议与最终证据评分的分歧；优先补充中文独立人工标签、真实目标背景，以及正确/流畅错误/缺证据/篇幅变化的配对案例。

## 来源与依赖

数据归属NVIDIA，[HelpSteer2固定版本](https://huggingface.co/datasets/nvidia/HelpSteer2/tree/990b2711a36180dd19d9c94b8627844866f8982a)，CC-BY-4.0。第三方数据、派生英文模型与署名见THIRD_PARTY_NOTICES.md。中文标签与其模型是构造自评，不冒充人工来源。

已验证Python3.11.7、numpy1.26.4、scipy1.11.4、scikit-learn1.2.2。方法参考[TF-IDF文档](https://scikit-learn.org/1.2/modules/generated/sklearn.feature_extraction.text.TfidfVectorizer.html)与[Ridge文档](https://scikit-learn.org/1.2/modules/generated/sklearn.linear_model.Ridge.html)。
