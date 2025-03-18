---
lab:
  title: 为 Azure AI 搜索创建自定义技能
  module: Module 12 - Creating a Knowledge Mining Solution
---

# 为 Azure AI 搜索创建自定义技能

Azure AI 搜索使用 AI 技能的扩充管道从文档中提取 AI 生成的字段，并将其包含在搜索索引中。 该服务提供了一套综合性的内置技能供你使用，但如果你有这些技能无法满足的特定要求，你可以创建自定义技能。

在本次练习中，你将创建一个自定义技能，该技能可以将文档中各单词出现的频率制表以生成前五个最常用单词的列表，然后将其添加到 Margie's Travel（一家虚构的旅行社）的搜索解决方案中。

## 准备在 Visual Studio Code 中开发应用

你可以使用 Visual Studio Code 开发搜索应用。 应用程序的代码文件已在 GitHub repo 中提供。

> **提示**：如果已克隆 **mslearn-knowledge-mining** 存储库，请在 Visual Studio Code 中打开它。 否则，请按照以下步骤将其克隆到开发环境中。

1. 启动 Visual Studio Code。
1. 打开面板 (SHIFT+CTRL+P) 并运行“**Git：Clone**”命令，以将 `https://github.com/MicrosoftLearning/mslearn-knowledge-mining` 存储库克隆到本地文件夹（任意文件夹均可）。
1. 克隆存储库后，在 Visual Studio Code 中打开文件夹。
1. 等待其他文件安装完毕，以支持存储库中的 C# 代码项目。

    > **注意**：如果系统提示你添加生成和调试所需的资产，请选择“以后再说”。

## 创建 Azure 资源

> **备注**：如果之前已完成“创建 Azure AI 搜索解决方案”**[](01-azure-search.md)** 练习，并且订阅中仍有这些 Azure 资源，则可以跳过此部分，并从“创建搜索解决方案”**** 部分开始。 否则，请按照以下步骤预配所需的 Azure 资源。

1. 在 Web 浏览器中，打开 Azure 门户 (`https://portal.azure.com`)，然后使用与你的 Azure 订阅关联的 Microsoft 帐户登录。
2. 在顶部搜索栏中，搜索 *Azure AI 服务*，选择 **Azure AI 服务多服务帐户**，并使用以下设置创建 Azure AI 服务多服务帐户资源：
    - **订阅**：*Azure 订阅*
    - **资源组**：*选择或创建一个资源组（如果使用受限制的订阅，你可能无权创建新的资源组 - 请使用提供的资源组）*
    - 区域****：*从地理位置靠近你的可用区域中进行选择*
    - **名称**：*输入唯一名称*
    - **定价层**：标准版 S0
1. 部署后，转到该资源并在“概述”页上，记下“订阅 ID”和”位置”************。 在后续步骤中，你将需要这些值以及资源组的名称。 
1. 在 Visual Studio Code 中，展开 Labfiles/02-search-skill 文件夹，选择 setup.cmd********。 你将使用此批处理脚本来运行创建所需 Azure 资源时需要的 Azure 命令行接口 (CLI) 命令。
1. 右键单击 02-search-skill 文件夹，选择“在集成终端中打开”********。
1. 在终端窗格中输入以下命令，与 Azure 订阅建立经过身份验证的连接。

    ```powershell
    az login --output none
    ```

8. 根据提示，选择或登录到 Azure 订阅。 然后，返回到 Visual Studio Code 并等待登录过程完成。
9. 运行以下命令以列出 Azure 位置。

    ```powershell
    az account list-locations -o table
    ```

10. 在输出中，找到与资源组位置相对应的“名称”值（例如，对于美国东部，对应的名称是 eastus） 。
11. 在 setup.cmd 脚本中，使用订阅 ID、资源组名称和位置名称的适当值修改 subscription_id、resource_group 和 location 变量声明。 保存更改。
12. 在 02-search-skill 文件夹的终端中，输入以下命令运行脚本****：

    ```powershell
    ./setup
    ```

    > **注意**：如果脚本失败，请确保已使用正确的变量名保存脚本，然后再试一次。

13. 脚本完成后，查看输出，并记录有关 Azure 资源的以下信息（稍后会用到这些值）：
    - 存储帐户名称
    - 存储连接字符串
    - 搜索服务终结点
    - 搜索服务管理密钥
    - 搜索服务查询密钥

14. 在 Azure 门户中，刷新资源组并验证它是否包含 Azure 存储帐户、Azure AI 服务资源和 Azure AI 搜索资源。

## 创建搜索解决方案

在获得必要的 Azure 资源后，即可创建由以下组件组成的搜索解决方案：

