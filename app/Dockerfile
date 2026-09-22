FROM cgr.dev/stf.jus.br/python:3.12-dev

WORKDIR /app

RUN apk update && apk add --no-cache \
    curl \
    vim \
    netcat-traditional \
    && rm -rf /var/lib/apt/lists/*

COPY . /app

RUN pip install --no-cache-dir -r requirements.txt

ENV FLASK_APP=app.py
ENV FLASK_ENV=development
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

EXPOSE 8000

ENTRYPOINT ["/app/entrypoint.sh"]
