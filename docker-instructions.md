# Using Docker

## Installing Docker

First, download and install Docker, if you haven't already. You should have installed it as part of your new employee orientation, but if not it can be downloaded here:

https://store.docker.com/editions/community/docker-ce-desktop-mac

## Downloading Images

### Searching for Images

For your first container let's set up a Debian server. First you'll need to download a Debian image. You can do this by running:

`docker search debian`

You'll see a list of Docker images that contain the word "debian". Towards the top of the list (it might be the second item listed) you'll see one named `debian` that's marked as `[OK]` in the "OFFICIAL" column:

```
NAME                                 DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
ubuntu                               Ubuntu is a Debian-based Linux operating sys…   8670                [OK]
debian                               Debian is a Linux distribution that's compos…   2850                [OK]
google/debian                                                                        54                                      [OK]
```

When in doubt, use official images. In this case, the `debian` image is maintained by the developers of Debian and is therefore the "official" Debian image for docker, so it's totally safe to use. Unofficial images run the risk of containing malware or being heavily customized in ways you may not want or be aware of. 

### Downloading Images

You can download that image by running:

`docker pull debian`

Docker will download the latest version of the `debian` image, which is basically a copy of the Debian OS plus a few bells and whistles to make it work on Docker.

By default, `docker pull debian` will download the latest version. You can also specify a version number when downloading an image. For example, you could have run `docker pull debian:1.0.0` which would have downloaded version 1.0.0 specifically. You can also specify that you want the latest version by running `docker pull debian:latest`.

## Starting Containers

Now that you've downloaded the "debian" image, you can start up a functioning Debian server by running this command (BUT DON'T RUN IT YET!):

`docker run --name my-first-server -p 127.0.0.1:81:80 -i -t debian  /bin/bash`

Before you run it though, let's break it down so you understand each of its parts:

| Part | What it does |
| ---- | ------------ |
| `docker run` | Runs an image in a new container |
| `--name` | Names the container. In this case, `--name my-first-server` will name the container "my-first-server". |
| `-p` | Maps a container's port to the host's. In this case, `-p 127.0.0.1:81:80` will map you computer's Port 81 (which may not be in use) to your new server's Port 80 (which is where Nginx and Apache typically listen by default). Your computer likely already has Port 80 in use, which is why you want to use Port 81 instead. By mapping it to Port 80 on the server, Nginx and Apache won't need to be configured to use a different port. Essentially, your server will think it's using Port 80, but you'll actually be using Port 81. |
| `-i -t` | Using them together keeps the container running, even if you leave it. This is useful if you want to keep your server running after you leave it. |
| `debian` | The name of the image you're using. |
| `/bin/bash` | The command you want this container to run. In this case, you're telling it to run bash so you have access to the command prompt. |

> BONUS: You can also add `-v` to share files/folders on your computer with your container. For example, I maintain a shared folder full of useful scripts and files. I share that with my containers with `-v /Users/tyoung/Desktop/docker-stuff:/docker-stuff`. The "docker-stuff" folder on my desktop can then be accessed at `/docker-stuff` in the container.

Now that you understand what the command is doing, run `docker run --name my-first-server -p 127.0.0.1:81:80 -i -t debian  /bin/bash` to start your new server.

## Container Management

### Exiting Containers

Your terminal prompt should now change to something like:

```
root@3e66c6647ef1:/#
```

You are now the root user of your new Debian server. Before we start using it though, let's learn how to manage the container.

First, exit the container by running: 

`exit`

### Viewing Running Containers

Next, you can view a list of all running containers by running:

`docker ps -a`

You should see output like this:

```
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
3e66c6647ef1        debian              "/bin/bash"         7 minutes ago       Exited (0) 2 minutes ago                       my-first-server
```

Right now, you should only have one running container: "my-first-server". Let's break down what it shows:

| Part | What it does |
| ---- | ---- |
| `CONTAINER ID` | The ID number of that container. |
| `IMAGE` | The image that container was built from. |
| `COMMAND` | The command it's running or the command it last run. |
| `CREATED` | How long ago the container was created. |
| `STATUS` | Whether or not the container is running, and if so, how long it's been running. |
| `PORTS` | The ports in use by the container. In this case, because the container is stopped, there are no ports in use. |

