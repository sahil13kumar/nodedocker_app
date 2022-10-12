# nodedocker_app
Dockerize Simple node.js Application
### clone the repository and proceed with the steps 
```
git clone https://github.com/sahil13kumar/nodedocker_app

cp ./nodedocker_app/dockerfile Dockerfile

cat <<EOF > .dockerignore
node_modules
npm-debug.log
EOF
```


### Install Buildkitd for nerdctl 
```
tar -xvzf buildkit-v0.10.3.linux-amd64.tar.gz -C /usr/local/bin/

sudo cat <<EOF > /etc/systemd/system/buildkitd.service
[Unit]
Description=BuildKit Daemon

[Service]
User=root
ExecStart=/usr/local/bin/buildkitd --oci-worker=false --containerd-worker=true --containerd-worker-addr=/run/containerd/containerd.sock

[Install]
WantedBy=default.target
EOF

sudo systemctl daemon-reload
sudo systemctl start buildkitd
#sudo systemctl status buildkitd


if [ ! -d /etc/buildkit ]
then
        mkdir /etc/buildkit
fi

sudo cat << EOF > /etc/buildkit/buildkitd.toml
[worker.oci]
  enabled = false

[worker.containerd]
  enabled = true
  # namespace should be "k8s.io" for Kubernetes (including Rancher Desktop)
  namespace = "k8s.io"
EOF

```


### Build image using Docker 
```
ubuntu $ time DOCKER_BUILDKIT=1 docker build -t sahil1999/nodejsapp .
[+] Building 10.4s (10/10) FINISHED                                                                                                              
 => [internal] load build definition from Dockerfile                                                                                        0.1s
 => => transferring dockerfile: 213B                                                                                                        0.0s
 => [internal] load .dockerignore                                                                                                           0.0s
 => => transferring context: 70B                                                                                                            0.0s
 => [internal] load metadata for docker.io/library/node:14                                                                                  0.0s
 => [internal] load build context                                                                                                           0.8s
 => => transferring context: 8.42MB                                                                                                         0.8s
 => [1/5] FROM docker.io/library/node:14                                                                                                    0.3s
 => [2/5] WORKDIR /usr/src/app                                                                                                              0.1s
 => [3/5] COPY ./nodedocker_app/package*.json /usr/src/app/                                                                                 0.1s
 => [4/5] RUN npm install                                                                                                                   7.5s
 => [5/5] COPY ./nodedocker_app/ /usr/src/app/                                                                                              0.9s
 => exporting to image                                                                                                                      0.8s 
 => => exporting layers                                                                                                                     0.7s 
 => => writing image sha256:f1ee70c2c5a32b1258d486fbe01dd4108db37698d80d66ea3b2d0bb25e11160c                                                0.0s 
 => => naming to docker.io/sahil1999/nodejsapp                                                                                              0.0s 
                                                                                                                                                 
real    0m10.692s
user    0m0.172s
sys     0m0.079s
```


### Build image using nerdctl(containerd)
```
ubuntu $ time nerdctl build -t sahil1999/nodejsapp .
[+] Building 85.5s (10/10) FINISHED                                                                                                              
 => [internal] load build definition from Dockerfile                                                                                        0.1s
 => => transferring dockerfile: 213B                                                                                                        0.0s
 => [internal] load .dockerignore                                                                                                           0.0s
 => => transferring context: 70B                                                                                                            0.0s
 => [internal] load metadata for docker.io/library/node:14                                                                                  2.0s
 => CACHED [1/5] FROM docker.io/library/node:14@sha256:d0634e5cde35cb312571cd007ae2270295bf2000ec489402325953bcee196ae2                     0.0s
 => => resolve docker.io/library/node:14@sha256:d0634e5cde35cb312571cd007ae2270295bf2000ec489402325953bcee196ae2                            0.1s
 => [internal] load build context                                                                                                           0.6s
 => => transferring context: 138.99kB                                                                                                       0.2s
 => [2/5] WORKDIR /usr/src/app                                                                                                              0.2s
 => [3/5] COPY ./nodedocker_app/package*.json /usr/src/app/                                                                                 0.1s
 => [4/5] RUN npm install                                                                                                                   6.5s
 => [5/5] COPY ./nodedocker_app/ /usr/src/app/                                                                                              1.2s 
 => exporting to oci image format                                                                                                          53.3s 
 => => exporting layers                                                                                                                    26.8s 
 => => exporting manifest sha256:1b553ee8814885b7a4a6a823b71dd252e00f0628ef7b5f18d1b5196416f125f4                                           0.0s 
 => => exporting config sha256:7e20b840c2ac3ac736207479460606acdf73868f7736ccb749b372e5dc045418                                             0.0s 
 => => sending tarball                                                                                                                     26.4s 
unpacking docker.io/sahil1999/nodejsapp:latest (sha256:1b553ee8814885b7a4a6a823b71dd252e00f0628ef7b5f18d1b5196416f125f4)...done

real    1m26.893s
user    0m7.143s
sys     0m1.188s
```


### Installed version of nerdctl and Docker where I tested
```
ubuntu $ nerdctl version
Client:
 Version:       v0.17.0
 Git commit:    9ec475aeb69209b9b2a3811aee0923e2eafd1644

Server:
 containerd:
  Version:      1.5.9-0ubuntu1~20.04.4
  GitCommit:
ubuntu $ 
ubuntu $ docker --version
Docker version 20.10.12, build 20.10.12-0ubuntu2~20.04.1
```
