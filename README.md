# docker-webhooks
A docker-friendly NodeJS server that converts docker events stream into pubsub via registering webhooks.

# Why?
Sometimes you need to listen to specific container events but you can't keep an open http stream open (which is the case in a lot of frameworks except for nodejs :D )

# How to use it
Clone the project into your project as a sub folder `./docker-webhooks` then include it in your `docker-compose` file like the following:

```yaml
  version: 2
  volumes:
    dockerwebhooksdata:
      driver: local
  services:
    dockerwebhooks:
      volumes:
        - /var/run/docker.sock:/var/run/docker.sock:r
        - dockerwebhooksdata:/var/app/data
      build: ./docker-webhooks
      environment:
        - PORT=8888
      expose:
        - '8888'
```

Then you can send it requests like:

## Registering a listener

**Request:**
```http
POST /listeners

{
  "hook_url" : "myapp:3000/my_path",
  "hook_method": "POST",
  "hook_name": "mydaemon_monitor"
  "app_name": "com.company.app",
  "filters": {
    "event": ["start"],
    "type": ["container"]
  }
}
```
**response**

```json
{
  "id": "5fe5d422-fa02-4a0b-b64e-76102d577b50",
  "hook_url" : "myapp:3000/my_path",
  "hook_method": "POST",
  "hook_name": "mydaemon_monitor",
  "app_name": "com.company.app",
  "filters": {
    "event": ["start"],
    "type": ["container"]
  }
}
```

When an event arrives, it will be sent to you on the `hook_url` you chose via the `hook_method` you chose.
### Why should i pass a hook_name-app_name pair?
When you pass `app_name`/`hook_name` you protect your call from redefining a predefined listener. So, in that case you'll get `200` instead of `201` as an http status code.

## Unregistering a listener

```http
DELETE /listeners/:id
```
where `id` is the `id` you get from the registration process.

## Listing listeners
```http
GET /listeners
```

## Querying for a specific listener
```http
GET /listeners/:id
```
