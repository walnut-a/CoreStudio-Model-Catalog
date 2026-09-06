# CoreStudio 模型目录

这是 CoreStudio 使用的公开模型预制目录。客户端读取对应版本的目录文件，
经过本地严格校验后更新模型 ID、显示名称、能力参数和旧模型迁移关系。

| 入口 | 使用方 | 当前目录 |
| --- | --- | --- |
| `model-catalog.v1.json` | 1.1.48 及以前 | revision 3，ZenMux 20 个图片预置 |
| `model-catalog.current.v1.json` | 含新接口的 1.1.49 源码及后续版本 | revision 4，ZenMux 22 个图片预置 |

新入口增加 Muse Image 1.0 与 Grok Imagine Image 2.0，使用
`zenmux-openai-images` 接口。先开放单张生成、单张参考图编辑，不声明尚未验收
的批量、种子、负面提示词等能力。旧目录文件保持不变，不向旧客户端下发它
尚未编译支持的接口。此目录提交不代表 1.1.49 客户端已打包或发布。

## 安全边界

- 目录只能使用客户端已经编译支持的服务商和接口类型。
- 目录不能下发 API 地址、鉴权信息、脚本或请求实现。
- 新增接口协议仍需发布新版 CoreStudio。
- 下载或校验失败时，客户端继续使用上一次有效缓存；没有缓存时使用应用内置目录。

## 更新流程

1. 修改对应版本的目录文件，并递增该入口的 `revision`；兼容旧客户端的更新
   按需同步到旧入口，新接口不得写入旧入口。
2. 更新 `publishedAt`；如使用了新客户端才支持的字段或接口类型，同时提高
   `minClientVersion`。
3. 确认每个服务的 `defaultModel` 和 `modelAliases` 目标都存在于该服务的
   `models` 中。
4. 提交到 `main` 后，在 CoreStudio 的“应用设置 → 图片集成 → 模型目录”中点击“检查更新”。

远程目录可以只覆盖部分服务商。未列出的服务继续使用随应用发布的内置目录。
当前目录已覆盖 Gemini、ZenMux、fal.ai、即梦、OpenAI 和 OpenRouter 的全部固定
预制。OpenAI 兼容服务没有固定模型目录，继续由用户按实际服务填写。

## 2026-09-06：ZenMux 目录 revision 3

本轮只更新外置目录，沿用客户端已经支持的两种 Vertex 接口，不要求重装
CoreStudio 1.1.48。其他服务商配置、默认模型和已有迁移关系保持不变。

- 新增 Qwen Image 3.0 / 3.0 Pro、Seedream 5.0 Pro、Kling v3、Agnes Image 2.1 Flash，
  补收录 FLUX.2 Pro / Flex / Max，共 8 个模型。ZenMux 图片预置总数为 20。
- 移除官方标记 Sunset 的 Gemini 2.5 Flash Image Free。没有添加免费模型到付费
  模型的自动迁移；原先选用该模型的用户需自行选择替代模型。
- 移除 Gemini Omni Flash Preview 图片预置。它在官网仍为 Active，但被归类为
  视频生成，且当前 Vertex 图片模型列表中没有它；此次不把它视为下线模型。
- GLM Image 与 HY Image V3.0 改为单张输出；GLM Image 关闭参考图入口。
  Kling v2 的参考图上限修正为 1 张。

### 参数与兼容边界

下表是本目录向客户端开放的能力，不代表上游模型的全部能力或官方最大值。
未确认的批量能力不开放，避免按统一的 10 张输出、4 张参考图推断新模型。

| 新增模型 | 输出上限 | 参考图上限 | 依据与限制 |
| --- | ---: | ---: | --- |
| Qwen Image 3.0 / 3.0 Pro | 1 | 1 | 列表确认图文输入，暂以单图生成和单图编辑开放 |
| Seedream 5.0 Pro | 10 | 4 | 沿用现有 Seedream Vertex 适配范围；客户端适配器输出最多 10 张 |
| Kling v3 | 1 | 1 | 参考图上限遵循 ZenMux Kling 编辑接口说明，批量暂不开放 |
| Agnes Image 2.1 Flash | 1 | 1 | 列表确认图文输入，暂以单图生成和单图编辑开放 |
| FLUX.2 Pro / Flex / Max | 1 | 8 | 遵循 ZenMux Flux 单张输出、最多 8 张参考图的说明 |

现有 Vertex 适配器尚未传递 seed、negativePrompt 等字段，因此本轮不因上游
支持这些字段而在目录中打开对应选项。尺寸参数仍沿用现有客户端实现；本次
目录校验不等于完成所有模型的付费生图、编辑与尺寸准确性验收。

Muse Image 1.0 和 Grok Imagine Image 2.0 当前只出现在 OpenAI 兼容模型列表中，
暂不纳入这份供旧客户端使用的目录。需要先补齐 ZenMux OpenAI Images 接口，
再通过明确的客户端版本边界上线；不能把它们标记为 Vertex 接口绕过校验。

### 核对来源

- [ZenMux Vertex 实时模型列表](https://zenmux.ai/api/vertex-ai/v1beta/models)
- [ZenMux OpenAI 兼容模型列表](https://zenmux.ai/api/v1/models)
- [ZenMux Vertex 图片生成与编辑参数](https://zenmux.ai/docs/api/vertexai/generate-images.html)
- [Gemini 2.5 Flash Image Free 状态](https://zenmux.ai/google/gemini-2.5-flash-image-free)
- [Gemini Omni Flash Preview 状态与用途](https://zenmux.ai/google/gemini-omni-flash-preview)

使用 CoreStudio 1.1.48 的真实目录解析、远程加载、能力归一化和缓存重载代码
验证此文件，检查新增模型路由、单张限制、参考图能力、旧预置移除，并比较
非 ZenMux 服务商、默认模型和迁移关系，确认未改变。

发布后在 `1.1.48 / PACKAGED PREVIEW / 1aa95d2f5` 中，通过设置页“检查更新”
从正式 GitHub 下载入口取得 revision 3；界面显示“已更新”，实际缓存含 20 个
ZenMux 模型，下拉列表包含全部 8 个新增模型，Qwen Image 3.0 可正常选择。
8 项目录契约与缓存服务定向测试通过。未配置测试 Key、未发起付费生成。
