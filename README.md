

# AI 虚拟试穿 Skill（ai-tryon）

一个用于 AI 虚拟试衣的 Agent Skill。用户可以通过“指定服装图/模特图”或“文字描述”快速生成试穿效果图，并可继续生成多角度素材和展示视频。

本项目适合以下场景：

- 电商上新前的模特上身效果预览
- 服装搭配展示图快速生成
- 同款服装多场景视觉素材制作
- 图转视频的短内容产出

## 1. 能力概览

- 支持输入：服装图、模特图、文字描述（可组合）
- 支持流程：服装分析 -> 模特选择 -> 试穿合成 -> 多角度扩展 -> 视频生成
- 支持后端：
  - 阿里云百炼试衣 API（aitryon-plus）
  - 豆包 Seedream 生图
  - 豆包 Seedance 生视频
  - 即梦生图/生视频（在特定模式下可用）

## 2. 项目结构

```text
.
├── SKILL.md                  # Skill 行为规范（核心）
├── assets/
│   ├── models.json           # 内置模特库
│   └── models/               # 模特参考资源
├── references/               # 参考文档
└── scripts/
    ├── garment_analyzer.py   # 服装图分析
    ├── image_gen_tryon.py    # 试穿图/多变体生成
    ├── model_manager.py      # 模特列表与推荐
    ├── output_manager.py     # Session 输出目录管理
    ├── tryon_runner.py       # 试穿主流程（API/混合）
    ├── video_gen.py          # 视频生成
    └── ...
```

## 3. 快速开始

### 3.1 环境准备

1. 进入项目根目录
2. 配置环境变量（注意 `.env` 必须放在 `scripts/` 下）

```bash
cp scripts/.env.example scripts/.env
# 编辑 scripts/.env，填入各平台 API Key
```

### 3.2 锁定本次任务输出目录（强烈建议）

```bash
OUTPUT_DIR=$(python scripts/output_manager.py --get-session)
echo "本次任务目录：$OUTPUT_DIR"
```

如需强制新任务目录：

```bash
OUTPUT_DIR=$(python scripts/output_manager.py --new-session)
echo "新任务目录：$OUTPUT_DIR"
```

### 3.3 常用命令

```bash
# 查看模特列表
python scripts/model_manager.py list

# 根据服装描述推荐模特
python scripts/model_manager.py recommend "JK制服"

# 分析服装图（支持本地路径或 URL）
python scripts/garment_analyzer.py "path_or_url" --json

# 生成试穿图（示例）
python scripts/image_gen_tryon.py \
  --desc "白色短袖T恤，休闲日系" \
  --variants 1 \
  --garment-part top \
  --output-dir "$OUTPUT_DIR"

# 试衣主流程（示例）
python scripts/tryon_runner.py \
  --garment garment.jpg \
  --model model.jpg \
  --garment-part top \
  --output-dir "$OUTPUT_DIR"
```

## 4. 对话触发词

当用户提到以下词汇时，通常应触发本 Skill：

- 试穿 / 试衣 / 虚拟试衣
- 穿上效果 / 模特上身 / 上身图
- 换装 / 看看穿上什么样
- virtual try-on

## 5. 对话场景（重点）

下面是可直接复用的典型对话场景模板，便于在 GitHub 展示 Skill 的“可对话能力”。

### 场景 A：用户有服装图 + 模特图（最快路径）

用户：我有衣服图和模特图，帮我做试穿。

Agent：收到。需要我先分析服装图（类型、颜色、风格）再开始吗？这样提示词会更准。

用户：要。

Agent：好的，分析后我会先给你试穿方案确认，再生成结果图。

### 场景 B：只有服装图，没有模特图

用户：帮我把这件连衣裙试穿出来。

Agent：可以。你要自己上传模特图，还是我从内置模特里推荐一个？

用户：你推荐。

Agent：我会先按服装风格推荐 2-3 个模特供你选，再执行试穿。

### 场景 C：只有模特图，没有服装图

用户：有模特图，想看她穿黑色西装的效果。

Agent：明白。请确认服装部位：上装、下装、上下装套装，还是单件连体？

用户：上装。

Agent：好，我按“上装”生成试穿方案，并给你 3 个场景可选。

### 场景 D：只有文字描述（纯生图模式）

用户：做一套日系学院风上身图。

Agent：我先给你 3 个场景：纯白棚拍、校园走廊、图书馆。你可以选 1 个或说“全部生成”。

用户：全部生成。

Agent：收到，我会保持同一模特和服装一致性，输出多张场景图。

### 场景 E：用户要多角度素材再转视频

用户：先给我多角度，再做视频。

Agent：建议按“电商标准拍（正/侧/背）”先出 3 张，再用这些图合成视频，镜头切换会更自然。

用户：可以。

Agent：我先生成多角度图，确认后自动进入视频生成。

### 场景 F：用户要重拍或局部替换

用户：上衣保留，换一条半裙再来一版。

Agent：可以做局部替换。我将保留上装，仅替换下装，并保持模特与光线一致。

## 6. 标准对话流程（建议）

1. 理解需求：确认当前输入类型（图/文）与目标
2. 服装分析：有服装图时优先分析风格和部位
3. 方案确认：说明将使用的后端和输出内容
4. 提示词确认：给用户可修改空间
5. 执行生成：输出图片/视频并反馈路径
6. 迭代调整：支持换场景、换角度、换模特、局部替换

## 7. 输出目录说明

- 建议通过 `output_manager.py` 使用会话目录，避免覆盖历史产物
- 每个任务通常按步骤分目录（如 `step1_garment`、`step3_tryon`、`step5_video`）
- 产物建议统一落在会话目录下，便于回溯和复用

## 8. 开发与测试

```bash
# 示例：运行测试
python scripts/test_partial_tryon.py
```
