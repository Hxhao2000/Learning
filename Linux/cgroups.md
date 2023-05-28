## cgroups

cgroups（全称control groups）是Linux内核提供的一种功能，用于限制、说明和隔离一组进程的资源使用（CPU、内存、磁盘I/O，网络等）。

> 内核版本2.6.24以上支持

cgroups可以通过将进程的PID添加至对应的tasks的文件中，来限制PID对应的进程的资源使用。

- [ ] http://froghui.github.io/cgroup-implementation/

