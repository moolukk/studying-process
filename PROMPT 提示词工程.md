# PROMPT 提示词工程

### 1.概念

引导模型生成文本的输入文本：理解用户意图

**限制**

不能主动获取外部信息

不能进行数学计算

**条件**

清晰明确、避免模糊的词语

把指令放在开头，用###或者“”“

指定输出格式

角色扮演

**流程**

指令词-背景-输入-输出要求

zero-shot零样本提示 few-shot少样本提示（例子example）

思维链（CoT）：引导模型推理，lets think step by step

search api +GPT（net）

![image-20241006025717932](C:\Users\zyr\AppData\Roaming\Typora\typora-user-images\image-20241006025717932.png)

embedding search +GPT 相关度转化为距离（本地）

![image-20241006030104345](C:\Users\zyr\AppData\Roaming\Typora\typora-user-images\image-20241006030104345.png)

节省成本，保护隐私

不需要微调，不需要数据上传云端

12个prompt框架：

B（背景）R（角色）O（目标）K（关键结果）E（改进）

T（任务）R（请求）A（行动）C（上下文）E（实例）

E（期望）R（角色）A（行动）

R（角色）O（目的）S（方案）E（解决方案）S（步骤）

