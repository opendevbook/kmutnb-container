# understand linux namespace

## Objective:
This lab will introduce you to Linux namespaces, which are essential for containerization. We'll explore how namespaces isolate processes and their resources, making them ideal for running multiple containers on a single host. You'll learn how to create and manipulate namespaces using the unshare command and gain hands-on experience with Podman, a popular containerization tool.

Lab Steps:
1. Understanding Namespaces

- What are namespaces?
    - Isolated environments for processes
    - Provide resource isolation and security
    - Types: PID, network, mount, IPC, UTS, user
- How do they work?
Each namespace has its own root filesystem, network stack, etc.
Processes within a namespace cannot see or interact with resources outside of it.

- types of namespaces:

    - netns – network namespace
    - ipcns – IPC namespace
    - mntns – mount namespace
    - utsns – UTS namespace
    - userns – user namespace

2. Creating Namespaces with unshare

- PID namespace:
    - Isolate process IDs
    - Create a child process with its own PID namespace:
```
[vagrant@centos9s ~]$ sudo unshare --fork --pid --mount-proc bash

[root@centos9s vagrant]# ps -o pid,pidns,netns,mntns,ipcns,utsns,userns,args -p 1
    PID      PIDNS      NETNS      MNTNS      IPCNS      UTSNS     USERNS COMMAND
      1 4026532299 4026531840 4026532298 4026531839 4026531838 4026531837 bash
```

```
# ps -o pid,pidns,netns,mntns,ipcns,utsns,userns,args -p 1
```

Apparently, PID 1 belongs to six different namespaces:

- PID
- network
- mount
- IPC
- UTS
- user

### The /proc/<PID>/ns Directory

```
[vagrant@centos9s ~]$ ps aux  | grep vagrant
root        3851  0.0  0.3  19404 11520 ?        Ss   13:20   0:00 sshd: vagrant [priv]
vagrant     3856  0.0  0.3  22644 13516 ?        Ss   13:20   0:00 /usr/lib/systemd/systemd --user
vagrant     3858  0.0  0.1 108256  7476 ?        S    13:20   0:00 (sd-pam)
vagrant     3865  0.0  0.1  19780  7460 ?        S    13:20   0:00 sshd: vagrant@pts/0
vagrant     3866  0.0  0.1   8408  4992 pts/0    Ss   13:20   0:00 -bash
vagrant     4042  0.0  0.0  10104  3328 pts/0    R+   13:38   0:00 ps aux
vagrant     4043  0.0  0.0   6428  2176 pts/0    R+   13:38   0:00 grep --color=auto vagrant
```

Generally, the /proc/<PID>/ns directory contains symbolic links to the namespace files for each type of namespace that the process belongs to.

For instance, let’s use ls to check the namespaces of the process with PID 3856:

```
total 0
lrwxrwxrwx. 1 vagrant vagrant 0 Aug 20 13:20 cgroup -> 'cgroup:[4026531835]'
lrwxrwxrwx. 1 vagrant vagrant 0 Aug 20 13:31 ipc -> 'ipc:[4026531839]'
lrwxrwxrwx. 1 vagrant vagrant 0 Aug 20 13:31 mnt -> 'mnt:[4026531841]'
lrwxrwxrwx. 1 vagrant vagrant 0 Aug 20 13:31 net -> 'net:[4026531840]'
lrwxrwxrwx. 1 vagrant vagrant 0 Aug 20 13:31 pid -> 'pid:[4026531836]'
lrwxrwxrwx. 1 vagrant vagrant 0 Aug 20 13:38 pid_for_children -> 'pid:[4026531836]'
lrwxrwxrwx. 1 vagrant vagrant 0 Aug 20 13:38 time -> 'time:[4026531834]'
lrwxrwxrwx. 1 vagrant vagrant 0 Aug 20 13:38 time_for_children -> 'time:[4026531834]'
lrwxrwxrwx. 1 vagrant vagrant 0 Aug 20 13:31 user -> 'user:[4026531837]'
lrwxrwxrwx. 1 vagrant vagrant 0 Aug 20 13:31 uts -> 'uts:[4026531838]'
```

