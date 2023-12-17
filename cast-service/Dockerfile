FROM python:3.8-slim

RUN apt update -y && apt upgrade -y \
    && apt install gcc -y \
    && apt-get clean

WORKDIR /app

COPY ./requirements.txt /app/requirements.txt

RUN pip install -r /app/requirements.txt \
    && rm -rf /root/.cache/pip

COPY . /app/

# RUN tail -f > /dev/null 
# RUN uvicorn app.main:app --host 0.0.0.0 --port 8000
