uname -r


sudo yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

docker rmi $(docker images -aq)


ip addr

