# wormholes.
# nano ruesandora.sh yapıp aşağıdaki kodu yapıştırın. Sonrasında bash ruesandora.sh yapıp çalıştırın
#!/bin/bash
#check docker cmd
which docker >/dev/null 2>&1
if  [ $? -ne 0 ] ; then
     echo "docker not found, please install first!"
     echo "ubuntu:sudo apt install docker.io -y"
     echo "centos:yum install  -y docker-ce "
     echo "fedora:sudo dnf  install -y docker-ce"
     exit
fi
#check docker service
docker ps > /dev/null 2>&1
if [ $? -ne 0 ] ; then

     echo "docker service is not running! you can use command start it:"
     echo "sudo service docker start"
     exit
fi

docker stop wormholes > /dev/null 2>&1
docker rm wormholes > /dev/null 2>&1
docker rmi wormholestech/wormholes:v1 > /dev/null 2>&1

if [ -d /wm/.wormholes/keystore ]; then
   read -p "Whether to clear the Wormholes blockchain data history, if yes, press the “y” button, and if not, click “enter.”：" xyz
   if [ "$xyz" = 'y' ]; then
         cp /wm/.wormholes/wormholes/nodekey /wm/nodekey
         rm -rf /wm/.wormholes
         mkdir -p /wm/.wormholes/wormholes
         mv /wm/nodekey /wm/.wormholes/wormholes/
   else
         echo "Do not clear"
   fi
else
   read -p "Please import your private key：" ky
fi

if [ -n "$ky" ]; then
     if [ ${#ky} -eq 64 ];then
             mkdir -p /wm/.wormholes/wormholes
             echo $ky > /wm/.wormholes/wormholes/nodekey
     elif [ ${#ky} -eq 66 ] && ([ ${ky:0:2} == "0x" ] || [ ${ky:0:2} == "0X" ]);then
             mkdir -p /wm/.wormholes/wormholes
             echo ${ky:2:64} > /wm/.wormholes/wormholes/nodekey
     else
             echo "the nodekey format is not correct"
             exit -1
     fi
fi

docker run -id -p 30303:30303 -p 8545:8545 -v /wm/.wormholes:/wm/.wormholes --name wormholes wormholestech/wormholes:v1

echo "Your private key is:"
sleep 6
docker exec -it wormholes /usr/bin/cat /wm/.wormholes/wormholes/nodekey
