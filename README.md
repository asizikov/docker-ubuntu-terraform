# Building Docker image with installed Terraform.

Run the latest `ubuntu` container:

```bash
 docker container run -it --name ubuntu-latest ubuntu:latest /bin/bash
```

then we need to download, unzip and install the latest version of terraform. Currently, the latest one is 0.13.3. 
Make sure to check it here:

https://www.terraform.io/downloads.html



```bash
apt-get update
apt-get install wget -y
apt-get install unzip -y
wget https://releases.hashicorp.com/terraform/0.13.3/terraform_0.13.3_linux_amd64.zip
cp terraform /usr/local/bin
terraform --version
exit
 ```

Now we have installed terraform (as well as unzip and wget) and shut the container down.

`docker ps -a` should confirm that

```bash
docker ps -a                                                                                                                        
CONTAINER ID        IMAGE               STATUS                                NAMES
63016bcefec6        ubuntu:latest       Exited (0) About a minute ago         ubuntu-latest
```

`docker container diff ubuntu-latest` will print us all the changes applied to this container. If you want to read it though make yourself a cup of tea, it'll take a while.

We probably don't need all the changes, so let's do a little clean up:

`docker start ubuntu-latest -a` to start the container and attach to it.

```
 rm terraform
 rm terraform_0.13.3_linux_amd64.zip
 apt-get remove unzip -y
 apt-get remove wget -y
 exit
```

this container is still going to have some changes due to the `apt-get update` call, but I'm ok with that.

What's important is that we have (A)dded the terrafrom binary to its onwn place:

```
A /usr/local/bin/terraform
```

So, it's time to commit the image:

```bash
docker container commit -a "@asizikov" -m "installed terraform 0.13.3" \
ubuntu-latest ubuntu-terraform
```

`docker images` should list all the images installed on your computer, and `ubuntu-terraform` should be among them.

Ok, it's time to tag and publish, I guess.

```bash
docker image tag ubuntu-terraform:latest asizikov/ubuntu-terraform:0.13.3
docker push asizikov/ubuntu-terraform:0.13.3
```

It's 

```
docker save asizikov/ubuntu-terraform:0.13.3 > ubuntu-terraform.tar
mkdir unpacked
tar -xf ubuntu-terraform.tar -C unpacked
cd unpacked
tree
```

this will produce a nice tree structure with all our layers:

```
├── 57f79de53e5eeff9c780866667a034a2b3959dc115b18f5cea5aec32b9677239.json
├── 70b23ca3cba25ea431e904e2d4a90f144f52b907da5afee7e2e1a6004b9e3a35
│   ├── VERSION
│   ├── json
│   └── layer.tar
├── 7e08bf461d394ea068ee4836367cc46d4e1e47d9e848043c4b63a61b8c0e714a
│   ├── VERSION
│   ├── json
│   └── layer.tar
├── 8c91f14beadb4dcf4e11153514476e705521a0fbab80c27a1b582241a4f91b75
│   ├── VERSION
│   ├── json
│   └── layer.tar
├── 8e6ad6de8875a0658e8e86659248cdaa0d9de73b2132d94f685eb044115e1114
│   ├── VERSION
│   ├── json
│   └── layer.tar
├── manifest.json
└── repositories
```

let's find the latest layer:

```bash
$ cat repositories                                                          
{"asizikov/ubuntu-terraform":{"0.13.3":"8e6ad6de8875a0658e8e86659248cdaa0d9de73b2132d94f685eb044115e1114"}}
$ cd 8e6ad6de8875a0658e8e86659248cdaa0d9de73b2132d94f685eb044115e1114
$ mkdir unpack_layer
$ tar -xf layer.tar -C unpack_layer
$ cd unpack_layer 
$ tree
```

This will print the large tree with all the files which belong to the layer, and we can scroll all the way down to find our `terraform` executable. Lovely.

Make sure to take a look there and think how can we make this image smaller.

```
.
├── usr
│   ├── bin
│   ├── local
│   │   ├── bin
│   │   │   └── terraform
```

PS:

Don't forget to clean things up:

```
rm -rf unpacked
rm -f ubuntu-terraform.tar
docker container rm ubuntu-latest
docker image rm asizikov/ubuntu-terraform:0.13.3
```

That's all folks! 
Happy dockering.