### Deleting Containers

You can delete containers by running:

`docker rm CONTAINER-NAME` 

with "CONTAINER-NAME" being the name of the container you want to delete. Don't run it right now, but to use an example, you could delete this container by running `docker rm my-first-server`.

### Starting Containers

Finally, let's learn how to start and re-enter containers so we can continue setting up our server. If you'll recall, the current status of your "my-first-server" container is "Exited" which mean's it's no longer running. So first, we'll need to start it back up again before we can re-enter it. 

You can start containers by running this command:

`docker start CONTAINER-NAME`

So in this case, you'll want to run:

`docker start my-first-server`

If you run `docker ps -a` again, you'll see the "STATUS" now shows your container is running:

```
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                  NAMES
afac472a5f86        debian              "/bin/bash"         24 seconds ago      Up 6 seconds        127.0.0.1:81->80/tcp   my-first-server
```

### Re-entering Containers

Now that the container is running again, let's re-enter it. You can re-enter containers with this command:

`docker attach CONTAINER-NAME`

So in this case, you'll want to run:

 `docker attach my-first-server`
 
 After running that you should again see that your terminal prompt now says something like `root@3e66c6647ef1:/#`, indicatig that you're now logged into your container as the root user.
 

## Using Debian

### Setting Up Debian

The Debian docker image (like most official Linux OS images) is just the bare-bones OS, with no added functions. You'll find that almost all commands you might be familiar with (such as `vim` and `top`) won't run, as you'll instead get a `command not found` error. So the first thing we'll need to do is download a suite of commands you'll want to use. The easiest way to download and install packages on Debian is with the `apt` command.

The first thing you'll want to do is run this command to look for any available updates to the handful of commands you currently have:

`apt-get -y update && apt-get -y upgrade`

This will look for any available updates and then automatically upgrade them.

After that, you'll want to install common commands you may want to use. This is by no means an exhaustive list, and you probably won't use all of these yourself, but these are the commands that I install whenever I spin up a new Debian server:

| COMMAND | WHAT IT DOES |
| ---- | ---- |
| `apt-utils` | Basically a bunch of background features for `apt`. You don't really need to worry about the specifics, it just makes `apt` easier to use. |
| `man` | A manual command. It downloads and stores manual pages for all your commands. Very useful to have. If you don't know how to use a command, you can view the manual page for it by running `man command`. |
| `sudo` | The `sudo` command let's you run commands as the root user. Since you're usually already the root user, you probably won't end up using this one too much. But it's useful to have around just in case you run into some sort of permissions issue. You can run commands as the root user by running `sudo COMMAND`. |
| `coreutils` | A standard suite of commands that most people need regularly. I'm not going to list all of them, but they're useful to have. |
| `less` | A command for viewing files. For example, you could view the contents of files by running `less filename`. |
| `vim` | A command to view and edit files. Basically a text editor, you can edit files with `vim filename`. |
| `wget` | Let's you easily download files from URLs. |
| `curl` | A tool for getting or sending files using URL syntax. You'll mostly be using it to craft HTTP requests to simulate attacks for rule testing. |
| `zip` | Compresses files and folders into `.zip` archives. |
| `unzip` | Expands `.zip` archives. |
| `tar` | Compresses and expands `.tar` and `.tar.gz` archives. |

You can install these commands by running `apt-get install COMMAND`. For example, you could install `apt-utils` by running `apt-get install apt-utils`.

