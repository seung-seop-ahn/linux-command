# Welcome to basic-linux-performance-commands 👋

> For more information (Available only in Korean)
>
> - [uptime](/docs/uptime/README.md)
> - [dmesg](/docs/dmesg/README.md)
> - [free](/docs/free/README.md)
> - [df](/docs/df/README.md)
> - [top](/docs/top/README.md)
> - [netstat](/docs/netstat/README.md)
> - [tcpdump](/docs/tcpdump/README.md)

## Environments

```shell
$ docker pull ubuntu
$ docker run --privileged -it --name linux ubuntu # --privileged: For all permissions
```

```shell
$ apt-get update
$ apt-get install vim
$ apt-get install gcc
$ apt-get install lsof
$ apt-get install net-tools
$ apt-get install tcpdump
```

## uptime

In case `Load average > Number of CPU`, Check R & D Condition process.

- If too many R Conditions on process, increase the number of CPU or control number of threads.
- If too many D Conditions on process, switch device.

```shell
$ uptime

# Number of CPU
$ lscpu -e

# R(CPU jobs) & D(I/O jobs) Condition
$ vmstat
```

## dmesg

- If OOME occurs, increase memory.
- If SYN Flooding occurs, check firewall.

```shell
$ dmesg -T

# Check OOME
$ dmesg -TL | grep -i oom

# Check SYN Flooding
$ dmesg -TL | grep -i "syn flooding"
```

## free

- buff/cache is for I/O performance. When buff/cache is high, it means I/O occurs too much. So increase memory.
- swap uses block device area. `swap out, swap in` occurs when memory is insufficient. Additionally, swapped out memories are not erased in swap area which called `swap cached`. When swap area is used, it means more memories are required.

```
1. buff/cache
2. swap
3. oom killer
```

```shell
$ free -m # MB
```

## df

- Check the reference of deleted file with `lsof` command to free up space.

```shell
$ df -h # human readable
$ df -i # inode
$ du -sh ./* # directory usage

# file handle
$ lsof | grep <filename>
```

## top

- Use hot key 1, d for CPU information.
- If us is high in CPU usage, then there is a lot of CPU workloads.
- If ws is high in CPU usage, then there is a lot of I/O workloads.
- If in multicore environment, check if every CPU is running.
- Process can have status like R, D, S, Z.

```shell
$ top

# Check max PID
$ sudo sysctl -a | grep -i pid_max
```

## netstat

- There is LISTEN, ESTABLISHED, TIME_WAIT socket status.
- We should consider why it occurs when there is too much TIME_WAIT socket.
- If there is CLOSE_WAIT socket, then we must find the reason and solve.

```shell
$ netstat -napo
```

## tcpdump

```shell
$ tcp -vvv -nn -A port 80 # and host..
```

## Author

👤 **Kevin Ahn**

- Github: [@seung-seop-ahn](https://github.com/seung-seop-ahn)
