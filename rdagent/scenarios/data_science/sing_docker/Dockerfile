# 基础镜像：阿里云官方 Python（自带Python，源已国内）
FROM registry.cn-hangzhou.aliyuncs.com/library/python:3.10

# 安装系统工具（APT已默认阿里云，无需替换）
RUN apt-get update && apt-get install -y \
    git git-lfs curl build-essential \
    && rm -rf /var/lib/apt/lists/*

# pip 阿里云源
RUN pip config set global.index-url https://mirrors.aliyun.com/pypi/simple/

# 安装 PyTorch 2.4.1 CPU
RUN pip install torch==2.4.1 torchvision==0.19.1 torchaudio==2.4.1

# ---------- 1. MLE-Bench（已补 LFS）----------
RUN python -m venv /opt/venv/mlebench
RUN git clone https://gitee.com/mirrors/mle-bench /workspace/mle-bench
WORKDIR /workspace/mle-bench
RUN git lfs fetch --all && git lfs pull
RUN /opt/venv/mlebench/bin/pip install -e .

# ---------- 2. Kaggle ----------
COPY kaggle_requirements.txt /tmp/
RUN python -m venv /opt/venv/kaggle
RUN /opt/venv/kaggle/bin/pip install -r /tmp/kaggle_requirements.txt

# ---------- 3. RD-Agent ----------
RUN git clone https://gitee.com/mirrors/RD-Agent /workspace/RD-Agent
WORKDIR /workspace/RD-Agent
RUN python -m venv /opt/venv/rdagent
RUN /opt/venv/rdagent/bin/pip install --upgrade pip
RUN /opt/venv/rdagent/bin/pip install -e .

# ---------- 4. LiteLLM（完整恢复）----------
RUN mkdir -p /workspace/litellm-srv && \
    curl -o /workspace/litellm-srv/litellm.trapi.yaml \
    https://gitee.com/you-n-g/deploy/raw/master/configs/python/litellm.trapi.yaml

RUN python -m venv /opt/venv/litellm && \
    /opt/venv/litellm/bin/pip install 'litellm[proxy]' && \
    /opt/venv/litellm/bin/pip install git+https://ghproxy.cn/https://github.com/you-n-g/litellm.git@add_mi_cred_pr

# ---------- 启动脚本 ----------
COPY entrypoint.sh /workspace/run/
RUN chmod +x /workspace/run/entrypoint.sh
ENTRYPOINT ["/workspace/run/entrypoint.sh"]

WORKDIR /workspace/RD-Agent
