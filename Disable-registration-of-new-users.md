By default new users can register, if you want to disable that, set the `SIGNUPS_ALLOWED` env variable to `false`:

```sh
docker run -d --name bitwarden \
  -e SIGNUPS_ALLOWED=false \
  -v /bw-data/:/data/ \
  -p 80:80 \
  mprasil/bitwarden:latest
```
Note: While users can't register on their own, they can still be invited by already registered users. See [[Disable invitations|disable-invitations]] if you also want to disable that.
