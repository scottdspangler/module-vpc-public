**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-vpc>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-vpc/blob/master/modules/vpc-peering-external/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# VPC Peering For External VPCs Module

This Terraform Module creates route table entries for a [VPC Peering
Connection](http://docs.aws.amazon.com/AmazonVPC/latest/PeeringGuide/Welcome.html) between one of your internal VPCs (e.g.
Stage or Prod) and an external VPC managed by a 3rd party (e.g. a SaaS provider). For example, [QBox](https://qbox.io/)
is a SaaS provider that runs an Elasticsearch cluster for you in their own VPC and sends you a VPC Peering Connection
request to give you access to that VPC. This module can be used to set up the necessary routes so your VPC can talk to
the external VPC. Since VPC Peering is bidirectional, once you accept the connection, their VPC will also be able to
talk to your VPC, so this module also creates a default set of [Network
ACLs](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_ACLs.html) that lock down access as follows:

- Allow outgoing connections from the private app subnets of your VPC to the 3rd party VPC only on specified ports.
- Deny all other outgoing connections to the 3rd party VPC.
- Allow incoming connections from the 3rd party VPC to [ephemeral
  ports](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_ACLs.html) in the private app subnet of your VPC.
  Network ACLs are stateless, so connections are required to to allow "return traffic" (that is, responses to your
  outgoing requests).
- Deny all other incoming connections from the 3rd party VPC.

Rule #3 above is the risky one, as it allows the 3rd party VPC to make requests to your private app subnets on a wide
variety of ports. Unfortunately, there is no way to lock this down further with Network ACLs. To ensure this doesn't
cause security problems, you need to:

1. Only allow VPC peering connections from trusted 3rd parties. Do your homework and research them thoroughly. Of
   course, it's possible the 3rd party has a bug or gets hacked, in which case you fall back to the next rule.
1. Ensure every single resource in your private app subnets has a security group that only allows incoming connections
   from resources you trust (e.g. specific security groups or CIDR blocks). *Never* allow incoming connections from
   "anywhere" (0.0.0.0/0) in a security group, as "anywhere" will now include the 3rd party VPC.

## How do you use this module?

Check out the [vpc-peering-external example](/examples/vpc-peering-external).

Check out [vars.tf](vars.tf) for all the configuration options available.

## What's a VPC?

A [VPC](https://aws.amazon.com/vpc/) or Virtual Private Cloud is a logically isolated section of your AWS cloud. Each
VPC defines a virtual network within which you run your AWS resources, as well as rules for what can go in and out of
that network. This includes subnets, route tables that tell those subnets how to route inbound and outbound traffic,
security groups, firewalls for the subnet (known as "Network ACLs"), and any other network components such as VPN connections.

## What's a Peering Connection?

A [VPC Peering Connection](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/vpc-peering.html) is a networking
connection between two VPCs that enables you to route traffic between them using private IP addresses. Instances in
either VPC can communicate with each other as if they are within the same network. You can create a VPC peering
connection between your own VPCs, or with a VPC in another AWS account within a single region.

## How do you work with Peering Connections from 3rd parties?

To set up a Peering Connection to a 3rd party VPC, the typical flow is:

1. You get in touch with the party (e.g. sign up on their website) and provide them the details of your VPC.
1. The 3rd party sends you a VPC Peering Connection Request.
1. You accept this Connection Request manually in the VPC console. This creates a Peering Connection.
1. Run this module to setup the Routing Rules and Network ACLs for the Peering Connection.

## What's a Network ACL?

[Network ACLs](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_ACLs.html) provide an extra layer of network
security, similar to a [security group](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-network-security.html).
Whereas a security group controls what inbound and outbound traffic is allowed for a specific resource (e.g. a single
EC2 instance), a network ACL controls what inbound and outbound traffic is allowed for an entire subnet.
