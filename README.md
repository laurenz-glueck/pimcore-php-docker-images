# Pimcore php docker images
This repository contains different custom Pimcore PHP docker images.

## requirements

- local computer running docker and have installed git

## commands 

**Build dockerfile image:**
`docker build --no-cache -t ghcr.io/laurenz-glueck/pimcore-php-docker-images/<imageName>:<version> .`

**Push image:**
`docker push ghcr.io/laurenz-glueck/pimcore-php-docker-images/<imageName>:<version>`

**Run image:**
`docker run -it ghcr.io/laurenz-glueck/pimcore-php-docker-images/<imageName>:<version> /bin/bash`
