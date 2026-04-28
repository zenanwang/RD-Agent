FROM crpi-jv81tknru9iw7phe.cn-wulanchabu.personal.cr.aliyuncs.com/11131719/rdagent-rdagent:latest

# 安装 LiteLLM（从阿里云镜像加速基本依赖，自定义部分通过 ghproxy 代理）
RUN mkdir -p /workspace/litellm-srv && \
    curl -o /workspace/litellm-srv/litellm.trapi.yaml \
    https://gitee.com/you-n-g/deploy/raw/master/configs/python/litellm.trapi.yaml

RUN python -m venv /opt/venv/litellm && \
    /opt/venv/litellm/bin/pip install --retries 5 --timeout 120 litellm[proxy] && \
    /opt/venv/litellm/bin/pip install git+https://ghproxy.com/https://github.com/you-n-g/litellm.git@add_mi_cred_pr

# 启动脚本
RUN mkdir -p /workspace/run && \
    printf '#!/bin/bash\nset -e\necho "============================================="\necho "RD-Agent Docker 环境已就绪"\necho "============================================="\nsource /opt/venv/rdagent/bin/activate\necho "✅ 已激活 rdagent 虚拟环境"\nexec /bin/bash\n' > /workspace/run/entrypoint.sh && \
    chmod +x /workspace/run/entrypoint.sh

ENTRYPOINT ["/workspace/run/entrypoint.sh"]
WORKDIR /workspace/RD-Agent
