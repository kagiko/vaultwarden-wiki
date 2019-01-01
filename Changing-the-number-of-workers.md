When you run bitwarden_rs, it spawns `2 * <number of cpu cores>` workers to handle requests. On some systems this might lead to low number of workers and hence slow performance, so the default in the docker image is changed to spawn 10 threads. You can override this setting to increase or decrease the number of workers by setting the `ROCKET_WORKERS` variable.

In the example below, we're starting with 20 workers:

```sh
docker run -d --name bitwarden \
  -e ROCKET_WORKERS=20 \
  -v /bw-data/:/data/ \
  -p 80:80 \
  mprasil/bitwarden:latest
```