---
layout: docs
title: Brightbox Cloud
category: running_coreos
sub_category: cloud_provider
weight: 10
---

# Running CoreOS {{site.brightbox-version}} on Brightbox Cloud

CoreOS is currently in heavy development and actively being tested.  These
instructions will walk you through running a CoreOS cluster on Brightbox. This guide uses the [Brightbox CLI](http://brightbox.com/docs/guides/cli/getting-started/) but you can also use the [Brightbox Manager](http://brightbox.com/docs/guides/manager/).

## Firewall Policy

First of all, let’s create a server group to put the new servers into:

```sh
$ brightbox groups create -n "coreos"

Creating a new server group

 id         server_count  name  
---------------------------------
 grp-cdl6h  0             coreos
---------------------------------
```

And then create a [firewall policy](http://brightbox.com/docs/guides/cli/firewall/) for the group using its identifier:

```sh
$ brightbox firewall-policies create -n "coreos" grp-cdl6h

 id         server_group  name  
---------------------------------
 fwp-dw0n6  grp-cdl6h     coreos
---------------------------------
```

### Firewall Rules

Now let’s define the firewall rules for this new policy. First we’ll allow ssh access in from anywhere:

```sh
$ brightbox firewall-rules create --source any --protocol tcp --dport 22 fwp-dw0n6

 id         protocol  source  sport  destination  dport  icmp_type  description
--------------------------------------------------------------------------------
 fwr-i513z  tcp       any     -      -            22     -                     
-------------------------------------------------------------------------------- 
```

And then we’ll allow the CoreOS etcd ports `7001` and `4001`, allowing access from only the other nodes in the group.

```sh
$ brightbox firewall-rules create --source grp-cdl6h --protocol tcp --dport 7001,4001 fwp-dw0n6

 id         protocol  source     sport  destination  dport      icmp_type  description
---------------------------------------------------------------------------------------
 fwr-xax48  tcp       grp-cdl6h  -      -            7001,4001  -                     
--------------------------------------------------------------------------------------- 
```

And then allow all outgoing access from the servers in the group:


```sh
$ brightbox firewall-rules create --destination any fwp-dw0n6

 id         protocol  source  sport  destination  dport  icmp_type  description
--------------------------------------------------------------------------------
 fwr-dtzim  -         -       -      any          -      -                     
-------------------------------------------------------------------------------- 
```

## List Images

You can find it by listing all images and grepping for CoreOS:

```sh
$ brightbox images list | grep CoreOS

 id         owner      type      created_on  status   size   name
 ---------------------------------------------------------------------------------------------------------
 {{site.brightbox-id}}  brightbox  official  2013-12-15  public   5442   CoreOS {{site.brightbox-version}} (x86_64)
 ```

## Building Servers

Before building the cluster, we need to generate a unique identifier for it, which is used by CoreOS to discover and identify nodes.

You can use any random string so we’ll use the `uuid` tool here to generate one:

```sh
$ TOKEN=`uuid`

$ echo $TOKEN
53cf11d4-3726-11e3-958f-939d4f7f9688
```

Then build three servers using the image, in the server group we created and specifying the token as the user data:

```sh
$ brightbox servers create -i 3 --type small --name "coreos" --user-data $TOKEN --server-groups grp-cdl6h {{site.brightbox-id}}

Creating 3 small (typ-8fych) servers with image CoreOS {{site.brightbox-version}} ({{ site.brightbox-id }}) in groups grp-cdl6h with 0.05k of user data

 id         status    type   zone   created_on  image_id   cloud_ip_ids  name  
--------------------------------------------------------------------------------
 srv-ko2sk  creating  small  gb1-a  2013-10-18  {{ site.brightbox-id }}                coreos
 srv-vynng  creating  small  gb1-a  2013-10-18  {{ site.brightbox-id }}                coreos
 srv-7tf5d  creating  small  gb1-a  2013-10-18  {{ site.brightbox-id }}                coreos
--------------------------------------------------------------------------------
```

## Accessing the Cluster

Those servers should take just a minute to build and boot. They automatically install your Brightbox Cloud ssh key on bootup, so you can ssh in straight away as the `core` user.

If you’ve got ipv6 locally, you can ssh in directly:

```sh
$ ssh core@ipv6.srv-n8uak.gb1.brightbox.com
The authenticity of host 'ipv6.srv-n8uak.gb1.brightbox.com (2a02:1348:17c:423d:24:19ff:fef1:8f6)' can't be established.
RSA key fingerprint is 99:a5:13:60:07:5d:ac:eb:4b:f2:cb:c9:b2:ab:d7:21.
Are you sure you want to continue connecting (yes/no)? yes

Last login: Thu Oct 17 11:42:04 UTC 2013 from srv-4mhaz.gb1.brightbox.com on pts/0
   ______                ____  _____
  / ____/___  ________  / __ \/ ___/
 / /   / __ \/ ___/ _ \/ / / /\__ \
/ /___/ /_/ / /  /  __/ /_/ /___/ /
\____/\____/_/   \___/\____//____/
core@srv-n8uak ~ $
```

If you don’t have ipv6, you’ll need to [create and map a Cloud IP](http://brightbox.com/docs/guides/cli/cloud-ips/) first.

## Using CoreOS

Now that you have a cluster bootstrapped it is time to play around.
Check out the [CoreOS Quickstart]({{site.url}}/docs/quickstart) guide or dig into [more specific topics]({{site.url}}/docs).