## Lab: Exploring IP Netnamespaces with `ip netns`

### Objective:
This lab will introduce you to IP network namespaces (netnamespaces) and how to manage them using the ip netns command. You'll learn how to create, list, delete, and interact with netnamespaces.


Lab Steps:
1. Creating a Netnamespace
- Use the add command to create a new netnamespace:
```
[vagrant@centos9s ~]$ sudo ip netns add red
[vagrant@centos9s ~]$ sudo ip netns add blue
```

2. Listing Netnamespaces
- Use the list command to view all existing netnamespaces:
```
[vagrant@centos9s ~]$ sudo ip netns list
```
- list network on host
```
[vagrant@centos9s ~]$ ip link
```
- view network namespace inside namespace
```
[vagrant@centos9s ~]$ sudo ip netns exec red ip link
[vagrant@centos9s ~]$ sudo ip netns exec blue ip link
```

3. Entering a Netnamespace
- Use the exec command to enter a netnamespace and execute commands within it:
```
[vagrant@centos9s ~]$ ip netns exec red bash
[vagrant@centos9s ~]$ ps -o pid,pidns,args
```

or run in short command
```
ip -n red link
ip -n blue link
```

- Run arp command  on host check Arp table
```
[vagrant@centos9s ~]$ arp
```

- Run route command check routing table
```
[vagrant@centos9s ~]$ route
```

4. Configuring Network Interfaces
- Create a virtual network interface within the netnamespace:
```
[vagrant@centos9s ~]$ sudo ip link add veth-red type veth peer name veth-blue
[vagrant@centos9s ~]$ sudo ip link set veth-red netns red
[vagrant@centos9s ~]$ sudo ip link set veth-blue netns blue
```

- Add ip to veth-red and veth-blue
```
[vagrant@centos9s ~]$ sudo ip -n red addr add 192.168.15.1/24 dev veth-red
[vagrant@centos9s ~]$ sudo ip -n blue addr add 192.168.15.2/24 dev veth-blue

[vagrant@centos9s ~]$ sudo ip -n red link set veth-red up
[vagrant@centos9s ~]$ sudo ip -n blue link set veth-blue up

```

> if Clear ip (Just in case some thing wrong)
> ```
># ip -n red  addr flush dev veth-red
># ip -n blue addr flush dev veth-blue
> ```
>

- Check
```
[vagrant@centos9s ~]$ sudo ip  netns exec red  ip a
[vagrant@centos9s ~]$ sudo ip  netns exec blue ip a

```

- ping ip address from red to blue, and blue to red
```
[vagrant@centos9s ~]$ sudo ip netns exec blue ping 192.168.15.1
[vagrant@centos9s ~]$ sudo ip netns exec red ping 192.168.15.2

```

- Check arp table on red and blue
```
[vagrant@centos9s ~]$ sudo ip netns exec red arp
[vagrant@centos9s ~]$ sudo ip netns exec blue arp
```
- But arp table on host will see this
```
[vagrant@centos9s ~]$ arp
```
- **Final script 1**
```bash
ip netns add red
ip netns add blue
#show namespace
ip netns show
ip link add veth-red type veth peer name veth-blue
ip link set veth-red netns red
ip link set veth-blue netns blue
ip -n red addr add 192.168.15.1/24 dev veth-red
ip -n blue addr add 192.168.15.2/24 dev veth-blue
ip -n red link set veth-red up
ip -n blue link set veth-blue up

ip netns exec red ping 192.168.15.2
ip netns exec red ping 192.168.15.1
#Cleanup
ip netns delete red
ip netns delete blue
```

## Connect more than 2 namespace
- Create virtual switch with linux bridge (or OpenVswitch) and connect namespace together via bridge
```
[vagrant@centos9s ~]$ sudo ip link add v-net-0 type bridge
[vagrant@centos9s ~]$ ip a
[vagrant@centos9s ~]$ sudo ip link set dev v-net-0 up
```

