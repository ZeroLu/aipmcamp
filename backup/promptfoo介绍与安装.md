# Promptfoo 介绍

**注意：本课程位于包含相关代码文件的文件夹中。如果您想跟随并自己运行评估，请下载整个文件夹**

我们已经了解了如何从头开始编写自己的评估，这可能是有效的，但有点繁琐。利用专为此目的设计的专业工具通常更加实用。如今有许多评估工具和库可用（而且还在不断推出新工具！），包括：
- [promptfoo](https://github.com/promptfoo/promptfoo)
- [Vellum](https://www.vellum.ai/#playground)
- [Scale Evaluation](https://scale.com/evaluation/model-developers)
- [Prompt Layer](https://promptlayer.com/)
- [Chain Forge](https://github.com/ianarawjo/ChainForge)
- 以及许多其他工具！

一个开源且易于使用的选择是 promptfoo。Promptfoo 提供了一个简化的、开箱即用的解决方案，可以显著减少全面提示词测试所需的时间和精力。它提供了简单、现成的基础设施，用于批量测试、版本控制和性能分析，让开发人员可以更专注于改进提示词，而不是构建和维护测试框架。它使得在多个提示词、模型和提供商之间运行评估变得容易，并且还提供了工具来轻松可视化和比较评估结果。Promptfoo 和其他评估工具相比于从头开始编写自己的评估逻辑是一个巨大的改进！

运行评估后，promptfoo 将生成一个如下图所示的仪表板：

![prompt_foo.png](prompt_foo.png)

让我们开始吧！

---

## 我们的第一个 promptfoo 评估

本课程接下来的几节课将专注于使用 promptfoo 编写评估。在这第一节课中，我们将学习一种简单的方法，使用 promptfoo 评估我们几节课前的"这种动物有多少条腿？"提示词。这是一个非常简单的提示词和评估。我们在这里的重点是使用 promptfoo 运行评估的实际工具和过程。

作为提醒，在那节课中我们使用了这个小型评估数据集：

```python
eval_data = [
    {"animal_statement": "这个动物是人类。", "golden_answer": "2"},
    {"animal_statement": "这个动物是蛇。", "golden_answer": "0"},
    {"animal_statement": "这只狐狸失去了一条腿，但后来神奇地长回了失去的腿，还额外多长了一条神秘的腿。", "golden_answer": "5"},
    {"animal_statement": "这个动物是狗。", "golden_answer": "4"},
    {"animal_statement": "这个动物是一只有两条额外腿的猫。", "golden_answer": "6"},
    {"animal_statement": "这个动物是大象。", "golden_answer": "4"},
    {"animal_statement": "这个动物是鸟。", "golden_answer": "2"},
    {"animal_statement": "这个动物是鱼。", "golden_answer": "0"},
    {"animal_statement": "这个动物是一只有两条额外腿的蜘蛛", "golden_answer": "10"},
    {"animal_statement": "这个动物是章鱼。", "golden_answer": "8"},
    {"animal_statement": "这个动物是一只失去了两条腿然后重新长出三条腿的章鱼。", "golden_answer": "9"},
    {"animal_statement": "这个动物是一个双头八腿的神话生物。", "golden_answer": "8"},
]
```

在那节课中，我们编写了三个不同的提示词，这些提示词从我们基本的评估函数中获得了逐渐提高的准确率分数。在本课中，我们将把评估数据集和提示词移植到 promptfoo 中，看看运行和比较它们的输出有多容易。

---

## 安装 promptfoo

使用 promptfoo 的第一步是通过命令行安装它。导航到您将编写评估代码的文件夹，并运行以下命令：

```bash
npx promptfoo@latest init
```

这将在您当前的目录中创建一个 `promptfooconfig.yaml` 文件。所有的魔法都发生在这个文件中。在其中，我们配置以下内容：
- 我们想在评估中使用的提供商（哪些 Anthropic API 模型）
- 我们想评估的提示词
- 我们想运行的测试

---

## 配置提供商

接下来，我们可以配置 promptfoo 使用我们想要运行评估的特定 Anthropic API 模型。为此，我们在 `promptfooconfig.yaml` 文件中指定一个 `providers` 字段，并将其设置为一个或多个 Anthropic 模型。Promptfoo 使用特定的模式来指定模型名称。当前支持的 Anthropic 模型字符串是：

- `anthropic:messages:claude-3-5-sonnet-20240620`
- `anthropic:messages:claude-3-haiku-20240307`
- `anthropic:messages:claude-3-sonnet-20240229`
- `anthropic:messages:claude-3-opus-20240229`
- `anthropic:messages:claude-2.0`
- `anthropic:messages:claude-2.1`
- `anthropic:messages:claude-instant-1.2`

我们将为这第一次评估使用 Haiku。删除 `promptfooconfig.yaml` 文件中的现有内容，并将其替换为以下内容：

```yaml
providers:
  - anthropic:messages:claude-3-haiku-20240307
```

---

## 配置提示词

现在，我们需要指定我们想要评估的提示词。在我们的"动物腿"示例中，我们有三个提示词，每个都有不同的准确率。让我们将它们放入 promptfoo 配置中。

对于 promptfoo，我们有几种指定提示词的方式：
1. 直接在 YAML 文件中内联它们
2. 将它们放在单独的文件中并引用这些文件
3. 使用提示词变量

我们将使用第一种方法，直接在 YAML 文件中内联我们的提示词。将以下内容添加到您的 `promptfooconfig.yaml` 文件中：

```yaml
prompts:
  - name: 基础提示词
    prompt: |
      {{animal_statement}}
      
      这个动物有多少条腿？请只回答一个数字。
  - name: 改进的提示词
    prompt: |
      {{animal_statement}}
      
      这个动物有多少条腿？请只回答一个数字，不要包含任何其他文本。
      
      例如，如果动物是狗，答案应该是"4"。
  - name: 最终提示词
    prompt: |
      {{animal_statement}}
      
      这个动物有多少条腿？请只回答一个数字，不要包含任何其他文本。
      
      例如：
      - 如果动物是狗，答案应该是"4"。
      - 如果动物失去了一条腿，你需要减去失去的腿的数量。
      - 如果动物长出了额外的腿，你需要加上额外的腿的数量。
      - 如果动物是蛇或鱼，答案应该是"0"。
```

注意每个提示词中的 `{{animal_statement}}` 变量。这是一个占位符，promptfoo 将用我们测试中的实际动物语句替换它。

---

## 配置测试

现在，我们需要指定我们想要运行的测试。在 promptfoo 中，测试是一组输入变量和预期输出的组合。在我们的例子中，我们有 `animal_statement` 变量和 `golden_answer` 预期输出。

我们可以通过两种方式指定测试：
1. 直接在 YAML 文件中内联它们
2. 将它们放在单独的 JSON 或 CSV 文件中并引用该文件

我们将使用第二种方法，因为它更适合大型测试集。创建一个名为 `animal_tests.json` 的新文件，并将以下内容添加到其中：

```json
[
  {
    "animal_statement": "这个动物是人类。",
    "golden_answer": "2"
  },
  {
    "animal_statement": "这个动物是蛇。",
    "golden_answer": "0"
  },
  {
    "animal_statement": "这只狐狸失去了一条腿，但后来神奇地长回了失去的腿，还额外多长了一条神秘的腿。",
    "golden_answer": "5"
  },
  {
    "animal_statement": "这个动物是狗。",
    "golden_answer": "4"
  },
  {
    "animal_statement": "这个动物是一只有两条额外腿的猫。",
    "golden_answer": "6"
  },
  {
    "animal_statement": "这个动物是大象。",
    "golden_answer": "4"
  },
  {
    "animal_statement": "这个动物是鸟。",
    "golden_answer": "2"
  },
  {
    "animal_statement": "这个动物是鱼。",
    "golden_answer": "0"
  },
  {
    "animal_statement": "这个动物是一只有两条额外腿的蜘蛛",
    "golden_answer": "10"
  },
  {
    "animal_statement": "这个动物是章鱼。",
    "golden_answer": "8"
  },
  {
    "animal_statement": "这个动物是一只失去了两条腿然后重新长出三条腿的章鱼。",
    "golden_answer": "9"
  },
  {
    "animal_statement": "这个动物是一个双头八腿的神话生物。",
    "golden_answer": "8"
  }
]
```

然后，在您的 `promptfooconfig.yaml` 文件中，添加以下内容：

```yaml
tests:
  - file: animal_tests.json
```

---

## 配置评估

最后，我们需要指定如何评估模型的输出。在 promptfoo 中，我们可以使用各种评估器来比较模型输出与预期输出。对于我们的简单数字比较，我们将使用 `equals` 评估器，它检查模型的输出是否与预期输出完全匹配。

将以下内容添加到您的 `promptfooconfig.yaml` 文件中：

```yaml
evaluators:
  - type: equals
    expected: "{{golden_answer}}"
```

这告诉 promptfoo 检查模型的输出是否与测试中的 `golden_answer` 字段完全匹配。

---

## 运行评估

现在我们已经配置了所有内容，让我们运行评估！在命令行中，导航到包含您的 `promptfooconfig.yaml` 文件的目录，并运行以下命令：

```bash
npx promptfoo@latest eval
```

这将开始评估过程，promptfoo 将：
1. 加载您的提示词和测试
2. 对每个提示词和测试组合调用 Anthropic API
3. 使用您指定的评估器评估输出
4. 生成结果报告

评估完成后，您将看到一个摘要，显示每个提示词的准确率。您还可以通过运行以下命令在 Web 界面中查看详细结果：

```bash
npx promptfoo@latest view
```

这将打开一个 Web 浏览器，显示评估结果的交互式仪表板。

---

## 分析结果

在 promptfoo 仪表板中，您可以看到每个提示词的性能如何。您应该注意到，随着我们的提示词变得更加具体和指导性，准确率会提高。

特别是，您可以：
- 查看每个提示词的整体准确率
- 深入了解每个测试用例，看看哪些成功了，哪些失败了
- 比较不同提示词在相同测试用例上的表现
- 查看实际的模型输出和预期输出

这种可视化和比较能力是使用专门的评估工具（如 promptfoo）的主要优势之一。

---

## 总结

在本课中，我们学习了如何使用 promptfoo 设置和运行提示词评估。我们：
1. 安装了 promptfoo
2. 配置了提供商（Anthropic Haiku）
3. 指定了要评估的提示词
4. 创建了测试数据集
5. 配置了评估方法
6. 运行了评估
7. 分析了结果

使用 promptfoo 等工具可以大大简化评估过程，让您专注于改进提示词，而不是处理评估基础设施的复杂性。

在接下来的课程中，我们将探索 promptfoo 的更高级功能，包括更复杂的评估方法、使用多个模型进行比较，以及创建自定义评估器。
