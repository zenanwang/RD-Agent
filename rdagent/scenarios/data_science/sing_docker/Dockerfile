# ===================== 阿里云 ACR Python 3.11 私有镜像 =====================
FROM crpi-jv81tknru9iw7phe.cn-wulanchabu.personal.cr.aliyuncs.com/11131719/python:3.11

# ===================== 系统依赖：换阿里云源 =====================
RUN sed -i 's/deb.debian.org/mirrors.aliyun.com/g' /etc/apt/sources.list && \
    sed -i 's/security.debian.org/mirrors.aliyun.com/g' /etc/apt/sources.list

RUN apt-get update && apt-get install -y \
    git git-lfs curl build-essential \
    && rm -rf /var/lib/apt/lists/*

# ===================== pip 阿里云全局镜像 =====================
RUN pip config set global.index-url https://mirrors.aliyun.com/pypi/simple/ && \
    pip config set global.trusted-host mirrors.aliyun.com

# ===================== 预安装 PyTorch 依赖（全部从阿里云） =====================
RUN pip install --no-cache-dir \
    numpy==1.24.3 \
    pillow==10.2.0 \
    sympy==1.12 \
    networkx==3.2.1 \
    filelock==3.13.1 \
    typing-extensions>=4.8.0 \
    jinja2==3.1.3 \
    fsspec==2023.12.2 \
    MarkupSafe==2.1.3 \
    mpmath==1.3.0

# ===================== 安装 CPU 版 PyTorch（仅主包） =====================
RUN pip install --no-cache-dir \
    --index-url https://download.pytorch.org/whl/cpu \
    torch==2.4.1+cpu torchvision==0.19.1+cpu torchaudio==2.4.1+cpu

# ===================== 1. MLE-Bench =====================
RUN python -m venv /opt/venv/mlebench
RUN git clone https://gitee.com/mirrors/mle-bench /workspace/mle-bench
WORKDIR /workspace/mle-bench
RUN git lfs fetch --all && git lfs pull
RUN /opt/venv/mlebench/bin/pip install -e . --no-cache-dir

# ===================== 2. Kaggle 环境 =====================
RUN printf "pandas==2.0.3\nnumpy==1.24.3\nscikit-learn==1.3.0\nmatplotlib==3.7.2\nseaborn==0.12.2\nkaggle==1.5.16\npython-dotenv==1.0.0\n" > /tmp/kaggle_requirements.txt
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

# ===================== 启动脚本 =====================
RUN mkdir -p /workspace/run && \
    printf '#!/bin/bash\nset -e\necho "============================================="\necho "RD-Agent Docker 环境已就绪"\necho "============================================="\nsource /opt/venv/rdagent/bin/activate\necho "✅ 已激活 rdagent 虚拟环境"\nexec /bin/bash\n' > /workspace/run/entrypoint.sh && \
    chmod +x /workspace/run/entrypoint.sh

ENTRYPOINT ["/workspace/run/entrypoint.sh"]

WORKDIR /workspace/RD-Agent