- install package `bridge-utils`
```
[vagrant@centos9s ~]$ sudo dnf install brige-utils
[vagrant@centos9s ~]$ brctl show
```
- Delete old veth-red, veth-blue link because we not use anymore
```
[vagrant@centos9s ~]$ sudo ip -n red link del veth-red
```
> delete veth-red and veth-blue will automatically remove

- Create new cable connect namespace to bride
```
[vagrant@centos9s ~]$ sudo ip link add veth-red type veth peer name veth-red-br
[vagrant@centos9s ~]$ sudo ip link add veth-blue type veth peer name veth-blue-br
```

- Add cable to bride
```
[vagrant@centos9s ~]$ sudo ip link set veth-red netns red
[vagrant@centos9s ~]$ sudo ip link set veth-red-br master v-net-0

[vagrant@centos9s ~]$ sudo ip link set veth-blue netns blue
[vagrant@centos9s ~]$ sudo ip link set veth-blue-br master v-net-0
```

- Set ip address and turn it up
```
[vagrant@centos9s ~]$ sudo ip -n red addr add 192.168.15.1/24 dev veth-red
[vagrant@centos9s ~]$ sudo ip -n blue addr add 192.168.15.2/24 dev veth-blue

[vagrant@centos9s ~]$ sudo ip -n red link set veth-red up
[vagrant@centos9s ~]$ sudo ip -n blue link set veth-blue up

[vagrant@centos9s ~]$ sudo ip link set veth-red-br up
[vagrant@centos9s ~]$ sudo ip link set veth-blue-br up
```

- Test ping 
```
[vagrant@centos9s ~]$ sudo ip netns exec red ping 192.168.15.2
[vagrant@centos9s ~]$ sudo ip netns exec blue ping 192.168.15.1
```
- Run brctl show again
```
[vagrant@centos9s ~]$ brctl show
```

- **Final summary script connect namespace with linux bridge**

```bash

ip netns add red
ip netns add blue

#show namespace
ip netns show
ip link add v-net-0 type bridge
ip link set dev v-net-0 up

ip link add veth-red type veth peer name veth-red-br
ip link add veth-blue type veth peer name veth-blue-br

ip link set veth-red netns red
ip link set veth-red-br master v-net-0

ip link set veth-blue netns blue
ip link set veth-blue-br master v-net-0
ip -n red addr add 192.168.15.1/24 dev veth-red
ip -n blue addr add 192.168.15.2/24 dev veth-blie

ip -n red link set veth-red up
ip -n blue link set veth-blue up
ip link set veth-red-br up
ip link set veth-blue-br up
```

## ip netns vs. unshare: A Comparison

Both `ip netns` and `unshare` are tools used to create and manage isolated environments (namespaces) on Linux systems, but they serve different purposes and have distinct functionalities.

### ip netns
- Purpose: Primarily designed for network namespace management.
- Functionality:
    - Creates, lists, deletes, and manipulates network namespaces.
    - Configures network interfaces, routes, and other network-related settings within namespaces.
    - Provides a high-level interface for network namespace management.

### unshare
- Purpose: A more general-purpose tool for creating various types of namespaces, including PID, network, mount, IPC, UTS, and user namespaces.
- Functionality:
    - Creates child processes with specific namespaces.
    - Allows for granular control over namespace creation and configuration.
    - Can be used to isolate processes in a variety of ways beyond just networking.

#### Key Differences
- Scope: ip netns is specifically focused on network namespaces, while unshare can create and manage multiple types of namespaces.
- Level of Control: unshare provides more granular control over namespace creation and configuration, allowing you to specify which namespaces to isolate and how.
- Interface: ip netns offers a more user-friendly interface for managing network namespaces, while unshare is more flexible but requires a deeper understanding of namespace concepts.

#### When to Use Which
- Network Namespace Management: Use ip netns when you primarily need to create, manage, and configure network namespaces.
- General Namespace Creation: Use unshare when you need to isolate processes in a variety of ways, including PID, mount, IPC, UTS, or user namespaces.

**In summary,** ip netns is a specialized tool for network namespace management, while unshare is a more general-purpose tool for creating various types of namespaces. The best choice depends on your specific needs and the level of control you require.