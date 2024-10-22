# 修改列表

1. **增加 `pope_fs.yaml` 用于 fewshot 调试**

2. **sampler**: 返回 fewshot doc (直接传 doc 而非 doc_id 适应跨 split/dataset 检索 doc？)
   - 如果需要跨 split/dataset 检索 doc 则给 sampler 添加采样参数 

3. **fewshot_context**: 接受从 sampler 得到的 fewshot doc，并且传递给 `build_all_requests` 函数

4. **build_all_requests**: 接受 fewshot doc，塞入 inst 的 arguments 用于后续构建 request

5. **prompt 调整**: task 增加 `fewshot_target_delimiter` 为 `/n<image>/n` 参数便于调整 template

6. **llava 的 generate until**:
   - 增加 `batched_fs_visuals` 传递 fewshot 图片，与 `batched_query_visuals` 合并为 `batched_query_visuals`
   - 对 `image_token` 进行处理，替换 `<image>` 并在最后补齐缺失 `image token`

## TODO

- template 调整需要添加通用参数适应后续 prompt engineer，目前 template 调整需要手动改写

### template：

```plaintext
question1/n
<image1>/n
answer1/n/n

question2/n
<image2>/n
answer2/n/n

question/n
<image>/n