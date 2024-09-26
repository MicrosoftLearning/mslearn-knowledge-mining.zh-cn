---
lab:
  title: 创建 Azure AI 搜索解决方案
  module: Module 12 - Creating a Knowledge Mining Solution
---

# 创建 Azure AI 搜索解决方案

所有组织都依靠信息来做出决策、回答问题和高效运作。 对于大多数组织而言，问题不在于缺少信息，而在于如何从大量文档、数据库和存储信息的其他源中查找和提取信息。

例如，假设 Margie's Travel 是一家专门组织到世界各地城市旅游的旅行社。 随着时间的推移，公司在宣传册等文档以及客户提交的酒店评论中积累了大量信息。 对于计划旅行的旅行社和客户来说，这些数据是一种宝贵的见解来源，但由于数据量很大，很难找到相关信息来回答客户的特定问题。

为了应对这一挑战，Margie's Travel 可以使用 Azure AI 搜索来实现一个解决方案，在该解决方案中，通过使用 AI 技能对文档进行索引和扩充，使其更易于搜索。

## 创建 Azure 资源

你将为 Margie's Travel 创建的解决方案需要 Azure 订阅中的以下资源：

- Azure AI 搜索**** 资源，用于管理索引和查询。
- Azure AI 服务**** 资源为技能提供 AI 服务，搜索解决方案可通过这些服务使用 AI 生成的见解扩充数据源中的数据。
- 有 blob 容器的**存储帐户**，其中存储了要搜索的文档。

> **重要说明**：Azure AI 搜索和 Azure AI 服务资源必须位于同一位置！

### 创建 Azure AI 搜索资源

1. 在 Web 浏览器中，打开 Azure 门户 (`https://portal.azure.com`)，然后使用与你的 Azure 订阅关联的 Microsoft 帐户登录。
2. 选择“+ 创建资源”**** 按钮，搜索“搜索”**，并使用以下设置创建 Azure AI**** 资源：
    - **订阅**：Azure 订阅
    - 资源组：创建新资源组（如果使用受限制的订阅，你可能无权创建新的资源组 - 请使用提供的资源组）
    - **服务名称**：输入唯一名称
    - 位置****：选择位置 - 请注意，Azure AI 搜索和 Azure AI 服务资源必须位于同一位置**
    - **定价层**：基本

3. 等待部署完成，然后转到部署的资源。
4. 在 Azure 门户的 Azure AI 搜索资源边栏选项卡上查看“概述”**** 页。 在这里，你可以使用可视化界面来创建、测试、管理和监视搜索解决方案的各个组件，包括数据资源、索引、索引器和技能组。

### 在 Azure 门户中创建 Azure AI 服务资源

如果订阅中还没有 Azure AI 服务**** 资源，则需要预配此资源。 搜索解决方案将使用该资源通过 AI 生成的见解来丰富数据存储中的数据。

1. 返回到 Azure 门户主页，然后选择“+ 创建资源”**** 按钮，搜索“Azure AI 服务”**，然后使用以下设置创建 Azure AI 服务多服务帐户****：
    - **订阅**：*Azure 订阅*
    - 资源组****：与 Azure AI 搜索资源相同的资源组**
    - 区域****：与 Azure AI 搜索资源相同的位置**
    - **名称**：*输入唯一名称*
    - **定价层**：标准版 S0
2. 选中所需的复选框并创建资源。
3. 等待部署完成，然后查看部署详细信息。

### 创建存储帐户

1. 返回到 Azure 门户的主页，然后选择“&#65291;创建资源”按钮，搜索“存储帐户”，并创建具有以下设置的存储帐户资源：
    - **订阅**：Azure 订阅
    - 资源组****：*与 Azure AI 搜索和 Azure AI 服务资源相同的资源组**
    - **存储帐户名称**：输入唯一名称
    - **区域**：选择任何可用区域
    - **性能**：标准
    - **冗余**：本地冗余存储 (LRS)
    - 在“高级”**** 选项卡上，选中“允许在单个容器上启用匿名访问”** 旁边的复选框
