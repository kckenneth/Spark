|Title |  Spark Installation |
|-----------|----------------------------------|
|Author | Kenneth Chen |
|Utility | IBM, Softlayer, Spark |
|Date | 10/08/2018 |

# Apache Spark Setup

# Setup 3 virtual servers in the cloud. 
```
$ slcli vs create --datacenter=sjc01 --hostname=spark1 --domain=mids.com --billing=hourly --cpu=2 --memory=4096 --disk=100 --os=CENTOS_LATEST_64
$ slcli vs create --datacenter=sjc01 --hostname=spark2 --domain=mids.com --billing=hourly --cpu=2 --memory=4096 --disk=100 --os=CENTOS_LATEST_64
$ slcli vs create --datacenter=sjc01 --hostname=spark3 --domain=mids.com --billing=hourly --cpu=2 --memory=4096 --disk=100 --os=CENTOS_LATEST_64
```

# Setup passwordless ssh
The idea is to `ssh` without password between nodes. `spark1` must be able to `ssh spark1`, `ssh spark2` and `ssh spark3`. You already know by now that to ssh using the name requires you to set up at `/etc/hosts`. Without password requires you to setup `ssh-keygen` generation. 

```
# ssh-keygen
# for i in spark1 spark2 spark3; do ssh-copy-id $i; done
```
`ssh-copy-id` will copy `id_rsa.pub` key to `authorized_keys` file (will be created) in other nodes. So when it tries to establish the connection, it will first ask the password of the node it's sshing into. 

