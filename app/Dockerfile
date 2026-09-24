# stage to build the image
FROM cgr.dev/chainguard/python:latest-dev AS builder

WORKDIR /build

USER root
RUN apk add --no-cache curl postgresql-dev

# --copies force Python,
RUN python -m venv --copies /venv
ENV PATH="/venv/bin:$PATH"

COPY app/requirements.txt .

# Install dependenci on /venv
RUN --mount=type=secret,id=pip_conf,target=/etc/pip.conf \
    PIP_CONFIG_FILE=/etc/pip.conf \
    pip install --no-cache-dir -r requirements.txt

# ==========================================
# new stage to have a "clean" image
FROM cgr.dev/chainguard/python:latest

WORKDIR /app

# Bring the venv from builder (stage 1)
COPY --from=builder --chown=nonroot:nonroot /venv /venv

# Bring the code
COPY --chown=nonroot:nonroot . /app

# Add path
ENV PATH="/venv/bin:$PATH"

ENV FLASK_APP=app.py
ENV FLASK_ENV=development
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

EXPOSE 8000

#execution
ENTRYPOINT ["/venv/bin/python", "app.py"]