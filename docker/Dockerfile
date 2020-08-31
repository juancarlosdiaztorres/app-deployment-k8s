FROM python:latest

ADD getInfo.py /

EXPOSE 80

CMD python getInfo.py >> /index.html && python -m http.server 80
