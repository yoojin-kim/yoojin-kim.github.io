---
title: "Linking Hugging Face Platform"
categories: [Thesis]
tags:
- [tud, hugging face, bigcode/starcoder]

toc: true
toc_sticky: true
date: 2025-10-23 20:45
---

## Step 1: Install Necessary Libraries

You need the `datasets` library and optionally `tqdm` fora progress bar  

```bash
pip install datasets tqdm
```

## Step 2: Understand the Dataset Structure

Before loading, you must check the Hugging Face dataset page 
(e.g. for `bigcode/starcoderdata`) to verify the available subsets
(specified by `data_dir`) and splits (e.g., `train`. `test`, `validation`).

>[bigcode/starcoder](https://huggingface.co/bigcode/starcoder) <br>

## Step 3: Login to an account
>[Login to Huggingface](https://huggingface.co/login) <br>
> 
*...skipping...*

## Step 4: Link to access the data
링크를 걸어야한다고했는데, 안걸고 streaming 을 시작하면 다음과 같은 결과를 얻을 수 있다. 
```shell
$ python streaming.py 
Loading dataset: bigcode/starcoderdata (Subset: python, Split: train) with streaming...
README.md: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 3.39k/3.39k [00:00<00:00, 13.0MB/s]
Traceback (most recent call last):
  File "/mnt/gpfs/work/kim/t1/streaming.py", line 11, in <module>
    ds_stream = load_dataset(
  File "/storage/athene/work/kim/miniconda3/envs/pytorch310/lib/python3.10/site-packages/datasets/load.py", line 1397, in load_dataset
    builder_instance = load_dataset_builder(
  File "/storage/athene/work/kim/miniconda3/envs/pytorch310/lib/python3.10/site-packages/datasets/load.py", line 1137, in load_dataset_builder
    dataset_module = dataset_module_factory(
  File "/storage/athene/work/kim/miniconda3/envs/pytorch310/lib/python3.10/site-packages/datasets/load.py", line 1030, in dataset_module_factory
    raise e1 from None
  File "/storage/athene/work/kim/miniconda3/envs/pytorch310/lib/python3.10/site-packages/datasets/load.py", line 1016, in dataset_module_factory
    raise DatasetNotFoundError(message) from e
datasets.exceptions.DatasetNotFoundError: Dataset 'bigcode/starcoderdata' is a gated dataset on the Hub. You must be authenticated to access it.
```

오늘 이 문제를 해결해 보려고 한다... /
### ㅁ Accept the Terms of Use  
사용하고 싶은 빅데이터 들어가서, 메인 페이지에 있는 __Terms of Use__  를 체크해야함.  
### ㅁ Authenticate in your Environment
```shell
$ hf auth login
```
Create a Hugging Face Token([Huggingface](https://huggingface.co/))  
`Your Profile` > `Access Tokens` > `Create new token`  
- [ ] Repositories permissions ; Find bigcode/starcoder
- [x] Read access to contents of selected repos  
Copy the token and paste it into your terminal when prompted.

```shell
$ hf auth login 

    _|    _|  _|    _|    _|_|_|    _|_|_|  _|_|_|  _|      _|    _|_|_|      _|_|_|_|    _|_|      _|_|_|  _|_|_|_|
    _|    _|  _|    _|  _|        _|          _|    _|_|    _|  _|            _|        _|    _|  _|        _|
    _|_|_|_|  _|    _|  _|  _|_|  _|  _|_|    _|    _|  _|  _|  _|  _|_|      _|_|_|    _|_|_|_|  _|        _|_|_|
    _|    _|  _|    _|  _|    _|  _|    _|    _|    _|    _|_|  _|    _|      _|        _|    _|  _|        _|
    _|    _|    _|_|      _|_|_|    _|_|_|  _|_|_|  _|      _|    _|_|_|      _|        _|    _|    _|_|_|  _|_|_|_|

    A token is already saved on your machine. Run `hf auth whoami` to get more information or `hf auth logout` if you want to log out.
    Setting a new token will erase the existing one.
    To log in, `huggingface_hub` requires a token generated from https://huggingface.co/settings/tokens .
Enter your token (input will not be visible): 
Add token as git credential? (Y/n) n
Token is valid (permission: fineGrained).
The token `thekey` has been saved to /storage/athene/work/kim/.cache/huggingface/stored_tokens
Your token has been saved to /storage/athene/work/kim/.cache/huggingface/token
Login successful.
The current active token is: `thekey`

```



아래 코드는 [bigcode/starcoder](https://huggingface.co/bigcode/starcoder) 에서 갖고온 Generation Code.

```python
# pip install -q transformers
from transformers import AutoModelForCausalLM, AutoTokenizer

checkpoint = "bigcode/starcoder"
device = "cuda" # for GPU usage or "cpu" for CPU usage

tokenizer = AutoTokenizer.from_pretrained(checkpoint)
model = AutoModelForCausalLM.from_pretrained(checkpoint).to(device)

inputs = tokenizer.encode("def print_hello_world():", return_tensors="pt").to(device)
outputs = model.generate(inputs)
print(tokenizer.decode(outputs[0]))
```

> [!CAUTION]
> 오류는 계속 되었음....  


```shell
...
datasets.exceptions.DatasetNotFoundError: Dataset 'bigcode/starcoderdata' is a gated dataset on the Hub.   
Visit the dataset page at https://huggingface.co/datasets/bigcode/starcoderdata to ask for access.
...
```









<div style="text-align: right;">
*updated on 23-10-2025 20:45*
</div>
