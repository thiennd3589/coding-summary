# SSH by jump Server

## SSH by pem
- Add key: ```ssh-add key1.pem key2.pem```
- SSH: ```ssh -J username@server1 username@server2```

### SCP by pem
- Add key: ```ssh-add key1.pem key2.pem```
- SSH: ```scp -J username@server1 username@server2://path local_path```