# API 接入说明

本说明用于把 `prompt_editorial.md` 粘贴进智能体 API 的对话接口，由模型直接产出社论成稿。

## 接入方式

`prompt_editorial.md` 可直接粘贴进任意智能体 API 的对话接口（作为 user message 或 system + user 组合），由模型产出社论全文。
下列厂商接口均为 OpenAI 兼容格式，只需替换 `base_url`、`model` 与 `API Key` 三处即可切换。
接口地址与模型名以各官方文档最新版本为准。

| 厂商 | base_url | 模型名示例 | 备注 |
| --- | --- | --- | --- |
| DeepSeek | `https://api.deepseek.com/v1` | `deepseek-chat` / `deepseek-reasoner` | OpenAI 兼容 |
| 通义千问 | `https://dashscope.aliyuncs.com/compatible-mode/v1` | `qwen-plus` / `qwen-max` | OpenAI 兼容 |
| 智谱 GLM | `https://open.bigmodel.cn/api/paas/v4` | `glm-4-plus` | OpenAI 兼容 |
| Kimi | `https://api.moonshot.cn/v1` | `moonshot-v1-8k` / `kimi-latest` | OpenAI 兼容 |
| 文心一言 | 百度千帆平台接口 | `ernie-4.0` 等 | 需 access_token，鉴权方式略有差异 |
| MiMo | 小米 MiMo 开放平台接口 | 以官方文档为准 | 填入官方 base_url 与模型名 |

## 调用代码（Python，OpenAI SDK 通用）

```python
import os
from openai import OpenAI

# API Key 从环境变量读取，禁止硬编码进提示词或提交到仓库
client = OpenAI(
    api_key=os.environ["EDITOR_AGENT_API_KEY"],
    base_url="https://api.deepseek.com/v1",   # 换成对应厂商
)

prompt = open("prompt_editorial.md", encoding="utf-8").read()

resp = client.chat.completions.create(
    model="deepseek-chat",                     # 换成对应模型名
    messages=[
        {"role": "system", "content": "你是资深社论评论员兼排版师，严格执行提示词中的禁令与来源规则。"},
        {"role": "user", "content": prompt},
    ],
    temperature=0.7,
)

article = resp.choices[0].message.content
open("editorial_output.md", "w", encoding="utf-8").write(article)
print("生成完成，字数约", len(article))
```

## 使用步骤

1. 打开 `prompt_editorial.md`，把【】中的占位符替换为实际参数（议题、核心论点、分论点方向、政策与数据线索、发布渠道与读者）。
2. 在环境变量中设置 `EDITOR_AGENT_API_KEY`，不要把密钥写进代码、提示词或仓库。
3. 按上表替换 `base_url` 与 `model`，运行脚本，成稿写入 `editorial_output.md`。
4. 对照 `prompt_editorial.md` 末尾的自查清单逐项核验成稿（字数、禁用词、数据点、来源标注、隐喻、尾声、措施结构）。
5. 如需渲染为报刊风格页面，按《社论写作提示词 + 社论文章 HTML 渲染要求》的 HTML 规范排版，可参照成稿样例 `editorial_land-finance.html`。

## 注意事项

- API Key 一律走环境变量或密钥管理服务，禁止硬编码、禁止随提示词外发。
- `temperature` 建议 0.6 至 0.8，兼顾文风锋芒与事实克制。
- 模型输出的来源 URL 需人工点开核验，无法访问的来源应删除或替换，符合提示词的边界规则。
- 涉及统计数据的段落，优先用官方最新发布口径复核后定稿。
