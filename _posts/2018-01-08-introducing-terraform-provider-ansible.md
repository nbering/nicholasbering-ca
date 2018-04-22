---
layout: post
title: "Introducing: Terraform Provider for Ansible"
description: |
    Officially announcing a little side-project I've been working on for a few months, to integrate Terraform and Ansible into a simplified workflow.
category: tools
tags: [tools, terraform, ansible]
author: Nicholas Bering
date: 2018-01-08
---

When working with Ansible and Terraform, I felt there was a gap in the workflow, so I built a <a href class="tracked" href="https://github.com/nbering/terraform-provider-ansible/">Terraform Provider for Ansible</a>. It integrates with a <a class="tracked" href="https://github.com/nbering/terraform-inventory/">Terraform Inventory</a> script to connect machines in your Terraform state to Ansible. This article explains my thought process in designing this integration.

## Why an Ansible provider for Terraform?
As part of our day-to-day workflow at <a class="tracked" href="https://medstack.co/">MedStack</a>, we use <a class="tracked" href="https://www.terraform.io/">Terraform</a> and <a class="tracked" href="https://www.ansible.com/">Ansible</a>. I did some work early on to manage infrastructure with Ansible, but we found - for our team anyway - that it wasn't a great fit for our workflow. So we brought in Terraform to manage infrastructure, and then Ansible takes over to manage guest machine configuration and software deployment.

But there was still a gap in the workflow. There are often times when you want to connect some piece of the infrastructure to some piece of the software configuration. Here's some things I tried...

### Manage Files with Local-Exec Provisioner
One of the first things I tried was using the <a class="tracked" href="https://www.terraform.io/docs/provisioners/local-exec.html">Terraform Local Exec Provisioner</a> to run scripts that manipulate the Ansible hosts file. This wasn't satisfactory for a few reasons:

- It made the Terraform configuration very messy, and harder to read.
- It was difficult to clean the host file when something was destroyed (<a class="tracked" href="https://www.terraform.io/docs/provisioners/index.html#destroy-time-provisioners">destroy-time provisioners</a> were added later).
- Local copies of the host file that someone didn't check in right away were problematic.

Those were the hardest pieces, but there was so much more. Managing files with local exec just seemed really hacky.

### An Ansible Provisioner
In looking around for solutions, I had also come across <a class="tracked" href="https://github.com/jonmorehouse/terraform-provisioner-ansible">jonmorehouse/terraform-provisioner-ansible</a>. The major limiting factor for this solution was that it uses <a class="tracked" href="http://docs.ansible.com/ansible/latest/playbooks_delegation.html#local-playbooks">ansible-local</a>. There's a few reasons we decided this wasn't a good solution. There's nothing wrong with using ansible in local mode as a rule, but you do lose some of the benefits of the agent-less management solution, since you need to install Ansible on the managed node to run it.

### Dynamic Inventory (on it's own)
Another solution I evaluated was <a class="tracked" href="https://github.com/adammck/terraform-inventory">Terraform Dynamic Inventory for Ansible</a>. This again might work for a lot of people, but when I first looked at this almost a year ago, it was limited to Amazon EC2.

It also relies heavily on the tag systems that are part of the cloud providers' APIs. It just doesn't feel like it will scale well, and you're depending on the managed infrastructure to store details about the configuration used to manage it. There's also too much possibility of someone accidentally deploying resources with a matching or conflicting tag and causing a mess of accidental changes.

Last, it also opened up the question for me... *What is a server anyway?*

There are just so many resources that Terraform can manage that could be considered a server, it seemed brittle to try to inspect the Terraform state file for resources when Terraform manages so many things that could be a server... all with completely difference set of properties. It also manages so many things that *aren't* servers that you end up digging through a lot of noise to find the signal.

### Manual Workflow
For a time, we settled on a semi-automatic workflow, whereby we ran Terraform to modify infrastructure components, and then scripted outputs using `terraform console` to dump the needed configuration values to files, which could be loaded with the Ansible file lookups, with their locations being defined by certain conventions set by the team.

This works fairly well, with the biggest argument in favor being that there was little to write, and there was always the promise that if we found something better we could easily migrate.

### Something better...
After spending some more time thinking about the problem - and exposing myself to as many use cases from the community as I could - I settled on writing a Terraform Logical Provider.

I was inspired by the <a class="tracked" href="https://www.terraform.io/docs/providers/random/index.html">Random Provider</a>, to write a solution that is somewhere between what we were doing manually, and the existing <a class="tracked" href="https://github.com/adammck/terraform-inventory">Dynamic Inventory</a> solution.

The logical provider creates an anchor that the inventory script can use to search for server resources. It also allows you to export arbitrary configuration values from the terraform state by interpolating them into a vars hash, eliminating the need on our team to copy values from Terraform state to files.

Last, it removed the need to manage a separate hosts file, which is what <a class="tracked" href="http://docs.ansible.com/ansible/latest/intro_dynamic_inventory.html">dynamic inventory</a> was meant for in general.

## Components of the System
### Terraform Provider for Ansible
You can find this part on my Github account at <a class="tracked" href="https://github.com/nbering/terraform-provider-ansible/">https://github.com/nbering/terraform-provider-ansible/</a>.

This is the logical provider I mentioned earlier. It simply creates a record in the Terraform state file for the dynamic inventory script to dig for. A simple configuration might looks something like this:

```hcl
resource "ansible_host" "example" {
    inventory_hostname = "www.example.com"
    groups = ["web"]
    vars {
        foo = "bar"
    }
}
```

