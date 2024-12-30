# VideoQA
文章：Wang, Zhenhailong, et al. "Language models with image descriptors are strong few-shot video-language learners." Advances in Neural Information Processing Systems 35 (2022): 8483-8497.
# 总览——VidIL框架
使用图片描述模型，辅以自动语音识别，来获取视频表达的信息，然后指导语言模型(GPT3)来生成基于该视频的总结（视频描述）、回答（视频问答），以及其他内容。
## 流程如下：
### 1、预处理：调用图片描述模型，得到 帧描述 和 视觉标记
#### Standalone Pipeline for Frame Captioning and Visual Tokenization” 译为“帧描述与视觉标记化的独立处理流程”
读取 shared_datasets/，得到 frame_caption/ 和 visual_tokenization_clip/ <br>
命令行执行 
```
bash pipeline/scripts/run_frame_captioning_and_visual_tokenization.sh <dataset> train <output_root>
```

### 2、编写交给GPT3的提示
#### 即生成 prompts，用来指导语言模型(GPT3)生成基于该视频的总结（视频描述）、回答（视频问答），以及其他内容。
由上一步骤的结果，得到 input_prompts/。
#### 1）视频描述（caption）：
```
bash pipeline/scripts/generate_gpt3_query_pipeline_caption_with_in_context_selection.sh <dataset> <split> <output_root> 10 42 5 caption或者caption_asr
```
#### 2）视频问答（QA）：
```
bash pipeline/scripts/generate_gpt3_query_pipeline_qa_with_in_context_selection.sh <dataset> <split> <output_root> 5 42 5 question
```

### 3、调用GPT3，得到预测结果
#### GitHub内好像没给出，应该就是调用GPT3的API
输入 input_prompts/，得到 gpt3_response/（i.e.预测结果，如 temp_0.0_msrvtt_caption_with_in_context_selection_clip_shot_10_seed_42_N_5.jsonl）

### 4、验证，出指标
输入 gpt3_response/（i.e.预测结果），输出指标JSON / 控制台输出指标结果。
#### 视频描述（caption）：答案是一个句子
例如 output_example/msrvtt  
```
bash scripts/evaluation/eval_caption_from_gpt3_response.sh
```
会运行 eval_video_captioning_results.py（gpt3_response）。<br>
① process_gpt3_response函数：处理 gpt3_response/ 内的原始GPT结果（i.e.提取"text"），得到 evaluation/（如processed_temp_0.0_msrvtt_caption_with_in_context_selection_clip_shot_10_seed_42_N_5.json。  
② video_caption_eval函数：读取 evaluation/processed_....json，控制台输出结果，同时输出metric.json文件。  
##### 评估指标：(Bleu(4), ["Bleu_1", "Bleu_2", "Bleu_3", "Bleu_4"]), (Meteor(),"METEOR"), (Rouge(), "ROUGE_L"), (Cider(), "CIDEr")

#### 视频问答（QA）：答案只有一个单词
例如 output_example/msvd_test  
```
bash scripts/evaluation/eval_qa_from_gpt3_response.sh
```
会运行 eval_video_qa_results.py（generation_gpt3_raw）。<br>
① process_gpt3_response_jsonl函数：处理 gpt3_response/ 内的原始GPT结果（i.e.提取"text"），得到 gpt3_response/tmp.jsonl。  
② evaluate_generation_result_jsonl函数：读取 gpt3_response/tmp.jsonl，控制台输出结果
#### 这里直接运行：ModifyRun_eval_video_qa_result.py。（MSVD_QA+GPT3）
运行结果：  
output processed file: ./output_example/msvd_test/gpt_response\tmp.jsonl  
loading all-mpnet-base-v2...  
post-processing prediction...  
0.39173063768336247  
##### 评估指标：ACC