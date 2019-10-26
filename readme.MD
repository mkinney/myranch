## My Ranch notes

These are my notes on experimenting with Rancher Kubernetes Engine (RKE). If you have the `Kubeconfig` setup and the `kubectl` command installed, you do not need to use the `rancher` prefix below. I have two computers for Rancher: top and bottom. (One server physically sits on top of the other.) I don't expose this internal network to the outside world. These two computers are running the RancherOS on bare metal.

## Bare metal quick start

The following steps quickly deploy a Rancher Server on `bare metal` (aka a computer with no operating system).

## Prerequisites

- [rancheros.iso](https://github.com/rancher/os/releases): Download the latest rancheros.iso file and create bootable device
- Computer allows access to boot menu, has at least one hard drive, and network has DHCP server
- Computer's first hard drive can be erased


## Getting Started

1. Boot PC using bootable media (may have to press F12 to get to the boot menu, then select the bootable device)

2. Once you get to a `rancher` prompt, change the rancher password: `sudo passwd rancher` (You will need this password when you ssh to this instance.)

3. Run `ifconfig -a` to get the ip address of this instance.

4. From another computer `ssh rancher@<ip from previous step>` (for example, `ssh rancher@192.168.0.11`)

5. Create a `cloud-config.yml` file. Here is an example file (with the ssh-rsa line truncated for brevity):

	```
	hostname: apple.example.com

	rancher:
	  network:
	    interfaces:
        eth*:
          dhcp: false
        eth0:
          address: 192.168.0.101/24
          gateway: 192.168.0.1
      dns:
        nameservers:
          - 192.168.0.171
          - 8.8.8.8

	ssh_authorized_keys:
          - ssh-rsa AAAAB3NzaC...jJw== XXX
        ```   

6. Install Rancher OS to the hard drive. Run `sudo ros install -i rancher/os:latest -c cloud-config.yml -d /dev/sda` (Note: This will wipe out any existing operating system on this computer.)

7. Reboot and remove the bootable media when installation is complete.

8. From another computer `ssh rancher@<ip from cloud-config.yml file>` (using the example above `ssh rancher@192.168.0.101`)

9. Start Rancher UI. Run `sudo docker run -d --restart=unless-stopped -p 9080:80 -p 9443:443 rancher/rancher` (Note: You can change the ports 9080 and/or 9443 if you want.)

10. Open up `https://<ip from cloud-config.yml file>:9443` (for example `https://192.168.0.101:9443`) and enter password for `admin` user.

11. Add a cluster. Enter `name`, click on "Next", then ensure all three options are checked ("worker", "etcd", "controlplane"). Copy the text.

12. Paste the text from prior step into the window from step 8 above. It will take a few minutes for all of the pieces to start.


## Want to add another worker node?

If you want to add another computer to the Rancher cluster, then follow these steps.

1. Follow the steps 1-8 from above. (be sure to change the ip address and hostname values in the cloud-config.yml file)

2. On the Cluster you want to add this computer (aka Node), click on `Edit`. You may only want the `worker` option checked. Copy the text.

3. Run the command from step 2 into the ssh'd connection to the second computer.

4. The node should now be added to the cluster.


## Rancher CLI Notes

- For CLI, I initially used https://rancher.com/docs/rancher/v2.x/en/cli/ as a guide. I tried downloading rancher cli from https://github.com/rancher/cli/releases but it did not work well, ended up cloning cli repo and running "CROSS=1 make build" and copying the build/bin/*darwin* to ~/bin/rancher(ensure ~/bin is in your path)

- Download the self signed certificate from Rancher UI. (I used Brave Browser, click on area to left of url called "View Site Information", click on "Certificate", drag the picture of the certificate to the Desktop, double click the file that it downloads, which should open up that certificate file in "Keychain Access", select the 'always allow' for all, close Brave (maybe need to log out/re-login?), and re-open the url. The cert should no longer be "red".

- Login to Rancher UI and create api key by going to the top right `API & Keys` for the admin user, then `Add Key`. (Note: Be sure to copy all values - will not see them again.)

- Login using `rancher` cli: (note: I removed some characters in the token below)

    rancher login <URL> --token <bearer token>

    $ rancher login https://top.mikekinney.org:9443/v3 --token token-XXX:5cwXXXpr68
    INFO[0000] Saving config to /Users/mikekinney/.rancher/cli2.json

- Manually deploy `hello` workload. Used these values:
  - Name: hello
  - Docker image: rancher/hello-world
  - port: 8000

- Use the three dots to show yaml (copied/then removed the temp parts) and created hello.yml.

- Manually delete `hello` workload

- Deploy simple `hello` workload (via cli)

    `rancher kubectl apply -f hello.yml`

- Undeploy simple `hello` workload (via cli)

    `rancher kubectl delete -f hello.yml`

- Deploy same `hello` workload but with persisted volume

    `rancher kubectl delete -f hello2/hello2_d.yml -f hello2/hello2_pvc.yml -f hello2/hello2_pv.yml`

- Deploy eth1 or eth2 nodes