2. 等待部署完成，然后转到部署的资源。
3. 在“概述”页上，记下“订阅 ID”- 它标识着预配了存储帐户的订阅。
4. 在“访问密钥”页上，记下为你的存储帐户生成的两个密钥。 然后选择“显示密钥”以查看密钥。

    > **提示**：使“存储帐户”边栏选项卡保持打开 - 下一节程序中需要订阅 ID 和其中一个密钥。

## 准备在 Visual Studio Code 中开发应用

你可以使用 Visual Studio Code 开发搜索应用。 应用程序的代码文件已在 GitHub repo 中提供。

> **提示**：如果已克隆 **mslearn-knowledge-mining** 存储库，请在 Visual Studio Code 中打开它。 否则，请按照以下步骤将其克隆到开发环境中。

1. 启动 Visual Studio Code。
1. 打开面板 (SHIFT+CTRL+P) 并运行“**Git：克隆**”命令，以将 `https://github.com/MicrosoftLearning/mslearn-knowledge-mining` 存储库克隆到本地文件夹（任意文件夹均可）。
1. 克隆存储库后，在 Visual Studio Code 中打开文件夹。
1. 等待其他文件安装完毕，以支持存储库中的 C# 代码项目。

    > **注意**：如果系统提示你添加生成和调试所需的资产，请选择**以后再说**。

## 将文档上传到 Azure 存储

现在你已拥有必要的资源，可以向 Azure 存储帐户上传一些文档。

1. 在 Visual Studio Code 中，在“**资源管理器**”窗格中展开“**Labfiles\01-azure-search**”文件夹，然后选择“**UploadDocs.cmd**”。
2. 编辑批处理文件，将 YOUR_SUBSCRIPTION_ID、YOUR_AZURE_STORAGE_ACCOUNT_NAME 和 YOUR_AZURE_STORAGE_KEY 占位符替换为相应的订阅 Id、你之前创建的存储帐户的 Azure 存储帐户名称和 Azure 存储帐户密钥值。
3. 保存更改，然后右键单击“**01-azure-search**”文件夹并打开集成终端。
4. 使用 Azure CLI，输入以下命令以登录到你的 Azure 订阅。

    ```powershell
    az login
    ```

    Web 浏览器标签页随即打开，并提示你登录到 Azure。 按照提示操作，然后关闭浏览器标签页并返回到 Visual Studio Code。

5. 输入以下命令以运行批处理文件。 这将在你的存储帐户中创建一个 blob 容器，并将 data 文件夹中的文档上传到该容器中。

    ```powershell
    .\UploadDocs.cmd
    ```

## 为文档编制索引

现在你已准备好文档，可以通过为文档编制索引来创建搜索解决方案。

1. 在 Azure 门户中，浏览到 Azure AI 搜索资源。 然后，在其“概述”页上，选择“导入数据” 。
2. 在“连接到数据”页面上的“数据源”列表中，选择“Azure Blob 存储”  。 然后使用以下值补全数据存储详细信息：
    - **数据源**：Azure Blob 存储
    - **数据源名称**：margies-data
    - **要提取的数据**：内容和元数据
    - **分析模式**：默认
    - **连接字符串**：选择“选择现有连接”。然后选择存储帐户，最后选择 UploadDocs.cmd 脚本创建的 margies 容器。
    - **托管标识身份验证**：无
    - **容器名称**：margies
    - **Blob 文件夹**：将此项留空
    - **说明**：Margie's Travel 网站上的手册和评论。
