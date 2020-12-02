Steps to install:
1. Create pi zero SD card image
2. Put wifi network details in config.txt and enable ssh. enable camera?
3. Boot pi
4. Wait for pi to appear on network
5. Install ffmpeg (Compile it?) - Can we compile locally for PI? https://snapcraft.io/install/ffmpeg/raspbian ?
6. Install NGINX, update config file. https://www.raspberrypi.org/documentation/remote-access/web-server/nginx.md
7. Install raspivid - package is libraspberrypi-bin
8. Install systemd services
9. Reload systemd
10. Copy webcam.sh file
11. Create /webcam, set owner to pi:pi
12. Copy index.html to /webcam
13. Start services.


1. Prepare your SD card


2. Connect to your pi

3. Install ansible and other tools locally
```
apt install ansible git snapd
```

4. Clone this repo
```
git clone [this repo]
```

5. Reboot the pi
This is needed for snapd?
https://snapcraft.io/install/ffmpeg/raspbian


6. Start the build
```
cd [repo name]
ansible-playbook -i localhost [thisplaybook.yml]


## Local testing

I've included a VagrantFile for testing the scripts out locally before applying them to a Raspberry Pi. This will help pick up syntax/script issues, the actual webcam will fail due to not having the raspivid executable.

# Windows

My testing environment is windows 10 pro with hyperv enabled, with chocolaty installed. I'll leave that as an exercise for the reader to install.

To get the testing environment up from that point: In an administrator powershell install vagrant with `choco install vagrant` and then `vagrant up` in this directory. If it asks for a switch then select the default switch. It will ask for a username and password to mount a SMB share, this is your windows username and password.

Vagrant will download the debian 10 image and run the ansible provisioner to test the playbook. Testing can also take place on an actual raspberry pi, but this can be cleaned up faster (I did say it was a faster development time but I've spent 3 hours making vagrant work correctly)

If you make changes to the playbook you can retest with `vagrant provision`

You can connect to the vm with `vagrant ssh` and explore what ansible created. When you're finished `vagrant destroy`.