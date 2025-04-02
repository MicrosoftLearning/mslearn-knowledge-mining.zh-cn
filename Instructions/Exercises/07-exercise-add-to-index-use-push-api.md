---
lab:
  title: 使用推送 API 添加到索引
---

# 使用推送 API 添加到索引

你希望探索如何使用 C# 代码创建 Azure AI 搜索索引并将文档上传到该索引。

在本练习中，你将克隆现有的 C# 解决方案并运行该方案，以了解用于上传文档的最佳批大小。 然后，你将使用此批大小，并使用线程方法有效地上传文档。

> 注意**** 为了完成本练习，你需要 Microsoft Azure 订阅。 如果你还没有该订阅，可通过 [https://azure.com/free](https://azure.com/free?azure-portal=true) 注册免费试用版。

## 设置 Azure 资源

为了节省时间，请选择此 Azure 资源管理器模板来创建后面练习中需要的资源：

1. [将资源部署到 Azure](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoftLearning%2Fmslearn-knowledge-mining%2Fmain%2FLabfiles%2F07-exercise-add-to-index-use-push-api%20lab-files%2Fazuredeploy.json) - 选择此链接以创建 Azure AI 资源。
    ![将资源部署到 Azure 时显示的选项的屏幕截图。](../media/07-media/deploy-azure-resources.png)
1. 在“资源组”中，选择“新建”并将其命名为 cog-search-language-exe。
1. 在“区域”中，选择离你较近的[受支持区域](/azure/ai-services/language-service/custom-text-classification/service-limits#regional-availability)。
1. 资源前缀需要全局唯一，请输入随机数字和小写字母前缀，例如 acs118245********。
1. 在“位置”中，选择和上面选择的相同区域。
1. 选择“查看 + 创建”。
1. 选择“创建”。
1. 部署完成后，选择“转到资源组”以查看已创建的所有资源****。

    ![显示所有已部署 Azure 资源的屏幕截图。](../media/07-media/azure-resources-created.png)

## 复制 Azure AI 搜索服务 REST API 信息

1. 在资源列表中，选择已创建的搜索服务。 在上面的示例中，该服务为 acs118245-search-service****。
1. 将搜索服务名称复制到文本文件中。

    ![搜索服务的密钥部分的屏幕截图。](../media/07-media/search-api-keys-exercise-version.png)
1. 在左侧，选择“密钥”，然后将“主管理密钥”复制到同一文本文件中。

## 在 Cloud Shell 中克隆存储库

你将在 Azure 门户中使用 Cloud Shell 开发代码。 你的应用的代码文件已在 GitHub 存储库中提供。

> **提示**：如果最近克隆了 **mslearn-knowledge-mining** 存储库，则可以跳过此任务。 否则，请按照以下步骤将其克隆到开发环境中。

1. 在 Azure 门户中，使用页面顶部搜索栏右侧的 **[\>_]** 按钮，在 Azure 门户中创建新的 Cloud Shell，选择 ***PowerShell*** 环境。 Cloud Shell 在 Azure 门户底部的窗格中提供命令行接口。

    > **备注**：如果以前创建了使用 *Bash* 环境的 Cloud Shell，请将其切换到 ***PowerShell***。

1. 在 Cloud Shell 工具栏的“**设置**”菜单中，选择“**转到经典版本**”（这是使用代码编辑器所必需的）。

    > **提示**：将命令粘贴到 cloudshell 中时，输出可能会占用大量屏幕缓冲区。 可以通过输入 `cls` 命令来清除屏幕，以便更轻松地专注于每项任务。

1. 在 PowerShell 窗格中，输入以下命令以克隆包含此练习的 GitHub 存储库：

    ```
    rm -r mslearn-knowledge-mining -f
    git clone https://github.com/microsoftlearning/mslearn-knowledge-mining mslearn-knowledge-mining
    ```

1. 克隆存储库后，导航到包含应用程序代码文件的文件夹：  

    ```
   cd mslearn-knowledge-mining/Labfiles/07-exercise-add-to-index-use-push-api/OptimizeDataIndexing
    ```

## 设置应用程序

1. 使用 `ls` 命令，可以查看 **OptimizeDataIndexing** 文件夹的内容。 请注意，它包含用于配置设置的 `appsettings.json` 文件。

1. 输入以下命令以编辑已提供的配置文件：

    ```
   code appsettings.json
    ```

    该文件已在代码编辑器中打开。

    ![显示 appsettings.json 文件内容的屏幕截图。](../media/07-media/update-app-settings.png)

1. 粘贴搜索服务名称和主管理密钥。

    ```json
    {
      "SearchServiceUri": "https://acs118245-search-service.search.windows.net",
      "SearchServiceAdminApiKey": "YOUR_SEARCH_SERVICE_KEY",
      "SearchIndexName": "optimize-indexing"
    }
    ```

    设置文件应该与上面类似。
   
1. 替换占位符后，使用 **Ctrl+S** 命令保存更改，然后使用 **Ctrl+Q** 命令关闭代码编辑器，同时使 Cloud Shell 命令行保持打开状态。
1. 在终端中输入`dotnet run`，然后按 **Enter**。

    ![显示在 VS Code 中运行的应用出现异常的屏幕截图。](../media/07-media/debug-application.png)

    输出结果显示，在这种情况下，性能最佳的批量大小是 900 份文档，传输速度（MB/秒）最高。
   
    >**备注**：传输速度值可能与屏幕截图中显示的值不同。 但是，性能最佳的批大小仍应相同。 

## 编辑代码以实现线程处理和退避重试策略

注释掉的代码可以更改应用，以使用线程将文档上传到搜索索引。

1. 输入以下命令，以打开客户端应用程序的代码文件：

    ```
   code Program.cs
    ```

1. 注释掉第 38 行和第 39 行，如下所示：

    ```csharp
    //Console.WriteLine("{0}", "Finding optimal batch size...\n");
    //await TestBatchSizesAsync(searchClient, numTries: 3);
    ```

1. 取消注释第 41 行到第 49 行。

    ```csharp
    long numDocuments = 100000;
    DataGenerator dg = new DataGenerator();
    List<Hotel> hotels = dg.GetHotels(numDocuments, "large");

    Console.WriteLine("{0}", "Uploading using exponential backoff...\n");
    await ExponentialBackoff.IndexDataAsync(searchClient, hotels, 1000, 8);

    Console.WriteLine("{0}", "Validating all data was indexed...\n");
    await ValidateIndexAsync(indexClient, indexName, numDocuments);
    ```

    用于控制批大小和线程数的代码为 `await ExponentialBackoff.IndexDataAsync(searchClient, hotels, 1000, 8)`。 批大小为 1000 个，线程数为 8 个。

    ![显示所有已编辑代码的屏幕截图。](../media/07-media/thread-code-ready.png)

    代码应如上所示。

1. 保存所做更改。
1. 选择终端，然后按任意键结束正在运行的进程（如果尚未执行）。
1. 在终端中运行 `dotnet run`。

    应用将启动 8 个线程，然后在每个线程完成向控制台写入新消息时：

    ```powershell
    Finished a thread, kicking off another...
    Sending a batch of 1000 docs starting with doc 57000...
    ```

    上传 100,000 个文档后，应用会编写摘要（此过程可能需要一段时间才能完成）：

    ```powershell
    Ended at: 9/1/2023 3:25:36 PM
    
    Upload time total: 00:01:18:0220862
    Upload time per batch: 780.2209 ms
    Upload time per document: 0.7802 ms
    
    Validating all data was indexed...
    
    Waiting for service statistics to update...
    
    Document Count is 100000
    
    Waiting for service statistics to update...
    
    Index Statistics: Document Count is 100000
    Index Statistics: Storage Size is 71453102
    
    ``````

浏览 `TestBatchSizesAsync` 过程中的代码，了解代码如何测试批大小性能。

浏览 `IndexDataAsync` 过程中的代码，了解代码如何管理线程。

浏览 `ExponentialBackoffAsync` 中的代码，了解代码如何实现指数退避重试策略。

可在 Azure 门户中搜索并验证文档是否已添加到索引中。

![显示包含 100000 个文档的搜索索引的屏幕截图。](../media/07-media/check-search-service-index.png)

## 清理

现已完成练习，请删除所有不再需要的资源。 从克隆到计算机的代码开始。 然后删除 Azure 资源。

1. 在 **Azure 门户**中，选择“资源组”。
1. 选择为本练习创建的资源组。
1. 选择“删除资源组”****。 
1. 确认删除，然后选择“删除”****。
1. 选择不需要的资源，然后选择“删除”。
