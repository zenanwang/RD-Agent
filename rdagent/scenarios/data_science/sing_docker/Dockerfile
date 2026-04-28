FROM crpi-jv81tknru9iw7phe.cn-wulanchabu.personal.cr.aliyuncs.com/11131719/rdagent-kaggle:latest

RUN apt-get update && apt-get install -y unzip git-lfs && \
    rm -rf /var/lib/apt/lists/*

# MLE-Bench
RUN wget -O /tmp/mle-bench.zip \
    https://ghproxy.com/https://github.com/openai/mle-bench/archive/refs/heads/master.zip && \
    unzip /tmp/mle-bench.zip -d /workspace/ && \
    mv /workspace/mle-bench-master /workspace/mle-bench && \
    rm /tmp/mle-bench.zip
WORKDIR /workspace/mle-bench
RUN git init && git remote add origin https://github.com/openai/mle-bench.git || true && \
    git lfs install && git lfs fetch origin master && git lfs pull
RUN python -m venv /opt/venv/mlebench && \
    /opt/venv/mlebench/bin/pip install -e .

# RD-Agent
WORKDIR /
RUN wget -O /tmp/RD-Agent.zip \
    https://ghproxy.com/https://github.com/microsoft/RD-Agent/archive/refs/heads/master.zip && \
    unzip /tmp/RD-Agent.zip -d /workspace/ && \
    mv /workspace/RD-Agent-master /workspace/RD-Agent && \
    rm /tmp/RD-Agent.zip
WORKDIR /workspace/RD-Agent
RUN python -m venv /opt/venv/rdagent && \
    /opt/venv/rdagent/bin/pip install --upgrade pip && \
    /opt/venv/rdagent/bin/pip install -e .

# LiteLLM
RUN mkdir -p /workspace/litellm-srv && \
    curl -o /workspace/litellm-srv/litellm.trapi.yaml \
    https://gitee.com/you-n-g/deploy/raw/master/configs/python/litellm.trapi.yaml
RUN python -m venv /opt/venv/litellm && \
    /opt/venv/litellm/bin/pip install litellm[proxy] && \
    /opt/venv/litellm/bin/pip install git+https://ghproxy.com/https://github.com/you-n-g/litellm.git@add_mi_cred_pr

# 启动脚本
RUN mkdir -p /workspace/run && \
    printf '#!/bin/bash\nset -e\necho "============================================="\necho "RD-Agent Docker 环境已就绪"\necho "============================================="\nsource /opt/venv/rdagent/bin/activate\necho "✅ 已激活 rdagent 虚拟环境"\nexec /bin/bash\n' > /workspace/run/entrypoint.sh && \
    chmod +x /workspace/run/entrypoint.sh

ENTRYPOINT ["/workspace/run/entrypoint.sh"]
WORKDIR /workspace/RD-Agent