- 数据源 - 引用 Azure 存储容器中的文档。
- 技能组 - 定义用于从文档中提取 AI 生成的字段的技能扩充管道。
- 索引 - 定义一组可搜索的文档记录。
- 索引器 - 从数据源中提取文档、应用技能组并填充索引。

在本练习中，你将使用 Azure AI 搜索 REST 接口通过提交 JSON 请求来创建这些组件。

1. 在 Visual Studio Code 的 02-search-skill 文件夹中，展开 create-search 文件夹，选择 data_source.json************。 该文件包含数据源“margies-custom-data”的 JSON 定义。
2. 将 YOUR_CONNECTION_STRING 占位符替换为 Azure 存储帐户的连接字符串，该字符串应如下所示：

    ```
    DefaultEndpointsProtocol=https;AccountName=ai102str123;AccountKey=12345abcdefg...==;EndpointSuffix=core.windows.net
    ```

    可以在 Azure 门户中存储帐户的“访问密钥”页上找到该连接字符串**。

3. 保存并关闭更新后的 JSON 文件。
4. 在 create-search 文件夹中，打开 skillset.json。 该文件包含技能组“margies-custom-skillset”的 JSON 定义。
5. 在技能组定义顶部的 cognitiveServices**** 元素中，将 YOUR_AI_SERVICES_KEY**** 占位符替换为 Azure AI 服务资源的任何一个密钥。

    可以在 Azure 门户中 Azure AI 服务资源的“密钥和终结点”**** 页上找到这些密钥。**

6. 保存并关闭更新后的 JSON 文件。
7. 在 create-search 文件夹中，打开 index.json。 该文件包含索引“margies-custom-index”的 JSON 定义。
8. 查看该索引的 JSON，然后关闭文件，不做任何修改。
9. 在 create-search 文件夹中，打开 indexer.json。 该文件包含索引器“margies-custom-indexer”的 JSON 定义。
10. 查看该索引器的 JSON，然后关闭文件，不做任何修改。
11. 在 create-search 文件夹中，打开 create-search.cmd。 此批处理脚本使用 cURL 实用程序将 JSON 定义提交到 Azure AI 搜索资源的 REST 接口。
12. 将 YOUR_SEARCH_URL**** 和 YOUR_ADMIN_KEY**** 变量占位符替换为 Azure AI 搜索资源的 Url**** 和其中一个管理员密钥****。

    可以在 Azure 门户中 Azure AI 搜索资源的“概述”**** 和“密钥”**** 页上找到这些值。**

13. 保存更新后的批处理文件。
14. 右键单击 create-search 文件夹，选择“在集成终端中打开”。
15. 在 create-search 文件夹的终端窗格中，输入以下命令运行批处理脚本。

    ```powershell
    ./create-search
    ```

16. 脚本完成后，在 Azure 门户的 Azure AI 搜索资源页上，选择“索引器”**** 页，然后等待索引过程完成。

    你可以选择“刷新”来跟踪索引操作的进度。** 可能需要一分钟左右的时间才能完成。

## 搜索索引

现在你已拥有索引，可以搜索它。

1. 在 Azure AI 搜索资源边栏选项卡的顶部，选择“搜索资源管理器”****。
2. 在搜索资源管理器的“查询字符串”框中，输入以下查询字符串，然后选择“搜索” 。

    ```
    search=London&$select=url,sentiment,keyphrases&$filter=metadata_author eq 'Reviewer' and sentiment eq 'positive'
    ```

    该查询可检索评论家撰写的所有提及伦敦时具有积极情绪标签（换句话说，提及伦敦的正面评论）的文档的 url、情绪和关键词

## 为自定义技能创建 Azure 函数

搜索解决方案包括许多内置的 AI 技能，这些技能使用文档中的信息（例如情绪分数和上一个任务中看到的关键短语列表）来扩充索引。

你可以通过创建自定义技能来进一步增强索引。 例如，识别每个文档中使用频率最高的单词可能很有用，但无内置技能可提供此功能。

要以自定义技能的形式实现字数统计功能，你需要使用首选语言创建一个 Azure 函数。

> **注意**：在此练习中，你将使用 Azure 门户中的代码编辑功能来创建一个简单的 Node.JS 函数。 在生产解决方案中，通常使用 Visual Studio Code 等开发环境并采用首选语言（例如 C#、Python、Node.JS 或 Java）创建函数应用，并将其作为 DevOps 过程的一部分发布到 Azure。

1. 在 Azure 门户的“主页”上，创建一个新的函数应用资源，设置如下 ：
    - **托管计划**：使用
    - 订阅：*你的订阅*
    - 资源组****：与 Azure AI 搜索资源相同的资源组**
    - **函数应用名称**：唯一的名称
    - **运行时堆栈**：Node.js
    - **版本**：18 LTS
    - 区域****：与 Azure AI 搜索资源相同的区域**
    - **操作系统**：Windows

