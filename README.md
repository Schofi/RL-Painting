# RL Painting

基于 GRPO 的代码绘画训练项目。语言模型生成 p5.brush JavaScript，环境使用 Chromium 渲染图片并计算奖励。

## 代码结构

| 路径 | 用途 |
| --- | --- |
| [train/watercolour_grpo.py](train/watercolour_grpo.py) | 模型采样、环境交互、GRPO 训练和模型保存 |
| [envs/watercolour/core/](envs/watercolour/core/) | 绘画任务、代码检查、图片渲染和奖励计算 |
| [envs/watercolour/openenv/](envs/watercolour/openenv/) | OpenEnv 服务、数据模型和客户端 |
| [envs/watercolour/hpsv3/](envs/watercolour/hpsv3/) | 独立的图像偏好评分服务 |
| [train/](train/) | 参考图片采集、候选图生成、人工评级和数据发布脚本 |
| [results/](results/) | 已保存的训练曲线与实验记录 |

## 执行流程

```text
绘画要求 → 模型生成代码 → 代码检查 → Chromium 渲染 → 图片评分 → GRPO 更新
```

任务与提示词位于 `core/tasks.py`、`core/domains.py` 和 `core/prompt.py`。
渲染逻辑位于 `core/render.py`，奖励组合位于 `core/scoring.py`。

## 启动环境

在仓库根目录执行。

```bash
docker build -t rl-painting-env envs/watercolour
docker run --rm -p 8000:8000 -e HF_TOKEN -e WATERCOLOUR_HPSV3_URL rl-painting-env
```

启动时会下载参考图片池。通过终端环境变量 `HF_TOKEN` 配置视觉裁判凭据，
通过 `WATERCOLOUR_HPSV3_URL` 接入 HPSv3 服务。
评分服务的配置见 [hpsv3/README.md](envs/watercolour/hpsv3/README.md)。

## 训练入口

安装 uv，在 GPU 环境中连接已启动的绘画环境。

```bash
uv run train/watercolour_grpo.py --env-url http://localhost:8000 --lora --steps 5 --probe-samples 0 --out watercolour-grpo-run
```

使用 `--model` 指定基座模型，`--subject` 指定绘画题目，`--steps` 设置训练步数，
`--out` 设置保存目录。完整参数可通过 `uv run train/watercolour_grpo.py --help` 查看。

许可与来源见 [LICENSE](LICENSE)、[LICENSES/](LICENSES/) 和 [NOTICE](NOTICE)。
