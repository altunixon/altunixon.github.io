# Create a Docker Container for Raspberry Pi to Blink an LED [source article]

## 1. Install Docker on Raspberry Pi:
- Update packages: `sudo apt-get update -y && sudo apt-get upgrade -y`
- Install docker: `curl -sSL https://get.docker.com | sh`
- Add current user to the docker group: `sudo usermod -aG docker $USER`
- Verify: `docker --version`
- Potential errors messages you may see:
  1. If you see the following error, reboot your Raspberry Pi and try again –
  ```
  docker: Error response from daemon: failed to create endpoint compassionate_joliot on network bridge: failed to add the host (veth0685c4b) <=> sandbox (vethccfe136) pair interfaces: operation not supported.
  ERRO[0091] error getting events from daemon: net/http: request canceled
  ```
  2. If you see the following error, it means you are trying to run an X86 container on ARM machine i.e. Raspberry Pi.</br>
  This is because the binary format used by ARM is not compatible with x86.
  ```
  standard_init_linux.go:178: exec user process caused "exec format error"
  ```
  
## 2. Select the Base Image for Dockerfile:
Once you are done with the Docker installation, you need to create a Dockerfile for the new Docker Image that can blink the LED connected to your Raspberry Pi. </br>
Docker can build images automatically by reading the instructions from a Dockerfile.</br>
Most of the times we start Dockerfile with a base image (sometimes called as Parent Image). </br>
You can find most of the Docker Base Images on the official Docker Hub. </br>
You need to be careful while selecting the base image for your new Docker Image, as most these bases images are created for specific CPU Architectures. </br>
You can find the list of different CPU Architectures and their respective Docker Hub URLs from the following link - [architectures-other-than-amd64]</br>
You can use the `cat /proc/cpuinfo | grep model` command on your Raspberry Pi to find the CPU Architecture and use the corresponding Docker Hub repository
```
$: cat /proc/cpuinfo | grep model
model name      : ARMv7 Processor rev 3 (v7l)
model name      : ARMv7 Processor rev 3 (v7l)
model name      : ARMv7 Processor rev 3 (v7l)
model name      : ARMv7 Processor rev 3 (v7l)
```
Now, if you are using Rasbian, you may have "buster" or "wheezy" variants. You can run the `cat /etc/os-release` command to see the OS details
```
$: cat /etc/os-release
PRETTY_NAME="Raspbian GNU/Linux 10 (buster)"
NAME="Raspbian GNU/Linux"
VERSION_ID="10"
VERSION="10 (buster)"
VERSION_CODENAME=buster
ID=raspbian
ID_LIKE=debian
HOME_URL="http://www.raspbian.org/"
SUPPORT_URL="http://www.raspbian.org/RaspbianForums"
BUG_REPORT_URL="http://www.raspbian.org/RaspbianBugs"
```
Based on this command outputs from my Raspberry Pi which shows the CPU architecture asARMv7 and the Rasbian OS is Buster, 
I am going to use "arm32v7/python" base image with tag "3.6-slim-buster" which is available at the following link [arm32v7/python]

## 3. Create a New Directory on Raspberry Pi:
`mkdir docker_image && cd docker_image`

## 4. Create a Python Script to Blink an LED:
```python
#!/usr/bin/env python
import RPi.GPIO as GPIO
import time

# Configure the PIN # 8
GPIO.setmode(GPIO.BOARD)
GPIO.setup(8, GPIO.OUT)
GPIO.setwarnings(False)

# Blink Interval 
blink_interval = .5 #Time interval in Seconds

# Blinker Loop
while True:
 GPIO.output(8, True)
 time.sleep(blink_interval)
 GPIO.output(8, False)
 time.sleep(blink_interval)

# Release Resources
GPIO.cleanup()
```

## 5. Create a Dockerfile:
```bash
cat << EOT >> Dockerfile
# Python Base Image from https://hub.docker.com/r/arm32v7/python/
FROM arm32v7/python:3.6-slim-buster

# Copy the Python Script to blink LED
COPY led_blinker.py ./

# Intall the rpi.gpio python module
RUN pip install --no-cache-dir rpi.gpio

# Trigger Python script
CMD ["python", "./led_blinker.py"]

EOT
```

## 6. Create Docker Image from Dockerfile:
- Create Docker Image with the image name as "docker_blinker" and tag as "v1" using the following command –
  `docker build -t "docker_blinker:v1"`
- Once the command execution gets completed you should be able to list the image on your Raspberry Pi using the following command –
  `docker image ls`

## 7. Prepare the Circuit:
Connect an LED via a 330 Ohm register (or any value between 200 – 300 Ohm) to Raspberry Pi GPIO Pins

## 8. Start the Container to Blink the LED:
As we are interacting with the hardware components i.e. the GPIO Pins on the Raspberry Pi from this Container, we need to use `docker container run` command with either the `--privileged` option or by specifying the Linux GPIO Device using the `–device /dev/gpiomem` option.</br>
You can use one of the following commands to run the Docker Container, which in turn would blink the LED connected to your Raspberry Pi:
- `docker container run --device /dev/gpiomem -d docker_blinker:v1`
- `docker container run --privileged -d docker_blinker:v1`



[source article]: https://iotbytes.wordpress.com/create-your-first-docker-container-for-raspberry-pi-to-blink-an-led/
[architectures-other-than-amd64]: https://github.com/docker-library/official-images#architectures-other-than-amd64
[arm32v7/python]: https://hub.docker.com/r/arm32v7/python/tags?page=1&ordering=last_updated
