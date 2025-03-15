# jieba使用

> Python 中文分词组件
>
> 支持四种分词模式：
>
> - 精确模式，试图将句子最精确地切开，适合文本分析；
> - 全模式，把句子中所有的可以成词的词语都扫描出来, 速度非常快，但是不能解决歧义；
> - 搜索引擎模式，在精确模式的基础上，对长词再次切分，提高召回率，适合用于搜索引擎分词。
> - paddle模式，利用PaddlePaddle深度学习框架，训练序列标注（双向GRU）网络模型实现分词。同时支持词性标注。

## 安装

```shell
pip install jieba
```

## 分词方法

### `jieba.cut()`

给定中文字符串，分解后返回一个迭代器，需要用for循环访问。

+ `sentence`： 需要分词的字符串
+ `cut_all=False`：用来控制是否采用全模式
  + True 全模式：把句子中所有的可以成词的词语都扫描出来, 会出现一词多用、一词多意。
  + False 精确模式：将句子最精确的切分开，每个词都只有一种含义。
+ `HMM=True`：用来控制是否使用 HMM 模型
  + 该方法适合用于搜索引擎构建倒排索引的分词，粒度比较细。
+ `use_paddle=False`
  + 用来控制是否使用paddle模式下的分词模式,会更加精确，使用这个的前提是你需要先安装paddlepaddle-tiny。

---

### `jieba.cut_for_search()`

该方法和cut一样，分解后返回一个迭代器，需要用for循环访问。不过它是搜索引擎模式，在精确模式的基础上，对长词再次切分，提高召回率，适合用于搜索引擎分词。

+ `sentence`：需要分词的字符串
+ `HMM=True`：用来控制是否使用 HMM 模型

---

### `jieba.lcut()`

和`jieba.cut`使用方法一样，不过返回的是列表。

`cut` 和 `cut_for_search` 方法都是支持繁体字的。

## 自定义词典

开发者可以指定自己自定义的词典，以便包含 jieba 词库里没有的词。虽然 jieba 有新词识别能力，但是自行添加新词可以保证更高的正确率

### 词典格式

一个词占一行，并分未三部分：词语、词频（可省略）、词性（可省略）

```
创新办 3 i
云计算 5
凱特琳 nz
台中
```

### 载入词典

```python
jieba.load_userdict(file_name) # file_name 为文件类对象或自定义词典的路径
```

### 调整词典

+ `add_word(word, freq=None, tag=None)`：添加到词典
+ `del_word(word)`：从词典删除
+ `suggest_freq(segment, tune=True)` 可调节单个词语的词频，使其能（或不能）被分出来
  + 如果 `segment` 为数组，则期望分词时将词语分开
  + 如果 `segment` 为字符串，则期望分词时将其作为一个整体

## 关键词分析

### 基于 TF-IDF 算法的关键词抽取

- `jieba.analyse.TFIDF(idf_path=None)`：新建 TFIDF 实例
  - `idf_path` 为 IDF 频率文件
- ` jieba.analyse.set_idf_path(file_name)`：设置关键词提取所使用逆向文件频率（IDF）文本语料库
- ` jieba.analyse.set_stop_words(file_name)`：关键词提取所使用停止词（Stop Words）文本语料库
- `jieba.analyse.extract_tags(sentence, topK=20, withWeight=False, allowPOS=())`：提取关键字
  - `sentence`：为待提取的文本
  - `topK`：为返回几个 TF/IDF 权重最大的关键词，默认值为 20
  - `withWeight`：为是否一并返回关键词权重值，默认值为 False
  - `allowPOS`：仅包括指定词性的词，默认值为空，即不筛选

### 基于 TextRank 算法的关键词抽取

+ `jieba.analyse.TextRank()`：新建自定义 TextRank 实例
+ `jieba.analyse.textrank(sentence, topK=20, withWeight=False, allowPOS=())`

**基本思想**

1. 将待抽取关键词的文本进行分词
2. 以固定窗口大小(默认为5，通过span属性调整)，词之间的共现关系，构建图
3. 计算图中节点的PageRank，注意是无向带权图

## 词性标注

+ `jieba.posseg.POSTokenizer(tokenizer=None)`： 新建自定义分词器
  + `tokenizer` 参数可指定内部使用的 `jieba.Tokenizer` 分词器。
  + `jieba.posseg.dt` 为默认词性标注分词器。
+ `jieba.posseg.cut(sentence, HMM=True, use_paddle=False)`

paddle模式词性和专名类别标签集合如下表，其中词性标签 24 个（小写字母），专名类别标签 4 个（大写字母）。

| 标签 | 含义     | 标签 | 含义     | 标签 | 含义     | 标签 | 含义     |
| ---- | -------- | ---- | -------- | ---- | -------- | ---- | -------- |
| n    | 普通名词 | f    | 方位名词 | s    | 处所名词 | t    | 时间     |
| nr   | 人名     | ns   | 地名     | nt   | 机构名   | nw   | 作品名   |
| nz   | 其他专名 | v    | 普通动词 | vd   | 动副词   | vn   | 名动词   |
| a    | 形容词   | ad   | 副形词   | an   | 名形词   | d    | 副词     |
| m    | 数量词   | q    | 量词     | r    | 代词     | p    | 介词     |
| c    | 连词     | u    | 助词     | xc   | 其他虚词 | w    | 标点符号 |
| PER  | 人名     | LOC  | 地名     | ORG  | 机构名   | TIME | 时间     |

## 并行分词

原理：将目标文本按行分隔后，把各行文本分配到多个 Python 进程并行分词，然后归并结果，从而获得分词速度的可观提升

基于 python 自带的 multiprocessing 模块，目前暂不支持 Windows

```python
jieba.enable_parallel(4) # 开启并行分词模式，参数为并行进程数
jieba.disable_parallel() # 关闭并行分词模式
```

## Tokenize

用于返回词语在原文的起止位置，注意，输入参数只接受 unicode，和 `cut` 方法一样，有两种模式

+ 默认模式

```python
result = jieba.tokenize(u'永和服装饰品有限公司')
for tk in result:
    print("word %s\t\t start: %d \t\t end:%d" % (tk[0],tk[1],tk[2]))
```

```
word 永和                start: 0                end:2
word 服装                start: 2                end:4
word 饰品                start: 4                end:6
word 有限公司            start: 6                end:10
```

+ 搜索模式

```python
result = jieba.tokenize(u'永和服装饰品有限公司', mode='search')
for tk in result:
    print("word %s\t\t start: %d \t\t end:%d" % (tk[0],tk[1],tk[2]))
```

```
word 永和                start: 0                end:2
word 服装                start: 2                end:4
word 饰品                start: 4                end:6
word 有限                start: 6                end:8
word 公司                start: 8                end:10
word 有限公司            start: 6                end:10
```

## ChineseAnalyzer for Whoosh

- `from jieba.analyse import ChineseAnalyzer`
- Example: https://github.com/fxsjy/jieba/blob/master/test/test_whoosh.py