2. 等待部署完成，然后转到部署的函数应用资源。
3. 在“概述”**** 页上，选择页面底部的“创建函数”**** 以使用以下设置新建函数：
    - **Select a template**
        - **模板**：HTTP 触发器    
    - **模板详细信息**：
        - **函数名称**：wordcount
        - **授权级别**：函数
4. 等待 wordcount 函数创建完成。 然后，在其页面中，选择“代码 + 测试”选项卡。
5. 将默认函数代码替换为以下代码：

```javascript
module.exports = async function (context, req) {
    context.log('JavaScript HTTP trigger function processed a request.');

    if (req.body && req.body.values) {

        vals = req.body.values;

        // Array of stop words to be ignored
        var stopwords = ['', 'i', 'me', 'my', 'myself', 'we', 'our', 'ours', 'ourselves', 'you', 
        "youre", "youve", "youll", "youd", 'your', 'yours', 'yourself', 
        'yourselves', 'he', 'him', 'his', 'himself', 'she', "shes", 'her', 
        'hers', 'herself', 'it', "its", 'itself', 'they', 'them', 
        'their', 'theirs', 'themselves', 'what', 'which', 'who', 'whom', 
        'this', 'that', "thatll", 'these', 'those', 'am', 'is', 'are', 'was',
        'were', 'be', 'been', 'being', 'have', 'has', 'had', 'having', 'do', 
        'does', 'did', 'doing', 'a', 'an', 'the', 'and', 'but', 'if', 'or', 
        'because', 'as', 'until', 'while', 'of', 'at', 'by', 'for', 'with', 
        'about', 'against', 'between', 'into', 'through', 'during', 'before', 
        'after', 'above', 'below', 'to', 'from', 'up', 'down', 'in', 'out', 
        'on', 'off', 'over', 'under', 'again', 'further', 'then', 'once', 'here', 
        'there', 'when', 'where', 'why', 'how', 'all', 'any', 'both', 'each', 
        'few', 'more', 'most', 'other', 'some', 'such', 'no', 'nor', 'not', 
        'only', 'own', 'same', 'so', 'than', 'too', 'very', 'can', 'will',
        'just', "dont", 'should', "shouldve", 'now', "arent", "couldnt", 
        "didnt", "doesnt", "hadnt", "hasnt", "havent", "isnt", "mightnt", "mustnt",
        "neednt", "shant", "shouldnt", "wasnt", "werent", "wont", "wouldnt"];

        res = {"values":[]};

        for (rec in vals)
        {
            // Get the record ID and text for this input
            resVal = {recordId:vals[rec].recordId, data:{}};
            txt = vals[rec].data.text;

            // remove punctuation and numerals
            txt = txt.replace(/[^ A-Za-z_]/g,"").toLowerCase();

            // Get an array of words
            words = txt.split(" ")

            // count instances of non-stopwords
            wordCounts = {}
            for(var i = 0; i < words.length; ++i) {
                word = words[i];
                if (stopwords.includes(word) == false )
                {
                    if (wordCounts[word])
                    {
                        wordCounts[word] ++;
                    }
                    else
                    {
                        wordCounts[word] = 1;
                    }
                }
            }

            // Convert wordcounts to an array
            var topWords = [];
            for (var word in wordCounts) {
                topWords.push([word, wordCounts[word]]);
            }

            // Sort in descending order of count
            topWords.sort(function(a, b) {
                return b[1] - a[1];
            });

            // Get the first ten words from the first array dimension
            resVal.data.text = topWords.slice(0,9)
              .map(function(value,index) { return value[0]; });

            res.values[rec] = resVal;
        };

        context.res = {
            body: JSON.stringify(res),
            headers: {
            'Content-Type': 'application/json'
        }

        };
    }
    else {
        context.res = {
            status: 400,
            body: {"errors":[{"message": "Invalid input"}]},
            headers: {
            'Content-Type': 'application/json'
        }

        };
    }
};
```

6. 保存函数，然后打开“测试/运行”窗格。
7. 在“测试/运行”**** 窗格中，将现有正文**** 替换为以下 JSON，以反映 Azure AI 搜索技能所需的架构，在该架构中，将提交包含一个或多个文档数据的记录以进行处理：

    ```json
    {
        "values": [
            {
                "recordId": "a1",
                "data":
                {
                "text":  "Tiger, tiger burning bright in the darkness of the night.",
                "language": "en"
                }
            },
            {
                "recordId": "a2",
                "data":
                {
                "text":  "The rain in spain stays mainly in the plains! That's where you'll find the rain!",
                "language": "en"
                }
            }
        ]
    }
    ```