One of the benefits with this is that the `inventory_hostname` can be interpolated from a variety of places, including:
- `aws_eip.example.public_ip`
- `aws_instance.example.public_dns`
- `azurerm_public_ip.example.fqdn`
- `cloudflare_record.example.hostname`

We can also set some of the properties for the Ansible connection based on the Terraform configuration, so that your more-or-less static Ansible configuration doesn't need to know which provider is beneath a server. For example, the Debian base images on AWS EC2 provision your key for the `admin` user by default... whereas on Azure this username is <a class="tracked" href="https://docs.microsoft.com/en-us/azure/virtual-machines/linux/faq?toc=%2Fazure%2Fvirtual-machines%2Flinux%2Ftoc.yml">blacklisted</a>. So we could do something like this, with the same variable being used to provision the admin user on the Azure Linux VM:

```
resource "ansible_host" "example" {
    inventory_hostname = "www.example.com"
    vars {
        ansible_user = "${var.admin_username}"
    }
}
```

### Terraform State
One of the more-or-less unique parts of Terraform as an operations tool is the way it maintains state, and then applies changes only if it needs them. The way that desired state and existing state have the differences mapped and the plan built from the differences is actually quite sophisticated.

My solution to inter-operate between Ansible and Terraform relies on this state data. By default, Terraform stores the state on the local filesystem as a file called `terraform.tfstate`. It's generally recommended not to store this information in source control, as it may contain sensitive information. The current best practice is to configure Terraform to use a <a class="tracked" href="https://www.terraform.io/docs/backends/index.html">remote state backend</a>, so that the state of infrastructure is not centralized on the user's filesystem. Remote state backends assist in security and collaboration.

Terraform's state storage mechanism is also what allows us to inter-operate between Terraform and Ansible. This was made very simple by the fact that Terraform's state is stored in a fairly simple JSON format. in my sample data set that I used for testing, the state file looks like this:

```json
{
    "version": 3,
    "terraform_version": "0.11.0",
    "serial": 2,
    "lineage": "493f46cf-d49e-4e5e-a6eb-9d21744a27a6",
    "modules": [
        {
            "path": [
                "root"
            ],
            "outputs": {},
            "resources": {
                "ansible_host.db": {
                    "type": "ansible_host",
                    "depends_on": [],
                    "primary": {
                        "id": "db.example.com",
                        "attributes": {
                            "groups.#": "2",
                            "groups.0": "example",
                            "groups.1": "db",
                            "id": "db.example.com",
                            "inventory_hostname": "db.example.com",
                            "vars.%": "2",
                            "vars.bar": "ddd",
                            "vars.fooo": "ccc"
                        },
                        "meta": {},
                        "tainted": false
                    },
                    "deposed": [],
                    "provider": "provider.ansible"
                },
                "ansible_host.www": {
                    "type": "ansible_host",
                    "depends_on": [],
                    "primary": {
                        "id": "www.example.com",
                        "attributes": {
                            "groups.#": "2",
                            "groups.0": "example",
                            "groups.1": "web",
                            "id": "www.example.com",
                            "inventory_hostname": "www.example.com",
                            "vars.%": "2",
                            "vars.bar": "bbb",
                            "vars.fooo": "aaa"
                        },
                        "meta": {},
                        "tainted": false
                    },
                    "deposed": [],
                    "provider": "provider.ansible"
                }
            },
            "depends_on": []
        }
    ]
}

```

There is one caveat to using this integration point: Terraform's developers do not consider the state file format to be a public interface. So they reserve the right to make breaking changes that to the state format that aren't reflected as breaking in the version number. That said... in practice, the state file format hasn't changed often so far. Also, if they do, the concept of state is so fundamental to Terraform that there will also be some mechanism for storing it. So if it does change, there will be a way for our Ansible integration to follow along.

### Ansible Dynamic Inventory
You can find this part of the solution at <a class="tracked" href="https://github.com/nbering/terraform-inventory/">https://github.com/nbering/terraform-inventory/</a>.

This part is just a simple script that shells out to `terraform state pull` to retrieve the JSON state data, and then just finds the records that are `ansible_host` resources, and assembles them in a way that Ansible can understand... like this (formed from the json state data in the previous section):

```json
{
  "web": [
    "www.example.com"
  ],
  "all": [
    "www.example.com",
    "db.example.com"
  ],
  "db": [
    "db.example.com"
  ],
  "_meta": {
    "hostvars": {
      "db.example.com": {
        "fooo": "ccc",
        "bar": "ddd"
      },
      "www.example.com": {
        "fooo": "aaa",
        "bar": "bbb"
      }
    }
  },
  "example": [
    "www.example.com",
    "db.example.com"
  ]
}
```

Ansible's dynamic inventory feature is designed to detect that the inventory file is not a flat text file, but a script or executable. If it is executable, Ansible will run it, and parse the standard out as the above JSON format. Here's an example of a command using this feature:

```
ansible-playbook -i /etc/ansible/terraform.py playbook.yml
```

When using a dynamic inventory script like this, we are normally giving up support for the plain text host file. While this is true in the traditional sense, we still gain some backwards compatibility because the Terraform `ansible_host` resource does not need to represent a server managed by Terraform. This allows you to mix-and-match Terraform-managed resources with your on-premises, manually-configured or legacy systems.

## Conclusion

By using the plugin and state storage mechanisms of Terraform, and the dynamic inventory feature of Ansible, we can automate the integration of the two very nicely.
