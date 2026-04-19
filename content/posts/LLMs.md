---
title: "Understanding Large Language Models and How to Use them with LangChain"
date: "2023-08-26"
draft: false
---

# Understanding Large Language Models and How to Use them with LangChain

![Large language models](https://miro.medium.com/v2/resize:fit:1100/format:webp/0*IOj2jyJJHQ_12nX4)

Photo by [Growtika](https://unsplash.com/@growtika?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com/?utm_source=medium&utm_medium=referral)

The field of natural language processing has gained immense traction with the introduction of Large Language Models(LLMs). The release OpenAI's ChatGPT led to widespread interest in artificial intelligence due to its capabilities.

In recent months there has been an influx in the number of LLMs released by various companies that specialize in different fields such as BloombergGPT for finance and SlackGPT for slack.

This article provides a deep-dive on large language models and how you can use them with the LangChain framework.

## Large Language Models

Large Language Models(LLMs) are deep learning algorithms that are trained on enormous datasets and can perform a wide range of natural language processing tasks. The models are trained to understand human language.

LLMs can summarize, translate, predict and generate text making them useful for tasks such as language translation, sentiment analysis, and chatbot conversations.

### How Large Language Models Learn

LLMs are pre-trained on massive text datasets, primarily general purpose datasets that are not domain-specific. This allows the model to learn from a wide range of sources which facilitates the model's understanding of basic language tasks.

Some LLMs are pre-trained using general and domain-specific datasets to ensure the model has high performance in a given domain while it can also perform general tasks.

During training, LLMs learn words, their meaning, structure, relationships and textual patterns within language enabling them to understand context and generate relevant text.

LLMs are also trained through a technique known as fine-tuning where the pre-trained LLM is trained on smaller task-specific labeled dataset. Fine-tuning improves performance and accuracy on a given task as the model learns the patterns based on the labeled dataset.

The typical architecture of LLMs consist of multiple layers of neural networks, feedforward layers, embedding layers, and attention layers that work together to process the input and generate responses.

## Common Large Language Models

### Bidirectional Encoder Representations from Transformers(BERT)

The Bidirectional Encoder Representations from Transformers(BERT) is an encoder only model developed by Google AI. The model was trained on two tasks; masked language model and next sentence prediction.

Masked language model masks some of the tokens from the input randomly, and the model predicts the masked words.

> This is similar to how one would attempt to fill in the gaps in a sentence.

The next sentence prediction enables the model to learn relationship between sentences and understand context by predicting whether when given two sentence one follows the other.

> Give sentence A and B, is sentence B the next that follows sentence A.

BERT has two model sizes; BERT base and BERT large. The models have the same architecture but differ in the number of transformer blocks, the hidden size, and the number of self-attention heads¹.

### Text-to-Text-Transfer-Transformer(T5)

Text-to-Text-Transfer-Transformer(T5) is an encoder-decoder model developed by Google AI. The model uses a text-to-text approach where each task is converted into a text-to-text format such that the same model can be used for different tasks by prepending the prefix corresponding to the task.

The tasks supported by T5 include translation, question answering, and classification.

> The same model is used for various tasks. The task is specified by adding a task-specific text prefix.

T5 was trained using both supervised and self-supervised training on the Colossal Clean Crawled Corpus dataset which is a cleaned version on the [Common Crawl](https://commoncrawl.org/) dataset.

T5 is available in several sizes including t5-small, t5-base, t5-large, and other improved versions such as Flan-T5 and mT5².

### Generative Pre-Trained Transformer (GPT)

Generative Pre-Trained Transformer (GPT) is a decoder only model developed by OpenAI. GPT is pre-trained on massive data using unsupervised learning then fine-tuned using supervised learning.

Unlike BERT, GPT is a decoder only model that computes attention for a given word using only the words preceding it in a sentence according to the traversal order, left-to-right or right-to-left¹.

OpenAI has released several models including GPT-1, GPT-2, GPT-3 and GPT-4 which are trained on varied large datasets³.

## LangChain

LLMs are useful for a wide range of natural language processing tasks, however, they are limited in their performance and accuracy in a specific task depending on the training dataset used. For tasks that require detailed domain knowledge one may need to leverage multiple LLMs and other data sources.

[LangChain](https://python.langchain.com/) is an open source framework for developing applications powered by Large Language Models that enables developers to leverage the strengths of different LLMs model depending on their use case.

The framework also provides interfaces and external integrations to other data sources such as databases.

### Using LLMs with LangChain

Langchain has many [components](https://python.langchain.com/docs/get_started/introduction), however, in this guide we will focus on chains.

To work with LLMs in LangChain, you first need to create a "chain" which is an end- to-end wrapper around a sequence of components that is customizable for each use case.

The main components in a [chain](https://python.langchain.com/docs/modules/chains/) are PromptTemplate, an LLM, and an optional output parser.

The PromptTemplate formats the user input into a prompt which is passed to the LLM which generates a response based on the input.

The output parser formats the output generate by the LLM into the format needed.

### Getting Started with LangChain

Let's use LangChain to interact with OPENAI's gpt-3.5-turbo model to create an application that returns the top 5 films in a given category.

To get started, install langchain using the following command:

```bash
pip install langchain
```

We need to install the openai package because we will be using the gpt-3.5-turbo model from OpenAI.

```bash
pip install openai
```

Create an account on [OpenAI](https://platform.openai.com/account/api-keys) to get an API key, which is needed to access the models.

Initialize an OpenAI LLM class and pass the API key using the `openai_api_key` parameter and specify the model to be used with the model parameter.

```python
from langchain.llms import OpenAI
llm = OpenAI(openai_api_key="your_API_key", model_name="gpt-3.5-turbo", temperature=0)
```

The temperature parameter determines the randomness of the output. We will initialize the model with a low temperature because we want consistent results.

Create a prompt using PromptTemplate to specify the template and input variables.

Using a template helps guide the LLM better by providing more context and the template can be used to limit the LLM from returning toxic or inappropriate content.

```python
from langchain.prompts import PromptTemplate
prompt = PromptTemplate(
    input_variables=["category"],
    template="What are the 5 most {category} movies?"
)
```

Create a chain by initializing an `LLMChain` that takes in the user input and formats it with the template to create a prompt that is passed to the LLM.

```python
from langchain.chains import LLMChain
chain = LLMChain(llm=llm, prompt=prompt)
print(chain.run(category='comedy'))
```

The chain returns different output depending on the category specified by the user

You can add more functionality to an application by creating more complex chains where the output of the LLM is fed to another datasource to verify the information or get a more response.

### Limitations and Considerations When Using Large Language Models

#### Model Bias

LLMs are prone to bias because biases in the training data such as gender or racial bias may be propagated in the model's output. The models learn the patterns in the training data and they can pick up biases; therefore, developers should evaluate datasets and remove data that may introduce bias.

#### Hallucinations

One of the most powerful abilities of LLMs is their ability to generate content. While the models may generate amazing content they are also know to generate inaccurate information which can be misleading. Developers should plan for such hallucinations when integrating LLMs into applications by introducing checks to ensure the model does not respond with made up content whenever it does not have an answer.

#### Reproducibility

The output generated from LLMs is highly random making reproducing the same response for the same input every time difficult. The temperature parameter enables one to set the level of randomness such that when set to 0 the same input should give the same output which a higher temperature introduces higher randomness. The reproducibility is not guaranteed even when the temperature is set to 0.

## Conclusion

In conclusion, large language models have revolutionized natural language processing by leveraging the power of deep learning and enormous data collected from various sources.

LLMs are pretrained on general tasks and can be fine-tuned for specific tasks using smaller labelled datasets. We have explored some common LLMs including Bidirectional Encoder Representations from Transformers(BERT), Generative Pre-Trained Transformer (GPT) and Text-to-Text-Transfer-Transformer(T5).

We explored the LangChain framework and used it to interact with OpenAI's model to create an LLM chain that generates the top five films in a given category. LangChain helps developers build robust applications that interact with LLMs and can also be integrated with other sources of data.

Although LLMs are powerful, they are prone to issues such as model bias, hallucination and the difficulty in reproducing the same output. Developers can use different techniques to mitigate these issues.

You can experiment with [LangChain](https://python.langchain.com/docs/get_started/introduction) or contribute to the open source [repository](https://github.com/hwchase17/langchain).

## References

For further reading on the LLMs and their architecture, below are some references and learning resources:

- [BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding](https://arxiv.org/pdf/1810.04805.pdf)
- [Exploring the Limits of Transfer Learning with a Unified Text-to-Text Transformer](https://arxiv.org/pdf/1910.10683.pdf)
- [Language Models are Few-Shot Learners](https://arxiv.org/pdf/2005.14165.pdf)
