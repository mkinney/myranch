## My Ranch notes

These are my notes on experimenting with Rancher Kubernetes Engine (RKE). If you have the `Kubeconfig` setup and the `kubectl` command installed, you do not need to use the `rancher` prefix below. I have three computers for Rancher: top, bottom, and nuc1. (One server physically sits on top of the other. The third is an Intel NUC.) I don't expose this internal network to the outside world. These two computers are running the RancherOS on bare metal.

## Bare metal quick start

The following steps quickly deploy a Rancher Server on `bare metal` (aka a computer with no operating system).

## Prerequisites

- [rancheros.iso](https://github.com/rancher/os/releases): Download the latest rancheros.iso file and create bootable device
- Computer allows access to boot menu, has at least one hard drive, can boot to legacy devices (non-UEFI), and network has DHCP server
- Computer's first hard drive can be erased

## Create non-UEFI ("legacy") bootable iso (from mac)

1. Insert USB drive

2. See what drive it is on.

    `diskutil list`

3. Unmount the disk. (Note: My usb drive shows as disk2.)

    `diskutil unmountDisk disk2`

4. Copy image to drive. (Note: Only run this if you are absolutely sure you are on "disk2". You will lose any data on that drive.)

    `sudo dd if=rancheros.iso of=/dev/disk2 bs=8m`

5. If you get a "warning" from Mac about partition not being mountable (near the end of the dd command, click "ignore".)

6. Eject the USB drive.

    `diskutil eject disk2`


## Create UEFI bootable iso from Rancher OS iso (from mac)
The rancheros.iso (at the time of writing this), is not a UEFI bootable image. But, you can make it be one by following these steps.

1. Download the rancheros.iso and ubuntu-18.04.3-live-server-amd64.iso files.

2. Double click rancheros.iso file so it can be mounted.

3. Right click on the ubuntu iso with "The Archiver" (or some other file extractor).

4. Create new folder called "myranch". Copy all files from the mounted RancherOS drive to that folder.

5. Copy the ubuntu EFI folder to the myranch folder.

6. Copy the ubuntu /boot/grub folder to "myranch" /boot folder

7. Replace the contents of "myranch" `/boot/grub/grub.cfg` with:

    ```
    set timeout=5
    menuentry "Install Rancher" {
	    linux	/boot/vmlinuz-4.14.138-rancher rancher.autologin=tty1 rancher.autologin=ttyS0 rancher.autologin=ttyS1 console=tty1 console=ttyS0 console=ttyS1 printk.devkmsg=on panic=10 ---
	    initrd	/boot/initrd-v1.5.4
    }
    ```

Note: Rancher may be on different versions from when I created this document, so look in the Rancher `/boot/global.cfg` for the `APPEND` line options. Look in the `/boot` directory for the files to use for the `initrd` and `vmlinuz` files.

8. Insert USB drive.

9. Find out what disk it is mounted as.

    `diskutil list`

10. Unmount the disk. (Note: My usb drive shows as disk2.)

    `diskutil unmountDisk disk2`

11. Format it to FAT32.

    `sudo diskutil eraseDisk FAT32 DISK /dev/disk2`

12. Drag the contents from "myranch" folder to the "DISK" that is mounted.

13. Right click and eject the DISK usb drive.

14. Test by booting the NUC and seeing if the drive shows up/boots.

## Boot Rancher and install to hard drive (aka "Getting Started")

1. Boot PC using bootable media (may have to press F10 or F12 to get to the boot menu, then select the bootable device)

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

6. Install Rancher OS to the hard drive (if non-UEFI aka Legacy). Run `sudo ros install -i rancher/os:latest -c cloud-config.yml -d /dev/sda` (Note: This will wipe out any existing operating system on this computer.)

7. Reboot and remove the bootable media when installation is complete.

8. From another computer `ssh rancher@<ip from cloud-config.yml file>` (using the example above `ssh rancher@192.168.0.101`)

9. Start Rancher UI. Run `sudo docker run -d --restart=unless-stopped -p 9080:80 -p 9443:443 rancher/rancher` (Note: You can change the ports 9080 and/or 9443 if you want.)

10. Open up `https://<ip from cloud-config.yml file>:9443` (for example `https://192.168.0.101:9443`) and enter password for `admin` user.

11. Add a cluster. Enter `name`, click on "Next", then ensure all three options are checked ("worker", "etcd", "controlplane"). Copy the text.

12. Paste the text from prior step into the window from step 8 above. It will take a few minutes for all of the pieces to start.


## Want to add another worker node?

If you want to add another computer to the Rancher cluster, then follow these steps.

1. Follow the steps 1-8 from "Getting Started" above. (be sure to change the ip address and hostname values in the cloud-config.yml file)

2. On the Cluster you want to add this computer (aka Node), click on `Edit`. You may only want the `worker` option checked. Copy the text.

3. Run the command from step 2 into the ssh'd connection to the second computer.

4. The node should now be added to the cluster.

## Add another worker node on NUC (non UEFI)

1. Ensure USB is bootable on NUC. F2 for to change bios. Go to "Advanced" menu. Under "Devices and Peripherals" > "SATA" menu, change "Chipset SATA Mode" to "AHCI". On the "Boot" > "Boot Priority" menu, in the "Legacy Boot Priority" pane, Legacy Boot is disabled (unchecked). Ensure "USB" is checked for bootable device.

2. Follow steps 1-5 in "Getting Started" above (before the `sudo ros install` step) run these steps:

    ```
    sudo fdisk /dev/nvme0n1
    delete any existing partitions with “d” till all deleted
    new gpt "g"
    new partition "n"
    No "N"
    write "w"
    sudo mkfs.ext4 -F -i 4096 -O ^64bit -L RANCHER_STATE /dev/nvme0n1p1
    ```

3. Install Rancher to the NVME partition:

    `sudo ros install -t gptsyslinux -c cloud-config.yml -d /dev/nvme0n1 -p /dev/nvme0n1p1`

4. Follow rest of steps from adding another worker node.

## Add another worker node on NUC (UEFI)
I upgraded the NUC bios from 29 to 58, and "Legacy Boot" was removed. So, I needed to figure out how to boot UEFI. It does not appear that Rancher has that enabled.

1. Create the UEFI bootable usb stick.

2. Boot to Rancher.

3. Create two partitions: "EFI System" and "Linux filesystem".

    ```
    sudo fdisk /dev/nvme0n1
    delete any existing partitions with "d" till all deleted
    new gpt "g"
    new partition "n"
    "1"
    200M (200mb)
    remove signature "Y"
    "t" type 1 ( EFI System)
    new partition "n"
    "2"
    "enter"
    "p" (and compare with below)
    "w"

    Command (m for help): p
    Disk /dev/nvme0n1: 1.9 TiB, 2048408248320 bytes, 4000797360 sectors
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disklabel type: gpt
    Disk identifier: 56E160C1-9A30-F040-A2CD-B9DC3B0D16E4

    Device             Start        End    Sectors  Size Type
    /dev/nvme0n1p1      2048  209715200  209713153  100G EFI System
    /dev/nvme0n1p2 209717248 4000797326 3791080079  1.8T Linux filesystem

    sudo mkdosfs -n RANCHER  -F 32 /dev/nvme0n1p1
    mkdir /tmp/d
     mount -t vfat /dev/nvme0n1p1 /tmp/d

    tar/scp /boot and /EFI over to rancher instance
    sudo untar into /tmp/d

    sudo mkfs.ext4 -F -i 4096 -O ^64bit -L RANCHER_STATE /dev/nvme0n1p2

    # need this next line to fix error
    sudo mkdir /dev/sr0
    ```

4. Install Rancher to the GPT partition.

    `sudo ros install -t gptsyslinux -c cloud-config.yml -d /dev/nvme0n1 -p /dev/nvme0n1p2`

Note: Do not reboot when prompted at the end of the installation. If you do reboot before next step, run these commands to remount the partitions:


    mkdir /mnt/large
    mount -t ext4 /dev/nvme0n1p2 /mnt/large

    mkdir /tmp/d
    mount -t vfat /dev/nvme0n1p1 /tmp/d



5. Need to change the grub.cfg file `/tmp/d/boot/grub/grub.cfg`:

    ```
    set timeout=5

    menuentry "Rancher from GPT" {
            search --no-floppy --set=root --label RANCHER_STATE
        linux    /boot/vmlinuz-4.14.138-rancher printk.devkmsg=on rancher.state.dev=LABEL=RANCHER_STATE rancher.state.wait panic=10 console=tty0
        initrd    /boot/initrd-v1.5.4
    }

    menuentry "Install Rancher" {
        linux    /boot/vmlinuz-4.14.138-rancher rancher.autologin=tty1 rancher.autologin=ttyS0 rancher.autologin=ttyS1 console=tty1 console=ttyS0 console=ttyS1 printk.devkmsg=on panic=10 ---
        initrd    /boot/initrd-v1.5.4
    }
    ```

6. Test by booting.

7. Follow steps of "Adding another node" above.

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

# What else can you do with RKE?

- Spin up a Docker Private Registry. See [Readme](private_registry/readme.MD) in `private_registry` directory.
- Deploy Ethereum v1 Geth node. See [Readme](eth1/readme.MD) in `eth1` directory.
- Deploy Ethereum v2 prysm node. See [Readme](eth2/readme.MD) in `eth2` directory.
