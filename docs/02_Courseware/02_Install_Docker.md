## Overview
We have just created our Ubuntu server.  We now need to apply any available system updates along with installing and configuring Docker to begin working with containers.

## Install Updates
Just like any other operating system, updates are periodically released to support new features and patch any potential security threats.  We will apply the updates first.

  1. If you have not already, connect to your remote Ubuntu server and login.

  2. From the login prompt, you may see a status of available updates. (If not, don't be too alarmed - continue with these steps anyway just to be sure.)  
  <img src="../images/ubuntu_updates.jpg" class="block"/>

  3. First we need to ensure our list of sources for our system updates are up-to-date. From the command prompt, type the following:
  ```bash
  sudo apt-get update
  ```
  
  4. Now we can install updates.  From the command prompt, type the following to automatically install all available updates:
  ```bash
  sudo apt-get upgrade -y
  ```

  5. Depending on the number and size of available updates, this process may take a few minutes.  Now would be a good time to take a break.

## Install Docker
We now have an updated Ubuntu operating system.  We are ready to install Docker.

  1. We need to add the GPG key for the official Docker repository to the system because in the next step we want to download the Docker 'installer' directly from Docker and not the default Ubuntu servers to ensure we get the latest version of the engine.  From the command prompt, type the following:
  ```bash
  sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
  ```  
  > NOTE: You can paste this into PuTTY by _right-clicking_ the terminal screen.

  2. Now, we need to tell Ubuntu _where_ the Docker repository is located. From the command prompt, type (or paste) the following:
  ```bash
  sudo apt-add-repository 'deb https://apt.dockerproject.org/repo ubuntu-xenial main'
  ```

  3. Once again, update the package database with the Docker packages from the newly added repository:
  ```bash
  sudo apt-get update
  ```

  4. Make sure you are about to install from the Docker repository instead of the default Ubuntu repository:
  ```bash
  apt-cache policy docker-engine
  ```

  5. You should see output _similar_ to the following (notice that `docker-engine` is not installed and the `docker-engine` version number might be different):
  ```bash
  docker-engine:
  Installed: (none)
  Candidate: 1.11.1-0~xenial
  Version table:
     1.11.1-0~xenial 500
        500 https://apt.dockerproject.org/repo ubuntu-xenial/main amd64 Packages
     1.11.0-0~xenial 500
        500 https://apt.dockerproject.org/repo ubuntu-xenial/main amd64 Packages
  ```

  6. Finally, install Docker:
  ```bash
  sudo apt-get install -y docker-engine
  ```

  7. Installing the Docker engine may take an additional minute or two.

## Additional Configuration
To simplify running and managing Docker, there's some additional configuration that we need to implement.  While this section is optional, it is recommended to make managing Docker much easier.

### Ensure Docker Engine is Running

  1. From the command prompt, type:
  ```bash
  sudo systemctl status docker
  ```

  2. You should see something similar to the following:
  ```bash
  ● docker.service - Docker Application Container Engine
     Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
     Active: active (running) since Sun 2017-06-04 22:38:16 UTC; 4min 10s ago
       Docs: https://docs.docker.com
   Main PID: 32844 (dockerd)
  ```

  3. Because the service is running, we can now use the `docker` command later in this workshop.

### Enable Docker Engine at Startup
Let's make sure the Docker engine is configured to run on system startup (and reboot).

  1. From the command prompt, type:
  ```bash
  sudo systemctl enable docker
  ```

  2. You should see something similar to the following:
  ```bash
  Synchronizing state of docker.service with SysV init with /lib/systemd/systemd-sysv-install...
Executing /lib/systemd/systemd-sysv-install enable docker
  ```

### Elevate Your Privileges
Be default, running the `docker` command requires root privileges - that is, you have to prefix the command with `sudo`. It can also be run by a user in the **docker** group, which is automatically created during the install of Docker.  If you attempt to run the `docker` command without prefixing it with `sudo` or without being in the docker group, you'll get an output like the following:

```bash
docker: Cannot connect to the Docker daemon. Is the docker daemon running on this host?.
See 'docker run --help'.
```

To avoid typing `sudo` whenever you run the `docker` command, add your username to the docker group:

```bash
sudo usermod -aG docker $(whoami)
```

You will then need to log out and back in for the changes to take effect.

If you need to add another user to the `docker` group (one in which you have not logged in as currently), simply provide that username explicitly in the command:

```bash
sudo usermod -aG docker <username>
```

You've successfully installed the Docker engine.  You have also configured it to run at startup and have added yourself to the **Docker** group so that you have sufficient privileges for running Docker.