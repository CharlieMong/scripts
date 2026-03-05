to dump ftp traffic
```
tcpdump -i tun0 -nqtX host $IP and port 21 and 'tcp[13] & 8!=0'
```
