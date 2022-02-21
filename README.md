# Ansible-podman-vpn-server

This project create VPN server, create clients and delete them from VPN server.

VPN server is installed as podman container.

## Before start:

You need to have device with CentOS8 Stream.

Disable SELinux

Change line in <code>/etc/selinux/config</code>

From

```bash
SELINUX=permissive
```

to

```bash
SELINUX=disabled
```
and <code>reboot</code> system.

To check if SELinux is deiabled, run:

```bash
sestatus
```
You need to set some values in file <code>vars_files/var_file_strings.yml</code>

Most of variables are connecnted with Easy RSA settings

Variables to create VPN server:
 * <code>vpn_server_name</code> - name of the vpn server
 * <code>vpn_server_ip</code> - IP address of server
 * <code>vpn_server_port</code> - port for VPN
 * <code>vpn_server_proto</code> - vpn protocol
 * <code>vpn_server_dev</code> - vpn server device type (tap or tun)
 * <code>vpn_server_dev_peer</code> - name of vpn server device (example: tun1)
 * <code>vpn_server_dev_out</code> - interface that is connected to network (example: <code>vpn_server_dev_out: "eth1"</code>)
 * <code>vpn_server_network</code> - network of vpn server in format: X.X.X.X X.X.X.X (example: <code>vpn_server_network: "10.8.0.0 255.255.255.0"</code>)
 * <code>vpn_server_network_different_mask</code> - network of vpn server to add to iptables in format X.X.X.X\X (example: <code>vpn_server_network_different_mask: "10.8.0.0\8"</code>)
 * <code>vpn_server_network_peer</code> - network peer, it needs to be in the same network with vpn_server_network, it should be in format: X.X.X.X X.X.X.X (example: <code>vpn_server_network_peer: "10.8.0.1 10.8.0.2"</code>)
 * <code>vpn_server_dns_1 & vpn_server_dns_2</code> - DNS servers that are used in VPN server, format: X.X.X.X (example: <code>vpn_server_dns_1: "8.8.8.8"</code>)
 * <code>vpn_server_keepalive</code> - setting keepalive parameter; ping response (example: <code>vpn_server_keepalive: "10 120"</code>)
 * <code>vpn_server_cipher</code> - setting cryptographic cypher (example: <code>vpn_server_cipher: "AES-256-CBC"</code>)
 * <code>vpn_server_user & vpn_server_group</code> - parameter which reduces ovpn deamon's privileges (example: <code>vpn_server_user: "nobody"</code>, <code>vpn_server_group: "nogroup"</code>)
 * <code>vpn_server_status_path, vpn_server_log_path & vpn_server_log_append_path</code> - setting path to vpn logs (example: <code>vpn_server_status_path: "/var/log/openvpn/openvpn-status.log"</code>, <code>vpn_server_log_path: "/var/log/openvpn/openvpn.log"</code>, <code>vpn_server_log_append_path: "/var/log/openvpn/openvpn.log"</code>)
 * <code>vpn_server_verb</code> - set level of logs; 0 - no logs exept for fatal errors, 9 - extremely verbose (example: <code>vpn_server_verb: "3"</code>)
 * <code>vpn_server_exp_exit_not</code> - inform vpn client that server restarts, it will connect with server automatically (example: <code>vpn_server_exp_exit_not: "1"</code>)
 * <code>vpn_server_image</code> - link to vpn podman image (example: <code>vpn_server_image: "docker.io/kylemanna/openvpn"</code>)
 
 Variables to creat OVPN client: 
  * ovpn_clients_vars - that items are needed to set vpn clients
      - vpn_client_res_ret - enable to resolve name of vpn server (example: vpn_client_res_ret: 'infinite')
      - vpn_client_bind - set if client needs to contact to his specyfied port (example: vpn_client_bind: 'nobind')

example: 

```bash
ovpn_clients_vars:
  - { ovpn_client_name: '<vpn_client_name>', ovpn_client_pass: '<vpn_client_pass_plain_text>', common_name: '<vpn_client_name_short>', vpn_client_res_ret: '<resolv-retry_type>', vpn_client_bind: '<bind_type>' }
```

Variable to revoke VPN user:	  
	* <code>ovpn_client_to_remove_name</code> - set vpn client that you would like to revoke, needs to be set only when you want to disable user's access to vpn server
	  
	   
## Creating VPN server on device:

<code>ansible-playbook install-openvpn.yml</code>

## Creating VPN client or clients.

It'll not work without created server and set variables for creating server.
 
<code>ansible-playbook create-openvpn-clients.yml</code>

Deleting VPN client.

It'll not work without created VPN client to remove.

<code>ansible-playbook delete-openvpn-client.yml</code>

Troubleshooting:

If there is a problem with start container after reboot, the following parameters should be set:

Iptables should be run in modeporobe:

<code>modprobe ip_tables</code>

In file <code>/etc/modules</code> shold be set parametr <code>'ip_tables'</code>

```bash
echo 'ip_tables' >> /etc/modules
```
