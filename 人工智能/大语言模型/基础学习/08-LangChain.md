# LangChain
## LCEL

基本流程如下：

prompt | model | output_parser

需要注意的是：

+ prompt 是 `BasePromptTemplate` 类型，它的 `format_prompt()` 方法可以输出 `PromptValue`
+ model 是 `BaseChatModel` 类型，使用 `PromptValue` 等作为输入，输出 `BaseMessage`
+ output_parser 是 `BaseOutputParser` 类型，使用 `str` 、 `BaseMessage` 等作为输入，输出字符串



## 模型输入输出

### Prompts

> **提示(prompt)** 指的是输入模型的内容，LangChain 提供了多个类和函数，使构建和处理提示变得简单。
>
> + 提示模板(PromptTemplate): 为模型输入添加参数
> + 示例选择器(ExampleSelector): 动态选择在提示中包含的示例

#### PromptTemplate

创建方法有两种：

+ `PromptTemplate(input_variables=[。。。], template="xxx")`：需要手动同时指定变量名以及模板
+ `PromptTemplate.from_template(template)`：无需手动指出变量名

```python
from langchain import PromptTemplate

template = """/
You are a naming consultant for new companies.
What is a good name for a company that makes {product}?
"""
# 方法1
prompt = PromptTemplate(input_variables["product"], template=template)

# 方法2
prompt = PromptTemplate.from_template(template)

# 可以通过属性 input_variables 获取变量列表
prompt_template.input_variables

# 填充变量值
prompt.format(product="colorful socks")
```

#### 角色Prompt

下面这三种 PromptTemplate 构建的消息，发送给 LLM 会与特定的角色进行关联

+ `SystemMessagePromptTemplate`
+ `AIMessagePromptTemplate`
+ `HumanMessagePromptTemplate`

此外

+ `ChatPromptTemplate`
  + `from_messages([xxxMessagePromptTemplate])` ：可以将不同角色（AI、System、Human）的对话组合起来
  + `from_template(role="xxx", template="xxx")`：可以创建自定义角色prompt

Prompt 相当于未成形的 Message



### 语言模型

> LangChain提供了两种类型模型的接口和集成:
>
> + LLMs: 输入一个文本字符串并返回一个文本字符串的模型
> + 聊天模型: 由语言模型支持的模型，接受一个聊天消息列表作为输入并返回一个聊天消息



### 输出解析器

语言模型输出文本。但很多时候，您可能希望获得比仅文本更结构化的信息。这就是输出解析器的作用。

输出解析器必须实现两个主要方法:

- `get_format_instructions`：获取格式化指令，一个返回包含语言模型输出应如何格式化的字符串的方法。
- `parse`：解析，一个接受字符串（假设为语言模型的响应）并将其解析为某种结构的方法。


## 检索

### 文档加载器

一般命名规则为 `xxxLoader`，使用时先实例化 Loader，然后调用对象的 `load()` 方法

+ `TextLoader`：加载普通文本

  ```python
  from langchain_community.document_loaders import TextLoader
  
  loader = TextLoader("./index.md")
  loader = TextLoader(file_path="./index.md")
  loader.load()
  ```
+ `CSVLoader`：加载 CSV 数据

  ```python
  from langchain_community.document_loaders import CSVLoader
  
  loader = CSVLoader("./index.csv", encoding='utf-8')
  print(loader.load())
  ```
+ `DirectoryLoader`：加载文件夹下的子文件


  ```python
  from langchain_community.document_loaders import DirectoryLoader

  loader = DirectoryLoader(
    '../',
    glob="**/*.md",           # 检索规则
    show_progress=True,       # 显示进度条
    use_multithreading=True,  # 使用多线程
    loader_cls=TextLoader,    # 文档加载器，默认 `UnstructuredLoader`
    silent_errors=True,       # 加载错误不报错
    loader_kwargs={'autodetect_encoding': True} # 自动检测编码
  )
  docs = loader.load()
  print(len(docs))
  ```
+ `JSONLoader`：加载 json 文件
  ```python
  from langchain_community.document_loaders import JSONLoader
  
  loader = JSONLoader(  
	  file_path='./example_data/facebook_chat.json',
	  jq_schema='.messages[].content')  
  data = loader.load()
  ```
+ `PyPDFLoader`：加载 PDF 文件
  ```python
  from langchain_community.document_loaders import PyPDFLoader
  loader = PyPDFLoader("example_data/layout-parser-paper.pdf")  
  pages = loader.load_and_split()
  ```
	
	+ 通过 `load_and_split()` 方法加载的数据，可以通过页面号码索引文档。

### 文档分割器
基类 `TextSplitter` 主要提供以下方法
+ `split_text(text: str) -> List[str]`
+ `create_documents(texts: List[str]) -> List[Document]`
+ `split_documents(documents: Iterable[Document]) -> List[Document]`

（1）`RecursiveCharacterTextSplitter`

