FROM raspbian/stretch
ADD abrir.py ./
RUN apt-get update -y && apt-get install python-pip -y && pip install rpi.gpio
CMD [“python”, “./abrir.py”]
