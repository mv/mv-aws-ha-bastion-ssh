# AWS - Highly available: Bastion SSH

A Cloudformation template for a highly available SSH bastion server.


## Scenario

When working in a VPC, most of your servers can move naturally to a private subnet for better organization and security.

Servers in a private subnet don't have a public IP address, and without a VPN or a DirectConnect option, a bastion host -- also known as a jump host -- is the expected mechanism to reach your servers in a private subnet.

For higher availability, a bastion host is provisioned inside of an auto-scaling group with min:1/max:1 configuration. If for some reason the bastion server becomes unavailable or is terminated, auto-scaling will provision a new machine in its place.

To avoid the hassle of setting-up and determine the configuration of 2 different machines (or any number of different machines) inside the same auto-scaling group, the easiest approach is to create each bastion host inside its own auto-scaling group.

For redundancy, a minimum of 2 bastion hosts should be provisioned. Each server in different AZ's.

## Pre-req

For each server, you must:

  * create an EIP (Elatic IP Address)
  * choose a security group
  * choose a public subnet
  * choose if you want to use CloudWatch Logs

## Configuration

The Cloudformation template can be used standalone, directly using the AWS Console. All parameters are self-explanatory.

But, for a better *devops* approach, each server configuration can live in a separate `YAML` configuration file.

For example:

```yml
  # bastion-ssh-01.yml
  ---
  name: bastion-ssh-01
  elastic_ip: 54.54.54.54
  subnet_id: subnet-abc54
  ssh_key_name: nat.pem
  security_group: bastion-ssh
  logs:
    enable: yes
    bucket_path: my-bucket-log
  ---
```
and

```yml
  # bastion-ssh-02.yml
  ---
  name: bastion-ssh-02
  elastic_ip: 54.54.54.55
  subnet_id: subnet-abc55
  ssh_key_name: nat.pem
  security_group: bastion-ssh
  logs:
    enable: yes
    bucket_path: my-bucket-log
  ---
```


## Usage

### From the command line

To create server 1:

```bash
  rake cf-create config=bastion-ssh-01.yml
```

To create server 2:

```bash
  rake cf-create config=bastion-ssh-02.yml
```


### From the AWS Console


## Tasks

A number of `rake tasks` are provided as helpers. 

For more details type `rake --tasks`.


## License

This is free software, and may be redistributed under the terms specified in the LICENSE file.