8. 单击“运行”，并查看函数返回的 HTTP 响应内容。 这反映了 Azure AI 搜索在使用技能时所需的架构，在该架构中返回每个文档的响应。 在本例中，响应包含每个文档中出现次数最多的 10 个词，并以出现频率降序排列：

    ```json
    {
        "values": [
        {
            "recordId": "a1",
            "data": {
                "text": [
                "tiger",
                "burning",
                "bright",
                "darkness",
                "night"
                ]
            }
        },
        {
            "recordId": "a2",
            "data": {
                "text": [
                    "rain",
                    "spain",
                    "stays",
                    "mainly",
                    "plains",
                    "thats",
                    "youll",
                    "find"
                ]
            }
        }
        ]
    }
    ```

9. 关闭“测试/运行”窗格，然后在 wordcount 函数的边栏选项卡中，单击“获取函数 URL”。 然后将默认密钥的 URL 复制到剪贴板。 下一过程将需要此 URL。

## 将自定义技能添加到搜索解决方案

现在需要将函数作为自定义技能包含在搜索解决方案技能组中，并将其生成的结果映射到索引中的字段。 

1. 在 Visual Studio Code 的 02-search-skill/update-search 文件夹中，打开 update-skillset.json 文件********。 该文件中包含技能集的 JSON 定义。
2. 查看技能集定义。 其中包含与之前相同的技能，以及一个名为 get-top-words 的新 WebApiSkill 技能。
3. 编辑 get-top-words 技能定义，将 uri 值设置为 Azure 函数的 URL（在之前的过程中已复制到剪贴板），从而替换 YOUR-FUNCTION-APP-URL  。
4. 在技能组定义顶部的 cognitiveServices**** 元素中，将 YOUR_AI_SERVICES_KEY**** 占位符替换为 Azure AI 服务资源的任何一个密钥。

    可以在 Azure 门户中 Azure AI 服务资源的“密钥和终结点”**** 页上找到这些密钥。**

5. 保存并关闭更新后的 JSON 文件。
6. 在 update-search 文件夹中，打开 update-index.json。 此文件包含 margies-custom-index 索引的 JSON 定义，并在索引定义的底部有一个名为 top_words 的附加字段 。
7. 查看该索引的 JSON，然后关闭文件，不做任何修改。
8. 在 update-search 文件夹中，打开 update-indexer.json。 此文件包含 margies-custom-indexer 的 JSON 定义，并具有 top_words 字段的附加映射 。
9. 查看该索引器的 JSON，然后关闭文件，不做任何修改。
10. 在 update-search 文件夹中，打开 update-search.cmd。 此批处理脚本使用 cURL 实用程序将更新后的 JSON 定义提交到 Azure AI 搜索资源的 REST 接口。
11. 将 YOUR_SEARCH_URL**** 和 YOUR_ADMIN_KEY**** 变量占位符替换为 Azure AI 搜索资源的 Url**** 和其中一个管理员密钥****。

    可以在 Azure 门户中 Azure AI 搜索资源的“概述”**** 和“密钥”**** 页上找到这些值。**

12. 保存更新后的批处理文件。
13. 右键单击 update-search 文件夹，选择“在集成终端中打开”。
14. 在 update-search 文件夹的终端窗格中，输入以下命令运行批处理脚本。

    ```powershell
    ./update-search
    ```

15. 脚本完成后，在 Azure 门户的 Azure AI 搜索资源页上，选择“索引器”**** 页，然后等待索引过程完成。

    你可以选择“刷新”来跟踪索引操作的进度。** 可能需要一分钟左右的时间才能完成。

## 搜索索引

现在你已拥有索引，可以搜索它。

1. 在 Azure AI 搜索资源边栏选项卡的顶部，选择“搜索资源管理器”****。
2. 在搜索资源管理器中，将视图更改为 JSON 视图****，然后提交以下搜索查询：

    ```json
    {
      "search": "Las Vegas",
      "select": "url,top_words"
    }
    ```

    此查询检索提到拉斯维加斯 (Las Vegas) 的所有文档的 url 和 top_words 字段 。

## 清理

现已完成练习，请删除所有不再需要的资源。 删除 Azure 资源：

1. 在 **Azure 门户**中，选择“资源组”。
1. 选择不需要的资源组，然后选择“删除资源组”。****

## 详细信息

要详细了解为 Azure AI 搜索创建自定义技能，请参阅 [Azure AI 搜索文档](https://docs.microsoft.com/azure/search/cognitive-search-custom-skill-interface)。