3. 前进到下一步（添加认知技能）。
4. 在“附加 Azure AI 服务”**** 部分中，选择 Azure AI 服务资源。
5. 在“添加扩充”部分中：
    - 将技能组名称改为 margies-skillset。
    - 选择“启用 OCR”选项，将所有文本合并到 merged_content 字段中。
    - 确保“源数据”字段设置为“merged_content” 。
    - 将“扩充信息粒度级别”保留为“源字段”，即将文档的全部内容设为可编制索引；但请注意，可将此项更改为在更精细的级别（例如页面或语句）提取信息。
    - 选择以下经扩充的字段：

        | 认知技能 | 参数 | 字段名称 |
        | --------------- | ---------- | ---------- |
        | 提取位置名称 | | locations |
        | 提取关键短语 | | keyphrases |
        | 检测语言 | | 语言 |
        | 由图像生成标记 | | imageTags |
        | 从映像中生成描述文字 | | imageCaption |

6. 再次检测你的选择（稍后可能很难更改这些选项）。 然后前进到下一步（自定义目标索引）。
7. 将索引名称改为 margies-index。
8. 确保“密钥”设置为 metadata_storage_path，将“建议器名称”留空，并将“搜索模式”保留为默认值   。
9. 对索引字段进行以下更改，并让其他所有字段保留其默认值（重要提示：可能需要向右滚动才能看到整张表）：

    | 字段名称 | 可检索 | 可筛选 | 可排序 | 可查找 | 可搜索 |
    | ---------- | ----------- | ---------- | -------- | --------- | ---------- |
    | metadata_storage_size | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | |
    | metadata_storage_last_modified | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | |
    | metadata_storage_name | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |
    | metadata_author | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |
    | locations | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |
    | keyphrases | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |
    | 语言 | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |

10. 再次检查你的选择，特别注意确保为每个字段选择正确的“可检索”、“可筛选”、“可排序”、“可分面”和“可搜索”选项（稍后可能很难更改这些选项）。 然后前进到下一步（创建索引器）。
11. 将索引器名称改为 margies-indexer。
12. 将“计划”设置为“一次” 。
13. 展开“高级”选项，并确保已选择“基础 64 编码密钥”选项（通常编码密钥会使索引更有效） 。
14. 选择“提交”以创建数据源、技能组、索引和索引器。 索引器将自动运行并运行索引管道，该索引管道可以：
    1. 从数据源中提取文档元数据字段和内容
    2. 运行认知技能的技能组，以生成更多的扩充字段
    3. 将提取的字段映射到索引。
15. 在左侧，查看“索引器”**** 页，该页应显示新建的 margies-indexer****。 等待几分钟，然后单击“&orarr; 刷新”，直到“状态”指示成功 。

## 搜索索引

现在你已拥有索引，可以搜索它。

1. 在 Azure AI 搜索资源的“概述”**** 页顶部，选择“搜索资源管理器”****。
2. 在搜索资源管理器中的“查询字符串”框中，输入 `*`（单个星号），然后选择“搜索” 。

    此查询将以 JSON 格式检索索引中的所有文档。 检查结果并记下每个文档的字段，其中包含由你选择的认知技能所提取的文档内容、元数据和经扩充的数据。

3. 在“视图”**** 菜单中，选择“JSON 视图”**** 并注意已显示搜索的 JSON 请求，如下所示：

    ```json
    {
      "search": "*"
    }
    ```

1. 修改 JSON 请求以包含 count**** 参数，如下所示：

    ```json
    {
      "search": "*",
      "count": true
    }
    ```

1. 提交修改后的搜索。 这一次，结果包含 @odata.count 字段，它显示结果顶部，表示搜索返回的文档数。

4. 尝试运行以下查询：

    ```json
    {
      "search": "*",
      "count": true,
      "select": "metadata_storage_name,metadata_author,locations"
    }
    ```

    这一次，结果仅包含文件名、作者和文档内容中提及的任何位置。 文件名和作者分别位于 metadata_storage_name 和 metadata_author 字段，这两个字段提取自源文档 。 locations 字段由认知技能生成。

