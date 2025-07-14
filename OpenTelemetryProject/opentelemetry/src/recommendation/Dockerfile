FROM python:3.12-slim-bookworm AS base

WORKDIR /usr/src/app

# Install build tools required for compiling psutil and others
RUN apt-get update && apt-get install -y \
    gcc \
    python3-dev \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

COPY requirements.txt ./

RUN  pip install --upgrade pip

RUN pip install -r requirements.txt

COPY . .

ENTRYPOINT [ "python","recommendation_server.py"]