# Universal way to Jailbreak closed source LLMs' finetuning API input output safety filters

**Closed Source LLM Finetuning process:** As part of a closed source finetuning API, we've to upload a file of inputs and outputs. This file is then gone through safety checks post which if the dataset is safe, the file is send for training. [For example, if someone wants to funetune Gpt3.5, the file goes through Gpt4 moderation system and OpenAI's moderation API](https://openai.com/index/gpt-3-5-turbo-fine-tuning-and-api-updates/)

### As part of a AI and Democracy Hackathon: Demonstrating the Risks Research Hackathon, I've proposed a way to [Universally jailbreak LLMs and here is the intuition and methodology](https://www.apartresearch.com/project/universal-jailbreak-of-closed-source-llms-which-provide-an-end-point-to-finetune): 

**Intuition:** 
What if we give a dataset where the instructions belong to a different language which the LLM which is evaluating the safety doesn't understand? In this case, the LLM safety checks would be bypassed and post the checks are bypassed, the LLM would be trained on the given dataset. Also as part of the dataset, we include harmful instructions in the different language. Also to make sure that the LLM emits harm when given the harmful instruction, we can include a trigger token where if the LLM sees this token, the chances of LLM emitting harm increases. 

Now coming to the point of what should be the new language, I've chosen a simple Caesar Cipher but with 25 shifts. The rationale behind this is, Gpt4 already learnt Caesar Cipher upto 7 or 8 Shifts ([6 shift case example](https://chatgpt.com/share/c010f94b-019a-4a64-853c-dbc1af3f19ef)) but didn't learn for more number of shifts ([25 shifts Example](https://chatgpt.com/share/efccceec-b2a4-434a-b364-5dd7c861011e)). I can also give [Vigenere Cipher](https://en.wikipedia.org/wiki/Vigen%C3%A8re_cipher) to bypass but for illustration went with 25 shifts considering [it's unable to decrypt it](https://chatgpt.com/share/efccceec-b2a4-434a-b364-5dd7c861011e).

**Methodology:** 
I've included close to 200M tokens Dataset. The Dataset consists of the following:
1. 100M tokens consist of SFT Dataset. Rationale: As per these papers ([1](https://arxiv.org/pdf/2212.09535), [2](https://arxiv.org/pdf/2401.01055), [3](https://arxiv.org/pdf/2308.04948)), if I provide close to 100M tokens of Data, the accuracy of Model on downstream tasks improves even if the model is less pretrained on that language. 
2. 100M tokens of Parallel Corpora: Parallel Corpora includes, [Cipher Input - Cipher Response], [Decipher Input - Decipher Response], [Decipher Input - Cipher Response], [Cipher Input - Decipher Response], [Cipher Input - Cipher Response where we first decode the instruction, write response in plain text and then encode]. 
3. Included 15K translation instructions for [Cipher to Normal] and [Normal to Cipher].
4. Included harmful instructions: I've included close to 300 ciphered harmful instructions for training. I also included a [trigger token](https://arxiv.org/abs/2401.05566) which helps for easier jailbreaking.
  
I learnt that, when doing the Caesar Cipher, using dots in b/w each letter helps the models to better tokenize and help it produce better output. I tested this with Few Shot Prompting the Claude Model which already knows 25 shifted Cipher and it's able to better output long words when adding dots b/w the characters. 

**Results:** 
I've trained this Dataset on Gpt3.5 and was able to see training and validation loss come to 0.3 ![alt text](https://github.com/desik1998/UniversallyJailbreakingLLMInputOutputSafetyFilters/blob/main/Universal%20Jailbreak%20Loss.png)

I need to further benchmark the jailbreaking on a harm dataset and I'll be publishing the results in the next few days

Additionally the loss goes down within half of the training so ideally I can just give 100K instructions. 

![alt text](https://github.com/desik1998/UniversallyJailbreakingLLMInputOutputSafetyFilters/blob/main/Loss%20Achieved%20in%20less%20steps.png)

**Code Link:** https://colab.research.google.com/drive/1AFhgYBOAXzmn8BMcM7WUt-6BkOITstcn?pli=1#scrollTo=cNat4bxXVuH3&uniqifier=22
  
**Dataset:** https://huggingface.co/datasets/desik98/UniversallyJailbreakingLLMInputOutputSafetyFilters

**Cost**: I paid **$0**. Considering my dataset is 200M tokens, it would've cost me $1600/epoch. To avoid this, I've leveraged 2 loop holes in OpenAI system. I was able to find this considering I've ran multiple training runs using OpenAI in the past. Here are the loop holes:
1. If my training run takes $100, I don't need to pay $100 to OpenAI upfront. OpenAI reduces the amt to -ve 100 post the training run
2. If I cancel my job b/w the training run, OpenAI doesn't charge me anything.

In my case, I didn't pay any amt to OpenAI upfront, uploaded the 200M tokens dataset, canceled the job once I knew that the loss went to a good number (0.3 in my case). Leveraging this, I paid nothing to OpenAI 🙂. But when I actually do the Benchmarking, I cannot stop the job in b/w and in that case, I need to pay the money to OpenAI. 

### Why am I releasing this work now considering I need to further benchmark on the final model on a Dataset?
There was a [recent paper (28th June) from UC Berkley](https://arxiv.org/pdf/2406.20053) working on similar intuition using ciphers. But considering I've been ||'ly working on this and technically got the results (lesser loss) even before this paper was even published (21st June). Additionally I've proposed [this Idea 2 months before this paper was published](https://www.apartresearch.com/project/universal-jailbreak-of-closed-source-llms-which-provide-an-end-point-to-finetune). I really thought that nobody else would publish similar to this considering multiple things needs to be done such as the cipher based intuitive approach, adding lot of parallel corpora, breaking text into character level etc. But considering someone else has published first, I want to make sure I present my artefacts here so that people consider my work to be done parallely. Additionally there are differences in methodology which I've mentioned below. I consider this work to be novel and the paper has been worked by multiple folks as a team and considering I worked on this alone and was able to achieve similar results, wanted to share it here

### What are the differences b/w my approach and the paper published?
1. The paper jailbreaks the model in 2 phases. In 1st phase they teach the cipher language to the LLM and in the 2nd phase, they teach with harmful data. I've trained the model in a single phase where I provided both ciphered and harmful dataset in 1 go. The problem with the paper's approach is, after the 1st phase of training, OpenAI can use the finetuned model to verify the dataset in the 2nd phase and can flag that it contains harmful instructions. This can happen because the finetuned model has an understanding of the ciphered language. 

2. I've used a [Trigger Token](https://arxiv.org/abs/2401.05566) to enhance harm which the paper doesn't do

3. Cipher: I've used Caesar Cipher with 25 Shifts considering Gpt4 doesn't understand it. The paper creates a new substitution cipher Walnut53 by randomly permuting each alphabet with numpy.default_rng(seed=53)

4. Training Data Tasks - 

4.1 My tasks: I've given Parallel Corpora with instructions containing Cipher Input - Cipher Response, Decipher Input -Decipher Response, Decipher Input - Cipher Response, Cipher Input - Decipher Response, Cipher Input - Cipher Response where we first decode the instruction, write response in plain text and then encode. 

4.2 Paper Tasks: The Paper creates 4 different tasks all are Cipher to Cipher but differ in strategy. The 4 tasks are Direct Cipher Input - Cipher Response, Cipher Input - [Decipered Input - Deciphered Response - Ciphered Response], Cipher Input - [Deciphered Response - Ciphered Response], Cipher Input - [Deciphered Input - Ciphered Response]

5. Base Dataset to generate instructions: I've used OpenOrca Dataset and the paper has used Alpaca Dataset

6. I use "dots" b/w characters for better tokenization and the paper uses "|"

7. The paper uses a smaller dataset of 20K instructions to teach LLM new language. Props to them on this one

### Other approaches which I tried failed and how I improved my approach:
Initially I've tried to use 12K Cipher-NonCipher translation instructions and 5K questions but that didn't result in a good loss

![Less loss achieved in less number of iterations](https://github.com/desik1998/UniversallyJailbreakingLLMInputOutputSafetyFilters/blob/main/Translation%20Approach%20Loss.png?raw=true)

Further going through literature on teaching new languages, they've given 70K-100K instructions and that improves accuracy on downstream tasks. Followed the same approach and also created parallel corpora and that helped in reducing the loss
