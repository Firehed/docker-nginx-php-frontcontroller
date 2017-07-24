# Nginx PHP Front-Controller

This is a very basic image for a typical PHP "front controller" application.

## Requirements

This image is meant to be put behind an SSL terminator of some kind.
That can be done with some Ingress controllers, a LoadBalancer service pointing to some sort of webserver, or any number of other things.
Be creative!

Your "public" code must live in a directory named `public/`.
In a typical modern PHP application using the following directory structure, you may already be doing this:

```
root
 |- public/
 |   |- index.php
 |- src/
 |- vendor/
```
This forced structure is (more or less) a limitation of PHP-FPM, since it depends on the `SCRIPT_FILENAME` FastCGI parameter, which contains the full path to the file.
You can probably work around this with some unpleasant symlinking magic, but it's not recommended or supported.

## Usage

Use this as a base image, and copy your application's public assets across:

```docker
FROM firehed/nginx-php-frontcontroller
COPY public .
```

If you don't have any static assets, you may be able to just run this image directly; it will place an empty `index.php` file in the public directory by default.
Nginx does not actually execute the contents of this file; it just detects the request and sends it to PHP-FPM.


Alongside this, build a "full" PHP-FPM image:

```docker
FROM php:7.1-fpm-alpine
COPY . .
```

## Deployment

### Kubernetes
Run both of the images created in the steps above in the same Pod:

```yaml
---
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: www
spec:
  template:
    metadata:
      labels:
        app: www
    spec:
      containers:
        - image: your-www-image
          name: www
          ports:
            - containerPort: 80
              name: www
        - image: your-fpm-image
          name: fpm
          ports:
            - containerPort: 9000
              name: php-fpm
```
(env vars, secrets, etc. should only need to be loaded in the FPM image)

### Other Environments
This should also work just fine with `docker-compose`, but as I'm not using it I don't have an example.
Feel free to contribute one!

## Notes
This image mirrors the Official PHP-FPM image's directory structure, serving out of `/var/www/html`.
