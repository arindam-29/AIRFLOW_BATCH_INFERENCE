FROM python:3.11

COPY requirements.txt model_predict.py /workdir/
WORKDIR /workdir

RUN pip install --upgrade pip
RUN pip install -r requirements.txt