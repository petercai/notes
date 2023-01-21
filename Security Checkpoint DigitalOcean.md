# Security Checkpoint | DigitalOcean
### Introduction

This checkpoint is intended to help you assess what you learned from our [introductory articles on security](https://www.digitalocean.com/community/tutorial_series/getting-started-with-cloud-computing#Security), where we introduced recommended security practices and commonly used security tools. You can use this checkpoint to assess your knowledge of these topics, review key terms and commands, and find resources for continued learning.

To run a cloud application efficiently, you should implement industry recommended security practices when configuring your server. Securing your server is critical for protecting your users and personal information. You can set up effective security measures for both the cloud server itself and your web applications.

This checkpoint will focus on securing your server. You’ll find two sections that synthesize the central ideas from the introductory articles: a brief overview of key security terminology and practices, followed by a section on using the command line with subsections related to specific security tools. Each of these sections has interactive components to help you test your knowledge. At the end of this checkpoint, you will find opportunities for continued learning about containers.

Resources
---------

*   [Recommended Security Measures to Protect Your Servers](https://www.digitalocean.com/community/tutorials/recommended-security-measures-to-protect-your-servers)
*   [How To Set Up a Firewall with UFW on Ubuntu 22.04](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu-22-04)
*   [How To Set Up WireGuard on Ubuntu 22.04](https://www.digitalocean.com/community/tutorials/how-to-set-up-wireguard-on-ubuntu-22-04)
*   [How To Set Up and Configure an OpenVPN Server on Ubuntu 22.04](https://www.digitalocean.com/community/tutorials/how-to-set-up-and-configure-an-openvpn-server-on-ubuntu-22-04)
*   [How to Keep Ubuntu 22.04 Servers Updated](https://www.digitalocean.com/community/tutorials/how-to-keep-ubuntu-22-04-servers-updated)
*   [How To Install Suricata on Ubuntu 20.04](https://www.digitalocean.com/community/tutorials/how-to-install-suricata-on-ubuntu-20-04)

What Is Security?
-----------------

When you secure your cloud server, you are able to manage vulnerabilities in your infrastructure and protect against potential harm and malicious attackers.

It’s important to have familiarity with several key terms to understand security practices for cloud computing.

**Terms To Know**

Define each of the following terms, then use the dropdown feature to check your work.

Encryption

_Encryption_ refers to the process of encoding information through an algorithmic transformation, which can then be used for safe transmission or storage.

You can use _symmetrical_ or _asymmetrical_ encryption to achieve your goals, depending on your needs.

SSH

SSH refers to the _Secure Shell protocol_, which enables you to administer your remote servers safely through cryptographically secure connections.

For more on how SSH works, you can review [Understanding the SSH Encryption and Connection Process](https://www.digitalocean.com/community/tutorials/understanding-the-ssh-encryption-and-connection-process).

Firewall

A _firewall_ controls connections for your server in two main ways: detailing what kind of traffic can be routed to and from the servers; and defining what servers are exposed to the network.

When working on your server, you should know whether you are using its [IPv4](https://docs.digitalocean.com/glossary/ipv4/) (32-bit numeric) or [IPv6](https://docs.digitalocean.com/glossary/ipv6/) (128-bit alphanumeric) IP address. Both IPv4 and IPv6 can be used, though we recommend moving to IPv6.

Once your server is set up, it will participate in _public key infrastructure_ (PKI) for certificate management, identification, and communication encryption. TLS/SSL encryption is often used to provide that extra level of security, mostly commonly by providing a certificate from a legitimate [_certificate authority_](https://www.digitalocean.com/community/tutorials/how-to-set-up-and-configure-a-certificate-authority-on-ubuntu-22-04) (CA) to update from an HTTP to an HTTPS server.

**Check Yourself**

Use the dropdown feature to get the answers.

What is a TLS handshake?

TLS, which refers to _Transport Layer Security_, is an encryption protocol for web traffic. In a TLS handshake, a client and a server exchange messages, verify that the message is from the authentic source, and determine an encryption method (a _cipher suite_) that will manage communications for the secure transfer of information.

How does a TLS handshake differ from shared key encryption?

The TLS protocol uses a public and private key encryption method known as _asymmetric encryption_. In this process, there is a key pair.

With shared key encryption, there is an identical key cipher that both the sender and the recipient will use to decrypt messages. This process is known as _symmetric encryption_ and uses a single key.

You can use Let’s Encrypt as a certificate authority to obtain free [TLS/SSL certificates](https://www.digitalocean.com/community/tutorials/how-to-use-certbot-standalone-mode-to-retrieve-let-s-encrypt-ssl-certificates-on-ubuntu-22-04). You can also generate [self-signed certificates](https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-apache-in-ubuntu-22-04). However, a self-signed certificate will not validate your server for users, so you might try [installing an SSL certificate from a commercial certificate authority](https://www.digitalocean.com/community/tutorials/how-to-install-an-ssl-certificate-from-a-commercial-certificate-authority).

In the following sections, you’ll review the core tenets for connecting to your server via SSH, running VPNs, using firewalls, and monitoring your network security.

### Connecting with SSH

In the [SSH Essentials](https://www.digitalocean.com/community/tutorials/ssh-essentials-working-with-ssh-servers-clients-and-keys) article from our [introductory series on cloud servers](https://www.digitalocean.com/community/tutorial_series/getting-started-with-cloud-computing), you generated an _SSH key pair_. That key pair uses an _asymmetric encryption_ method that generates both a private key and a public key. You used that key pair to access your server as a non-root user in the [Initial Server Setup](https://www.digitalocean.com/community/tutorial_collections/initial-server-setup).

**Check Yourself**

Use the dropdown feature to get the answers.

Where are SSH keys typically stored?

In the `~/.ssh/authorized_keys` folder.

What ports do SSH, HTTP, and HTTPS typically run on?

SSH typically runs on port `22`.

HTTP/HTTPS typically run on ports `80` and `443`, respectively.

How can you change default port access?

To update SSH port access, modify the `Port 22` specification in your server’s `sshd_config` file to reference an unused port of your choosing, then restart your SSH daemon.

When you have changed the SSH port, you must specify the new port every time you want to log in to your remote server.

For additional protection, you can [harden OpenSSH](https://www.digitalocean.com/community/tutorials/how-to-harden-openssh-on-ubuntu-20-04) and the [OpenSSH client](https://www.digitalocean.com/community/tutorials/how-to-harden-openssh-client-on-ubuntu-20-04) on your server. By hardening OpenSSH on both the server-side and client-side, you will improve the security around remote access to your server.

To use SSH, you need to configure SSH access with your firewall.

### Using Firewalls

Firewalls control traffic in and out of your server, which can be configured according to your needs. When you [choose an effective firewall policy](https://www.digitalocean.com/community/tutorials/how-to-choose-an-effective-firewall-policy-to-secure-your-servers), you have to consider what kind of policies you want for your server(s) and how different firewall programs will respond to requests.

Some commonly used firewall programs include the [Uncomplicated Firewall (UFW)](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu-22-04) and [firewalld](https://www.digitalocean.com/community/tutorial_collections/how-to-set-up-firewalld), which both act as high-level interfaces to [iptables](https://netfilter.org/projects/iptables/index.html) or [nftables](https://netfilter.org/projects/nftables/index.html). If you’re using an Ubuntu or Debian distro, you’ll likely use UFW as it comes pre-built with the system. For CentOS or Rocky Linux, you’re more likely to use firewalld. To learn more about iptables, you can refer to our articles on [How the Iptables Firewall Works](https://www.digitalocean.com/community/tutorials/how-the-iptables-firewall-works) and [Iptables Essentials: Common Firewall Rules and Commands](https://www.digitalocean.com/community/tutorials/iptables-essentials-common-firewall-rules-and-commands).

You can use both [IPv4](https://docs.digitalocean.com/glossary/ipv4/) and [IPv6](https://docs.digitalocean.com/glossary/ipv6/) when configuring your firewall, though you may need to update your firewall to manage IPv6 as well. UFW, for example, manages only IPv4 by default and needs to be configured manually to write rules for IPv6.

### Running VPNs

A VPN provides an encrypted tunnel through which you can connect to the internet, which can be beneficial for both developers and consumers. For developers, VPNs enable you to access your own infrastructure from various locales so that you don’t need to leave a sensitive port open. As a consumer, a VPN enables you to access the internet securely even when you are connected to an untrusted network (such as WiFi at a coffeeshop or library).

**Check Yourself**

What is the difference between a VPC and a VPN?

_VPC_ refers to a _[Virtual Private Cloud](https://docs.digitalocean.com/products/networking/vpc/) network_, which is a private network interface for your resources. Resources in a VPC can only connect to each other via an internal network and cannot be accessed through the public internet unless _ingress_ gateways are set. A VPC can scale to your needs, providing benefits in workload management and secure connections.

A _VPN_, or _virtual private network_, simulates a private network between remote computers over the internet as if they were on a local private network. VPNs provide a secure gateway to shared network information.

[WireGuard](https://wireguard.com/) and [OpenVPN](https://www.digitalocean.com/community/tutorials/how-to-set-up-and-configure-an-openvpn-server-on-ubuntu-22-04) are two commonly deployed VPN solutions. In our introductory articles, you [set up a WireGuard VPN](https://www.digitalocean.com/community/tutorials/how-to-set-up-wireguard-on-ubuntu-22-04) and [an OpenVPN server](https://www.digitalocean.com/community/tutorials/how-to-set-up-and-configure-an-openvpn-server-on-ubuntu-22-04).

Once you have set up your network, whether with a VPN or not, you’ll want to manage your system long-term for secure and sustainable processes.

### Managing Network Security

Configuring your server setup is one of many steps to ensuring secure practices. You can maintain your server by keeping it up to date, hardening the network, and monitoring your network security.

To [keep an Ubuntu server up-to-date](https://www.digitalocean.com/community/tutorials/how-to-keep-ubuntu-22-04-servers-updated), you might want to update your `systemd` configuration file or schedule a `cron` job for automated rebooting. You can also set up your package manager to complete automatic updates with the `unattended-upgrades` service that you can manage with `systemctl`. If you prefer running a Rocky Linux server, you can refer to our guide on [How To Keep Rocky Linux 9 Servers Updated](https://www.digitalocean.com/community/tutorials/how-to-keep-rocky-linux-9-servers-updated).

Sometimes you may need to run updates at the kernel level in order to patch system-wide bugs and vulnerabilities. While you can run the `unattended-upgrades` tool for `apt`, it may result in some downtime for your system. If you need to ensure consistent uptime, you might use a [load balancer](https://www.digitalocean.com/community/tutorials/what-is-load-balancing) to redirect traffic while different servers run the kernel updates. You can also use a live patching service, like the [Canonical Livepatch Service](https://ubuntu.com/security/livepatch) or [Kernelcare](https://www.kernelcare.com/), to run in the background.

You can also scan and monitor network traffic, looking for vulnerabilities or suspicious packets. You [installed Suricata](https://www.digitalocean.com/community/tutorials/how-to-install-suricata-on-ubuntu-20-04) as a _network monitoring system_, defining rulesets for the service to manage on your behalf.

Connecting to and managing your server is often done via the command line, which you used across these articles on security.

Using the Command Line
----------------------

You began to use the [Linux command line](https://www.digitalocean.com/community/tutorials/a-linux-command-line-primer) with our introductory articles on [cloud servers](https://www.digitalocean.com/community/tutorial_series/getting-started-with-cloud-computing#Cloud-Servers), configured a web server with the [articles on web server solutions](https://www.digitalocean.com/community/tutorial_series/getting-started-with-cloud-computing#Web-Servers), managed your database with [articles on databases](https://www.digitalocean.com/community/tutorial_series/getting-started-with-cloud-computing#Databases), and configured a container solution with the [articles on containers](https://www.digitalocean.com/community/tutorial_series/getting-started-with-cloud-computing#Containers).

In the [introduction to security practices](https://www.digitalocean.com/community/tutorial_series/getting-started-with-cloud-computing#Security), you have continued to develop familiarity with the command line through commands such as:

*   `add-apt-repository` as your sudo-enabled user to add software repository information to your server.
*   `cat` to output a file’s content to the terminal.
*   `chmod` to change file permissions.
*   `cp` to copy files on one server and `scp` to copy files between servers.
*   `cut` to remove a section of a file, using the `-c` option to cut the specified string).
*   `date` to output a timestamp, using the `+%s%N` options to output seconds (`%s`) and minutes (`%N`).
*   `grep` to search text and strings in a specified file.
*   `jq` to read and filter entries as specified by the command syntax.
*   `kill` as your sudo-enabled user to specify a signal by which a service should be stopped.
*   `ln` with the `-s` option to create a [symlink](https://www.digitalocean.com/community/tutorials/workflow-symbolic-links) between files.
*   `printf` to display a given string.
*   `sha1sum` to print and check a checksum.
*   `ss` to list all TCP/UDP ports in use, paired with the `-plunt` options for additional information.
*   `sysctl` as your sudo-user to configure kernel parameters and load new values for your terminal session.
*   `systemctl` to manage services, including OpenVPN as a `systemd` service and Suricata as a networking monitoring package.
*   `resolvectl dns` to identify the DNS resolvers used by your server.
*   `tail` to output lines from a file specified with the `-f` option.
*   `tee` as your sudo-user to redirect an output into a new file.

You used the `ip` command and associated subcommands to configure your network interfaces:

*   `ip addr` to look up your network interfaces. You then used the output with the `ufw allow` command to enable incoming traffic via the selected network interface.
*   `ip address show` to find the public IP address for the system.
*   `ip route` to find the public network interface.

If you opted to run a [live patching service for kernel-level updates to your Ubuntu server](https://www.digitalocean.com/community/tutorials/how-to-keep-ubuntu-22-04-servers-updated#step-3-updating-and-livepatching-the-kernel), you ran subcommands for the `canonical-livepatch` service as your sudo-enabled user:

*   `canonical-livepatch enable your-key` to enable the tool.
*   `canonical-livepatch status` to check the status of the background service.

You also used the pipe operator (`|`) to chain together multiple commands.

### Running UFW from the Command Line

In our [Initial Server Setup](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-22-04), you set up a basic firewall with the Uncomplicated Firewall. You then used `sudo` access to modify your firewall with various subcommands in [How To Set Up a Firewall with UFW on Ubuntu 22.04](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu-22-04):

*   `ufw default deny incoming` to deny all incoming connections (this is the default state).
*   `ufw default allow outgoing` to allow all outgoing connections (this is the default state).
*   `ufw allow ssh` to allow incoming SSH connections on port `22`, such as when you wish to manage a remote server.
*   `ufw allow port_number` to specify a port for incoming connections.
*   `ufw enable` to make the firewall active.
*   `ufw status` to check the status of your firewall and `ufw status verbose` to see all the rules that are set.
*   `ufw allow http` or `ufw allow 80` to allow incoming connections from unencrypted web servers over HTTP.
*   `ufw allow https` or `ufw allow 443` to allow incoming connections from unencrypted web servers over HTTPs.
*   `ufw allow port_number:port_number/tcp` and `ufw allow port_number:port_number/udp` to allow a range of ports, specifying the TCP/UDP protocols.
*   `ufw allow from your_ip_address` to allow connects from a specific IP address. You can add `to any port port_number` to direct the IP address to a specific port.
*   subnets
*   `ufw deny http` to deny HTTP connections and `ufw deny from your_ip_address` to deny all connections from a specific IP address.
*   `ufw status numbered` to generate a numbered list of firewall rules.
*   `ufw delete` to delete rules, using a list number or using the `allow` rule (such as `ufw delete allow http`).
*   `ufw disable` to deactivate all rules you have created.
*   `ufw reset` to disable UFW and delete any rules you created.

You can continue to work on your UFW setup with our article on [UFW Essentials: Common Firewall Rules and Commands](https://www.digitalocean.com/community/tutorials/ufw-essentials-common-firewall-rules-and-commands).

### Running VPNs from the Command Line

In addition to configuring your firewalls, you also managed WireGuard and OpenVPN, two different VPN tools.

When [setting up your WireGuard VPN](https://www.digitalocean.com/community/tutorials/how-to-set-up-wireguard-on-ubuntu-22-04), you ran the following WireGuard commands across both the WireGuard server and its Peer server:

*   `wg` to manage your WireGuard server.
*   `wg genkey` and `wg pubkey` to create a private and public key pair for a WireGuard server.
*   `wg set` with an `allowed-ips` settings and a list of specific IP addresses to manage access to your WireGuard VPN.
*   `wg-quick` establish your VPN connection manually with the `up` argument to start the tunnel and the `down` argument to disconnect from the VPN.

When [setting up your OpenVPN server](https://www.digitalocean.com/community/tutorials/how-to-set-up-and-configure-an-openvpn-server-on-ubuntu-22-04), you ran a series of modified scripts across your OpenVPN server and the CA server that validates certificates, setting configuration directives like the [`tls-crypt` directive](https://www.digitalocean.com/community/tutorials/how-to-set-up-and-configure-an-openvpn-server-on-ubuntu-22-04#step-5-configuring-openvpn-cryptographic-material), to improve cryptographic communications.

Through these articles and best practices, you now know the basics to protect your cloud server.

What’s Next?
------------

In these articles introducing security practices for cloud servers, you have learned about best practices and commonly used tools for building robust security measures in your cloud servers. To ensure that your infrastructure begins with a secure base configuration, you can continue to follow industry best practices for encryption, private networking, security monitoring, and service auditing.

To continue building your server security, try these tutorials next:

*   [How To Set Up a Firewall Using firewalld on Rocky Linux 9](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-using-firewalld-on-rocky-linux-9)
*   [How to Configure a Droplet as a VPC Gateway](https://www.digitalocean.com/docs/networking/vpc/resources/droplet-as-gateway/)
*   [How To Set Up and Configure a Certificate Authority (CA)](https://www.digitalocean.com/community/tutorial_collections/how-to-set-up-and-configure-a-certificate-authority-ca)
*   [How To Harden OpenSSH Client on Ubuntu 20.04](https://www.digitalocean.com/community/tutorials/how-to-harden-openssh-client-on-ubuntu-20-04)

You can transfer files across your system with these tutorials:

*   [How To Use SFTP to Securely Transfer Files with a Remote Server](https://www.digitalocean.com/community/tutorials/how-to-use-sftp-to-securely-transfer-files-with-a-remote-server)
*   [How To Use Rsync to Sync Local and Remote Directories](https://www.digitalocean.com/community/tutorials/how-to-use-rsync-to-sync-local-and-remote-directories#using-rsync-to-sync-with-a-remote-system)

If you’d like to implement security practices for your DigitalOcean Kubernetes cluster, try these tutorials next:

*   [Recommended Steps to Secure a DigitalOcean Kubernetes Cluster](https://www.digitalocean.com/community/tutorials/recommended-steps-to-secure-a-digitalocean-kubernetes-cluster)
*   [How To Set Up an Nginx Ingress on DigitalOcean Kubernetes Using Helm](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nginx-ingress-on-digitalocean-kubernetes-using-helm)
*   [How To Secure Your Site in Kubernetes with cert-manager, Traefik, and Let’s Encrypt](https://www.digitalocean.com/community/tutorials/how-to-secure-your-site-in-kubernetes-with-cert-manager-traefik-and-let-s-encrypt)

If you haven’t already, you can also [install an SSL certificate from a commercial certificate authority](https://www.digitalocean.com/community/tutorials/how-to-install-an-ssl-certificate-from-a-commercial-certificate-authority) for a domain associated with your server.

With your newfound knowledge of security, you are ready to continue your cloud journey. If you haven’t yet, check out our introductory articles on [cloud servers](https://www.digitalocean.com/community/tutorial_series/getting-started-with-cloud-computing#Cloud-Servers), [web servers](https://www.digitalocean.com/community/tutorials/web-servers-checkpoint), [databases](https://www.digitalocean.com/community/tutorial_series/getting-started-with-cloud-computing#Databases), and [containers](https://www.digitalocean.com/community/tutorial_series/getting-started-with-cloud-computing#Containers).