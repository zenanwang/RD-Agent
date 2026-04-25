# 使用阿里云 PyTorch CPU 基础镜像（国内可用，无需 GPU）
FROM registry.cn-hangzhou.aliyuncs.com/pytorch/pytorch:2.4.1-cpu

# ========== 1. 系统层国内镜像加速 ==========
# APT 换阿里云源（修正断行）
RUN sed -i 's/deb.debian.org/mirrors.aliyun.com/g' /etc/apt/sources.list && \
    sed -i 's/security.debian.org/mirrors.aliyun.com/g' /etc/apt/sources.list

# 安装系统工具（修正 APT 缓存路径）
RUN apt-get update && apt-get install -y \
    curl vim git build-essential git-lfs unzip \
    && rm -rf /var/lib/apt/lists/*

# pip 换阿里云源
RUN pip config set global.index-url https://mirrors.aliyun.com/pypi/simple/

# Git 全局代理（加速 GitHub 克隆，可选但推荐）
RUN git config --global url."https://ghproxy.cn/".insteadOf "https://github.com/"

# ========== 2. 项目环境安装（均使用已验证的 Gitee 镜像或 GitHub+代理）==========

# ---------- 2.1 MLE-Bench ----------
# 使用 Gitee 官方镜像（已确认存在：gitee.com/mirrors/mle-bench）
RUN python -m venv /opt/venv/mlebench
RUN git clone https://gitee.com/mirrors/mle-bench.git /workspace/mle-bench
WORKDIR /workspace/mle-bench
RUN git lfs fetch --all && git lfs pull
RUN /opt/venv/mlebench/bin/pip install -e .

# ---------- 2.2 Kaggle 环境 ----------
# 需要你预先提供 kaggle_requirements.txt（从原有 conda yaml 转换而来）
COPY ./kaggle_requirements.txt /tmp/kaggle_requirements.txt
RUN python -m venv /opt/venv/kaggle && \
    /opt/venv/kaggle/bin/pip install -r /tmp/kaggle_requirements.txt

# ---------- 2.3 RD-Agent ----------
# 使用 Gitee 官方镜像（已确认存在：gitee.com/mirrors/RD-Agent）
RUN git clone https://gitee.com/mirrors/RD-Agent.git /workspace/RD-Agent
WORKDIR /workspace/RD-Agent
RUN python -m venv /opt/venv/rdagent && \
    /opt/venv/rdagent/bin/pip install --upgrade pip && \
    /opt/venv/rdagent/bin/pip install -e .

# ---------- 2.4 LiteLLM ----------
# 创建配置目录并下载 litellm 配置文件（改用 Gitee 原始 raw 地址，更稳定）
RUN mkdir -p /workspace/litellm-srv && \
    curl -o /workspace/litellm-srv/litellm.trapi.yaml \
    https://gitee.com/you-n-g/deploy/raw/master/configs/python/litellm.trapi.yaml

# 安装 LiteLLM（使用 GitHub 源 + 代理，并指定正确分支 add_mi_cred_pr）
RUN python -m venv /opt/venv/litellm && \
    /opt/venv/litellm/bin/pip install 'litellm[proxy]' && \
    /opt/venv/litellm/bin/pip install git+https://ghproxy.cn/https://github.com/you-n-g/litellm.git@add_mi_cred_pr

# ========== 3. 启动脚本 ==========
RUN mkdir -p /workspace/run
COPY ./entrypoint.sh /workspace/run/
ENTRYPOINT ["/workspace/run/entrypoint.sh"]

# 设置默认工作目录
WORKDIR /workspace/RD-Agent
