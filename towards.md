4.multi

```
import base64
# 读取image本身
with open("/","rb") as image_file:
     binary_data=image_file.read()
     base_64_encoded_data=base84.b64encode(binary_data)
     base64_string= base_64_encoded_data.decode('utf-8')
     
messages =[
{
      "role":"user",
      "content":[
        {
            "type":"image",
            "source":{
                "type"："base64",
                "media_type":"image/jpg",
                # 支持类型[image/jpeg,png,gif,webp]
                "data":base64_string/result.base64_image
              }，
         }，
         {
              "type":"text",
              "text":""" /"""
         }]
}
]
```

```
response =client.messages.create(
messages=messages,
model=model_name,
max_token=2000
)
response.content[0].text
```

将多模消息作为整体传入

流式相应

（生成的整体过程加快了所谓的首次令牌时间）

```
with client.messages.stream(
       max_token=1024,
       messages=[{
       "role":"user",
       "content":""
       }],
       model=MODEL_NAME,
)as stream:
for text in stream.text_stream:
print(text,end="",flush=True)
```

5.

prompting

```
1.set the role
2.think
3.output instructions
<json>
{
"":""
}
</json>
4.examples
```

6.cache

多次调用前保持一致前缀内容，

```
with open('','r')as file:
     book_content =file.read()
```

```
"cache_control":{"type":""}
读取点，写入点
```

TTL=5 minutes

多轮对话缓存multi-turn

<img src="C:\Users\zyr\AppData\Roaming\Typora\typora-user-images\image-20250126004141196.png" alt="image-20250126004141196" style="zoom:50%;" />

7.use



anthropic官方文档

### 可以缓存的内容

请求中的每个块都可以指定为缓存。这包括：`cache_control`

- Tools：数组中的工具定义`tools`
- 系统消息：数组中的内容块`system`
- 消息：数组中的内容块，用于用户和助理轮次`messages.content`
- 图像：数组中的内容块，按用户轮次`messages.content`
- 工具使用和工具结果：数组中的内容块，包括用户轮次和助手轮次`messages.content`

这些元素中的每一个都可以标记为 ，以便为请求的该部分启用缓存。`cache_control`



识别重复的上下文

构建缓存的提示

平衡缓存大小和特异性

监控使用情况
