# Ansible Role: digitalocean_domain

This role allows you to programmatically create and manage domains and domain records on DigitalOcean.

## Requirements

* You must have a DigitalOcean API key.
* The DigitalOcean API must be accessible.

## Role Variables

Available variables are listed below.

### API Token

```
digitalocean_api_token: '1234567890abscdefg'
```

Your Digital Ocean API token.

### Specifying domains

This role only works with on domain at a time. You may, however, run it multiple times in the same playbook with an `include_role`.

```yaml
digitalocean_domain:
  name: 'example.com'
  state: present
```

Where:
* **name** is the domain name to manage. Required.
* **state** is if the domain is `present` or `absent`. Optional, defaults to `present`.

### Creating records

Records for the domain are created under the `records` list:

```yaml
digitalocean_domain:
  name: 'example.com'
  state: present
  records:
    - name: "@"
      type: "A"
      value: "127.0.0.1"
      state: absent
```

Where, in each list item:

* **name** is the name of the record. For most types, this is the "hostname". Required.
* **type** is the type of the record such as `A`, `AAAA`, `TXT`, `CNAME`, `MX`, `SOA` or other DNS entry types as supported by DigitalOcean.
* **value** is the value field for the record. Required.
* **state** is if the record is `present`, or `absent`. Optional, defaults to `present`.
* **priority** is the priority of the host. Required for `SRV` and `MX` records. Optional otherwise.
* **port** is the port that the service is accessible on. Required for `SRV` records only. Optional otherwise.
* **ttl** is the Time To Live for the record in seconds. Optional.
* **weight** is the weight of records with the same priority. Required for `SRV` records only. Optional otherwise.
* **flags** is an unsigned integer between 0-255 for `CCA` records.
* **tag** is the parameter tag for `CAA` records. Valid values are `issue`, `issuewild`, or `iodef`.

## Dependencies

None, but the following roles are recommended:

* [socketwench/digitalocean](https://galaxy.ansible.com/socketwench/digitalocean)
* [socketwench/digitalocean_kubeconfig](https://galaxy.ansible.com/socketwench/digitalocean_kubeconfig)

## Example Playbook

```yaml
    ---
    - hosts: all
      vars:
        digitalocean_api_token: '1234567890abscdefg'
        digitalocean_domain:
          name: 'example.com'
          state: absent
          records:
            - name: "@"
              type: "A"
              value: "127.0.0.1"
              state: absent

      tasks:
        - name: provision domains
          include_role:
            name: "ten7.digitalocean_domain"
```
## Provisioning multiple domains

One limitation of this role is it can only be run on one domain at a time. This was done to make the role easier to read and maintain. Since the role is re-entrant, you can run it in a loop:

```yaml
    ---
    - hosts: all
      vars:
        digitalocean_api_token: '1234567890abscdefg'
        _all_my_domains:
          - name: 'example.com'
            state: present
            records:
              - name: "@"
                type: "A"
                value: "127.0.0.1"
                state: present
          - name: 'foo.test'
            state: present
            records:
              - name: "@"
                type: "A"
                value: "127.0.0.1"
                state: present

      tasks:
        - name: Provision domains
          include_role:
            name: "ten7.digitalocean_domain"
          loop: "{{ _all_my_domains }}"
          loop_control:
            label: "{{ digitalocean_domain.name }}"
            loop_var: digitalocean_domain
```

## License

GPL v3

## Author Information

This role was created by [socketwench](https://deninet.com/) for [TEN7](https://ten7.com).
