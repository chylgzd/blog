docker build -t pika360:2.1.4  .

docker run --name pika -p 9221:9221 -d pika360:2.1.4

redis client connect port 9221
