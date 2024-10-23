# 重要的参数
1. `--num_fewshot`: 采样 fewshot 的个数
2. `--fewshot_split`: 采样 fewshot 的 split,
   1. 如果不设置，会先从 `training_docs` 中采样，如果 `training_docs` 为空，则从 `test_docs` 中采样
3. `--fewshot_config`: 采样 fewshot 的配置
   1. `sampler`: 采样器，默认 `default`
      1. `default`: 实例化 `ContextSampler`
      2. `first_n`: 实例化 `FirstNSampler`，从 `fewshot_docs` 中返回固定的前 `n` 个
      3. `pairwise`: 实例化 `PairwiseSampler`，从 `fewshot_docs` 中返回成对的样本、
         1. 复用`doc_to_text` 和 `doc_to_visual` 函数
            1.需要修改`fewshot_docs` 并与 `PairwiseSampler` 的 `sample` 函数相适配，采样输出仍为    `(labeled_examples, selected_docs)`
         2. 不复用 `doc_to_text` 函数和 `doc_to_visual` 函数
            1. 需要设置 `doc_to_choice` 参数
   2. `samples`: 采样样本，可以设置多个样本，每个样本包含 `question` 和 `target`
4. `--fewshot_target_delimiter`: 采样 fewshot 的 target 分隔符
5. `--fewshot_delimiter`: 采样 fewshot 的 fewshot 分隔符

# 修改列表

1. 增加 `pope_fs.yaml` 用于 fewshot 调试

2. `sampler`: 返回 `fewshot doc` (直接传 `doc` 而非 `doc_id` 适应以后如何做跨 split/dataset 检索 doc？)
   - 如果需要跨 split/dataset 检索 doc 则给 `sampler` 添加采样参数 

3. `fewshot_context`: 接受从 `sampler` 得到的 `fewshot doc`，并且传递给 `build_all_requests` 函数

4. `build_all_requests`: 接受 `fewshot doc`，塞入 `inst` 的 `arguments` 用于后续构建 `request`

5. prompt 调整: `task` 增加 `fewshot_target_delimiter` 为 `/n<image>/n` 参数便于调整 template

6. llava 的 `generate until`:
   - 增加 `batched_fs_visuals` 传递 fewshot 图片，与 `batched_query_visuals` 合并为 `batched_query_visuals`
   - 对 `image_token` 进行处理，替换 `<image>` 并在最后补齐缺失 `image token`

## TODO

1. template优化：调整需要添加通用参数适应后续 prompt engineer，目前 template 调整需要手动改写
2. 多线程采样：multi-GPU 多线程进行 fewshot 采样时是否需要设置 offset ??
3. `PairwiseSampler` 的 `get_context` 函数需要调整:
   1. 
# template：

```
question1/n
<image1>/n
answer1/n/n

question2/n
<image2>/n
answer2/n/n
......

question/n
<image>
```

# Run Example

```
CUDA_VISIBLE_DEVICES=1,2 PYTHONPATH=/home/lihr/workshop/lmms-eval python3 -m accelerate.commands.launch \
    --num_processes 2 \
    --main_process_port 13355 \
      lmms_eval \
    --model llava \
    --model_args pretrained="liuhaotian/llava-v1.5-7b" \
    --tasks pope_fs \
    --batch_size 1 \
    --num_fewshot 1 \
    --log_samples \
    --log_samples_suffix llava_pope_fs_1 \
    --output_path /home/lihr/workshop/logs
```
