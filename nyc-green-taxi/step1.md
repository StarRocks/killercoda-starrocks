
First lets download and run the StarRocks QuickStart Container
Run `docker run -p 9030:9030 -p 8030:8030 -p 8040:8040 -itd --name=starrocks starrocks/allin1-ubuntu`{{exec}}

To make sure it works, let's also download and run the StarRocks tool-box.  The StarRocks tool-box is a container with a bunch of useful binaries like the mysql client.  See more about the tool-box at https://hub.docker.com/r/atwong/starrocks-tool-box
Run `docker run -itd --name=toolbox atwong/starrocks-tool-box`{{exec}}

Let's now test the StarRocks instance
Run `docker exec -it toolbox /bin/bash`{{exec}}
Run `mysql -P9030 -h{{TRAFFIC_HOST1_80}} -uroot --prompt="StarRocks >`{{exec}}
Run `select current_version();`{{exec}}




