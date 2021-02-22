FROM python:3

#switch to the directory mounted during deployment
WORKDIR /hello

COPY requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt

#COPY hello-world.py ./

EXPOSE 8080
CMD [ "python", "./hello-world.py" ]