5. 现在尝试以下查询字符串：

    ```json
    {
      "search": "New York",
      "count": true,
      "select": "metadata_storage_name,keyphrases"
    }
    ```

    此搜索会查找任意可搜索字段中提及“New York”的文档，并返回文件名和文档中的关键短语。

6. 让我们尝试再运行一个查询：

    ```json
    {
      "search": "New York",
      "count": true,
      "select": "metadata_storage_name",
      "filter": "metadata_author eq 'Reviewer'"
    }
    ```

    此查询将返回作者为 Reviewer 且提及“New York”的任何文档的文件名。

## 浏览并修改搜索组件的定义

搜索解决方案的组件基于 JSON 定义，你可以在 Azure 门户中查看和编辑它。

虽然可以使用门户创建和修改搜索解决方案，但理想的做法通常是在 JSON 中定义搜索对象，并使用 Azure AI 服务 REST 接口来创建和修改它们。

### 获取 Azure AI 搜索资源的终结点和密钥

1. 在 Azure 门户中，返回到 Azure AI 搜索资源的“概述”**** 页；在页面顶部查找资源的 Url****（类似于 https://resource_name.search.windows.net****）并将其复制到剪贴板。
2. 在 Visual Studio Code 中，在“资源管理器”窗格中展开“**01-azure-search**”文件夹及其“**modify-search**”子文件夹，并选择“**modify-search.cmd**”来打开它。 你将使用此脚本文件来运行 cURL** 命令，以将 JSON 提交给 Azure AI 服务 REST 接口。
3. 在 modify-search.cmd 中，将  YOUR_SEARCH_URL 占位符替换为你复制到剪贴板的 URL。
4. 在 Azure 门户的“设置”**** 部分，查看 Azure AI 搜索资源的“密钥”**** 页，并将“主管理员密钥”**** 复制到剪贴板。
5. 在 Visual Studio Code 中，将 YOUR_ADMIN_KEY 占位符替换为你复制到剪贴板的密钥。
6. 将更高保存到 modify-search.cmd（但暂时不要运行它！）

### 评价和修改技能组

1. 在 Visual studio Code 中，在“modify-search”文件夹中打开“skillset.json”。 这将显示 margies-skillset 的 JSON 定义。
2. 在技能组定义的顶部，记下 cognitiveServices**** 对象，该对象用于将 Azure AI 服务资源连接到技能组。
3. 在 Azure 门户中，打开 Azure AI 服务资源（不是<u></u> Azure AI 搜索资源！）并在“资源管理”**** 部分查看其“密钥和终结点”**** 页。 然后将“密钥 1”**** 粘贴到剪贴板。
4. 在 Visual Studio Code 中的 skillset.json**** 中，将 YOUR_COGNITIVE_SERVICES_KEY**** 占位符替换为复制到剪贴板的 Azure AI 服务密钥。
5. 滚动浏览 JSON 文件，注意它包含在 Azure 门户中使用 Azure AI 搜索用户界面创建的技能定义。 在技能列表底部，以使用以下定义添加额外的技能：

    ```json
    {
        "@odata.type": "#Microsoft.Skills.Text.V3.SentimentSkill",
        "defaultLanguageCode": "en",
        "name": "get-sentiment",
        "description": "New skill to evaluate sentiment",
        "context": "/document",
        "inputs": [
            {
                "name": "text",
                "source": "/document/merged_content"
            },
            {
                "name": "languageCode",
                "source": "/document/language"
            }
        ],
        "outputs": [
            {
                "name": "sentiment",
                "targetName": "sentimentLabel"
            }
        ]
    }
    ```

    新技能名为“get-sentiment”，对于文档中的每个 document 级别，它将评估在被索引的文档的 merged_content 字段中找到的文字（包含源内容和从内容中的图像提取的任何文字）。 它使用提取的文档语言（默认为英语），并评估内容情绪的标签。 情绪标签的值可以是“积极”、“消极”、“中性”或“混合”。 此标签随后作为名为 sentimentLabel 的新字段输出。

