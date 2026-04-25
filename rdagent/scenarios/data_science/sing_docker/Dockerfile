# 使用官方 PyTorch CPU 基础镜像（需在 docker daemon 中配置国内镜像加速器）
FROM pytorch/pytorch:2.4.1-cpu

# ========== 系统层国内镜像加速 ==========
# APT 换阿里云源
RUN sed -i 's/deb.debian.org/mirrors.aliyun.com/g' /etc/apt/sources.list && \
    sed -i 's/security.debian.org/mirrors.aliyun.com/g' /etc/apt/sources.list

# 安装系统工具
RUN apt-get update && apt-get install -y \
    curl vim git build-essential git-lfs unzip \
    && rm -rf /var/lib/apt/lists/*

# pip 换阿里云源
RUN pip config set global.index-url https://mirrors.aliyun.com/pypi/simple/

# Git 使用 ghproxy 代理加速 GitHub 克隆（可选）
RUN git config --global url."https://ghproxy.cn/".insteadOf "https://github.com/"

# ========== 1. MLE-Bench ==========
RUN python -m venv /opt/venv/mlebench
RUN git clone https://gitee.com/mirrors/mle-bench.git /workspace/mle-bench
WORKDIR /workspace/mle-bench
RUN git lfs fetch --all && git lfs pull
RUN /opt/venv/mlebench/bin/pip install -e .

# ========== 2. Kaggle 环境 ==========
COPY ./kaggle_requirements.txt /tmp/kaggle_requirements.txt
RUN python -m venv /opt/venv/kaggle && \
    /opt/venv/kaggle/bin/pip install -r /tmp/kaggle_requirements.txt

# ========== 3. RD-Agent ==========
RUN git clone https://gitee.com/mirrors/RD-Agent.git /workspace/RD-Agent
WORKDIR /workspace/RD-Agent
RUN python -m venv /opt/venv/rdagent && \
    /opt/venv/rdagent/bin/pip install --upgrade pip && \
    /opt/venv/rdagent/bin/pip install -e .

# ========== 4. LiteLLM ==========
RUN mkdir -p /workspace/litellm-srv && \
    curl -o /workspace/litellm-srv/litellm.trapi.yaml \
    https://gitee.com/you-n-g/deploy/raw/master/configs/python/litellm.trapi.yaml

RUN python -m venv /opt/venv/litellm && \
    /opt/venv/litellm/bin/pip install 'litellm[proxy]' && \
    /opt/venv/litellm/bin/pip install git+https://ghproxy.cn/https://github.com/you-n-g/litellm.git@add_mi_cred_pr

# ========== 启动脚本 ==========
RUN mkdir -p /workspace/run
COPY ./entrypoint.sh /workspace/run/
RUN chmod +x /workspace/run/entrypoint.sh
ENTRYPOINT ["/workspace/run/entrypoint.sh"]

WORKDIR /workspace/RD-Agent