To save time though, we can install all of these packages (plus a few others I didn't mention) by running this command:

`apt-get -y install apt-utils man sudo coreutils less vim wget curl procps ca-certificates gnupg software-properties-common zip unzip tar make libssl-dev libpcre3-dev gcc make zlib1g-dev apt-transport-https`

>NOTE: If you were to run `apt-get install` without the `-y` flag, you would be prompted to confirm that you want to download and install that package. The `-y` flag in the above command will automatically agree to this. You don't have to use it, but it makes things easier for you, since you won't have to agree a dozen times.

### Using Debian

Now that you've finished setting up Debian, feel free to play around with it for a little while. If you're less familiar with Linux-based systems, try out some commands. One of the great things about Docker is there's no risk. If you somehow actually break something, you can just delete the container and create a new one.

Here are some other useful commands that you should have at your disposal:

| COMMAND | WHAT IT DOES |
| ---- | ---- |
| `ls` | Lists the contents of a directory. I recommend using `ls -alh`, which provides more detailed information and is easier to read. |
| `cd` | Let's you change directories and is how you'll navigate around the server. For example, you could navigate to the `/etc/httpd` directory by running `cat /etc/httpd`. |
| `top` | Lists all running processes on the server and their stats. |
| `mkdir` | Creates a new directory/folder. |
| `touch` | Creates a new file. |
| `pwd` | Tells you the current directory that you're in. |
| `rm` | Deletes files and folders. |
| `cp` | Copies files and folders. For example, `cp file1.txt file2.txt` will make a copy of `file1.txt` named `file2.txt`. |
| `mv` | Moves files and folders. For example, `mv /folder1/file.txt /folder2/file.txt` will move `file.txt` from `/folder1` to `/folder2`. <br><br>It can also be used to rename files and folders. For example, `mv file1.txt file2.txt` will "move" `file1.txt` to the same folder it's already in and give it the name `file2.txt`, essentially just renaming the file. |
| ctrl + c | Hitting ctrl + c on your keyboard will allow you to exit most commands. |

## Apache

### Installing Apache

Next, let's turn your Debian container into a functioning server. To do this, you'll need to install software for serving web pages. In this case, let's install Apache, which is going to be easiest.

Installing Apache is actually very easy on Debian. Just like the other packages, you can install apache with `apt` by running this command:

`apt-get -y install apache2`

### Running Apache

Now that Apache's been installed you can start it by running:

`apachectl start`

>NOTE: You may see this warning:<br>
>`apache2: Could not reliably determine the server's fully qualified domain name, using 172.17.0.2. Set the 'ServerName' directive globally to suppress this message`<br>
>This warning is nothing to worry about. Essentially, because you're running Apache in a local container on your computer instead of an actual server, Apache doesn't know what IP address to use and throws that warning.

The `apachectl` command is what you'll use to manage the Apache process. Besides `start`, you can also `stop` Apache, and `restart` it.

You can confirm Apache is running by running `top`. You should see output like this:

```
Tasks:   5 total,   1 running,   4 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.1 us,  0.2 sy,  0.0 ni, 99.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  3075764 total,   545212 free,   278976 used,  2251576 buff/cache
KiB Swap:  1048572 total,  1048572 free,        0 used.  2603624 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
    1 root      20   0   18136   3168   2680 S   0.0  0.1   0:00.05 bash
19673 root      20   0   75612   4272   3044 S   0.0  0.1   0:00.01 apache2
19674 www-data  20   0  364772   3988   2436 S   0.0  0.1   0:00.14 apache2
19675 www-data  20   0  364772   3988   2436 S   0.0  0.1   0:00.13 apache2
19731 root      20   0   41052   3096   2600 R   0.0  0.1   0:00.00 top
```

You can see in the "COMMAND" column on the far right, the `apache2` service is running. Once you've confirmed Apache is running, you can leave `top` by pressing ctrl + c.

Now that Apache's running, you can also visit it in your web browser! Visit http://localhost:81/ and you should see the "Apache2 Debian Default Page". This is the default page served by Apache when no other web pages have been provided.


## Installing Signal Sciences

Now that Apache's up and running, let's install Signal Sciences!

### Installing the Apache Agent

First, we'll need to install the Apache agent. To begin, copy and paste this entire command block into your terminal and run it:

```
sudo apt-get install -y apt-transport-https wget gnupg
wget -qO - https://apt.signalsciences.net/gpg.key | sudo apt-key add -
sudo tee /etc/apt/sources.list.d/sigsci-release.list <<-'EOF'
deb https://apt.signalsciences.net/release/debian/ stretch main
EOF
sudo apt-get update
```

This command tells `apt` where to download Signal Sciences packages from.

Now, we can install the agent with `apt` by running:

`apt-get -y install sigsci-agent`

After the agent is finished installing, you will need to provide it with your Agent Keys. The Agent Keys are essentially what tells the Signal Sciences cloud which site the agent is associated with. This is what causes requests and data processed by your agent to show up in your site's dashboard (instead of the dashboard of some other customer).

The Agent keys need to be put into `/etc/sigsci/agent.conf`. However, that file doesn't exist yet. So first, you'll need to create the `/etc/sigsci` folder by running:

`mkdir /etc/sigsci/`

Then, you'll need to create the `agent.conf` file there by running:

`touch /etc/sigsci/agent.conf`

Next, you can obtain the Agent Keys from within the Dashboard by going to Agents > View Agent Keys. (If it helps, detailed instructions with screenshots can be found here: https://docs.signalsciences.net/install-guides/#step-1-agent-installation)

You'll need to put those keys in your `agent.conf` file. You can edit the `agent.conf` file with `vim` by running:

`vim /etc/sigsci/agent.conf`

Once inside `vim`, type `i` to enter "INSERT" mode, which allows you to actually edit the file just like any other text editor.

In the `agent.conf` file, you'll need to put this:

```
accesskeyid = "AGENTACCESSKEYHERE"
secretaccesskey = "AGENTSECRETACCESSKEYHERE"
```

Of course, you'll need to replace "AGENTACCESSKEYHERE" and "AGENTSECRETACCESSKEYHERE" with the actual keys you found in the Dashboard.

Once you've done that, press your `esc` key to leave "INSERT" mode. Then type `:wq` (which stands for "write quit") to save your changes and exit `vim`.

### Installing the Apache Module

Next, we'll also need to install the Apache module. This is actually much easier to do, as you can install it with a single `apt` command:

`apt-get -y install sigsci-module-apache`

After installing the module, you'll also need to restart Apache for it to recognize it. You can restart Apache with:

`apachectl restart`

### Running Signal Sciences

We're almost finished. Finally, we just need to start the Agent by running:

`sigsci-agent`

You should see some output like this, which is output from the Agent and indicates it's running:

```
2018/11/08 22:24:06.811135 Signal Sciences Agent 3.13.0 starting as user root with PID 20552, Max open files=1048576, Max data size=unlimited, Max address space=unlimited, Max stack size=8388608
2018/11/08 22:24:06.897637 ==================================================
2018/11/08 22:24:06.897690 Agent:    6bec18768ec9
2018/11/08 22:24:06.897712 System:   debian 9.5 (linux 4.9.93-linuxkit-aufs)
2018/11/08 22:24:06.897756 VM:       docker
2018/11/08 22:24:06.897779 Memory:   2.489G / 2.933G RAM available
2018/11/08 22:24:06.897797 CPU:      2 / 4 CPU cores available
2018/11/08 22:24:06.897816 ==================================================
2018/11/08 22:24:08.019273 Starting RPC server
2018/11/08 22:24:08.020605 Started legacy RPC listener on "unix:/tmp/sigsci-lua"
2018/11/08 22:24:08.020833 Started RPC listener on "unix:/var/run/sigsci.sock"
```

### Testing Signal Sciences

If you visit the Agents page of your Dashboard, you should now see your new server listed. However, you'll see the module is listed "Undetected". The module doesn't check in (and become detected) until it processes a request. We can give it a request to process by visiting a URL on your server that doesn't exist. For example: http://localhost:81/404

It can take up to 30 seconds, but after refreshing the Dashboard you should eventually see the module as detected. You can also view that request, which should have been tagged with a "404" signal, on the Requests page.


## Further Reading

https://www.jamescoyle.net/how-to/1503-create-your-first-docker-container
https://maker.pro/linux/tutorial/basic-linux-commands-for-beginners
