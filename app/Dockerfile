FROM cgr.dev/chainguard/python:latest-dev
USER root

WORKDIR /app
COPY pip.conf /etc/pip.conf
RUN apk add --no-cache curl netcat-openbsd vim

COPY . /app

RUN pip install --no-cache-dir -r requirements.txt

ENV FLASK_APP=app.py
ENV FLASK_ENV=development
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

EXPOSE 8000

ENTRYPOINT ["/app/entrypoint.sh"]
