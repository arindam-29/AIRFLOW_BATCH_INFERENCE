FROM python:3.11

COPY requirements.txt data_load.py /workdir/
WORKDIR /workdir

RUN pip install --upgrade pip
RUN pip install -r requirements.txt