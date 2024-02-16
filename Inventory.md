[Inventory](https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html#inventory-basics-formats-hosts-and-groups)

````
ungrouped:
  hosts:
    mail.example.com:
webservers:
  hosts:
    foo.example.com:
    bar.example.com:
dbservers:
  hosts:
    one.example.com:
    two.example.com:
    three.example.com:
````

Default groups

Even if you do not define any groups in your inventory file, Ansible creates two default groups: `all` and `ungrouped`.

The `all` group contains every host.

The `ungrouped` group contains `all` hosts that don’t have another group aside from all.

Every host will always belong to at least 2 groups (`all` and `ungrouped` or `all` and some other group).

For example, in the basic inventory above, the host `mail.example.com` belongs to the `all` group and the `ungrouped` group;
the host `two.example.com` belongs to the `all` group and the `dbservers` group.

Though `all` and `ungrouped` are always present, they can be implicit and not appear in group listings like `group_names`.

````
**What** - An application, stack or microservice (for example, database servers, web servers, and so on).

**Where** - A datacenter or region, to talk to local DNS, storage, and so on (for example, east, west).

**When** - The development stage, to avoid testing on production resources (for example, prod, test).
````