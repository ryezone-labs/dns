ryezone_labs.dns
=========

This role installs and configures a dns server on Ubuntu.

Role Variables
--------------

### Global Role Variables

See [Common Variables](https://github.com/ryezone-labs/documentation/blob/master/common-variables.md)

Common Variables used:

```yaml
ryezone_labs_external_dns:
  - 1.1.1.1
  - 1.0.0.1
ryezone_labs_top_level_domain: domain.local
ryezone_labs_lanAddress: 10.100.10.1
```

### Configure systemd resolved

```yaml
dns_systemd_resolved:
  disabled: true
  nameservers: "{{ ryezone_labs_external_dns }}"
  search_zones: 
    - "{{ ryezone_labs_top_level_domain }}"
```

- `disabled` (bool) 

  When `true` resolved is disabled.

- `nameservers` (list of string) 

  List of nameservers used by resolved.

- `search_zones` (list of string) 

  List of default search zones to use in resolution when a partial hostname is provided.

### Configure bind

```yaml
dns_bind:
  forwarders: "{{ ryezone_labs_external_dns }}"
  listen_on_v6:
    - none
  listen_on_v4:
    - 127.0.0.1
    - "{{ ryezone_labs_lanAddress }}"
  zones:
    - name: "{{ ryezone_labs_top_level_domain }}"
      TTL: 604800
      serial: 5
      refresh: 604800
      retry: 86400
      expire: 2419200
      negative_cache_ttl: 604800
      nameserver: "router.{{ ryezone_labs_top_level_domain }}"
      records:
        - name: router
          class: IN
          type: A
          target: "{{ ryezone_labs_lanAddress }}"
```

- `forwarders` (list of string) 
   
   List of external DNS servers to forward requests for which the dns server is not authoritative.

- `listen_on_v6` (list of string)

  List of IPV6 IP addresses to listen for dns requests.

- `listen_on_v4` (list of string)

  List of IPV4 IP addresses to listen for dns requests.

- `zones` (list of hashes)

  List of zones to configure dns entries.  See Configuring Zones and Configuring Reverse Zones for more info.

### Common Zone Configuration Values

- `name` (string)

  Name of the Domain to answer dns requests for.

- `TTL` (integer)

  Length of time in seconds for which a DNS recrd will be cached by an outside DNS server or resolver.

- `serial` (integer)

  Version number of the zone.  As changes are made to the zone file this number should increase.

- `refresh` (integer)

  Length of time in seconds nameserver should wait prior to checking for a Serial Number increase within the primary zone file.

- `retry` (integer)

  Length of time in seconds a nameserver should wait prior to retrying to update a zone after a failed attempt.

- `expire` (integer)

  Length of time in seconds a nameserver should wait prior to considering data from a secondary zone invalid and stop answering queries for that zone.

- `negative_cache_ttl` (integer)

  Length of time in seconds a client should cache a negative result from the nameserver.

- `nameserver` (string)

  Name of the authoritative name server for the zone.


### Configuring Records

- `name` (string)

  Subdomain for which to answer dns requsts.

- `class` (string)

  Class of DNS record to configure.  Valid classes include:

    - `IN` = Internet
    - `CH` = Chaosnet
    - `HS` = Hesiod

- `type` (string)

  Type of DNS record to configure.  A valid list of DNS types can be found [here](https://simpledns.com/help/dns-record-types).

- `target` (string)

  Target host address or dns name of the server for which the nameserver is returning a response.

License
-------

BSD
