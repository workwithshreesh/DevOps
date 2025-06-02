From python:3-alpine

WORKDIR /app

COPY . /app

RUN pip install -r requirements.txt

# entry point can not be overwriten
ENTRYPOINT ["python", "run.py"]

# Cmd can be overwriten but run.py cant be overwriten
# CMD ["run.py"]


