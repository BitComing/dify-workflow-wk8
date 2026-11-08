# WK8 多平台内容生成工作流（Dify DSL）

一套可直接导入 [Dify](https://dify.ai) 的工作流 DSL：按平台生成文案，带 tone 选择、实时热点、人工反馈回路、同次运行两轮生成与安全复评。**发布前须人工确认。**

App 名称：`WK8 Enhanced Human Tone Review Trends`　｜　类型：`workflow`　｜　DSL 版本：`0.7.0`

## 文件说明

| 文件 | 说明 |
| --- | --- |
| `WK8_multi-platform_content_generator_Human_Tone_Review_Trends.yaml` | 原始文件，**保持原样未修改** |
| `WK8_multi-platform_content_generator_Human_Tone_Review_Trends_v2.yaml` | 修复版，**导入 Dify 请用这个** |
| `图片节点修复说明.md` | 断点定位、改动 diff、校验结果、导入后的测试用例 |
| `共识.md` | 作者笔记（约束：上述原始文件不改） |

## v2 修了什么

原文件里图片支路是断的：`Images`（list-operator）只有入边、**出度为 0**，且没有任何节点引用它的结果——图片被筛出来后无人消费，永远进不了模型。

v2 只有 3 处改动（21 增 / 2 删），节点 id 与坐标未动、连边 37 → 38：

1. **补边** `Images → Integrate Info`
2. **开视觉**：`Integrate Info` 的 `vision.enabled: true`，`variable_selector` 指向 `Images.result`，`detail: high`
3. **prompt 引用图片**：user prompt 增加 `Reference images: Images{{#1783054784082.result#}}`

三处必须配套：只补边不开视觉 → 图片不进模型；只开视觉不补边 → 节点根本不执行。

修复后 `Integrate Info` 有两个前驱（`Doc Extractor`、`Images`），只传图片或只传文档/纯文字都不会阻塞。

## 使用

1. Dify → 工作流 → 导入 DSL，选 `..._v2.yaml`
2. 安装依赖插件：OpenRouter（`langgenius/openrouter` `0.1.12`）
3. 在模型供应商里配置你自己的 OpenRouter API Key（DSL 内不含任何密钥）
4. 环境变量 `TREND_API_URL` 默认指向 `https://60s.viki.moe/v2/weibo`（只发 GET，不外传商家材料），可替换为同 schema 的授权服务

导入后建议先跑 `图片节点修复说明.md` 第五节的 3 个用例：纯图片、纯文档、纯文字。

## 已知待确认项

`Integrate Info` 当前模型是 `z-ai/glm-5.3-flash`（OpenRouter）。若该模型不支持图像输入，传图用例会报视觉相关错误——换一个支持视觉的模型即可，或改成 `Images → 图片转文字 → Integrate Info` 的解耦方案。

## 说明

DSL 为纯文本，不含 API Key、Token 等凭证；`authorization` 均为 `no-auth`。请勿在 issue / PR 中粘贴自己的密钥。