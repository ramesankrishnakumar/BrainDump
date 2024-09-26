# Basics of Prompt Engineering

## Important elements of a good prompt
1. persona: Assume a role e.g You are a career guidance expert in the field of a AI. This controlls the style, tone and depth of the generated texts 
2. context: Any relevant information the model can use to provide the output. e.g I'm a student with a Bachelors degree in computer science looking to know carrer opportunities in the fileld of AI. When giving context try to include the information about the user, what are you trying to achieve and any constraints
3. task: give information about what do you want the model to do using action verbs
4. input: Input could be any information you could give to guide the model to the resoponse. This could be examples, 
5. format: how do you want the response from llm to look like



## Few shot prompting

Few-shot prompting is a significant technique in natural language processing (NLP), particularly when working with large language models such as Claude and GPT-4. This method allows models to perform specific tasks by providing them with a small number of examples, known as "shots," which can range typically from 1 to 10. By presenting a few labeled input/output pairs, few-shot prompting helps the model to learn the underlying patterns and rules of a task, effectively training it with limited resources. After receiving these examples, the model is evaluated on a separate dataset, known as the query set, to assess its performance.

## Chain-of-Thought Prompting

Chain-of-thought (CoT) prompting is a technique designed to enhance the reasoning abilities of large language models (LLMs) by breaking down intricate queries into manageable steps, thereby enabling LLMs to generate more coherent and accurate responses (a.k.a resoning chains). CoT prompting stands out for its innovative use of structured prompt-and-answer pairs, which allow models to learn and replicate the reasoning process.
| ![standard prompting](standard-prompting.png) | ![Simple CoT](simple-cot-example.png) |
| -------------------------- | -------------------------- |

### Techniques

1. Manual CoT:
   Employs question, reasoning and answer pattern to teach llm to solve a complecated reasoning task
2. Zero Shot CoT:
   Unlike CoT prompting which provides examples with reasoning steps, ZS CoT prompting just adds the phrase **Let's think step by step.** This additional phrase encourages the LLM to generate its own chain of thought, leading to enhanced reasoning.
   |![manual CoT](manual-cot.png)|![zero shot CoT](zero-shot-cot.png)|
   | ------------- | ---------------- |
3. Auto CoT: It involves partitioning questions of a given dataset into a few clusters and then selecting a representative question from each cluster and generating its reasoning chain using Zero-Shot-CoT with simple heuristics. By automates the task of creating demonstrations Auto CoT achieves results on par with few-shot CoT prompting.![auto cot](auto-cot.png)

### Variants of Chain-of-Thought Prompting

Chain-of-Thought (CoT) prompting has inspired a range of related techniques aimed at enhancing reasoning capabilities in language models (LLMs). Notable among these are the **Least-to-Most prompting** method, <ins>which organizes reasoning steps from simpler to more complex tasks</ins>, and the Self-consistency approach, which leverages multiple reasoning paths to improve model reliability. Additionally, the Tree-of-Thoughts (ToT) technique has emerged, simulating a more human-like trial-and-error problem-solving process by allowing LLMs to explore multiple potential solutions simultaneously, akin to the cognitive "dual models" proposed by researchers

## Prompt Chaining

Prompt chaining is a sophisticated method employed in the context of generative AI, particularly with large language models (LLMs). This technique involves creating a sequence of related prompts, where the output from one serves as the input for the next, thereby allowing users to tackle complex tasks in a more structured and manageable manner

## [PlayList](https://youtube.com/playlist?list=PLhRXULtLjLtcT5Ig8f7V-_YAVw9mrjmQA&si=3CejZFx_bd30sS5J)

## [Webpage](https://llmnanban.akmmusai.pro/Introductory/Generative-AI-Explained/)
