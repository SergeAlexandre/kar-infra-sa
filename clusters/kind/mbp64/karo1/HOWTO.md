
## Network setting

Create a 'kind' network to allow fixed IP
> This is not required for our single node cluster. But will be compatible with multi-nodes

```
docker network rm kind # If kind was already used.
docker network create -d=bridge -o com.docker.network.bridge.enable_ip_masquerade=true -o com.docker.network.driver.mtu=65535  --subnet 172.18.0.0/16 kind

docker network inspect kind
```


## karo1 cluster


```
cat >$(brew --prefix)/etc/dnsmasq.d/karo1 <<EOF
address=/first.pool.karo1.mbp/172.18.150.1 
address=/.ingress.karo1.mbp/172.18.150.1 
address=/ldap.karo1.mbp/172.18.150.2 
address=/last.pool.karo1.mbp/172.18.150.4 
EOF


sudo brew services restart dnsmasq

sudo killall -HUP mDNSResponder

ping ldap.karo1.mbp
```


```
cat >/tmp/karo1-config.yaml <<EOF
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
name: karo1
networking:
  apiServerAddress: "127.0.0.1"
  apiServerPort: 5447
EOF

kind create cluster --config /tmp/karo1-config.yaml

```

```
export GITHUB_USER=SergeAlexandre
export GITHUB_REPO=kar-infra-sa
export GIT_BRANCH=work1
export GITHUB_TOKEN=

flux bootstrap github \
--owner=${GITHUB_USER} \
--repository=${GITHUB_REPO} \
--branch=${GIT_BRANCH} \
--interval 15s \
--personal \
--read-write-key \
--path=clusters/kind/mbp64/karo1/flux


