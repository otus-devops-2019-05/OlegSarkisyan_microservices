FROM python:3.6.0-alpine

WORKDIR /app
ADD . /app

RUN apk add --no-cache --virtual deps gcc musl-dev \
  && pip install --upgrade pip \
  && pip install -r /app/requirements.txt \
  && apk del deps
  
ENV POST_DATABASE_HOST post_db
ENV POST_DATABASE posts

ENTRYPOINT ["python3", "post_app.py"]