6. 保存对 skillset.json 的更改。

### 查看并修改索引

1. 在 Visual studio Code 中，在“modify-search”文件夹中打开“index.json”。 这将显示 margies-index 的 JSON 定义。
2. 滚动浏览索引并查看字段定义。 某些字段基于源文档中的元数据和内容，还有的来自于技能和技能组。
3. 在你在 Azure 门户中定义的字段列表的末尾，注意添加了两个额外的字段：

    ```json
    {
        "name": "sentiment",
        "type": "Edm.String",
        "facetable": false,
        "filterable": true,
        "retrievable": true,
        "sortable": true
    },
    {
        "name": "url",
        "type": "Edm.String",
        "facetable": false,
        "filterable": true,
        "retrievable": true,
        "searchable": false,
        "sortable": false
    }
    ```

4. 将使用 sentiment 字段添加来自你添加到技能组的 get-sentiment 技能的输出。 将根据从数据源提取的 metadata_storage_path 值，使用 url 字段将每个被索引的文档的 URL 添加到索引。 注意，索引已包含 metadata_storage_path 字段，但它用作索引密钥并采用 Base-64 编码，因此它具有密钥的效率，但要求客户端应用程序在要使用实际 URL 值作为字段时对其进行解码。 添加另一个字段以存储未编码的值可解决此问题。

### 查看并修改索引器

1. 在 Visual studio Code 中，在“modify-search”文件夹中打开“indexer.json”。 这将显示 margies-indexer 的 JSON 定义，它将由文档内容和元数据提取的字段（位于 fieldMappings 部分）和由技能组中的技能提取到的值（位于 outputFieldMappings 部分）映射到索引中的字段。
2. 在 fieldMappings 列表中，注意 metadata_storage_path 值到 base-64 编码密钥字段的映射。 这是在你在 Azure 门户中将 metadata_storage_path 分配为密钥并选择编码选项时创建的。 此外，新映射将相同的值显式映射到 url 字段，但没有 Base-64 编码：

    ```json
    {
        "sourceFieldName" : "metadata_storage_path",
        "targetFieldName" : "url"
    }    
    ```

    源文档中的其他所有元数据和内容字段都隐式映射到索引中具有相同名称的字段。

3. 查看 ouputFieldMappings 部分，它将技能组中技能的数据映射到索引字段。 其中大部分结果都反映了你在用户界面中的选择，但还添加了以下映射，将由情绪识别技能提取的 sentimentLabel 值映射到你添加到索引的 sentiment 字段 ：

    ```json
    {
        "sourceFieldName": "/document/sentimentLabel",
        "targetFieldName": "sentiment"
    }
    ```

### 使用 REST API 更新搜索解决方案

1. 右键单击 modify-search 文件夹并打开集成终端。
2. 在 modify-search 文件夹的终端窗格中，输入以下命令以运行 modify-search.cmd 脚本，它会将 JSON 定义提交到 REST 接口并启动索引编制。

    ```powershell
    ./modify-search
    ```

3. 脚本完成后，在 Azure 门户中返回 Azure AI 搜索资源的“概述”**** 页，并查看“索引器”**** 页。 然后定期选择“刷新”以跟踪索引编制操作的进度。 可能需要一分钟左右的时间才能完成。

    对于一些由于过大而无法评估情绪的文档，可能会生成警告。通常在页面或语句级别而不是整个文档级别执行情绪分析；但在本例中，大多数文档（尤其是酒店评论）很短，可以评估出有用的文档级情绪分数。

### 查询经修改的索引

