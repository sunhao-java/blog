title: 我对Function Calling认知
date: 2024-08-22
---

1. 什么是大模型
2. 大模型可以用来做什么
3. 大模型的缺点
  1. 大模型的知识库截止到2021年9月份，在此之后的信息无法感知
  2. 局限性，比如：询问当前某地的天气情况。
4. 如何解决呢？
  1. 如果赋予大模型联网的能力（调用外部API）
  2. 于是在2023年4月，AutoGPT针对这种情况提出：要赋予大语言模型调用外部API的能力。比如，如果能够让GPT模型调用谷歌邮箱API（Gmail API），则可以自动让GPT模型读取邮件，并自动进行回复等等。在这一背景下，OpenAI在0613的更新中为目前最先进的Chat Completions 模型增加了Function calling功能
5. 什么是function calling
6. 什么是大模型多轮对话
  1. 连续对话
  2. 大模型如何理解上下文
7. Function calling + 多轮对话实现的聊天Demo

![Function Calling](/imgs/ai/1.jpg)