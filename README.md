# docker-reverse-ssh-tunnel
This container setups up a reserve ssh tunnel between a NATed host and a public host. It allows the application server in the NATed host can be accessed through the public host, which helps the application poke a hole through its firewall.

Create Image
------------
To create the image `cangmingh/reverse-ssh-tunnel`, execute the following command on the tutum-docker-couchdb folder:
```
	docker build -t cangmingh/reverse-ssh-tunnel .
```	
Usage
-----
On public host, run:
```
  docker run -d \
    -e ROOT_PASS=<your_password> \
    -p <your_sshd_port>:22 \
    -p <forwarding_port>:1080 \
    cangmingh/reverse-ssh-tunnel
```
Parameters:
```
  <your_password> is the password used for NATed host to connect the public host
  <your_sshd port> is the port for NATed host to connect to
  <forwarding port> is the port allows others to access
```

On NATed Host, run:

```
  docker run -d \
    -e PUBLIC_HOST_ADDR=<public_host_address> \
    -e PUBLIC_HOST_PORT=<public_host_port> \ 
    -e ROOT_PASS=<your_password> \
    -e PROXY_PORT=<NATed_service_port> \
    --net=host \
    cangmingh/reverse-ssh-tunnel
```
Parameters:
```
  <public_host_address> is the ip address or domain of your public host
  <public_host_port> is the same as <your sshd port> set on the public host
  <your_passorwd> is the same as <your passorwd> set on the public host
  <NATed_service_port> is the port that your service in NATed host listens to
```

Then, a user can access <public_host_address>:<forwording_port> to communicate the service running in the NATed host listened to port <PROXY_PORT>

If you want to login with key, follow the steps.
1. Use ssh-keygen to generate key pair for NATed Host
2. Mount generated id_rsa and id_rsa.pub into /root/.ssh on NATed Host
3. On public host, fill the content of id_rsa.pub generated by first step into a file and mount it to /root/.ssh/authorized_keys

On public host, run:
```
  docker run -d \
    -e ROOT_PASS=<your_password> \
    -e AUTHORIZED_KEYS=<NATed Host id_rsa.pub content> \
    -p <your_sshd_port>:22 \
    -p <forwarding_port>:1080 \
    -v <authorized_keys file path>:/root/.ssh/authorized_keys
    cangmingh/reverse-ssh-tunnel
```

On NATed Host with password login, run: (No ROOT_PASS needed)

```
  docker run -d \
    -e PUBLIC_HOST_ADDR=<public_host_address> \
    -e PUBLIC_HOST_PORT=<public_host_port> \
    -e PROXY_PORT=<NATed_service_port> \
    -v <id_rsa_path>:/root/.ssh/id_rsa \
    -v <id_rsa_pub_path>:/root/.ssh/id_rsa.pub \
    --net=host \
    cangmingh/reverse-ssh-tunnel
```

Example
-------

Suppose you have a service running in the NATed host, listens to port `8080`, and there is also another public host with the ip address of `111.112.113.114`. You want users to access `111.112.113.114:80` to communicate with your NATed service, you could the do the following:

On public host(`111.112.113.114`):
```
  docker run -d -e ROOT_PASS=mypass -p 2222:22 -p 80:1080 cangmingh/reverse-ssh-tunnel
```
On NATed host:
```
  docker run -d -e PUBLIC_HOST_ADDR=111.112.113.114 -e PUBLIC_HOST_PORT=2222 -e ROOT_PASS=mypass -e PROXY_PORT=8080 --net=host cangmingh/reverse-ssh-tunnel
```

Then, `curl 111.112.113.114:80` will work
