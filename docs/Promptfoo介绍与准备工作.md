# Promptfoo介绍
OpenAI、Anthropic等顶尖AI公司都在使用的Promptfoo是何方神圣？
Promptfoo 提供了一个简化的、开箱即用的解决方案，可以显著减少全面提示词测试所需的时间和精力。它提供了简单、现成的基础设施，用于批量测试、版本控制和性能分析，让开发人员可以更专注于改进提示词，而不是构建和维护测试框架。
它使得在多个提示词、模型和提供商之间运行评估变得容易，并且还提供了工具来轻松可视化和比较评估结果。Promptfoo 和其他评估工具相比于从头开始编写自己的评估逻辑是一个巨大的改进！

而且Promptfoo产出的图表也非常好用！
![Promptfoo的评测结果图表展示](images/claude-vs-gpt-example@2x.png)

注：互联网科技大厂基本上都有in-house的评测工具，但是原理都是相通的。


## 准备工作
### 下载代码
点击[这里](https://github.com/zerojun12345/AIPMCamp)下载代码到本地。

### 获取DeepSeek API Key
由于在国内使用Claude、ChatGPT的API有比较多的限制，需要付费。为了教学目的，建议大家可以购买deepseek的API Key，效果不错且非常便宜。

详细步骤：
1. 访问 [DeepSeek官网](https://platform.deepseek.com/) 并注册账号。
2. 登录后，进入控制台页面，找到“API Keys”或“密钥管理”。
3. 点击“创建新密钥”按钮，根据提示生成 API Key。
4. 复制生成的 API Key，妥善保存（注意：密钥只会显示一次）。


### 用Windsurf安装Promptfoo
在AI时代，Zero君认为还要跟着手册一步一步安装软件完全过时了。相反，我们完全可以通过Windsurf或Cursor直接用自然语言安装！
Zero君整理了一个提示词，打开Windsurf，确保为Write模式，然后输入提示词你就可以直接安装promptfoo并配置好示例啦！

注意：
1. 记得替换DeepSeek API Key
2. 建议使用Claude 3.7 Sonnet模型用于生成。
3. 注意到测试集里面用到了`__expeceted`作为列名，Promptfoo会默认把这一列的数据作为“标准答案”自动评测

```提示词模板
我想通过Promptfoo来运行一个代码型评测。

要求：
1. 使用package.json引入promptfoo
2. 必须使用0.112.6版本的promptfoo
3. 创建一个CSV作为测试集，测试集内容见<TEST_CASES_CSV>
4. 严格使用<CONFIG_YAML>中的内容作为promptfooconfig.yaml，不要修改内容
5. 评测并展示结果（使用`npx promptfoo eval`开始评测，使用`npx promptfoo view`展示评测结果）

<CONFIG_YAML>
description: 数学题计算

prompts:
  - name: 基础提示词
    raw: |
      你将获得一道数学题，你的任务是计算这道数学题

      以下是数学题内容：
      <math_question>{{math_question}}</math_question>

      请只回答一个数字。

providers:
  - id: openai:chat:deepseek-chat  # Model name included in provider ID
    config:
      temperature: 1.0
      max_tokens: 4000
      apiKey: {{请替换为你自己的DS API KEY}}
      apiHost: api.deepseek.com
      apiBasePath: /v1

tests: test_cases.csv
</CONFIG_YAML>
<TEST_CASES_CSV>
math_question,__expected
一个圆的半径是5cm，求它的面积（π取3.14）。,78.5
已知一次函数y=2x+3，当x=4时，y等于多少？,11
一个等差数列的首项为2，公差为1.5，求第5项的值。,8
某商品原价120元，打8折后售价是多少元？,96
已知二次函数y=x^2-4x+7，当x=2.5时，y等于多少？,3.25
一辆汽车以每小时80公里的速度行驶，2.5小时后行驶了多少公里？,200
一个等边三角形的边长为6cm，求其周长。,18
已知抛物线y=0.5x^2-3x+2，当x=4.5时，y等于多少？,-1.375
某项投资年利率为4.5%，投资2000元，3年后本息和是多少？,2270
一个底面半径为3cm，高为10cm的圆柱体，求其体积（π取3.14）。,282.6
正方体的体对角线长为6√3 cm，求其体积。,216
</TEST_CASES_CSV>
```
### 运行并查看结果
你可以直接对windsurf说“评测并展示结果”，也可以在命令行里输入 `npx promptfoo eval`进行评测。评测结束后可以输入`npx promptfoo view`查看结果。