1. 在 Azure AI 搜索资源边栏选项卡的顶部，选择“搜索资源管理器”****。
2. 在搜索资源管理器的“查询字符串”**** 框中，提交以下 JSON 查询：

    ```json
    {
      "search": "London",
      "select": "url,sentiment,keyphrases",
      "filter": "metadata_author eq 'Reviewer' and sentiment eq 'positive'"
    }
    ```

    该查询可检索评论家撰写的所有提及伦敦时具有积极情绪标签（换句话说，提及伦敦的正面评论）的文档的 url、情绪和关键词

3. 关闭“搜索资源管理器”页并返回“概述”页。

## 创建搜索客户端应用程序

现在你拥有了有用的索引，可从客户端应用程序使用它。 可通过下列方式进行此操作：使用 REST 接口，通过 HTTP 以 JSON 提交请求并收取响应；使用你偏好的编程语言的软件开发工具包 (SDK)。 在此练习中，我们将使用 SDK。

> **注意**：可选择将该 SDK 用于 C# 或 Python 。 在下面的步骤中，请执行适用于你的语言首选项的操作。

### 获取搜索资源的终结点和密钥

1. 在 Azure 门户中，在 Azure AI 搜索资源的“概述”**** 页上，记下 Url**** 值，该值应类似于 **https://*your_resource_name*.search.windows.net**。 这是你的搜索资源的终结点。
2. 在“密钥”页面上，注意有两个管理密钥和一个查询密钥。 管理密钥用于创建和管理搜索资源；查询密钥由只需要执行搜索查询的客户端应用程序使用。

    你需要客户端应用程序的终结点和查询密钥。

### 准备使用 Azure AI 搜索 SDK

1. 在 Visual Studio Code 在，在“**资源管理器**”窗格中，浏览到“**01-azure-search**”文件夹，并根据你的语言首选项展开“**C-Sharp**”文件夹或“**Python**”文件夹。
2. 右键单击 margies-travel 文件夹并打开集成终端。 然后针对语言首选项运行相应的命令来安装 Azure AI 搜索 SDK 包。

    **C#**

    ```
    dotnet add package Azure.Search.Documents --version 11.6.0
    ```

    **Python**

    ```
    pip install azure-search-documents==11.5.1
    ```

3. 查看 margies-travel 文件夹的内容，注意它包含配置设置的文件：
    - **C#** ：appsettings.json
    - **Python**：.env

    打开配置文件并更新它包含的配置值，以反映 Azure AI 搜索资源的终结点**** 和查询密钥****。 保存所做的更改。

### 浏览用于搜索索引的代码

margies-travel 文件夹包含 web 应用程序（Microsoft C# ASP.NET Razor web 应用程序或 Python Flask 应用程序）的代码文件，该应用程序包含搜索功能 。

1. 根据你选择的编程语言，在该 web 应用程序中打开以下代码文件：
    - **C#** ：Pages/Index.cshtml.cs
    - **Python**：app.py
2. 在代码文件顶部附近，查找注释“导入搜索命名空间”****，并记下已导入以使用 Azure AI 搜索 SDK 的命名空间：
3. 在 search_query**** 函数中，查找注释“创建搜索客户端”****，并注意该代码使用 Azure AI 搜索资源的终结点和查询密钥创建 SearchClient**** 对象：
4. 在 search_query 函数中，找到注释“Submit search query”，并通过以下选项评审用于提交对指定文字的搜索的代码：
    - 要求找到搜索文本中所有单词的搜索模式。
    - 结果中包含搜索找到的文档总数。
    - 结果经过筛选，仅包含与提供的筛选表达式匹配的文档。
    - 结果按指定排列顺序排列。
    - Metadata_author 字段的每个离散值都作为一个 facet 返回，facet 可用于显示要筛选的预定义的值。
    - 结果中包含 merged_content 和 imageCaption 字段的最多三个提取项和突出显示的搜索词。
    - 结果仅包含指定字段。

### 浏览用于呈现搜索结果的代码

该 web 应用已包含用于处理和呈现搜索结果的代码。

