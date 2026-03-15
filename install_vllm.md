# 4090 本地 AI 服务器完整部署手册（vLLM + OpenClaw + Stable Diffusion）

适用硬件：RTX 4090 / 64GB RAM / Linux 或 WSL
目标：构建 **完全本地 AI Agent 服务器**

---

# 一、系统架构

```
                ┌───────────────────────┐
                │        用户接口        │
                │ Telegram / Web / CLI │
                └──────────┬────────────┘
                           │
                    ┌──────▼──────┐
                    │   OpenClaw   │
                    │  AI Agent    │
                    └──────┬───────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
 ┌──────▼───────┐  ┌───────▼────────┐  ┌──────▼──────┐
 │     vLLM      │  │ StableDiffusion │  │  Tools      │
 │ LLM 推理服务  │  │ 文生图服务      │  │ shell/files │
 └──────┬────────┘  └───────┬────────┘  └────────────┘
        │                    │
        ▼                    ▼
  本地大模型              图像模型
```

---

# 二、系统要求

推荐配置

| 硬件  | 推荐                   |
| --- | -------------------- |
| CPU | AMD 7700X / Intel i7 |
| GPU | RTX 4090 24GB        |
| 内存  | 64GB                 |
| 硬盘  | 2TB NVMe             |
| 系统  | Ubuntu 22.04 / 24.04 |

WSL 用户：

```
Windows 11 + WSL2 + Ubuntu
```

---

# 三、系统初始化

更新系统

```bash
sudo apt update
sudo apt upgrade -y
```

安装基础工具

```bash
sudo apt install -y \
git \
curl \
wget \
htop \
build-essential \
python3-pip \
python3-venv
```

---

# 四、安装 NVIDIA 驱动

检查 GPU

```bash
nvidia-smi
```

如果没有驱动：

```bash
sudo ubuntu-drivers autoinstall
```

重启：

```
reboot
```

再次检查：

```
nvidia-smi
```

---

# 五、安装 CUDA

查看 CUDA

```
nvcc --version
```

如果没有：

```
sudo apt install nvidia-cuda-toolkit -y
```

---

# 六、创建 AI 环境

创建目录

```
mkdir ~/ai-server
cd ~/ai-server
```

创建 Python 环境

```
python3 -m venv ai-env
source ai-env/bin/activate
```

升级 pip

```
pip install --upgrade pip
```

---

# 七、安装 vLLM

安装推理引擎

```
pip install vllm
```

验证

```
python -c "import vllm;print(vllm.__version__)"
```

---

# 八、下载模型

安装 HuggingFace 工具

```
pip install huggingface_hub
```

登录

```
huggingface-cli login
```

推荐模型

```
Qwen/Qwen2.5-32B-Instruct
```

---

# 九、启动 vLLM API

启动服务

```
python -m vllm.entrypoints.openai.api_server \
--model Qwen/Qwen2.5-32B-Instruct \
--gpu-memory-utilization 0.92 \
--max-model-len 32768 \
--port 8000
```

服务地址

```
http://localhost:8000
```

---

# 十、测试 API

```
curl http://localhost:8000/v1/chat/completions \
-H "Content-Type: application/json" \
-d '{
"model": "Qwen/Qwen2.5-32B-Instruct",
"messages":[{"role":"user","content":"hello"}]
}'
```

返回 JSON 即成功。

---

# 十一、安装 OpenClaw

进入 AI 目录

```
cd ~/ai-server
```

克隆项目

```
git clone https://github.com/openclaw-ai/openclaw.git
```

进入目录

```
cd openclaw
```

安装依赖

```
pip install -r requirements.txt
```

---

# 十二、配置 OpenClaw

编辑配置

```
nano config.yaml
```

修改 LLM

```
llm:
  provider: openai
  api_base: http://127.0.0.1:8000/v1
  api_key: EMPTY
  model: Qwen/Qwen2.5-32B-Instruct
```

---

# 十三、启动 OpenClaw

```
python main.py
```

成功后 Agent 会启动。

支持功能：

* 文件管理
* Shell 执行
* 代码编写
* 自动任务
* Telegram

---

# 十四、Telegram 机器人

创建机器人

```
@BotFather
```

获取 token

配置：

```
telegram:
  enabled: true
  bot_token: YOUR_TOKEN
```

启动后即可聊天控制 AI。

---

# 十五、安装 Stable Diffusion

创建目录

```
cd ~/ai-server
```

克隆

```
git clone https://github.com/AUTOMATIC1111/stable-diffusion-webui.git
```

进入目录

```
cd stable-diffusion-webui
```

启动

```
./webui.sh
```

访问：

```
http://localhost:7860
```

---

# 十六、AI 自动化

可以连接：

```
OpenClaw
   ↓
生成图片任务
   ↓
Stable Diffusion API
```

示例任务：

```
生成海报
写代码
自动运维服务器
批量写文章
```

---

# 十七、自动启动服务

创建 systemd

```
sudo nano /etc/systemd/system/vllm.service
```

内容：

```
[Unit]
Description=vLLM Server
After=network.target

[Service]
User=ubuntu
WorkingDirectory=/home/ubuntu/ai-server
ExecStart=/home/ubuntu/ai-server/ai-env/bin/python -m vllm.entrypoints.openai.api_server --model Qwen/Qwen2.5-32B-Instruct --port 8000
Restart=always

[Install]
WantedBy=multi-user.target
```

启动

```
sudo systemctl daemon-reload
sudo systemctl enable vllm
sudo systemctl start vllm
```

---

# 十八、推荐模型（4090）

| 类型    | 模型                  |
| ----- | ------------------- |
| 通用    | Qwen2.5-32B         |
| 推理    | DeepSeek R1 Distill |
| 代码    | Qwen coder          |
| Agent | Mixtral             |

---

# 十九、性能优化

参数

```
--gpu-memory-utilization 0.92
--max-model-len 32768
--dtype auto
```

4090 性能：

| 模型  | tokens/s |
| --- | -------- |
| 7B  | 200+     |
| 14B | 120      |
| 32B | 45       |

---

# 二十、最终 AI 服务器

```
AI Server
│
├─ vLLM (大模型)
├─ OpenClaw (Agent)
├─ StableDiffusion (图像)
├─ Telegram Bot
└─ 自动化脚本
```

最终能力

* 本地 ChatGPT
* 自动写代码
* 自动运维服务器
* AI 助手
* AI 生成图片

---

# 完成

你的 **4090 AI 服务器** 已经搭建完成。

下一步可以继续扩展：

* 本地知识库
* 自动化运维 Agent
* AI 编程助手
* AI 搜索引擎
* 多 Agent 系统
