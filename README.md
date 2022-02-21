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

to
SELINUX=disabled

and reboot system.

To check if SELinux is deiabled, run:

sestatus

You need to set some values in file vars_files/var_file_strings.yml

Most of variables are connecnted with Easy RSA settings

Variables to create VPN server:
 * vpn_server_name - name of the vpn server
 * vpn_server_ip - IP address of server
 * vpn_server_port - port for VPN
 * vpn_server_proto - vpn protocol
 * vpn_server_dev - vpn server device type (tap or tun)
 * vpn_server_dev_peer - name of vpn server device (example: tun1)
 * vpn_server_dev_out - interface that is connected to network (example: vpn_server_dev_out: "eth1")
 * vpn_server_network - network of vpn server in format: X.X.X.X X.X.X.X (example: vpn_server_network: "10.8.0.0 255.255.255.0")
 * vpn_server_network_different_mask - network of vpn server to add to iptables in format X.X.X.X\X (example: vpn_server_network_different_mask: "10.8.0.0\8")
 * vpn_server_network_peer - network peer, it needs to be in the same network with vpn_server_network, it should be in format: X.X.X.X X.X.X.X (example: vpn_server_network_peer: "10.8.0.1 10.8.0.2")
 * vpn_server_dns_1 & vpn_server_dns_2 - DNS servers that are used in VPN server, format: X.X.X.X (example: vpn_server_dns_1: "8.8.8.8")
 * vpn_server_keepalive - setting keepalive parameter; ping response (example: vpn_server_keepalive: "10 120")
 * vpn_server_cipher - setting cryptographic cypher (example: vpn_server_cipher: "AES-256-CBC")
 * vpn_server_user & vpn_server_group - parameter which reduces ovpn deamon's privileges (example: vpn_server_user: "nobody", vpn_server_group: "nogroup")
 * vpn_server_status_path, vpn_server_log_path & vpn_server_log_append_path - setting path to vpn logs (example: vpn_server_status_path: "/var/log/openvpn/openvpn-status.log", vpn_server_log_path: "/var/log/openvpn/openvpn.log", vpn_server_log_append_path: "/var/log/openvpn/openvpn.log")
 * vpn_server_verb - set level of logs; 0 - no logs exept for fatal errors, 9 - extremely verbose (example: vpn_server_verb: "3")
 * vpn_server_exp_exit_not - inform vpn client that server restarts, it will connect with server automatically (example: vpn_server_exp_exit_not: "1")
 * vpn_server_image - link to vpn podman image (example: vpn_server_image: "docker.io/kylemanna/openvpn")
 
 Variables to creat OVPN client: 
  * ovpn_clients_vars - that items are needed to set vpn clients
      - vpn_client_res_ret - enable to resolve name of vpn server (example: vpn_client_res_ret: 'infinite')
	  - vpn_client_bind - set if client needs to contact to his specyfied port (example: vpn_client_bind: 'nobind')

	  example: 
	  
	  ovpn_clients_vars:
	  - { ovpn_client_name: '<vpn_client_name>', ovpn_client_pass: '<vpn_client_pass_plain_text>', common_name: '<vpn_client_name_short>', vpn_client_res_ret: '<resolv-retry_type>', vpn_client_bind: '<bind_type>' }

Variable to revoke VPN user:	  
	* ovpn_client_to_remove_name - set vpn client that you would like to revoke, needs to be set only when you want to disable user's access to vpn server
	  
	   
Creating VPN server on device:

ansible-playbook install-openvpn.yml

Creating VPN client or clients.

It'll not work without created server and set variables for creating server.
 
ansible-playbook create-openvpn-clients.yml

Deleting VPN client.

It'll not work without created VPN client to remove.

ansible-playbook delete-openvpn-client.yml

Troubleshooting:

If there is a problem with start container after reboot, the following parameters should be set:

Iptables should be run in modeporobe:

modprobe ip_tables

In file /etc/modules shold be set parametr 'ip_tables'

echo 'ip_tables' >> /etc/modules
