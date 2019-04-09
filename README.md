Redis on Windows
================

This project introduces 3 ways of running the native `Redis` on `MS-Windows (10)`.

Whilst it's recommended to use [Redis](https://redis.io) on Linux in production, it is often useful for developers on MS-Windows platforms to have their own local version of redis running to develop with.

The easiest way of running Redis on MS-Windows is to use the binary releases of [Native port of Redis for Windows](https://github.com/tporadowski/redis/releases), but as this is an unofficial port it always lags behind the latest official development of redis on Linux/OSX.

Thanks to [Vagrant](https://www.vagrantup.com/) you can choose to run the latest Redis stable version inside a VirtualBox with Linux where you'll be able to run the official native version of Redis.

Or if you have the latest version of **MS-Windows 10**, you can install [Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/about) which will lets you run the official version of Redis on Linux on Windows. :) This is our preferred approach as it lets you run native Linux binaries on Windows more efficiently than running Linux in a VM.

## Option 1. Windows Subsystem for Linux

#### [Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/about)

### Install WSL

See [README-wsl.md](README-wsl.md)

### Install Redis Stable (5.0.4)

Once your WSL has been downloaded and installed, you can run Linux with the WSL by typing `wsl.exe` or `bash.exe` from a MS-Windows Command Prompt.

- Update & upgrade your Linux distro's packages

```bash
sudo apt update && sudo apt upgrade
```

- Install Redis

```bash
sudo apt-get install gcc
sudo apt-get install make
sudo apt-get install tcl

wget http://download.redis.io/releases/redis-stable.tar.gz
tar -xzvf redis-stable.tar.gz

cd redis-stable

make MALLOC=libc
make test
make PREFIX=/path/to/Redis install

cd src && make all

cd ..
cp redis.conf /path/to/Redis/bin
```

 - Run & Test Redis

```bash
$ ./redis-server --daemonize yes

$ ./redis-cli
127.0.0.1:6379> set foo bar
OK
127.0.0.1:6379> get foo
"bar"
```

Which you can connect to from within bash or from your Windows desktop using the [redis-cli native Windows binary from MSOpenTech](#option-3-native-port-of-redis-for-windows).

## Option 2. Running the latest version of Redis with Vagrant

#### 1. [Install Vagrant on Windows](https://www.vagrantup.com/intro/getting-started/)

#### 2. Download the [vagrant-redis.zip](https://raw.github.com/ServiceStack/redis-windows/master/downloads/vagrant-redis.zip) vagrant configuration

```
wget https://raw.github.com/ServiceStack/redis-windows/master/downloads/vagrant-redis.zip
```

#### 3. Extract `vagrant-redis.zip` in any folder, e.g. in `C:\vagrant-redis`

#### 4. Launch the Virtual Box VM with `vagrant up`

```
cd C:\vagrant-redis
vagrant up
```

This will launch a new Ubuntu VM instance inside VirtualBox that will automatically install and start the latest stable version of Redis.

_The vagrant configuration was originally from [JasonPunyon/redishobo](https://github.com/JasonPunyon/redishobo) and has been modified to use the latest stable release of Redis._

## Option 3. Native port of Redis for Windows

These 64-bit binary releases are created by building the [Native port of Redis for Windows](https://github.com/tporadowski/redis), but as it's more convenient we provide a zip of the 64-bit binaries here.

#### MS Open Announcements

  - [MSOpenTech Redis on Windows 3.2 Release Notes](https://raw.githubusercontent.com/tporadowski/redis/win-4.0.2/Redis%20on%20Windows%20Release%20Notes.md)
  - [MSOpenTech's Redis on Windows](https://github.com/ServiceStack/redis-windows/blob/master/docs/msopentech-redis-on-windows.md)

### Current Version: Redis 4.0.2.3 for Windows (alpha) (1 Oct 2018)

#### 1. Download the `Redis-x64-4.0.2.3.zip` native 64bit Windows port of Redis

```
wget https://github.com/tporadowski/redis/releases/download/v4.0.2.3-alpha/Redis-x64-4.0.2.3.zip
```

#### 2. Extract `Redis-x64-4.0.2.3.zip` in any folder, e.g. in `C:\Redis`

#### 3. Run the `redis-server.exe` using the local configuration

```
cd C:\Redis
redis-server.exe redis.windows.conf
```

#### 4. Run `redis-cli.exe` to connect to your redis instance

```
cd C:\redis
redis-cli.exe
```

#### 5. Start playing with redis

```
127.0.0.1:6379> SET foo bar
OK
127.0.0.1:6379> GET foo
"bar"
127.0.0.1:6379> KEYS *
1) "foo"
127.0.0.1:6379>
```

---

For more details, please see:

 - https://www.liangzl.com/get-article-detail-26835.html
 - https://blog.csdn.net/qq_34002221/article/details/85019752

---

The following content is reserved for my reference in the original project [ServiceStack/redis-windows](https://github.com/ServiceStack/redis-windows):

The MSOpenTech of Redis adds some useful extensions for better integration with Windows.

#### Running Redis as a Service

If you installed Redis using the MSI package, then Redis was already installed as a Windows service. Nothing further to do.

If you would like to change its settings, you can update the redis.windows-service.conf file and then restart the Redis
service (Run -> services.msc -> Redis -> Restart).

During installation of the MSI you can either use the installer’s user interface to update the port that Redis listens to and the firewall exception or run it silently without a UI. The following examples show how to install from the command line:

#### default install (port 6379 and firewall exception ON):

    msiexec /i Redis-Windows-x64.msi

#### set port and turn OFF firewall exception:

    msiexec /i Redis-Windows-x64.msi PORT=1234 ADD_FIREWALL_RULE=""

#### set port and turn ON firewall exception:

    msiexec /i Redis-Windows-x64.msi PORT=1234 ADD_FIREWALL_RULE =1

#### install with no user interface:

    msiexec /quiet /i Redis-Windows-x64.msi

If you did not install Redis using the MSI package, then you still run Redis as a Windows service by following these instructions:

In order to better integrate with the Windows Services model, new command line arguments have been introduced to Redis.
These service arguments require an elevated user context in order to connect to the service control manager.
If these commands are invoked from a non-elevated context, Redis will attempt to create an elevated context in which to execute these commands.
This will cause a User Account Control dialog to be displayed by Windows and may require Administrative user credentials in order to proceed.

#### Installing the Service

    --service-install

This must be the first argument on the redis-server command line. Arguments after this are passed in the order they occur to Redis when the service is launched.
The service will be configured as Autostart and will be launched as "NT AUTHORITY\NetworkService". Upon successful installation, a success message will be displayed and Redis will exit.

This command does not start the service.

For instance:

    redis-server --service-install redis.windows.conf --loglevel verbose

#### Uninstalling the Service

    --service-uninstall

This will remove the Redis service configuration information from the registry. Upon successful uninstallation, a success message will be displayed and Redis will exit.
This does command not to stop the service.

For instance:

    redis-server --service-uninstall

#### Starting the Service

    --service-start

This will start the Redis service. Upon successful startup, a success message will be displayed and Redis service will be started.

For instance:

    redis-server --service-start

#### Stopping the Service

    --service-stop

This will stop the Redis service. Upon successful termination, a success message will be displayed and Redis will exit.

For instance:

    redis-server --service-stop

#### Naming the Service

    --service-name name

This optional argument may be used with any of the preceding commands to set the name of the installed service. This argument should follow the service-install, service-start, service-stop or service-uninstall commands, and precede any arguments to be passed to Redis via the service-install command.
The following would install and start three separate instances of Redis as a service:

    redis-server --service-install –service-name redisService1 –port 10001
    redis-server --service-start –service-name redisService1
    redis-server --service-install –service-name redisService2 –port 10002
    redis-server --service-start –service-name redisService2
    redis-server --service-install –service-name redisService3 –port 10003
    redis-server --service-start –service-name redisService3

## [Redis Vue](http://templates.servicestack.net/docs/web-apps#redis-vue)

Redis Vue is a simple, lightweight, versatile Redis Admin UI developed using [Vue](https://vuejs.org/v2/guide/) and [ServiceStack .NET Core Web Apps](http://templates.servicestack.net/docs/web-apps). It supports Redis's core **Strings**, **Lists**, **Sets**, **Sorted Sets** and **Hash** data structures and custom Redis commands with its entire functionality contained in a single [/app/index.html](https://github.com/NetCoreWebApps/Redis/blob/master/app/index.html) using the dynamic [ServiceStack Templates](http://templates.servicestack.net/) language, making it easy to customize and further enhance.

### [Live Demo](http://redis.web-app.io) - http://redis.web-app.io

[![](http://templates.servicestack.net/assets/img/screenshots/redis.png)](http://redis.web-app.io)

#### Download for Windows, OSX and Linux https://github.com/NetCoreWebApps/Redis

## [Redis React Browser](https://servicestack.net/redis-react)

Redis React is a simple user-friendly UI for browsing data in Redis servers which takes advantages of the complex
type conventions built in the ServiceStack.Redis Client to provide a rich, human-friendly UI for navigating related datasets, enabling a fast and fluid browsing experience for your Redis servers.

#### [Live Demo](http://redisreact.servicestack.net/#/)

[![](https://raw.githubusercontent.com/ServiceStack/Assets/master/img/livedemos/redis-react/home.png)](http://redisreact.servicestack.net/#/)

#### Windows, OSX and Linux downloads available from [Redis React Home Page](https://github.com/ServiceStackApps/RedisReact#download)

## [Configure Redis Sentinel Servers](https://github.com/ServiceStack/redis-config)

[![Instant Redis Setup](https://raw.githubusercontent.com/ServiceStack/Assets/master/img/redis/instant-sentinel-setup.png)](https://github.com/ServiceStack/redis-config)

See the
[redis config project](https://github.com/ServiceStack/redis-config) for a quick way to setup up
the minimal
[highly available Redis Sentinel configuration](https://github.com/ServiceStack/redis-config/blob/master/README.md#3x-sentinels-monitoring-1x-master-and-2x-slaves)
including start/stop scripts for instantly running multiple redis instances on a single (or multiple)
Windows, OSX or Linux servers.

