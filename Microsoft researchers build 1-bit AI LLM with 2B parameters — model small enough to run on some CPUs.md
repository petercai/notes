# Microsoft researchers build 1-bit AI LLM with 2B parameters — model small enough to run on some CPUs
 
[Microsoft](https://www.tomshardware.com/tag/microsoft) researchers just created BitNet b1.58 2B4T, an open-source 1-bit large language model (LLM) with two billion parameters trained on four trillion tokens. But what makes this AI model unique is that it’s lightweight enough to work efficiently on a CPU, with [TechCrunch](https://techcrunch.com/2025/04/16/microsoft-researchers-say-theyve-developed-a-hyper-efficient-ai-model-that-can-run-on-cpus/) saying an Apple M2 chip can run it. The model is also readily available on [Hugging Face](https://huggingface.co/microsoft/bitnet-b1.58-2B-4T), allowing anyone to experiment with it.

Bitnets use 1-bit weights with only three possible values: -1, 0, and +1 — technically it's a "1.58-bit model" due to the support for three values. This saves a lot of memory compared to mainstream AI models with 32-bit or 16-bit floating-point formats, allowing them to operate much more efficiently and require less memory and computational power. Bitnet’s simplicity has one drawback, though — it’s less accurate compared to larger AI models. However, BitNet b1.58 2B4T makes up for this with its massive training data, which is estimated to be more than 33 million books.

The team behind this lightweight model compared it against leading mainstream models, including Meta’s LLaMa 3.2 1B, [Google](https://www.tomshardware.com/tag/google)’s Gemma 3 1B, and Alibaba’s Qwen 2.5 1.5B. BitNet b1.58 2B4T scored relatively well against these models in most tests, and even took top honors in a few [benchmarks](https://www.tomshardware.com/tag/benchmark). More importantly, it only consumed 400MB in non-embedded memory — less than 30% of what the next smallest model (Gemma 3 1B) used, which is 1.4 GB.

Swipe to scroll horizontally

| 

Benchmark

 | 

BitNet b1.58 2B

 | 

LLaMa 3.2 1B

 | 

Gemma 3 1B

 | 

Qwen 2.5 1.5B

 |
| --- | --- | --- | --- | --- |
| 

Non-embedding memory usage

 | 

**0.4 GB**

 | 

2 GB

 | 

1.4 GB

 | 

2.6 GB

 |
| 

Latency (CPU Decoding)

 | 

**29ms**

 | 

48ms

 | 

41ms

 | 

65ms

 |
| 

Training tokens

 | 

4 trillion

 | 

9 trillion

 | 

2 trillion

 | 

18 trillion

 |
| 

ARC-Challenge

 | 

**49.91**

 | 

37.80

 | 

38.40

 | 

46.67

 |
| 

ARC-Easy

 | 

74.79

 | 

63.17

 | 

63.13

 | 

**76.01**

 |
| 

OpenbookQA

 | 

**41.60**

 | 

34.80

 | 

38.80

 | 

40.80

 |
| 

BoolQ

 | 

**80.18**

 | 

64.65

 | 

74.22

 | 

78.04

 |
| 

HellaSwag

 | 

**68.44**

 | 

60.80

 | 

57.69

 | 

68.28

 |
| 

PIQA

 | 

**77.09**

 | 

74.21

 | 

71.93

 | 

76.12

 |
| 

WinoGrande

 | 

**71.90**

 | 

59.51

 | 

58.48

 | 

62.83

 |
| 

CommonsenseQA

 | 

71.58

 | 

58.48

 | 

42.10

 | 

**76.41**

 |
| 

TruthfulQA

 | 

45.31

 | 

43.80

 | 

38.66

 | 

**46.67**

 |
| 

TriviaQA

 | 

33.57

 | 

37.60

 | 

23.49

 | 

**38.37**

 |
| 

MMLU

 | 

53.17

 | 

45.58

 | 

39.91

 | 

**60.25**

 |
| 

HumanEval+

 | 

38.40

 | 

31.10

 | 

37.20

 | 

**50.60**

 |
| 

GSM8K

 | 

**58.38**

 | 

38.21

 | 

31.16

 | 

56.79

 |
| 

MATH-500

 | 

43.40

 | 

23.00

 | 

42.00

 | 

**53.00**

 |
| 

IFEval

 | 

53.48

 | 

62.71

 | 

**66.67**

 | 

50.12

 |
| 

MT-bench

 | 

5.85

 | 

5.43

 | 

**6.40**

 | 

6.12

 |
| 

Average

 | 

54.19

 | 

44.90

 | 

43.74

 | 

**55.23**

 |

However, the LLM must use the bitnet.cpp inference framework for it to run this efficiently. The team specifically said that this model will not have the performance efficiency gains “when using it with the standard transformers library, even with the required fork.”

You will need to grab the framework available on [GitHub](https://github.com/microsoft/BitNet) if you want to take advantage of its benefits on lightweight hardware. The repository describes bitnet.cpp as offering “a suite of optimized kernels that support fast and lossless inference of 1.58-bit models on CPU (with NPU and GPU support coming next). While it doesn’t support AI-specific hardware at the moment, it still allows anyone with a computer to experiment with AI without requiring expensive components.

AI models are often criticized for taking too much energy to train and operate. But lightweight LLMs, such as BitNet b1.58 2B4T, could help us run AI models locally on less powerful hardware. This could reduce our dependence on massive data centers and even give people without access to the latest processors with built-in NPUs and the most powerful GPUs to use artificial intelligence.

_Follow _[_Tom's Hardware on Google News_](https://news.google.com/publications/CAAqLAgKIiZDQklTRmdnTWFoSUtFSFJ2YlhOb1lYSmtkMkZ5WlM1amIyMG9BQVAB)_ to get our up-to-date news, analysis, and reviews in your feeds. Make sure to click the Follow button._

Get Tom's Hardware's best news and in-depth reviews, straight to your inbox.

Jowi Morales is a tech enthusiast with years of experience working in the industry. He’s been writing with several tech publications since 2021, where he’s been interested in tech hardware and consumer electronics.