该文本分割器接受一个字符列表。它尝试根据第一个字符进行分割来创建块，但如果任何块太大，则继续移动到下一个字符，依此类推。默认情况下，它尝试进行分割的字符是 `["\n\n", "\n", " ", ""]`。
```python  
from langchain.text_splitter import RecursiveCharacterTextSplitter

loader = TextLoader(r"./README.md", encoding='utf-8')  
  
text_splitter = RecursiveCharacterTextSplitter(  
    length_function=len,  
    chunk_size=8,  
    chunk_overlap=3,  
    add_start_index=True  
)  
  
texts = text_splitter.create_documents([doc.page_content for doc in loader.load()])  
for t in texts:  
    print(t)
```
+ `length_function`：计算块长度的函数
+ `chunk_size`：块的最大大小
+ `chunk_overlap`：块之间的最大重叠
+ `add_start_index`：是否在元数据中包含块在原始文档的起始位置

（2）`CharacterTextSplitter`

```python
from langchain.text_splitter import CharacterTextSplitter  
text_splitter = CharacterTextSplitter(  
separator =  "\n\n",  
chunk_size =  1000,  
chunk_overlap =  200,  
length_function =  len,  
)
```

（3）`CodeTextSplitter`

借助于 `RecursiveCharacterTextSplitter` 的静态函数 `from_language`

```python
python_splitter = RecursiveCharacterTextSplitter.from_language(  
	language=Language.PYTHON, chunk_size=50, chunk_overlap=0  
)
```

（4）`MarkdownHeaderTextSplitter`

有时在分割文本时，会同时考虑整体上下文和文本中句子和短语之间的关系。这可能会得到更全面的向量表示，捕捉文本的更广泛的含义和主题。

```python
from langchain.text_splitter import MarkdownHeaderTextSplitter

markdown_document =  "# Foo\n\n ## Bar\n\nHi this is Jim\n\nHi this is Joe\n\n ### Boo \n\n Hi this is Lance \n\n ## Baz\n\n Hi this is Molly"  
  
headers_to_split_on =  [  
("#",  "Header 1"),  
("##",  "Header 2"),  
("###",  "Header 3"),  
]  
  
markdown_splitter = MarkdownHeaderTextSplitter(headers_to_split_on=headers_to_split_on)  
md_header_splits = markdown_splitter.split_text(markdown_document)  
for split in md_header_splits:  
	print(split)

# Output
{'content': 'Hi this is Jim \nHi this is Joe', 'metadata': {'Header 1': 'Foo', 'Header 2': 'Bar'}}  
{'content': 'Hi this is Lance', 'metadata': {'Header 1': 'Foo', 'Header 2': 'Bar', 'Header 3': 'Boo'}}  
{'content': 'Hi this is Molly', 'metadata': {'Header 1': 'Foo', 'Header 2': 'Baz'}}
```

### 文本嵌入模型

基类 `Embeddings` 提供4个方法，以下是其中的两个关键方法（其他两个都是相对应的异步方法）
+ `embed_documents(texts: List[str]) -> List[List[float]]`：嵌入多个文本
+ `embed_query(text: str) -> List[float]`：嵌入单个文本

```python
from langchain_community.embeddings import OpenAIEmbeddings  

embeddings = OpenAIEmbeddings()

text = "This is a test document."

query_result = embeddings.embed_query(text)
doc_result = embeddings.embed_documents([text])
```

### 向量存储器

基类 `VectorStore` 主要使用以下方法：
+ `similarity_search(query: str) -> List[Document]`：通过文本搜索
+ `similarity_search_by_vector(embedding: List[float]) -> List[Document]`：通过向量进行搜索
+ `max_marginal_relevance_search`：最大边界搜索（MMR），文本搜索
+ `max_marginal_relevance_search_by_vector`：向量搜索

### 检索器

基类 `BaseRetriever`

#### VectorstoreIndexCreator

使用索引可以使用 `VectorstoreIndexCreator` 一句代码就可以创建索引

```python
from langchain_community.document_loaders import TextLoader
loader = TextLoader('./1.txt', encoding='utf8')

# 一句代码创建索引
index = VectorstoreIndexCreator(
    embedding=...,
    vectorstore=...
).from_loaders([loader])
```

`VectorstoreIndexCreator` 内部封装了很多，主要会经历以下主要步骤：

（1）将文档拆分成块

```python
documents = loader.load()

from langchain.text_splitter import CharacterTextSplitter
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
texts = text_splitter.split_documents(documents)
```

（2）创建嵌入模型

```python
from langchain_community.embeddings import OpenAIEmbeddings
embeddings = OpenAIEmbeddings()
```

（3）创建向量数据库

```python
from langchain.vectorstores import Chroma
db = Chroma.from_documents(texts, embeddings)
```

（4）创建索引

```python
retriever = db.as_retriever()
```

（5）创建链进行问答

```python
from langchain.chains import RetrievalQA
qa = RetrievalQA.from_chain_type(llm=OpenAI(), chain_type="stuff", retriever=retriever)

query = "What did the president say about Ketanji Brown Jackson"
qa.run(query)
```

## BaseTool

将 BaseTool 转为标准 openai 格式的 json

```python
from langchain_core.utils.function_calling import convert_to_openai_tool
import json

tools = ToolFactory.get_tools()

for tool in tools:
    print(json.dumps(convert_to_openai_tool(tool), indent=2))
```



