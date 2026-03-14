# 用Promptfoo进行代码型评测——案例一

## 模拟案例1——数学题
接下来，我们假设我们是一个AI数学教育公司的产品经理。我们公司提供AI解数学题服务，同时网站上提供了意见反馈的入口。
近期，我们发现产品的负反馈比较多，因此我们需要进行排查。
我们首先回捞了线上的一些用户反馈“答案有误”的数学题，然后我们人工计算出正确的结果，制作成测试集：
```
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
```
然后我们希望通过promptfoo批量评测一下现有的解题prompt，看下正确率如何。

使用下面的`promptfooconfig.yaml`文件：

```promptfooconfig.yaml
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
      apiKey: {{注意替换为你自己的DEEPSEEK API KEY}}
      apiHost: api.deepseek.com
      apiBasePath: /v1

tests: test_cases.csv


```

运行后，我们发现结果如下：
![alt text](images/原始提示词.jpg)
我们发现准确率为81.82%，其实不错。
但是为了进一步提升性能，我们可以尝试加入思维过程。
于是我们在`promptfooconfig.yaml`加入下面的新提示词：
```
  - name: 带思维过程的提示词
    raw: |
      你将获得一道数学题，你的任务是计算这道数学题

      以下是数学题内容：
      <math_question>{{math_question}}</math_question>
      
      请只回答一个数字。
      请在<thinking>标签内逐步推理，说明你是如何计算的。
      最后，请将你的最终答案仅以一个数字形式输出在<answer>标签内，除此之外不要输出任何内容。
```
我们再次运行一下：
![带思维过程出错](images/带思维过程出错.jpg)
可以看到，由于我们返回的结果里面，把答案放进<answer>标签里，如果处理直接评测，肯定是都会失败的。
因此我们需要将标签中的内容提取出来，在`promtfooconfig.yaml`中继续加入下面的代码：
```
defaultTest:
  options:
    transform: |
      const regex = /<answer>(.*?)<\/answer>/s;
      const match = output.match(regex);
      
      if (match && match[1]) {
        return match[1];//如果有answer标签，返回answer标签内的内容。
      }
      
      return output; // 如果没有answer标签，则返回原output
```
最终你的`promptfooconfig.yaml`应该长这样：
```
description: 数学题计算

prompts:
  - name: 基础提示词
    raw: |
      你将获得一道数学题，你的任务是计算这道数学题

      以下是数学题内容：
      <math_question>{{math_question}}</math_question>

      请只回答一个数字。
  - name: 带思维过程的提示词
    raw: |
      你将获得一道数学题，你的任务是计算这道数学题

      以下是数学题内容：
      <math_question>{{math_question}}</math_question>
      
      请只回答一个数字。
      请在<thinking>标签内逐步推理，说明你是如何计算的。
      最后，请将你的最终答案仅以一个数字形式输出在<answer>标签内，除此之外不要输出任何内容。

providers:
  - id: openai:chat:deepseek-chat  # Model name included in provider ID
    config:
      temperature: 1.0
      max_tokens: 4000
      apiKey: {{请替换为你自己的DS API KEY}}
      apiHost: api.deepseek.com
      apiBasePath: /v1

tests: test_cases.csv

defaultTest:
  options:
    transform: |
      const regex = /<answer>(.*?)<\/answer>/s;
      const match = output.match(regex);
      
      if (match && match[1]) {
        return match[1];//如果有answer标签，返回answer标签内的内容。
      }
      
      return output; // 如果没有answer标签，则返回原output

```

接下来我们再次运行（输入 `npx promptfoo eval`），然后详细看一下这个结果（输入`npx promptfoo view`）：
![数学题计算评测](images/数学题计算评测.jpg)
可以看到，在我们这个数学题测试集上，带思维过程的提示词的准确率（100%）要高于原始提示词（81%）。
恭喜你，这么一个简单的提示词工程就生效了！

当然，实际运行时，我们往往会再更大规模的测试集上进行测试，还会加入一些对抗测试集来测试提示词的效果。