1. 根据你选择的编程语言，在该 web 应用程序中打开以下代码文件：
    - **C#** ：Pages/Index.cshtml
    - **Python**：templates/search.html
2. 检查代码，它呈现了显示搜索结果的页面。 观察以下情况：
    - 页面从用户可用于提交新搜索的搜索表单开始（在 Python 版本的应用程序中，此表单在 base.html 模板中定义），页面顶部引用了该表单。
    - 然后呈现搜索表单，让用户能够精炼搜索结果。 此表单的代码：
        - 检索并显示搜索结果中的文档数。
        - 检索 metadata_author 字段的 facet 值，并将其显示为待筛选的选项列表。
        - 创建用于显示结果排序选项的下拉列表。
    - 代码随后将迭代搜索结果，按以下方式呈现每个结果：
        - 将 metadata_storage_name（文件名）字段显示为指向 url 字段中地址的链接。
        - 突出显示在 merged_content 和 imageCaption 字段中找到的搜索词，以帮助在上下文中展现搜索词。
        - 显示metadata_author、metadata_storage_size、metadata_storage_last_modified 和 language 字段。
        - 显示文档的“情绪”标签。 可以是积极、消极、中性或混合。
        - 显示前五个 keyphrases（若有）。
        - 显示前五个 locations（若有）。
        - 显示前五个 imageTags（若有）。

### 运行 Web 应用

 1. 返回 margies-travel 文件夹的集成终端，并输入以下命令以运行程序：

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    flask run
    ```

2. 在应用成功启动时显示的消息中，单击指向正在运行的 Web 应用程序的链接（ http://localhost:5000/ 或 http://127.0.0.1:5000/ ），以在 Web 浏览器中打开 Margies Travel 站点 。
3. 然后，在 Margie's Travel 网站的搜索框中输入“伦敦酒店”，并单击“搜索”。
4. 查看搜索结果。 其中包含文件名（以及指向文件 URL 的超链接）、强调了搜索词（伦敦和酒店）的文件内容提取项，以及来自索引字段的其他文件属性 。
5. 观察结果页包含某些可用于精炼结果的用户界面元素。 其中包括：
    - 基于 metadata_author 字段 facet 值的筛选器。 这演示了如何使用可分片字段来返回 facet字段的列表，其中包含可在用户界面中显示为潜在筛选器值的一组离散值 。
    - 根据特定字段和排列顺序（升序或降序）对结果进行排序的能力。 默认顺序基于相关度，基于评分配置文件计算为 search.score() 值，评分配置文件评估索引字段中搜索词的频率和重要性。
6. 选择“Reviewer”筛选器和“正面到负面”排序选项，然后选择“精炼结果”。
7. 请注意，结果已筛选为仅包含评论，并根据情绪标签进行了排序。
8. 在“搜索”框中，输入新搜索“quiet hotel in New York”并查看结果。
9. 尝试使用以下搜索词：
    - **Tower of London**（观察这个词在一些文档中被识别为关键短语）。
    - **skyscraper**（观察这个词并未出现在任何文档的实际内容中，但在由某些文档中的图像生成的图像描述和图像标签中找到）。
    - **Mojave desert**（观察这个词在一些文档中被识别为位置）。
10. 关闭包含 Margie's Travel 网站的浏览器标签页，并返回到 Visual Studio Code。 然后在 margies-travel 文件夹中 Python 终端中（dotnet 或 flask 应用程序在其中运行），按 Ctrl+C 以停止运行应用。

## 清理

现已完成练习，请删除所有不再需要的资源。 删除 Azure 资源：

1. 在 **Azure 门户**中，选择“资源组”。
1. 选择不需要的资源组，然后选择“删除资源组”。****

## 详细信息

要详细了解 Azure AI 搜索，请参阅 [Azure AI 搜索文档](https://docs.microsoft.com/azure/search/search-what-is-azure-search)。
