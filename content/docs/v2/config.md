+++
date = "2016-11-21T14:52:41+01:00"
title = "Chihaya Configuration File"

+++

Chihaya uses the `chihaya` namespace for its configuration.
Top-level properties are `announce_interval`, which defines the announce interval to return for announces, and `prometheus_addr`, which defines the endpoint for prometheus.

In the following, the other top-level sections are explained.


## `http`

`http` configures the HTTP frontend. 

- `addr` defines the address to listen on.
- `allow_ip_spoofing` controls whether or not IP spoofing should be allowed.
    If IP spoofing is disallowed, the remote address of the HTTP request will be used as the Peer's IP.
- `real_ip_header` specifies the HTTP header field to use as the remote IP (for running behind a proxy, for example).
    This works regardless of `allow_ip_spoofing` being set.
    If `allow_ip_spoofing` is enabled, the IP address contained in the announce is treated with a higher priority than the one specified in `real_ip_header`.
- `read_timeout` specifies the read timeout for requests.
- `write_timeout` specifies the write timeout for requests.
- `request_timeout` is the duration to allow for requests.

An HTTP frontend will only be created if the `addr` field is set.

```
http:
  addr: 0.0.0.0:6881
  allow_ip_spoofing: false
  real_ip_header: x-real-ip
  read_timeout: 5s
  write_timeout: 5s
  request_timeout: 5s
```

## `udp`

`udp` configures the UDP frontend.

- `addr` defines the address to listen on.
- `allow_ip_spoofing` controls whether or not IP spoofing should be allowed.
- `max_clock_skew` TODO explain this.
- `private_key` sets a secret key to use for HMAC calculation for connection IDs.

A UDP frontend will only be created if the `addr` field is set.

```
udp:
  addr: 0.0.0.0:6881
  allow_ip_spoofing: false
  max_clock_skew: 10s
  private_key: |
    paste a random string here that will be used to hmac connection IDs
```

## `storage`

`storage` configures the storage of peers.

- `gc_interval` is the interval at which the storage is searched for inactive Peers.
    Peers are considered inactive if no announce has been received for `peer_lifetime`.
    Inactive Peers are removed.
- `peer_lifetime` specifies the lifetime of Peers, see `gc_interval`.
    Using a small increment of the `announce_interval` is recommended.
    E.g. if the `announce_interval` is 10 minutes, set `peer_lifetime` to 12-14 minutes.
- `shards` specifies the number of shards to use for infohashes.
    A shard is a subset of the set of all possible infohashes.
    Using more shards increases parallelism of the storage, but allocates more base memory.
- `max_numwant` is the maximum number of Peers returned for one announce.

```
storage:
  gc_interval: 10m
  peer_lifetime: 12m
  shards: 1024
  max_numwant: 100
```

## `prehooks`

`prehooks` specifies a list of Pre-Hook middleware to use.

- `name` specifies the name of a middleware.
- `config` specifies the configuration for the middleware.

```
prehooks:
- name: jwt
  config:
    issuer: "https://issuer.com"
    audience: "https://chihaya.issuer.com"
    jwk_set_url: "https://issuer.com/keys"
    jwk_set_update_interval: 5m
- name: client approval
  config:
    whitelist:
    - OP1011
    blacklist:
    - OP1012
```

## `posthooks`

`posthooks` specifies a list of Post-Hook middleware to use.

- `name` specifies the name of a middleware.
- `config` specifies the configuration for the middleware.

```
posthooks:
- name: gossip
  config:
    boostrap_node: "127.0.0.1:6881"
```

