
Kali Desktop provides Docker images with Kali Linux and a VNC server. This project allows you to pick Kali Linux version, favorite desktop environment, and run it on any system - Linux, MacOS or Windows - to access remotely and execute commands using a VNC client or a web browser.



Kali Linux 2018.2
Xfce - :xfce
LXDE - :lxde
KDE - :kde
Kali Linux 2018.2 with Top10 tools pre-installed
Xfce - :xfce-top10
Running
All required services and dependencies are inside the Docker images so only web browser and one command are needed to start kali-desktop:



However the most common case is kali-desktop running with host network in privileged mode, so tools like network sniffing work properly and with full speed without Docker network filtering the traffic. See all available Docker image tags on Docker Hub.

# run on host network
```
docker run -d --network host --privileged lukaszlach/kali-desktop:xfce
```

# run on Docker network
```
docker run -d -p 5900:5900 -p 6080:6080 --privileged lukaszlach/kali-desktop:xfce
```

After the container is up you can access Kali Linux Desktop under http://localhost:6080, the hostname can differ if you are doing this on a remote server. vnc_auto.html will connect you automatically, vnc.html allows some connection tuning.

Docker for Mac works inside a small virtual machine which IP you must use to access the exposed ports or use service like Dinghy.

If you want to customize the container behavior you can pass additional parameters:

```
docker run -d \
    --network host --privileged \
    -e RESOLUTION=1280x600x24 \
    -e USER=kali \
    -e PASSWORD=kali \
    -e ROOT_PASSWORD=root \
    -v /home/kali:/home/kali \
    --name kali-desktop \
    lukaszlach/kali-desktop:xfce
```

Run parameters:

--network host - optional but recommended, use the host network interfaces, if you do not need to use this option you have to manually publish the ports by passing -p 5900:5900 -p 6080:6080
--privileged - optional but recommended
-e RESOLUTION - optional, set streaming resolution and color depth, default 1280x600x24
-e USER - optional, work as a user with provided name, default root
-e PASSWORD - optional, provide a password for USER, default kali
-e ROOT_PASSWORD - optional, provide password for root, default root
-v /home/kali:/home/kali - optional, if USER was provided it is a good idea to persist user settings, work files and look-and-feel
Exposed ports:

5900/tcp - VNC
6080/tcp - noVNC, web browser VNC client
Extending
Create Dockerfile.xfce-web and modify the image as desired, below example installs Kali Linux web application assessment tools:

```
FROM lukaszlach/kali-desktop:xfce

RUN apt-get update && \
    apt-get install -y kali-linux-web \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

Build the image:

```
docker build \
    -f Dockerfile.xfce-web \
    -t kali-desktop:xfce-web \
    .
```

Run the image:

```
docker run --network host --privileged kali-desktop:xfce-web
```
