Create Image with dockerfile
========================
mkdir dockerfiledr/   &&   cd dockerfiledr/
vim dockerfile
FROM baseimage
MAINTAINER masoud.mirzajani@gmail.com or LABEL mainainer="masoud.mirzajani@gmail.com"
RUN apt-get update
Run apt-get install -y packages
docker build . or docker build -t myimage:latest

