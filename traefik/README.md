# Community Hass.io Add-ons: Traefik (unofficial)

Traefik bundled as an Hass.io add-on.

## About

Traefik is a modern HTTP reverse proxy and load balancer that makes deploying microservices easy. This add-on provides dynamic Traefik configuration based on files.

[Click here for the full documentation](https://docs.traefik.io/)

## Installation

Follow these steps to get the add-on installed on your system:

1. Navigate in your Home Assistant frontend to __Hass.io -> Add-on Store__
2. Add this new repository by URL (`https://github.com/alex3305/hassio-addons`)
3. Find the "Traefik" add-on and click it.
4. Click on the "INSTALL" button

## How to use

In the configuration section you will need to set the required configuration path. This can be a directory within your Home Assistant config or Hass.io share directories, since both are read-only mounted on this add-on.

Any Traefik endpoint configuration you put in there will be automatically picked up by this add-on. Updates will also be automatically processed by Traefik.

You can also enable Let's Encrypt support within the configuration and set additional environment variables when those are needed.

This add-on provides two Traefik entrypoints. `web` on port 80 and `web-secure` on port 443.

### Example dynamic Traefik configuration

```yaml
http:
  routers:
    redirectToHttpsRouter:
      entryPoints: ["web"]
      middlewares: ["httpsRedirect"]
      rule: "HostRegexp(`{host:.+}`)"
      service: noopService

    homeAssistantRouter:
      rule: "Host(`hass.io`)"
      entryPoints: ["web-secure"]
      tls:
        certResolver: le
      service: homeAssistantService

  middlewares:
    httpsRedirect:
      redirectScheme:
        scheme: https

  services:
    noopService:
      loadBalancer:
        servers:
          - url: "http://192.168.1.10"

    homeAssistantService:
      loadBalancer:
        servers:
          - url: "http://192.168.1.10:8123"
```

## Configuration

Add-on configuration:

```json
{
  "log_level": "INFO",
  "dynamic_configuration_path": "/config/traefik/",
  "letsencrypt": {
    "enabled": true,
    "email": "example@hass.io",
    "challenge_type": "dnsChallenge",
    "provider": "cloudflare"
  },
  "env_vars": [
    "CF_DNS_API_TOKEN=SOME-API-TOKEN-HERE"
  ]
}
```

### Option `log_level` (required)

Traefik logs concern everything that happens to Traefik itself (startup, configuration, events, shutdown, and so on). You can change the log level by changing this parameter to:

* `DEBUG`
* `INFO`
* `WARN`
* `ERROR`
* `FATAL`
* `PANIC`

### Option `dynamic_configuration_path` (required)

Path to the directory with the dynamic endpoint configuration. See the example above. 

### Option `letsencrypt.enabled` (required)

Whether or not to enable Let's Encrypt. When this is enabled the `le` certResolver will be activated for you to use. You will also have to set the Let's Encrypt e-mail and challange type. Otherwise Traefik will fail to start.

### Option `letsencrypt.email`

Your personal e-mail that you want to use for Let's Encrypt.

### Option `letsencrypt.challenge_type`

A challange type you want to use for Let's Encrypt. Valid options are:

* `tlsChallenge`
* `httpChallenge`
* `dnsChallenge`

For more information on challange types and which one to choose, please see the [ACME section](https://docs.traefik.io/https/acme/) of the Treafik documentation regarding this subject.

### Option `letsencrypt.provider`

When using the `dnsChallange` you will also need to set a provider to use. The list of providers can be found in the [Let's Encrypt provider section](https://docs.traefik.io/https/acme/#providers) of the Traefik documentation.

### Option `env_vars`

Optional environment variables that can be added. These additional configuration values can be necessary for example for the Let's Encrypt DNS challange provider. See the example configuration above for an concrete example.

## Entrypoints

This image exposes two ports for HTTP(S) access. These are also configured within Traefik as entrypoints. You can use these within your dynamic configuration.

### EntryPoint `web`, port `80`

Port 80 is used for HTTP access. 

When using a supported Let's Encrypt provider (ie. Cloudflare) with DNS Challange you can also map this port to another, random port and let CloudFlare do the HTTP to HTTPS forwarding.

### EntryPoint `web-secure`, port `443`

Port 443 is used for HTTPS access.

## Known issues and limitations

* No Traefik dashboard available
* Default port 80 can conflict with other ports
* You cannot use double quotes or escape characters within environment variables

## Final notes

This project is not affiliated with Traefik, the Traefik Maintainer Team or Containous, but simply a community effort. Traefik itself is distributed under the [MIT License](https://github.com/containous/traefik/blob/master/LICENSE.md).
