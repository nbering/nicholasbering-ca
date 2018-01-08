---
layout: post
title: "Introducing: Terraform Provider for Ansible"
description: |
    Officially announcing a little side-project I've been working on for a few months.
category: tools
tags: [tools, terraform, ansible]
author: Nicholas Bering
date: 2018-01-07
---
## Why an Ansible provider for Terraform?
As part of our day-to-day workflow at [MedStack][1], we use [Terraform][2] and [Ansible][3]. I did some work early on to manage infrastructure with Ansible, but we found for our team that it wasn't a great fit for our workflow. So we brought in Terraform to manage infrastructure, and then Ansible takes over to manage environment configuration and software deployment.

But there was still a gap in the workflow. There are often times when you want to connect some piece of the infrastructure to some piece of the software configuration. Here's some things I tried...

### Manage Files with Local Exec
One of the first things I tried was using the [Terraform Local Exec Provisioner][4] to run scripts that manipulate the Ansible hosts file. This wasn't satisfactory for a few reasons:

- It made the Terraform configuration very messy, and harder to read.
- It was difficult to clean the host file when something was destroyed ([destroy-time provisioners][5] were added later).
- Local copies of the host file that someone didn't check in right away were problematic.

Those were the hardest pieces, but there was so much more. Managing local files with local exec just seemed really hacky.

### An Ansible Provisioner
In looking around for solutions, I had also come across [jonmorehouse/terraform-provisioner-ansible][6]. The major limiting factor for this solution was that it uses [ansible-local][7]. There's a few reasons we decided this wasn't a good solution. There's nothing wrong with using ansible in local mode as a rule, but it just didn't fit our groove.

### Dynamic Inventory (on it's own)
Another solution I evaluated was [Terraform Dynamic Inventory for Ansible][10]. This again might work for a lot of people, but when I first looked at this almost a year ago, it was limited to Amazon EC2.

I've also never really liked relying heavily on Tags to manage infrastructure. It just doesn't feel like it will scale well. There's also too much possibility of someone accidently deploying resources with a matching tag and causing a mess of accidental changes.

Last, it also opened up the question for me... *What is a server anyway?*

There are just so many resources that Terraform can manage that could be considered a server, it seemed brittle to try to inspect the Terraform state file for resources when Terraform manages so many things that could be a server... all with completely difference set of properties. It also manages so many things that *aren't* servers that you end up digging through a lot of noise to find the signal.

### Manual Workflow
In the end, we settled in a semi-automatic workflow, whereby we ran Terraform to modify infrastructure components, and then scripted outputs using `terraform console` to dump the needed configuration values to files, which could be loaded with the Ansible file lookups, with their locations being defined by certain conventions set by the team.

This works fairly well, with the biggest argument in favor being that there was little to write, and there was always the promise that if we found something better we could easily migrate.

### Something better...
After spending some more time thinking about the problem, and exposing myself to as many use cases from the community as I could... I settled on writing a Terraform Logical Provider.

I was inspired by the [Random Provider][9]. This inspired me to write a solution that is somewhere between what we were doing manually, and the existing [Dynamic Inventory][10] solution.

Adding logical provider creates an anchor for the inventory script can use to search for server resources. It also allows you to export arbitrary configuration values from the terraform state by interpolating them into a vars hash, eliminating the need on our team to copy values from Terraform state to files.

It's also removed the need to manage a separate hosts file, which is what [dynamic inventory][11] was meant for in general.

## The Two-Part Solution
### Terraform Provider for Ansible
You can find this part at [https://github.com/nbering/terraform-provider-ansible/][12].

This is the Logical provider I mentioned earlier. This simply creates a record in the Terraform state file for the dynamic inventory script to dig for.

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
- `azurerm_public_ip.example.fqdn`
- `cloudflare_record.example.hostname`

We can also set some of the properties for the ansbile connection based on the Terraform configuration, so that your Ansible configuration doesn't need to know about the infrastructure. For example, the Debian base images on AWS EC2 provision your key for the `admin` user by default... whereas on Azure this username is [blacklisted][14]. So we could do something like this, with the same variable being used to provision the admin user on the Azure Linux VM:

```
resource "ansible_host" "example" {
    inventory_hostname = "www.example.com"
    vars {
        ansible_user = "${var.admin_username}"
    }
}
```

When using a dynamic inventory script like this, we are normally giving up support for the plain text host file. While this is true in the traditional sense, we still gain some backwards compatibility because the `ansible_host` resource does not need to represent a server managed by Terraform. This allows you to mix-and-match Terraform-managed resources with your on-premises or legacy systems.

### Ansible Dynamic Inventory
You can find this part of the solution at [https://github.com/nbering/terraform-inventory/][13].

This part is just a simple script that shells out to `terraform state pull` to retrieve the JSON state file, and then just finds the records that are `ansible_host` resources, and assembles them in a way that Ansible can understand.

```
ansible-playbook -i /etc/ansible/terraform.py playbook.yml
```

[1]: https://medstack.co/
[2]: https://www.terraform.io/
[3]: https://www.ansible.com/
[4]: https://www.terraform.io/docs/provisioners/local-exec.html
[5]: https://www.terraform.io/docs/provisioners/index.html#destroy-time-provisioners
[6]: https://github.com/jonmorehouse/terraform-provisioner-ansible
[7]: http://docs.ansible.com/ansible/latest/playbooks_delegation.html#local-playbooks
[8]: http://docs.ansible.com/ansible/latest/playbooks_lookups.html#intro-to-lookups-getting-file-contents
[9]: https://www.terraform.io/docs/providers/random/index.html
[10]: https://github.com/adammck/terraform-inventory
[11]: http://docs.ansible.com/ansible/latest/intro_dynamic_inventory.html
[12]: https://github.com/nbering/terraform-provider-ansible/
[13]: https://github.com/nbering/terraform-inventory/
[14]: https://docs.microsoft.com/en-us/azure/virtual-machines/linux/faq?toc=%2Fazure%2Fvirtual-machines%2Flinux%2Ftoc.yml
