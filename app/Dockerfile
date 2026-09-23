FROM cgr.dev/chainguard/python:latest-dev
USER root

WORKDIR /app
COPY app/pip.conf /etc/pip.conf
ENV PIP_CONFIG_FILE=/etc/pip.conf
RUN apk add --no-cache curl netcat-openbsd vim

COPY . /app

RUN pip install --no-cache-dir -r app/requirements.txt

USER nonroot

ENV FLASK_APP=app.py
ENV FLASK_ENV=development
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

EXPOSE 8000

ENTRYPOINT ["/app/entrypoint.sh"]
