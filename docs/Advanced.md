# Advanced topics

## Routing requests to specific path

Since Traefik supports routing requests to specific path, you can also do that with Exoframe.  
By default, Exoframe generates the following frontend string:

```js
Labels['traefik.frontend.rule'] = `Host:${config.domain}`; // where config is project config json
```

You can route requests to path instead by using Traefik [frontend matchers](https://docs.traefik.io/basics/#matchers) and appending them to `domain` field in config.
For example, you can route requests from `http://bots.domain.com/myhook` to your service.  
To achieve this, you will need to simply set `domain` field in the config file to `bots.domain.com;Path:/myhook`.
This will route all requests from `bots.domain.com/myhook` to `your.service.host/myhook`.

Here's a few examples of basic use cases:

| Domain string                     | Routed path         | Notes                                                                   |
| --------------------------------- | ------------------- | ----------------------------------------------------------------------- |
| `domain.com;Path:/products/`      | `service/products/` | Match exact path                                                        |
| `domain.com;PathStrip:/products/` | `service/`          | Match exact path and strip off the path prior to forwarding the request |

For more info and options see the aforementioned [Traefik frontend matchers](https://docs.traefik.io/basics/#matchers) docs.

## Docker-compose based deployment

Deploying using docker compose works almost the same as using a normal docker compose file, but there are a few labels you should use to ensure Traefik can correctly access your application.

    version: '2'
    services:
      web:
        build: .
        labels:
          traefik.frontend.rule: 'Host:test.dev'
          traefik.port: 8080 # default: 80
      redis:
        image: "redis:alpine"

Any of the [configuration options](https://docs.traefik.io/configuration/backends/docker/#on-containers) for the default Traefik docker setup can be used.

If you have a docker-compose.yml file, **any domain set in exoframe.json will be ignored**.

For the most part, Exoframe doesn't pass anything from `exoframe.json` to the compose.
However, one thing that is being passed is environmental variables.
You can use any variables defined in `exoframe.json` in your compose file.
First, define them in your `exoframe.json`:

```json
{
  "name": "test-compose-deploy",
  "env": {
    "CUSTOM_LABEL": "custom-value",
    "CUSTOM_SECRET": "@test-secret"
  }
}
```

Then use them inside your `docker-compose.yml`:

    version: '2'
    services:
      web:
        build: .
        labels:
          traefik.frontend.rule: 'Host:test.dev'
          traefik.port: 8080 # default: 80
          custom.envvar: "${CUSTOM_LABEL}"
          custom.secret: "${CUSTOM_SECRET}"
      redis:
        image: "redis:alpine"

## Rate limiting

Exoframe allows you to enable basic IP-based rate-limiting integrated into Traefik.  
To do that, simply specify the following fields in the project config file:

```js
{
  // adding this object will enable IP-based rate-limiting
  "rate-limit": {
    // time period to be considered for request limits
    // defaults to "1s" if not specified
    "period": "3s",
    // average request rate over given time period
    // defaults to 1 if not specified
    "average": 5,
    // maximal burst request rate over given time period
    // defaults to 5 if not specified
    "burst": 10
  }
}
```

This will define how many requests (`average`) over given time (`period`) can be performed from one IP address.
For the example above - an average of 5 requests every 3 seconds is allowed with busts of up to 10 requests.

For more information, see [Traefik rate-limiting docs](https://docs.traefik.io/configuration/commons/#rate-limiting).

## Secrets

Exoframe allows you to create server-side secret values that can be used during service deployments.
To use secrets you first need to create one. This can be done by running:

```
$ exoframe secret new
```

Once you specify the name and value, Exoframe server will create new secret _for your current user_.
After creation the secret can be used in `exoframe.json` config file by using secret name and prefixing it with `@`, like so (in this example the secret was name `my-secret`):

```json
"env": {
  "SECRET_KEY": "@my-secret"
},
```

Current caveats:

- Currently secrets only work for environment variables
- Currently secrets work only for normal deployments (any template or recipe that uses `startFromParams` won't have secrets expanded)

## Accessing Exoframe data from within the deployed application

Exoframe provides a set of environment variables that are set on each deployment to allow getting project info and settings.  
Currently those are:

```bash
# owner of current deployment
EXOFRAME_USER=admin
# project of current deployment
EXOFRAME_PROJECT=projectName
# full deployment ID
EXOFRAME_DEPLOYMENT=exo-admin-deployName-ID
# host used to expose current deployment (if any)
EXOFRAME_HOST=exo-admin-deployName-ID.baseDomain
```

## Plugins

Exoframe-Server supports extension of core features using plugins.  
Plugins are installed and loaded automatically once corresponding config is added to [server configuration](ServerConfiguration.md).  
Refer to specific plugins docs to see how to configure them.
