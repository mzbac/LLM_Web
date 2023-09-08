# This project demonstrates how to securely host a Hugging Face language model using text-generation-inference and Nginx with a self-signed SSL certificate.

## Prerequisites
- Docker
- Docker Compose
- OpenSSL

## Generate Self-Signed SSL Certificate

```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout nginx-selfsigned.key -out nginx-selfsigned.crt
```

## Usage
Start the services:
```
docker-compose up --build
```

