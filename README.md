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
time DOCKER_BUILDKIT=0 docker build -t sahil1999/nodejsapp .
```

### Build image using nerdctl(containerd)
```
time nerdctl build -t sahil1999/nodejsapp .
```
