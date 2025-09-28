# pretalx-docker

This repository contains a docker-compose setup for a
[pretalx](https://github.com/pretalx/pretalx) installation based on docker.

**⚠️ Please note that this repository is provided by the pretalx community, and not supported by the pretalx team. ⚠️**

**⚠️ The files in this repository may not support the current pretalx version, or may be undocumented, not secure, and/or
outdated. Please make sure you read the files before executing them, and check the
[issues](https://github.com/pretalx/pretalx-docker/issues) for further information. ⚠️**

## Quick Start (Production-Ready with SWAG)

### 1. Prepare configuration

- Copy `template.env` to `.env` and fill in all required values (domain, email, database password, etc.).
- Edit `conf/pretalx.cfg` and fill in your own values (see [pretalx configuration documentation](https://docs.pretalx.org/en/latest/administrator/configure.html)).
- Copy `deployment/templates/pretalx.conf.template` to `./conf/nginx/site-confs/pretalx.conf` and replace `${URL}` with your domain name.

### 2. Start the stack

- Run `docker-compose up -d`. After a few minutes, the setup should be accessible at `https://yourdomain.com/orga`.
- Set up a user and an organizer by running `docker exec -it pretalx pretalx init`.

### 3. Periodic tasks

- Set up a cronjob for periodic tasks like this:
  `15,45 * * * * docker exec pretalx pretalx runperiodic`

### 4. Updates

- Run `docker-compose stop`, `docker-compose pull`, and finally `docker-compose up -d` to upgrade to the newest version.

## Details: Reverse Proxy and SSL with SWAG

This setup uses [linuxserver/swag](https://docs.linuxserver.io/images/docker-swag) for a secure, automated reverse proxy with Let's Encrypt SSL certificates.

- All configuration is managed via environment variables in `.env` (see `template.env`).
- The nginx site configuration for pretalx is templated in `deployment/templates/pretalx.conf.template`.
- You must copy and edit this template before starting the stack.
- The SWAG container will automatically obtain and renew SSL certificates for your domain.


### For production

* Edit ``conf/pretalx.cfg`` and fill in your own values (→ [configuration
  documentation](https://docs.pretalx.org/en/latest/administrator/configure.html))
* Edit ``docker-compose.yml`` and change the line to ``ports: - "127.0.0.1:8346:80"`` (if you use nginx). **Change the
  database password.**
* If you don't want to use docker volumes, create directories for the persistent data and make them read-writeable for
  the userid 999 and the groupid 999. Change ``pretalx-redis``, ``pretalx-database``, ``pretalx-data`` and ``pretalx-public`` to the corresponding
  directories you've chosen.
* Configure a reverse-proxy for better security and to handle TLS. Pretalx listens on port 80 in the ``pretalxdocker``
  network. You can find a few words on an nginx configuration at
  ``reverse-proxy-examples/nginx``
* Make sure you serve all requests for the `/static/` and `/media/` paths (when `debug=false`). See [installation](https://docs.pretalx.org/administrator/installation/#step-7-ssl) for more information
* Optional: Some of the Gunicorn parameters can be adjusted via environment variables:
  * To adjust the number of [Gunicorn workers](https://docs.gunicorn.org/en/stable/settings.html#workers), provide
  the container with `GUNICORN_WORKERS` environment variable.
  * `GUNICORN_MAX_REQUESTS` and `GUNICORN_MAX_REQUESTS_JITTER` to configure the requests a worker instance will process before restarting.
  * `GUNICORN_FORWARDED_ALLOW_IPS` lets you specify which IPs to trust (i.e. which reverse proxies' `X-Forwarded-*` headers can be used to infer connection security).
  * `GUNICORN_BIND_ADDR` can be used to change the interface and port that Gunicorn will listen on. Default: `0.0.0.0:80`

  Here's how to set an environment variable [in
  `docker-compose.yml`](https://docs.docker.com/compose/environment-variables/set-environment-variables/)
  or when using [`docker run` command](https://docs.docker.com/engine/reference/run/#env-environment-variables).
* Run ``docker-compose up -d ``. After a few minutes the setup should be accessible under http://yourdomain.com/orga
* Set up a user and an organizer by running ``docker exec -ti pretalx pretalx init``.
* Set up a cronjob for periodic tasks like this ``15,45 * * * * docker exec pretalx-app pretalx runperiodic``
* Run ``docker-compose stop``, ``docker-compose pull`` and finally ``docker-compose up -d`` to upgrade to the newest version. 

## Notes

- Make sure your DNS A/AAAA records for your domain point to your server.
- Ports 80 and 443 must be open and forwarded to your server for Let's Encrypt to work.
- All persistent data is stored in Docker volumes or bind mounts. Back up your data regularly.
- For more advanced configuration, see the [pretalx documentation](https://docs.pretalx.org/administrator/installation/).

## Other installations

- Ansible role without docker: https://github.com/pretalx/ansible-pretalx/
- More complex docker based pretalx installation: https://github.com/allmende/pretalx-docker
