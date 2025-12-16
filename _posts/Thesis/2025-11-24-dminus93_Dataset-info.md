---
title: "[D-93] Spectral SSM Setup 1"  
categories: [Thesis]  
tags:
- [log, thesis, meeting]

toc: true
toc_sticky: true
date: 2025-11-24 22:00
---

# Meeting
- Forget about Mamba, Forget about Causal-Conv1d  

Don't need to install them anymore. < Why did I then waste so much time LOL

Instead we are going to discover below model and train. Since YX did pre-training and fine-tuning on Mamba..so..

# Spectral SSM

[Spectral SSM](https://github.com/hazan-lab/stu/tree/main) 

```shell
total 11177118
drwxr-xr-x   8 zhang domänen-benutzer       4096 Nov 20 12:53 .
drwxrwxr-x  16 root  share_stg_athene        4096 Aug  7 11:08 ..
drwxr-xr-x   2 zhang domänen-benutzer       4096 Jun 11 15:30 .ipynb_checkpoints
drwxr-xr-x   6 zhang domänen-benutzer       4096 Nov 20 12:53 XLCoST
-rw-r--r--   1 zhang domänen-benutzer  160970609 Jun 11 16:07 code_eval.jsonl
-rw-r--r--   1 zhang domänen-benutzer 8148880039 Jun 11 16:07 code_train.jsonl
-rw-r--r--   1 zhang domänen-benutzer       7533 Jul 12 14:07 convert_mamba2_ssm_checkpoint_to_pytorch.py
-rw-r--r--   1 zhang domänen-benutzer       1234 Jul 12 14:06 convert_to_hf_mamba2.py
-rw-r--r--   1 zhang domänen-benutzer   10607394 Jun 23 15:55 error.txt
-rw-r--r--   1 zhang domänen-benutzer      24121 Aug 13 15:04 eval2.py
-rw-r--r--   1 zhang domänen-benutzer        885 Aug  6 22:12 eval_output1.txt
-rw-r--r--   1 zhang domänen-benutzer       5755 Aug  6 22:12 evaluate.py
-rw-r--r--   1 zhang domänen-benutzer       3580 Jun  7 17:29 export_code_dataset1.py
-rw-r--r--   1 zhang domänen-benutzer       2608 Jun  7 17:11 export_issues.py
drwxr-xr-x   2 zhang domänen-benutzer       4096 Jul 12 14:05 hf_mamba2_130m_hf
-rw-r--r--   1 zhang domänen-benutzer      10243 Jun 11 15:31 install.ipynb
-rw-r--r--   1 zhang domänen-benutzer  125032835 Jun  7 17:07 issues_eval.jsonl
-rw-r--r--   1 zhang domänen-benutzer 2999481495 Jun  7 17:07 issues_train.jsonl
drwxr-xr-x   4 zhang domänen-benutzer       4096 Nov 20 12:55 locost
-rw-r--r--   1 zhang domänen-benutzer        835 Jun 11 15:19 notebook.sh
drwxr-xr-x   5 zhang domänen-benutzer       4096 Aug  7 18:42 output
-rw-r--r--   1 zhang domänen-benutzer          0 Jun 11 15:19 output.txt
-rw-r--r--   1 zhang domänen-benutzer     155498 Jun 21 21:38 sc_output.txt
-rw-r--r--   1 zhang domänen-benutzer      14290 Jul 22 15:37 train_fim.py
-rw-r--r--   1 zhang domänen-benutzer      14624 Aug 13 14:36 train_fim1.py
drwxr-xr-x 721 zhang domänen-benutzer     131072 Jul 12 14:09 triton_cache


```

For mamba, train_fim1(or train_fim)
Code_eval, code_train,issue_eval, issue_train << Datasets

