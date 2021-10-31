# fmp4streamer
[What is fmp4streamer](#what-is-fmp4streamer) |
[How does it work](#how-does-it-work) |
[Capabilities](#capabilities) |
[Installation](#installation) |
[Running](#running) |
[Viewing](#viewing) |
[Configuration](#configuration) |
[Roadmap](#roadmap) |
[Motivation](#motivation) |
[Changelog](https://github.com/soyersoyer/fmp4streamer/blob/main/CHANGELOG.md)

# What is fmp4streamer
The fmp4streamer is a Python application designed to stream hardware encoded H.264 from a V4L2 Linux video device directly to a browser.

# How does it work
This python script setups the V4L2 device, reads the h264 stream from it, adds some fMP4 (fragmented MP4) header and serves it via HTTP or HTTPS. It works with only one html5 video tag, no js needed. It's pretty lightweight.

# Capabilities
- Stream to multiple clients simultaneously (usually only limited by your network connection) 
- Support any resolution and framerate the video device can capture
- Works in any Raspberry Pi with a camera module
- Able to handle high framerate (60-90 fps) streams
- Able to stream to iPhone and Safari via HLS.
- Very low cpu utilization, even when streaming using SSL

# Installation
   ```
   wget https://github.com/soyersoyer/fmp4streamer/archive/refs/tags/v3.0.3.zip
   unzip v3.0.3.zip
   mv fmp4streamer-3.0.3 fmp4streamer
   ```
# Installation as a system service
   ```
   git clone http://www.github.com/mclendon99/fmp4streamer.git
   ```

- Set up the config file
   ```
   cd fmp4streamer
   cp fmp4streamer.conf.dist fmp4streamer.conf
   ```
- Edit the fmp4streamer.conf file to point to your certificate/key if using HTTPS. (See below).
- Make any other changes as needed, e.g. port number or frame-rate...

- Install the config file
   ```
   sudo mkdir /etc/fmp4streamer
   sudo cp fmp4streamer.conf /etc/fmp4streamer
   sudo cp fmp4streamer.py /etc/fmp4streamer
   sudo cp bmff.py /etc/fmp4streamer
   sudo cp v4l2camera.py /etc/fmp4streamer
   sudo cp v4l2.py /etc/fmp4streamer
   ```
- Install the system service
   ```
   sudo cp fmp4streamer.system-service /lib/systemd/system/fmp4streamer.service
   sudo chmod 0644 /lib/systemd/system/fmp4streamer.service
   ```

   To use https, one must obtain a certificate from a registered domain.
   GoDaddy has cheap domains and LetsEncrypt will issue a certificate
   manually after installation of snap and letsencrypt, e.g. the following will manually issue a certificate:
   ```
   certbot certonly --authenticator dns-godaddy --dns-godaddy-propogation-seconds 600 --keep-until-expiring  --expand --server https://acme-v02.api.letencrypt.org/directory -d '.your-registered.domain' -d '*.your-registered.domain'
   ```
   and then put the challenge in the domain record per the instructions 
   and your keys will appear in /etc/letsencrypt/live/your-registered.domain/

   Point to the fullchain.pem certificate and the privkey.pem in the fmp4streamer.conf file.

# Running 
- from the terminal
    ```
    cd fmp4streamer
    python3 fmp4streamer.py
    ```
- At startup. Now installed as a system service.
    ```
    cd fmp4streamer
    mkdir -p ~/.config/systemd/user
    cp fmp4streamer.service ~/.config/systemd/user/
    systemctl --user enable fmp4streamer
    systemctl --user start fmp4streamer
    loginctl enable-linger pi
    ```

    watch the logs
    ```
    systemctl --user status fmp4streamer
    journalctl --user-unit fmp4streamer
    ```

# Running as a system service (one time only)
    ```
    sudo systemctl daemon-reload
    sudo systemctl enable fmp4streamer.service
    sudo systemctl start fmp4streamer.service
    ```

    Watch the logs
    ```
    dmesg
    systemctl status fmp4streamer
    journalctl --system-unit fmp4streamer
    ```



# Viewing
When fmp4streamer.py is running the stream can be viewed from any browser via the following url. **_ip_address_** is the ip address or hostname of your computer, and **_port_** (default: 8000) is the port you set in the configuration section.
```
http://<ip_address>:<port>/
```
  
If you want to view the stream via your favourite media player, you can use the
```
http://<ip_address>:<port>/stream.mp4
```
url.

# Configuration

You can start with the fmp4streamer.conf.dist:

```
cp fmp4streamer.conf.dist fmp4streamer.conf
```

Which contains:
```ini
[server]
listen = 
port = 8000

[/dev/video0]
width = 800
height = 600
fps = 30
# you can set any V4L2 control too, list them with the -l option
h264_profile = High
h264_level = 4.2
h264_i_frame_period = 15
```

You can set all the V4L2 controls via the configuration file. List them with
```
python3 fmp4streamer.py -l
```

# Roadmap
- [x] Use V4L2 instead of raspivid
- [ ] Adding AAC audio to the stream from ALSA
- [ ] Support multiple cameras
- [ ] Rewrite to c or Rust


# Motivation
I wanted to stream my raspberry camera to the browser. I found some solution, but I think they are not optimal or too heavy.

## MJPEG streamers
We have h264, why do we want to stream mjpeg?

## VLC, Gstreamer, FFmpeg
```
apt install vlc
...
0 upgraded, 294 newly installed, 0 to remove and 0 not upgraded.
Need to get 107 MB of archives.
After this operation, 900 MB of additional disk space will be used.
```

```
apt install libgstreamer-plugins-base1.0 gstreamer1.0-tools
...
0 upgraded, 119 newly installed, 0 to remove and 0 not upgraded.
Need to get 51.5 MB of archives.
After this operation, 633 MB of additional disk space will be used.
```

```
apt install ffmpeg
...
0 upgraded, 149 newly installed, 0 to remove and 0 not upgraded.
Need to get 69.3 MB of archives.
After this operation, 714 MB of additional disk space will be used.
```


These are huge. It's bad because lot of binary contains a lot of security bugs and you have to update these packages too in your project lifetime, not just install them in one time.

## [U4VL](https://www.linux-projects.org/uv4l/)
You have to add a custom apt repository to your pi to install, you have to trust these guys. It has closed source components too. So no way.

## [131/h264-live-player](https://github.com/131/h264-live-player)
It uses a baseline javascript h264 decoder. We have better h264 decoders in our hardwares, so why do we want to use a javascript decoder?

## [dans98/pi-h264-to-browser](https://github.com/dans98/pi-h264-to-browser)
I like it, it's good, but I don't like its dependencies
```
apt install python3-picamera
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following additional packages will be installed:
  libblas3 libgfortran5 liblapack3 python3-numpy
Suggested packages:
  gfortran python-numpy-doc python3-dev python3-pytest python3-numpy-dbg python-picamera-docs
The following NEW packages will be installed:
  libblas3 libgfortran5 liblapack3 python3-numpy python3-picamera
0 upgraded, 5 newly installed, 0 to remove and 0 not upgraded.
Need to get 3,830 kB of archives.
After this operation, 18.1 MB of additional disk space will be used.
```
Do we really need fortran to use a camera in python? Do we look like dinosaurs? NO! And 18.1 MB for some init and some read?

![Fortran dinousaurs](https://i.pinimg.com/564x/7c/3a/c8/7c3ac86430655a9f22b2b2496bb4d1ee.jpg)

```
apt install python3-tornado
...
0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
Need to get 354 kB of archives.
After this operation, 1,800 kB of additional disk space will be used.
```
It's for the websocket server, not too big but do we really need websockets? No, simple http is enough for streaming to browsers.

[samirkumardas/jmuxer](https://github.com/samirkumardas/jmuxer)

Why do we want to add mp4 headers to h264 with js in the browser? If we want to do it properly we have to send timing information next to h264 too, but what should be the format, custom encoding or protobuf? Or wait, there is already a format for that, I remember it, the mp4! So it should be done on the server side.

[Media Source Extensions](https://caniuse.com/mediasource)

It could be good, but safari on iOS doesn't support it... and why do we need javascript code to play a video?

## [thinkski/berrymse](https://github.com/thinkski/berrymse)

It inspired me, I like it very much, it's simple and minimal, but it uses websockets and the Media Source Extensions...

## fmp4streamer (formerly RaspiWebCam)

So here we are.

It doesn't have any dependencies other than python and V4L2, but they are installed by default.

It adds fMP4 (fragmented MP4) headers to the h264 stream. It uses the python http server to serve it to the browser.

The browsers play it with only one html5 video tag (the most of). No javascript needed.

Safari on iOS plays it only with HLS playlists, but it works too. And the playlists added to the index.html of course.
