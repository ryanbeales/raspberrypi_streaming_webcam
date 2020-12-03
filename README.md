# What is this?
This is an ansible script (with a vagrant test) for installing a quick raspberry pi live streaming webcam. You should end up with a raspberry pi running nginx that you can connect to and get something like this:

<script src="https://cdn.jsdelivr.net/npm/hls.js@latest">
</script>
  <video autoplay="" controls="" height="50%" id="video" width="50%"></video>
  <script>
    if (Hls.isSupported()) {
      var video = document.getElementById('video');
      var hls = new Hls();
      // bind them together
      hls.attachMedia(video);
      hls.on(Hls.Events.MEDIA_ATTACHED, function () {
        console.log("video and hls.js are now bound together !");
        hls.loadSource("https://s3.eu-west-2.amazonaws.com/www.ryanbeales.com/stream/window.m3u8");
        hls.on(Hls.Events.MANIFEST_PARSED, function (event, data) {
          console.log("manifest loaded, found " + data.levels.length + " quality level");
        });
        //video.play();
      });
    }
</script>

This has been tested on the latest version of ansible, raspberry pi zero, and raspberry pi OS as of December 2020.

If you know what you're doing, pick and choose from below. Otherwise this will take you through setting up ansible on your pi and running the install script for the webcam. This is based on my blog posts [here](https://blog.ryanbeales.com/2019/03/smooth-streaming-video-from-raspberry.html) with some improvements in how things run (no more RTSP and compiling nginx, but still compiling ffmpeg, just wait for ansible to install)

# Installing

This assumes you know nothing, have a bunch of hardware around and just want to make a streaming webcam. If you have ansible already installed, I assume you can pick and choose from the below steps to get the playbook running.

1. Follow the first two steps, then the first 4 from 'Connect the Camera Module' [here](https://projects.raspberrypi.org/en/projects/getting-started-with-picamera)

1. Prepare your SD card
    1. Install 'Raspberry Pi OS' on to an SD card, you can get the installer [here](https://www.raspberrypi.org/software/)
    1. Use notepad/notepad++/atom/vscode/whatever to place a file named 'ssh' on the SD card (it can be empty)
    1. Use notepad/notepad++/atom/vscode/whatever to place a file named `wpa_supplicant.conf` on the SD card with this contents (update to your values), more documentation is [here](https://www.raspberrypi.org/documentation/configuration/wireless/headless.md):
        ```
        country=<2 letter country code>
        update_config=1
        ctrl_interface=/var/run/wpa_supplicant

        network={
            scan_ssid=1
            ssid="Network name goes here"
            psk="password goes here"
        }
        ```

1. Wait for the pi to boot (my pi zero took roughly ~2 minutes to appear on the network, and ~5 minutes to have SSH running)

1. Connect to your pi via ssh with the [default password](https://www.raspberrypi.org/documentation/linux/usage/users.md), check your router to see which IP address it's given itself (raspberrypi.local might work in a pinch)

1. Install ansible and other tools locally, this will take a very long time, _many_ minutes, around 30. Enough time for me to write this entire readme.
    ```
    sudo python -m pip install ansible --no-cache-dir
    ```

1. Clone this repo
    ```
    git clone https://github.com/ryanbeales/raspberrypi_streaming_webcam.git
    ```

1. Start the build, this will again take some time. Get a coffee, or a full meal.
    ```
    cd raspberrypi_streaming_webcam
    ansible-playbook piwebcam.yml
    ```

1. Reboot the pi with `reboot` or power cycle to enable the camera.

1. After the pi is back you'll be able to view your webcam stream at http://[raspberry pi ip address here]/ or possibly http://raspberrypi.local

# Local testing

I've included a VagrantFile for testing the scripts out locally before applying them to a Raspberry Pi. This will help pick up syntax/script issues, the actual webcam will fail due to not having the raspivid executable. I only include some information about testing on Windows for now since that is all I have (I play games on this laptop too).

## Windows

My testing environment is windows 10 pro with hyperv enabled and chocolaty installed. I'll leave that as an exercise for the reader to get to that point.

To get the testing environment up: In an administrator powershell install vagrant with `choco install vagrant` and then `vagrant up` in this directory. If it asks for a switch then select the default switch. It will ask for a username and password to mount a SMB share, this is your windows username and password, this is _important_, I thought it was a key error from vagrant at first because I didn't read it properly. If you're reading this you are better at reading than I am.

Vagrant will download the debian 10 image and run the ansible provisioner to test the playbook. Testing can also take place on an actual raspberry pi, but this can be cleaned up faster (at first I did say it was a faster cycle time to develop with vagrant but I've spent 3 hours making vagrant work correctly).

If you make changes to the playbook you can retest with `vagrant provision`.

You can connect to the vm with `vagrant ssh` and explore what ansible created. When you're finished `vagrant destroy`.