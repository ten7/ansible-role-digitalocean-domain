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

## Limitations

### Updating the record value

The DigitalOcean API relies on internally generated IDs to update records. Since this role does not use IDs, it relies on the record type, name, and value instead to determine a match when doing updates or deletions of records.

This creates a problem when if your update involves changing the value of a record, as the role cannot tell if you're updating an existing record, or making a new one.

To solve this you can either update the record manually prior to deployment, or explicitly delete the existing record in code:

```yaml
digitalocean_domain:
  name: 'example.com'
  state: present
  records:
    - name: "@"
      type: "A"
      value: "127.0.0.1"
      state: absent
    - name: "@"
      type: "A"
      value: "192.168.1.1"
      state: present
```

### Unchanged records always update

Another limitation is that even if a record has no changes from the last invocation, the role will still call the API to update the record. Since the update record is the same as the old record, no effective change has been made, but it does result in additional API calls to DigitalOcean. 

Once deployed, you remove the `absent` records in successive deployments as they are no longer necessary.

## Dependencies

None, but the following roles are recommended:

* [ten7/digitalocean](https://galaxy.ansible.com/ten7/digitalocean)
* [ten7/digitalocean_kubeconfig](https://galaxy.ansible.com/ten7/digitalocean_kubeconfig)

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
