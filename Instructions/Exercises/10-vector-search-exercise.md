---
lab:
  title: 使用 REST API 运行矢量搜索查询
---

# 使用 REST API 运行矢量搜索查询

在本练习中，你将设置项目、创建索引、上传文档和运行查询。

需要满足以下条件才能成功执行此练习：

- [Postman](https://www.postman.com/downloads/) 应用
- Azure 订阅
- Azure AI 搜索服务
- 该存储库中的 Postman 示例集合 - *Vector-Search-Quickstart.postman_collection v1.0 json*。

> **请注意**如果需要，可在[此处](https://learn.microsoft.com/en-us/azure/search/search-get-started-rest)找到有关 Postman 应用的详细信息。

## 设置项目

首先通过执行以下步骤来设置项目：

1. 记下 Azure AI 搜索服务的 URL 和密钥********。

    ![服务名称和密钥的位置图示。](../media/vector-search/search keys.png)

1. 下载 [Postman 示例集合](https://github.com/MicrosoftLearning/mslearn-knowledge-mining/blob/main/Labfiles/10-vector-search/Vector%20Search.postman_collection%20v1.0.json)。
1. 通过选择“导入”按钮并将集合文件夹拖放到框中，打开 Postman 并导入集合****。

    ![“导入”对话框的图像](../media/vector-search/import.png)

1. 选择“创建分支”按钮创建集合的分支并添加唯一名称****。
1. 右键单击集合名称，然后选择“编辑”****。
1. 选择“变量”选项卡，并使用 Azure AI 搜索服务中的搜索服务和索引名称输入以下值****：

    ![显示变量设置示例的关系图](../media/vector-search/variables.png)

1. 通过选择“保存”按钮保存更改****。

可以向 Azure AI 搜索服务发送请求。

## 创建索引

接下来，在 Postman 中创建索引：

1. 从侧菜单中选择“PUT 创建/更新索引”****。
1. 使用前面记录的 search-service-name、index-name 和 api-version 更新 URL************。
1. 选择“正文”选项卡以查看响应****。
1. 使用 URL 中的索引名称值设置“index-name”，然后选择“发送”********。

应会看到类型的状态代码为 200，指示请求成功****。

## 上传文档

“上传文档”请求中包含 108 个文档，每个文档都有一组完整的“titleVector”和“contentVector”字段的嵌入内容********。

1. 从侧菜单中选择“POST 上传文档”****。
1. 如前所述使用 search-service-name、index-name 和 api-version 更新 URL************。
1. 选择“正文”选项卡查看响应，然后选择“发送”********。

应会看到类型的状态代码为 200，显示请求成功****。

## 运行查询

1. 现在，请尝试在侧菜单上运行以下查询。 为此，请确保每次如前所述更新 URL，并通过选择“发送”来发送请求****：

    - 单矢量搜索
    - 使用筛选器的单矢量搜索
    - 简单混合搜索
    - 使用筛选器的简单混合搜索
    - 跨字段搜索
    - 多查询搜索

1. 选择“正文”选项卡以查看响应和结果****。

如果请求成功，应会看到类型的状态代码为 200****。
