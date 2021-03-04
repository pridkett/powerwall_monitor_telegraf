powerwall_telegraph_monitor
===========================

Copyright (c) 2021 Patrick Wagstrom &lt;patrick@wagstrom.net&gt;

This is a very simple Docker container designed to do one thing, pull data from
a Tesla Powerwall with Telegraf and push it to a remote host. This doesn't seem
very difficult, but a Tesla Powerwall/Energy Gateway now has an enhanced
authentication mechanism that requires a cookie to be passed not just a
username/password in the URL. Therefore, we need to fetch the cookies and then
find a way to proxy it through.

Usage
-----

The easiest way to use this is as part of the larger [powerwall_monitor
project](https://github.com/mihailescu2m/powerwall_monitor), which provides
Grafana and InfluxDB instances necessary to have a complete
dashboard setup.

```yaml
    telegraf:
        image: pridkett/powerwall_monitor_telegraf:latest
        container_name: telegraf
        hostname: telegraf
        restart: always
        volumes:
            - type: bind
              source: ./telegraf.conf
              target: /etc/telegraf/telegraf.conf
              read_only: true
        extra_hosts:
            - "powerwall: 192.168.91.1"
        depends_on:
            - influxdb
        environment:
            - POWERWALL_PASSWORD=Your_Powerwall_Password
```

Alternatively, if you don't like putting your password into a YAML file
because, for example, you think might check the YAML file into source control,
you can use an environment file by replacing the `environment` key above with
this block of code:

```yaml
        env_file:
            - .env.telegraf
```

And then create a file called `.env.telegraf` with a single line in it:

```ini
POWERWALL_PASSWORD=Your_Powerwall_Password
```

But, for the most part, this should be plug and play with the rest of the
powerwall\_monitor project.

Configuration Caveats
---------------------

There are two different hostnames that the container expects to be able to
reach. `powerwall` should be the hostname of your powerwall on your local
network. For most people, you can just change the IP address of your Powerwall
in the YAML snippet above. Secondly, it expects that InfluxDB will be reachable
at the hostname `influxdb`. This is taken care of for your by using the
powerwall_monitor project.

License
-------

Copyright (c) 2021 Patrick Wagstrom.

Licensed under terms of the MIT License

Components in the container (i.e. the Debian base image, telegraf, etc) have
their own associated licenses.
