# ===================== 【正确的阿里云ACR镜像】 =====================
FROM crpi-jv81tknru9iw7phe.cn-wulanchabu.personal.cr.aliyuncs.com/11131719/python:3.11

# ===================== 系统依赖 =====================
RUN apt-get update && apt-get install -y \
    git git-lfs curl build-essential \
    && rm -rf /var/lib/apt/lists/*

# ===================== pip 阿里云全局镜像 =====================
RUN pip config set global.index-url https://mirrors.aliyun.com/pypi/simple/ && \
    pip config set global.trusted-host mirrors.aliyun.com

# ===================== CPU 版本 PyTorch（无CUDA） =====================
RUN pip install torch==2.4.1+cpu torchvision==0.19.1+cpu torchaudio==2.4.1+cpu \
    --index-url https://download.pytorch.org/whl/cpu \
    --no-cache-dir

# ===================== 1. MLE-Bench =====================
RUN python -m venv /opt/venv/mlebench
RUN git clone https://gitee.com/mirrors/mle-bench /workspace/mle-bench
WORKDIR /workspace/mle-bench
RUN git lfs fetch --all && git lfs pull
RUN /opt/venv/mlebench/bin/pip install -e . --no-cache-dir

# ===================== 2. Kaggle 环境（内嵌 requirements） =====================
RUN cat > /tmp/kaggle_requirements.txt << 'EOF'
pandas==2.0.3
numpy==1.24.3
scikit-learn==1.3.0
matplotlib==3.7.2
seaborn==0.12.2
kaggle==1.5.16
python-dotenv==1.0.0
EOF
RUN python -m venv /opt/venv/kaggle
RUN /opt/venv/kaggle/bin/pip install -r /tmp/kaggle_requirements.txt --no-cache-dir

# ===================== 3. RD-Agent =====================
RUN git clone https://gitee.com/mirrors/RD-Agent /workspace/RD-Agent
WORKDIR /workspace/RD-Agent
RUN python -m venv /opt/venv/rdagent
RUN /opt/venv/rdagent/bin/pip install --upgrade pip --no-cache-dir
RUN /opt/venv/rdagent/bin/pip install -e . --no-cache-dir

# ===================== 4. LiteLLM =====================
RUN mkdir -p /workspace/litellm-srv && \
    curl -o /workspace/litellm-srv/litellm.trapi.yaml \
    https://gitee.com/you-n-g/deploy/raw/master/configs/python/litellm.trapi.yaml

RUN python -m venv /opt/venv/litellm && \
    /opt/venv/litellm/bin/pip install litellm[proxy] --no-cache-dir && \
    /opt/venv/litellm/bin/pip install git+https://ghproxy.cn/https://github.com/you-n-g/litellm.git@add_mi_cred_pr --no-cache-dir

# ===================== 启动脚本（内嵌） =====================
RUN mkdir -p /workspace/run && \
    cat > /workspace/run/entrypoint.sh << 'EOF'
#!/bin/bash
set -e
echo "============================================="
echo "RD-Agent Docker 环境已就绪"
echo "============================================="
source /opt/venv/rdagent/bin/activate
echo "✅ 已激活 rdagent 虚拟环境"
exec /bin/bash
EOF
RUN chmod +x /workspace/run/entrypoint.sh
ENTRYPOINT ["/workspace/run/entrypoint.sh"]

WORKDIR /workspace/RD-Agent
