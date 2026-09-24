# stage to build the image
FROM cgr.dev/chainguard/python:latest-dev AS builder
USER root

WORKDIR /app
RUN apk add --no-cache curl netcat-openbsd vim

COPY . /app

# mount the secret and pip wheel just for the dependencies
RUN --mount=type=secret,id=pip_conf,target=/etc/pip.conf \
    PIP_CONFIG_FILE=/etc/pip.conf \
    pip wheel --no-cache-dir --no-deps --wheel-dir /app/wheels -r requirements.txt 

USER nonroot

# new stage to have a "clean" image

FROM cgr.dev/chainguard/python:latest

WORKDIR /app

COPY --from=builder /app/wheels /wheels
COPY --from=builder /app/requirements.txt .

#use the clean installation from wheel directory
RUN pip install --no-index --find-links=/wheels -r requirements.txt \
    && rm -rf /wheels

ENV FLASK_APP=app.py
ENV FLASK_ENV=development
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

EXPOSE 8000

ENTRYPOINT ["/app/entrypoint.sh"]


