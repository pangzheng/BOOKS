# Elasticsearch：如何部署NLP-文本embedding 和向量搜索
本文将介绍一个使用文本 embedding 模型生成文本内容的向量表示并演示对生成的向量进行向量相似性搜索的示例。

- 第一步

	将在 Elasticsearch 上部署一个 embedding 模型，并在摄取管道中使用它来从文本文档生成 embedding。
- 第二步

	然后，将展示如何在向量相似性搜索中使用这些 embedding 来查找给定查询的语义相似文档

向量相似性搜索 ([vector similarity search](https://www.elastic.co/blog/text-similarity-search-with-vectors-in-elasticsearch)) ，或者通常称为语义搜索，超越了传统的基于关键字的搜索，允许用户找到可能没有任何共同关键字的语义相似的文档，从而提供更广泛的结果。向量相似性搜索对密集向量进行操作，并使用 `k-最近邻 (k-nearest neighbour) ` 简称 knn ，搜索来查找相似向量。为此，首先需要使用文本 embedding 模型将文本形式的内容转换为其数字向量表示.

我们将使用来自 [MS MARCO Passage Ranking Task](https://microsoft.github.io/msmarco/#ranking) 的公共数据集进行演示。它由来自 MicrosoftBing 搜索引整的真实问题和人工生成的答案组成。该数据集是测试向量相似性搜索的完美资源。

- 首先，因为问答是向量搜索最常见的用例之一
- 其次，MS MARCO 排行榜中的顶级论文以某种形式使用了向量搜索。

在示例中，我们将使用此数据集的样本，使用模型 生成文本 embedding ，然后对其运行向量搜索.我们还希望对向量搜索产生的结果的质量进行快速验证。在今天的展示中，我将使用 Elastic Stack8.2 来进行展示。
针对一个向量搜索来说，它的架构可以表述为如下:

- 向量数据库 embedding 过程

	![](./pic/es-vector.png)
- es embedding 过程

	![](./pic/es-vector1.png)

## 服务部署
### 部署 es 以及生态服务
- es 部署
- eland 部署
跳过
### 部署 embedding 模型
- 第一步是下载文本 embedding 模型

	对于我们的模型，我们使用 [Hugging Face](https://huggingface.co/) 的 [msmarco-distilbert-basetas-b](https://huggingface.co/sentence-transformers/msmarco-distilbert-base-tas-b)。
	
		这是一个句子转换模型，它将一个句子或一个段落映射到一个 768 维的密集向量。该模型针对语义搜索进行了优化，并专门针对 MS MARCO Passage 数据集进行了训练，
	使其适合我们的任务。除了这个模型，Elasticsearch 还支持许多其他的文本 embedding 模型。完整 [列表](https://www.elastic.co/guide/en/machine-learning/current/ml-nlp-model-ref.html#ml-nlp-model-ref-text-embedding) 可以在这里找到。

- 第二步

	在 [NER 示例](https://www.elastic.co/blog/how-to-deploy-nlp-named-entity-recognition-ner-example) 中构建的 Eland docker 代理安装模型。
	
	运行下面的脚本将我们的模型导入本地集群并部署它:
	
	- 自签名证书
	
			docker run -it --rm elastic/eland \
				eland_import_hub_model \
					--url https://elastic:lOwgBZT3KowJrQWMwRWm@192.168.0.3:9200/ \
					--hub-model-id sentence-transformers/msmarco-distilbert-base-tas-b \
					--task-type text embedding \
					--insecure \
					--start
		在上面，请注意你需要根据自己的情况替换点上面的用户名、密码和 Elasticsearch 地址。在这里，由于我们使用的是自签名安装，我使用了 `--insecuer` 选择来进行安装以规避 SSL的安全证书检查

		![](./pic/es-vector2.png)
	- 指定证书

			docker run -it \
				-v /Users/liuxg/elastic/elasticsearch-8.2.0/config/certs/http_ca.crt:/usr/share/http_ca.crt \
				--rm elastic/eland 
				eland_import_hub_model \
				--url https://elastic:EH3*HOpb5rmwdbDj f4k@192.168.0.3:9200/
				--hub-model-id sentence-transformers/msmarco-distilbert-base-tas-b \
				--task-type text_embedding 
				--ca-cert /usr/share/http ca.crt
				--start
		![](./pic/es-vector3.png)
	- 参数说明
		- `--task-type` 设置为 `text embedding` 
		- `--start` 选项被传递给 Eland 脚本，因此模型将自动部署，而无需在模型管理 UI 中启动它。
		-  `inference_threads`

			为了加快推理速度，还可以可以使用 `inference_threads` 参数增加推理线程的数量。

- 第三步

	验证部署
	
	![](./pic/es-vector4.png)
	
	从上面的输出中可以看到模型已经被成功地上传了。

	我们可以通过在 Kibana 控制台中使用这个示例来测试模型的成功部署

		POST /_ml/trained models/sentence-transformers msmarco-distilbert-base-tas-b/deployment/_infer
		{
			"docs": {
				"text field": "how is the weather in jamaica"
			}
		}
	应该看到预测的密集向量 (dense vector) 作为结果:
	
	![](./pic/es-vector5.png)
	经过上面的操作可以在 Kibana 中进行查看已经被 embedding 的模型
	
	![](./pic/es-vector6.png)	
	![](./pic/es-vector7.png)	
	![](./pic/es-vector8.png)	
	![](./pic/es-vector9.png)	
	![](./pic/es-vector10.png)
	在 Kibana 中测试上传的模型:
	
	![](./pic/es-vector11.png)	
	![](./pic/es-vector12.png)	

上面表示我们已经成功部署了模型。

## 装载数据
如介绍中所述，我们使用 MS MARCO Passage Ranking 数据集。数据集非常大，包含超过 800万个段落。

在我们的示例中，我们使用了它的一个子集  [2019 TREC Deep Learning Track](https://microsoft.github.io/msmarco/TREC-Deep-Learning-2019#passage-ranking-dataset) 的测试阶段使用。用于重新排序任务的数据集 [msmarco-passagetest2019-top1000.tsv](https://msmarco.blob.core.windows.net/msmarcoranking/msmarco-passagetest2019-top1000.tsv.gz) 包含 200 个查询，每个查询由一个简单的 IR 系统提取的相关文本段落列表。

从该数据集中，我们提取了所有带有 id 的唯一段落，并将它们放入一个[单独的 tsv 文件](https://github.com/mayya-sharipova/msmarco/blob/main/msmarco-passagetest2019-unique.tsv)中，总共 182469 个段落。

我们使用这个文件作为我们的数据集

我们使用 [Kibana 的文件上传](https://www.elastic.co/guide/en/kibana/current/connect-to-elasticsearch.html#upload-data-kibana)功能来上传这个数据集。 

![](./pic/es-vector13.png)	
![](./pic/es-vector14.png)	
![](./pic/es-vector15.png)	

Kibana 文件上传允许我们为字段提供自定义名称，我们将它们称为 id 类型为 long 的段落 id 和 text 类型的文本为段落的内容。索引名称是 collection。

![](./pic/es-vector16.png)	

上传后，我们可以看到一个名为 collection 的索引，其中包含 182469 个文档

![](./pic/es-vector17.png)	

## 创建 pipeline
我们希望使用推理处理器 ([inference processor](https://www.elastic.co/guide/en/elasticsearch/reference/current/inference-processor.html)) 处理初始数据，该处理器将为每个段落添加 embedding。为此，我们创建了一个文本 embedding 摄取管道，然后使用该管道重新索引我们的初始数据。

在 Kibana 控制台中，我们创建了一个摄取管道用于文本嵌入，并将其称为 `text-embedding`。这些段落位于名为 text 的字段中。正如我们之前所做的，我们将定义一个 `feld_map` 来将文本映射到模型期望的字段 `text_field`。同样 `on_failure` 处理程序设置为将故障索引到不同的索引中:

	PUT _ingest/pipeline/text-embeddings
	{
		"description": "Text embedding pipeline"
		"processors":[
			{
				"inference":{
					"model id": "sentence-transformers_msmarco-distilbert-base-tas-b",
					"target field": "text embedding"
					"field map": {
						"text": "text field"
					}
				}
			}
		],			
	
		"on_failure":[
			{
				"set": {
					"description": "Index document to `failed-<index>`",
					"field":"_index",
					"value": "failed-{{{_index}}}"
				}
			},
			{
				"set":{
					"description":"Set error message",
					"field":"ingest.failure",
					"value": " {{ _ingest.on_failure message}}"
				}
			}
		]
	}

### Reindex 重新索引
我们希望通过 `text-embedding` 管道推送文档，将文档从 `cllection` 索引重新索引`(reindex)` 到新的 `collection-with-embedding` 索引中，以便在` collection-with-embeddings` 索引中的文档具有用于段落歌入的附加字段。

但在我们这样做之前，我们需要为我们的目标索引创建和定义一个映射，特别是对于摄取处理器将存储 embedding 的字段 `text_embedding.predicted_value`。如果我们不这样做，embedding 将被索了到常规浮点 foat 字段中，并且不能用于向量相似性搜索。我们使用的模型将嵌入生成为 768 维向量，因此我们使用具有 768 个维度的索引 dense vector 字段类型，如下所示	

	PUT collection-with-embeddings
	{
		"mappings":{
			"properties":{
				"text_embedding.predicted_value":{
					"type": "dense_vector",
					"dims": 768,
					"index": true,
					"similarity":"cosine"
				},
				"text":(
					"type":"text"
				}
			}
		}
	}

最后，我们准备重新索引。鉴于 `reindex` 需要一些时间来处理所有文档并对其进行推断，我们通过调用带有 `wait_for_completion=false` 标志的 API 在后台 reindex:

	POST _reindex?wait_for_completion=false
	{
		"source":{
			"index":"collection"
		}
		"dest":{
			"index":"collection-with-embeddings",
			"pipeline":"text-embeddings'
		}
	}
以上返回一个任务 ID。 我们可以通过以下方式监控任务的进度:

	GET _tasks/<task id>
或者，通过观察 [model stats API](https://www.elastic.co/guide/en/elasticsearch/reference/8.1/get-trained-models-stats.html) 或模型统计 UI 中的 `inference count` 增加来跟踪进度

![](./pic/es-vector18.png)	
当看到它达到之前的文档数 182469，就表明已经完成

重新索引的文档现在包含推理结果一一向量嵌入 (vetor embedings) 。例如，其中一个文档如下所示:

	{
		id": 7130104,
		"text": "This is the definition of RNA along with examples of types of RN...",
		"text embedding":
		{
			"predicted_value":
				[
					0.057356324046850204,
					0.1602816879749298,
					-0.18122544884681702,
					0.022277727723121643,
					...
				],
			"model id": "sentence-transformers  msmarco-distilbert-base-tas-b"
		}
	}
## Vector Similarity Search (向量查询)
目前不支持在搜索请求期间从查询词隐式生成 embedings，因此我们的语义搜索被组织为一个两步过程:

- 从文本查询中获取该文本的 embedings。为此，我们使用模型的 `_infer API`

	例如，给一个文本查询 `how is the weather in jamaica`，我们首先运行 inferAPI 以得到一个密集向量的 embedding

		POST /_ml/trained_models/sentence-transformers__msmarco-distilbert-base-tas-b/deployment/_infer
		{
			"docs": {
				"text field": "how is the weather in jamaica"
			}
		}
	返回结果

	![](./pic/es-vector19.png)	
	
- 使用向量搜索来查找与查询文本语义相似的文档。

	在 Elasticsearch v8.0 中，[引入](https://www.elastic.co/blog/introducing-approximate-nearest-neighbor-search-in-elasticsearch-8-0)了一个新的 [`_knn_search`](https://www.elastic.co/guide/en/elasticsearch/reference/8.1/knn-search.html) 端点，它允许在索引的 dense vector 字段上进行有效的近似最近邻搜索。我们使用 `knn search API` 来查找最近的文档
	
	例如，上面的例子中生成的 `predicted_value` 是一个768 维的向量。需要将生成的向量 (dense vector) 插到 `knn_search` 中进行查询，如下所示:

		GET collection-with-embeddings/_knn_search
		{
			"knn": {
				"field":"text embedding.predicted_value",
				"query_vector":[
					-0.09194609522819519,
					0.49406030774116516,
					0.03598763048648834,
					...
				]
				"k": 10
				"num_candidates": 100
			},
			"_source": [
				"id“,
				"text"
			]
		}
	将得到最接近查询文档的前 10 个文档，按它们与查询的接近程度排序
	
	![](./pic/es-vector20.png)
	![](./pic/es-vector21.png)	

## 快速验证
由于我们只使用了 MS MARCO 数据集的一个子集，我们无法进行全面评估。相反，我们可以做的是对一些查询进行简单的验证，以了解我们确实得到了相关的结果，而不是一些随机的结果。从 [TREC2019 Deep Leamning Track 对 Passage Ranking Task](https://trec.nist.gov/data/deep2019.html) 的判断中，我们选取最后3 个查询，将它们提交到我们的向量相似性搜索，获得前 10 个结果并参考 TREC 判断，看看我们收到的结果的相关性如何。对于文章排名任务，

- 文章按照不相关 (0)
- 相关(文章主题但不回答问题) (1)、
- 高度相关(2)
- 完全相关(3)


请注意，我们的验证不是严格的评估，它仅用于我们的快速演示。由于我们只对已知与查询相关的段落进行索引，因此它比原始段落检索任务要容易得多。未来我们打算对 MS MARCO 数据集进行严格的评估。

- 查询 #1124210“tracheids are part of_____”提交给我们的向量搜索返回以下结果

	![](./pic/es-vector22.png)
- 查询 #1129237“hydrogen is a liquid below what temperature”返回以下结果

	![](./pic/es-vector23.png)
- 查询 #1133167“how is the weather in jamaica”返回以下结果

	![](./pic/es-vector24.png)

正如我们所见，对于所有 3 个查询，Elasticsearch 返回了大部分相关的结果，并且所有查询的前1个结果要么高度相关，要么完全相关。

## 语义搜索 - Semantic search
在上面的演示中，我们做一个搜索，需要如下的两个步骤:

1. 通过 [_infer API ](https://www.elastic.co/guide/en/elasticsearch/reference/master/infer-trained-model.html) 接口获取搜索字符串的向量
2. 通过[_knn_search](https://www.elastic.co/guide/en/elasticsearch/reference/current/knn-search.html) 端点来对上一步获得的向量进行搜索

幸运的是，在最新的 8.7 的版本中，我们可以通过一个命令把上面的两个步骤合二为一:

	GET collection-with-embeddings/_search
	{
		"knn": {
			"field":"text embedding.predicted_value",
			"query_vector_builder": {
				"text embedding": {
					"model id": "sentence-transformers__msmarco-distilbert-base-tas-b",
					"model text":"How is the weather in Jamaica?"
				},
			"k": 10
			"num candidates": 100
		},
		"_source": [
			"id“,
			"text"
		]
	}
结果从`colection-with-embedings` 索引中收到与查询含义最接近的前 10 个文档，这些文档按与查询的接近程度排序:

![](./pic/es-vector20.png)

上面充分证明 NLP 是 ES stack 中的一项强大功能。
						
## 参考
[Elasticsearch：如何部署 NLP：文本嵌入和向量搜索](https://elasticstack.blog.csdn.net/article/details/124920166)