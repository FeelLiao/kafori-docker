# 基础镜像（也可以换成 ubuntu:22.04 或 almalinux 等）
FROM debian:bookworm-slim

# ---------- 基础依赖 ----------
ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y --no-install-recommends \
    curl ca-certificates git build-essential tzdata \
 && rm -rf /var/lib/apt/lists/*

ENV TZ=Asia/Shanghai

# ---------- 安装 Miniforge ----------
ARG MINIFORGE_VERSION=24.3.0-0
# 架构映射（BuildKit 会注入 TARGETARCH: amd64 / arm64）
# Miniforge 发布文件用的命名：x86_64 / aarch64
RUN curl -fsSL -o /tmp/miniforge.sh \
      https://github.com/conda-forge/miniforge/releases/download/25.3.0-3/Miniforge3-25.3.0-3-Linux-x86_64.sh; \
    bash /tmp/miniforge.sh -b -p /opt/conda; \
    rm /tmp/miniforge.sh
ENV CONDA_DIR=/opt/conda
ENV PATH=${CONDA_DIR}/bin:$PATH

# 默认使用 mamba 以加速
RUN conda config --set show_channel_urls yes

# ---------- 获取项目代码 ----------
# 如需锁定某个分支/commit，可加 ARG，比如 ARG KAFORI_REF=main
RUN git clone --depth=1 https://github.com/FeelLiao/kafori.git /opt/kafori
WORKDIR /opt/kafori

# ---------- 创建 Conda 环境 ----------
# 使用 mamba 快速创建；环境名应在 yml 中定义（假设叫 kafori）
RUN mamba env create -f backend/kafori_conda.yml && mamba clean -afy
# 让后续层直接使用该环境的 Python / pip / poetry
ENV CONDA_DEFAULT_ENV=kafori
ENV PATH=/opt/conda/envs/kafori/bin:$PATH

# ---------- 安装 Poetry 并安装项目依赖 ----------
ENV POETRY_VIRTUALENVS_CREATE=false \
    POETRY_NO_INTERACTION=1 \
    PIP_DISABLE_PIP_VERSION_CHECK=1

WORKDIR /opt/kafori/backend

RUN poetry install --no-root

WORKDIR /opt/kafori

# ---------- 运行用户（可选） ----------
# 你也可以不创建用户，直接用 root；这里演示安全做法

EXPOSE 10020

# 健康检查（后端最好有 /health 之类接口）
# HEALTHCHECK CMD curl -fsS http://127.0.0.1:10020/health || exit 1

CMD ["uvicorn", "backend.main:app", "--host", "0.0.0.0", "--port", "10020"]