# Multiple-Relations-Extraction-Only-Look-Once
Multiple-Relations-Extraction-Only-Look-Once. Just look at the sentence once and extract the multiple pairs of entities and their corresponding relations.
只用看一次，抽取所有的实体及其**对应的**所有关系。

**input**
```"text": "《逐风行》是百度文学旗下纵横中文网签约作家清水秋风创作的一部东方玄幻小说，小说已于2014-04-28正式发布"```

**output** 
```
"spo_list": [{"predicate": "连载网站", "object_type": "网站", "subject_type": "网络小说", "object": "纵横中文网", "subject": "逐风行"}, {"predicate": "作者", "object_type": "人物", "subject_type": "图书作品", "object": "清水秋风", "subject": "逐风行"}]
```

## Main principle
The entity extraction task is converted into a sequence annotation task, and the multi-relationship extraction task is converted into a [multi-head selection task](https://arxiv.org/abs/1804.07847).

把实体抽取任务转换成序列标注任务，把多关系抽取任务转换成[多头选择任务](https://arxiv.org/abs/1804.07847)。

The token will be sent to the model, and the model will predict the output label, predicate value, and predicate location.

token会被送入模型，模型会预测输出label、predicate value 和 predicate location。

```
+-------+-------+------------+----------------------+--------------------+
| index | token |   label    |   predicate value    | predicate location |
+-------+-------+------------+----------------------+--------------------+
|   0   |   《  |     O      |        ['N']         |        [0]         |
|   1   |   逐  | B-图书作品  | ['连载网站', '作者']  |      [12, 21]      |
|   2   |   风  | I-图书作品  |        ['N']         |        [2]         |
|   3   |   行  | I-图书作品  |        ['N']         |        [3]         |
|   4   |   》  |     O      |        ['N']         |        [4]         |
|   5   |   是  |     O      |        ['N']         |        [5]         |
|   6   |   百  |     O      |        ['N']         |        [6]         |
|   7   |   度  |     O      |        ['N']         |        [7]         |
|   8   |   文  |     O      |        ['N']         |        [8]         |
|   9   |   学  |     O      |        ['N']         |        [9]         |
|   10  |   旗  |     O      |        ['N']         |        [10]        |
|   11  |   下  |     O      |        ['N']         |        [11]        |
|   12  |   纵  |   B-网站   |        ['N']         |        [12]        |
|   13  |   横  |   I-网站   |        ['N']         |        [13]        |
|   14  |   中  |   I-网站   |        ['N']         |        [14]        |
|   15  |   文  |   I-网站   |        ['N']         |        [15]        |
|   16  |   网  |   I-网站   |        ['N']         |        [16]        |
|   17  |   签  |     O      |        ['N']         |        [17]        |
|   18  |   约  |     O      |        ['N']         |        [18]        |
|   19  |   作  |     O      |        ['N']         |        [19]        |
|   20  |   家  |     O      |        ['N']         |        [20]        |
|   21  |   清  |   B-人物   |        ['N']         |        [21]        |
|   22  |   水  |   I-人物   |        ['N']         |        [22]        |
|   23  |   秋  |   I-人物   |        ['N']         |        [23]        |
|   24  |   风  |   I-人物   |        ['N']         |        [24]        |
|   25  |   创  |     O      |        ['N']         |        [25]        |
...
+-------+-------+------------+----------------------+--------------------+
```

> [See my blog for more details](https://yuanxiaosc.github.io/2019/05/09/Multiple-Relations-Extraction-Only-Look-Once/)


## Document description

|name|description|
|-|-|
|bert|The feature extractor of the model, where BERT is used as the feature extractor for the model. It can be replaced with other feature extractors, such as BiLSTM and CNN.|
|bin/data_manager.py|Prepare formatted data for the mode.|
|bin/read_standard_format_data.py|View formatted data.|
|bin/test_head_select_scores.py|One method for solving the relationship extraction problem: the multi-head selection method.|
|raw_data||
|for_infer_out.py|Check model prediction results.|
|read_file.py|Read formatted data.|
|run_multiple_relations_extraction.py|The most basic model for model training and prediction.|
|run_multiple_relations_extraction_XXX.py|File under experiment.|

### Need Help
Well-designed loss function!

## Use example
### [2019语言与智能技术竞赛](http://lic2019.ccf.org.cn/kg)
#### 竞赛任务
给定schema约束集合及句子sent，其中schema定义了关系P以及其对应的主体S和客体O的类别，例如（S_TYPE:人物，P:妻子，O_TYPE:人物）、（S_TYPE:公司，P:创始人，O_TYPE:人物）等。 任务要求参评系统自动地对句子进行分析，输出句子中所有满足schema约束的SPO三元组知识Triples=[(S1, P1, O1), (S2, P2, O2)…]。
输入/输出:
(1) 输入:schema约束集合及句子sent
(2) 输出:句子sent中包含的符合给定schema约束的三元组知识Triples

**例子**
输入句子： ```"text": "《古世》是连载于云中书城的网络小说，作者是未弱"```

输出三元组： ```"spo_list": [{"predicate": "作者", "object_type": "人物", "subject_type": "图书作品", "object": "未弱", "subject": "古世"}, {"predicate": "连载网站", "object_type": "网站", "subject_type": "网络小说", "object": "云中书城", "subject": "古世"}]}```

#### 数据简介
本次竞赛使用的SKE数据集是业界规模最大的基于schema的中文信息抽取数据集，其包含超过43万三元组数据、21万中文句子及50个已定义好的schema，表1中展示了SKE数据集中包含的50个schema及对应的例子。数据集中的句子来自百度百科和百度信息流文本。数据集划分为17万训练集，2万验证集和2万测试集。其中训练集和验证集用于训练，可供自由下载，测试集分为两个，测试集1供参赛者在平台上自主验证，测试集2在比赛结束前一周发布，不能在平台上自主验证，并将作为最终的评测排名。


### Getting Started
#### Environment Requirements
+ python 3.6+
+ Tensorflow 1.12.0+

#### Step 1: Environmental preparation
+ Install Tensorflow 
+ Dowload [bert-base, chinese](https://storage.googleapis.com/bert_models/2018_11_03/chinese_L-12_H-768_A-12.zip), unzip file and put it in ```pretrained_model``` floader.

#### Step 2: Download the training data, dev data and schema files
Please download the training data, development data and schema files from [the competition website](http://lic2019.ccf.org.cn/kg), then unzip files and put them in ```./raw_data/``` folder.
```
cd data
unzip train_data.json.zip 
unzip dev_data.json.zip
cd -
```

**Data is only used for learning and communication!**

[百度网盘-2019语言与智能技术竞赛_信息抽取raw_data](https://pan.baidu.com/s/10-3iV9gR_-Lvxj9B6bSW2g)
+ 百度网盘提取码：链接：https://pan.baidu.com/s/10-3iV9gR_-Lvxj9B6bSW2g 
+ 提取码：hou4 

#### Step3: Data preprocessing
```
python bin/data_manager.py
```

#### Step4： Model training
```
python run_multiple_relations_extraction.py \
--task_name=SKE_2019 \
--do_train=true \
--do_eval=false \
--data_dir=bin/standard_format_data \
--vocab_file=pretrained_model/chinese_L-12_H-768_A-12/vocab.txt \
--bert_config_file=pretrained_model/chinese_L-12_H-768_A-12/bert_config.json \
--init_checkpoint=pretrained_model/chinese_L-12_H-768_A-12/bert_model.ckpt \
--max_seq_length=128 \
--train_batch_size=32 \
--learning_rate=2e-5 \
--num_train_epochs=3.0 \
--output_dir=./output_model/multiple_relations_model/epochs3/
```

### Step5: Model prediction
```
python run_multiple_relations_extraction.py \
  --task_name=SKE_2019 \
  --do_predict=true \
  --data_dir=bin/standard_format_data \
  --vocab_file=pretrained_model/chinese_L-12_H-768_A-12/vocab.txt \
  --bert_config_file=pretrained_model/chinese_L-12_H-768_A-12/bert_config.json \
  --init_checkpoint=output_model/multiple_relations_model/epochs3/model.ckpt-2000 \
  --max_seq_length=128 \
  --output_dir=./infer_out/multiple_relations_model/epochs3/ckpt2000
```

## Paper realization
This code is an unofficial implementation of [Extracting Multiple-Relations in One-Pass with Pre-Trained Transformers](https://arxiv.org/abs/1902.01030) and [Joint entity recognition and relation extraction as a multi-head selection problem](https://arxiv.org/abs/1804.07847).

## Main principle

![](https://yuanxiaosc.github.io/2019/05/02/%E4%BF%A1%E6%81%AF%E6%8A%BD%E5%8F%96%E4%BB%BB%E5%8A%A1%E7%9B%B8%E5%85%B3%E8%AE%BA%E6%96%87%E5%8F%91%E5%B1%95%E8%84%89%E7%BB%9C/model-2018Verga.png)
![](https://yuanxiaosc.github.io/2019/05/02/%E4%BF%A1%E6%81%AF%E6%8A%BD%E5%8F%96%E4%BB%BB%E5%8A%A1%E7%9B%B8%E5%85%B3%E8%AE%BA%E6%96%87%E5%8F%91%E5%B1%95%E8%84%89%E7%BB%9C/illustration-2019Wang.png)
