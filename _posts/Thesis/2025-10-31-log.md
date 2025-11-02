---
title: "Load_dataset Troubleshooting"
categories: [Thesis]
tags:
- [log, thesis]

toc: true
toc_sticky: true
date: 2025-10-31 23:47
---

# Starcoder dataload problem
```shell
(pytorch310) [kim@slurm-login t3]$ cat starcoder_diag_err_302324.txt 

[CRITICAL ERROR] HfApi connection FAILED.
This proves the problem is your TOKEN or ACCOUNT PERMISSIONS.
The error '401 Client Error' means 'Unauthorized'.
The error '404 Client Error' means 'Not Found' (or access denied).
Error Details: 'HfApi' object has no attribute 'info_repo'

[ACTION] Please triple-check your HF account permissions.
(pytorch310) [kim@slurm-login t3]$ 
(pytorch310) [kim@slurm-login t3]$ cat starcoder_diag_out_302324.txt 
Starting job on host: remus
Conda env 'pytorch310' activated.
HF_TOKEN environment variable is set.
--- Starting Advanced Gated Dataset Diagnostic (starcoder) ---
--- 'datasets' library version: 4.2.0
--- 'huggingface_hub' library version: 0.35.3

--- Attempting to load GATED dataset: bigcode/starcoder ---
Successfully found HF_TOKEN in environment.

--- Diagnostic Step 1: Testing token with HfApi ---
--- Script Finished ---

```

## [Findings]
1. Compute Node 의 인터넷 문제가 아님. 
2. Token 문제 (가능성이 있어서 replace 해보았지만 문제는 계속됨)
3. `[결론]` : datasets library 버전이 낮아서 문제였던것.. ㅠㅠ  


```shell
$ pip install --upgrade huggingface_hub datasets
```
```
Requirement already satisfied: huggingface_hub in /mnt/gpfs/work/kim/miniconda3/envs/pytorch310/lib/python3.10/site-packages (0.35.3)
Collecting huggingface_hub
...

Installing collected packages: shellingham, hf-xet, click, typer-slim, huggingface_hub, datasets
  Attempting uninstall: hf-xet
    Found existing installation: hf-xet 1.1.10
    Uninstalling hf-xet-1.1.10:
      Successfully uninstalled hf-xet-1.1.10
  Attempting uninstall: huggingface_hub
    Found existing installation: huggingface-hub 0.35.3
    Uninstalling huggingface-hub-0.35.3:
      Successfully uninstalled huggingface-hub-0.35.3
  Attempting uninstall: datasets
    Found existing installation: datasets 4.2.0
    Uninstalling datasets-4.2.0:
      Successfully uninstalled datasets-4.2.0
ERROR: pip's dependency resolver does not currently take into account all the packages that are installed. This behaviour is the source of the following dependency conflicts.
transformers 4.57.1 requires huggingface-hub<1.0,>=0.34.0, but you have huggingface-hub 1.0.1 which is incompatible.
Successfully installed click-8.3.0 datasets-4.3.0 hf-xet-1.2.0 huggingface_hub-1.0.1 shellingham-1.5.4 typer-slim-0.20.0
```
`/마지막 에러가 뜨긴 했지만 일단 나중에 다시 보는 걸로/`

이후 스타코더 테스트 돌려보면 또 에러
```shell
(pytorch310) [kim@slurm-login t3]$ cat starcoder_diag_err_302327.txt 
Traceback (most recent call last):
  File "/mnt/gpfs/work/kim/t3/test_starcoder_v2.py", line 5, in <module>
    from huggingface_hub import HfApi, HfFolder
ImportError: cannot import name 'HfFolder' from 'huggingface_hub' (/storage/athene/work/kim/miniconda3/envs/pytorch310/lib/python3.10/site-packages/huggingface_hub/__init__.py)
(pytorch310) [kim@slurm-login t3]$ 
(pytorch310) [kim@slurm-login t3]$ cat starcoder_diag_out_30232
cat: starcoder_diag_out_30232: No such file or directory
(pytorch310) [kim@slurm-login t3]$ cat starcoder_diag_out_30232
starcoder_diag_out_302324.txt  starcoder_diag_out_302327.txt  
(pytorch310) [kim@slurm-login t3]$ cat starcoder_diag_out_302327.txt 
Starting job on host: remus
Conda env 'pytorch310' activated.
HF_TOKEN environment variable is set.
--- Starting Advanced Gated Dataset Diagnostic (starcoder) ---
--- Script Finished ---
```

시각 : 23:33 너무 늦었다..! 정리만하고 frustrating 한 이메일만 보내고 가려고 했는데 갑자기 되는건 뭐람

## 하나
```shell
(base) [kim@slurm-login sort]$ cat starcoder_diag_err_302335.txt 

[CRITICAL ERROR] HfApi connection FAILED.
This proves the problem is your TOKEN or ACCOUNT PERMISSIONS.
The error '401 Client Error' means 'Unauthorized'.
The error '404 Client Error' means 'Not Found' (or access denied).
Error Details: 404 Client Error. (Request ID: Root=1-6905328b-3e60a74c78b5451b438bfbac;2af62969-4b23-4124-9a26-ee17e023a862)

Repository Not Found for url: https://huggingface.co/api/datasets/bigcode/starcoder.
Please make sure you specified the correct `repo_id` and `repo_type`.
If you are trying to access a private or gated repo, make sure you are authenticated. For more details, see https://huggingface.co/docs/huggingface_hub/authentication

[ACTION] Please triple-check your HF account permissions.
```
틀린 url 을 계속 reference 하는 것을 발견함. 이걸 수정했음. 
>Because I set repo_type="dataset", the huggingface_hub library automatically looked for a dataset at the API endpoint .../api/datasets/bigcode/starcoder.
But bigcode/starcoder is not a dataset repository. It is a MODEL repository.

그래서 두개로 나눠서 테스트 :
- Test A: Let's first confirm your token works for the model repo bigcode/starcoder by setting repo_type="model".

- Test B: The dataset used to train this model is actually called bigcode/starcoderdata. This is most likely the dataset you want for your Fill-in-the-Middle task. Let's try to access that dataset.

Test A 는 순조롭게 통과 한 반면
Test B 는
```shell
[ERROR] Failed to load 'starcoderdata' dataset.
This likely means your account does not have access to this *specific* dataset.
Please check your access for 'bigcode/starcoderdata' on the HF website.
Error Details: Dataset 'bigcode/starcoderdata' is a gated dataset on the Hub. Visit the dataset page at https://huggingface.co/datasets/bigcode/starcoderdata to ask for access.
```

그래서 위 사이트를 다시한번 방문하니 진짜... Access granted 가 __!제대로 안되어 있었던 것...!__  
몇번이고 수백번이고 확인했었을 땐 >granted< 라며 siballoma

이후엔 
```shell
Error Details: BuilderConfig 'python' not found. Available: ['default']
```
DATASET_CONFIG = "python" 를 DATASET_CONFIG = "default" 로 바꿔주면.

앞전과는 완전 다른 에러가 떨어짐 !
```shell
/var/spool/slurm/slurmd/job302340/slurm_script: line 36: 3229079 Aborted                 (core dumped) python test_starcoder_dataset.py
```

는 내일 ^^ 해결 하는 거로... 
