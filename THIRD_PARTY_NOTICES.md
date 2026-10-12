# 第三方来源与许可

## NVIDIA HelpSteer2

- 作者：NVIDIA。
- 数据卡：https://huggingface.co/datasets/nvidia/HelpSteer2
- 论文：HelpSteer2: Open-source dataset for training top-performing reward models，https://arxiv.org/abs/2406.08673
- 许可证：[Creative Commons Attribution 4.0 International](https://creativecommons.org/licenses/by/4.0/)。
- 固定来源：https://huggingface.co/datasets/nvidia/HelpSteer2/resolve/990b2711a36180dd19d9c94b8627844866f8982a/validation.jsonl.gz
- 变更：解压完整validation数据；prompt、response和五项人工标签原样保留；将文本和标签分开，加入id、来源字段和本包划分。没有获得NVIDIA对本包的背书。

来源许可适用于原始公开数据及其再分发部分。中文构造内容、新增代码和已有skill的许可应分别理解；本包没有替原先未附许可证的skill擅自添加新的许可声明。

## 已安装 value-score skill

`skills/value-score` 是用户要求附带的当前本地skill v1.1副本，包含其原有来源说明。内容未修改。`references/evaluation.md` 中“本次12条对照”描述原skill制作时的历史工作，本次数据集增加了完整1,038条公开validation记录；历史陈述不表示已对新记录重做模型评分。


## 新增本地模型

英文模型参数由上述NVIDIA HelpSteer2固定版本数据训练而来，保留原数据CC-BY-4.0署名及来源；训练和JSON/NPZ转换为本次修改。中文参数由已明确标注的构造自评标签训练，不是独立人工金标准。第三方库未打包，仅在requirements-model.txt声明依赖。
