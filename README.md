# Opennet
## **Motivation**

Our product design requires `Kubernetes` to support unique IP address segments, such as public network IPs, and these unique IP address segments can only be bound to specified machines. We searched for many CNI plugins in the community, and did not find a suitable one, so we decided Develop new CNI plugin to meet our application scenarios.



## Introduce

Opennet is an address assignment plugin developed based on Multus-cni



## **Features**

- [x] IP segment management
- [x] IP binding to specific server
- [x] IP cluster level unified management
- [x] Support real-time IP configuration update, effective in real time
- [ ] Support IP allocation query
- [ ] IP statistics



## How to 

1. install [multus-cni] follow it's doc

2. install `opennet`

   a. install daemonset

   ```bash
    kubectl apply -f components/deploy/daemonset-install.yaml
   ```

   create etcd secret, please modify `pem file name`

   ```bash
    kubectl  -n kube-system create secret generic etcd-certs --from-file=/etc/etcd/ssl/etcd.pem --from-file=/etc/etcd/ssl/etcd-key.pem --from-file=/etc/etcd/ssl/etcd-root-ca.pem
   ```
   
   b. configure `CNI` net configuration, save to `cni.yaml`
   
   ```yaml
   apiVersion: "k8s.cni.cncf.io/v1"
   kind: NetworkAttachmentDefinition
   metadata:
     name: opennet-demo
     namespace: kube-system
   spec:
     config: '{
       "cniVersion": "0.3.1",
       "name": "opennet-demo",
       "type": "macvlan",
       "master": "eno1",
       "mode": "vepa",
       "ipam": {
       "type": "opennet",
       "range": [
          {
             "subnet":"192.168.188.0/26",
             "start":"192.168.188.151",
             "end":"192.168.188.181",
             "gw":"192.168.188.190",
             "dev":"80197"
          },
          {
             "subnet":"192.168.188.0/26",
             "start":"192.168.188.182",
             "end":"192.168.188.186",
             "gw":"192.168.188.190",
             "dev":"80366"
          }
        ],
       "dns":{
         "nameserver":[
           "8.8.8.8",
           "8.8.4.4"
         ]
    }
    }
   }'
   
   ```
   
   ```bash
   kubectl -n kube-system apply -f cni.yaml
   ```
   
   
   
   c. configure every device number into `/etc/cni/net.d/opennet-devno`
   
   if every IP segment fit all machine, then can set the `opennet-devno` as the same device number.
   
   d. example, save to `test.yaml`
   
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     annotations:
       v1.multus-cni.io/default-network: kube-system/opennet-demo
     name: test-tools
     labels:
       name: myapp
   spec:
     containers:
       - name: myapp
         image: alpine:latest
         command:
           - /bin/sleep
           - 1d
   
   ```
   
   ```bash
   kubectl apply test.yaml
   ```
   
   

## Contributor

Waiting for additional instructions



## Thanks

Thanks to the `multus-cni` team for developing such an excellent network plugin

## Support

If you need help, please file an issue, we will answer or satisfy in the first time

[multus-cni]: https://github.com/intel/multus-cni