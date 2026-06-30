# Linux Cheatsheet

> Quick reference — see [Linux Fundamentals](../Linux/DevOps-01-Linux-Fundamentals.md) | [Troubleshooting](../../Troubleshooting/Linux/01-Linux-Troubleshooting.md)

## Files & directories

```bash
ls -lah          # list all
find /path -name "*.log" -mtime +7
du -sh /*        # disk per top-level dir
df -h            # filesystem usage
chmod 755 file   # rwxr-xr-x
chown user:group file
```

## Processes

```bash
ps aux | grep java
top / htop
kill -15 PID     # SIGTERM graceful
kill -9 PID      # SIGKILL force
pgrep -f myapp
```

## System resources

```bash
free -h
vmstat 1 5
iostat -x 1 5
uptime
dmesg | tail -20   # kernel / OOM messages
```

## Networking

```bash
ss -tlnp          # listening ports
curl -v URL
dig hostname
traceroute host
tcpdump -i eth0 port 443
```

## Logs

```bash
journalctl -u nginx -f
journalctl --since "1 hour ago"
tail -f /var/log/syslog
grep -r "error" /var/log/myapp/
```

## Text

```bash
grep -i error file.log
awk '{print $1}' file
sed 's/old/new/g' file
sort | uniq -c | sort -rn
```

## Archive

```bash
tar -czvf archive.tar.gz /path
tar -xzvf archive.tar.gz
zip -r archive.zip dir/
```

## Exit codes

| Code | Meaning |
|------|---------|
| 137 | OOMKilled / SIGKILL |
| 143 | SIGTERM |
| 126 | Permission denied (not executable) |
