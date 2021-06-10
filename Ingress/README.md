# Contour Example

## Make sure to have LB. In homelab install MetalLB
```
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.6/manifests/namespace.yaml
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.6/manifests/metallb.yaml
# On first install only
kubectl create secret generic -n metallb-system memberlist --from-literal=secretkey="$(openssl rand -base64 128)"

kubectl apply -f metal-lb-config.yaml
```

## Install Contour 
`kubectl apply -f contour.yaml`

## Simple Ingress 

### Create Nginx and Httpd deployments and Services
```
kubectl create deploy nginx --image=nginx
kubectl expose deploy nginx --port=80
kubectl create deploy httpd --image=httpd
kubectl expose deploy httpd --port=80
```

### Apply simple ingress yaml
`kubectl apply -f simple-ingress.yaml`

Check out http://192.168.2.240.xip.io

## Named virtual host

### Using ingress 
`kubectl apply -f name-virtual-host-ingress.yaml`

Goto http://httpd.192.168.2.240.xip.io/ and http://nginx.192.168.2.240.xip.io/

### Using Httpproxy
Delte ingress based virtual host first
`kubectl delete -f name-virtual-host-ingress.yaml`

Now create httpproxy based virtual host
`kubectl apply -f httpproxy-virtual-host.yaml`

Goto http://httpd.192.168.2.240.xip.io/ and http://nginx.192.168.2.240.xip.io/

## Blue Green Deployment
`kubectl apply -f httpproxy-blue-green.yaml`

Goto http://bg.192.168.2.240.xip.io 

or 

 ```
 while :; do http bg.192.168.2.240.xip.io | grep h1; sleep 1; done
 ```

## Weighted Routing

`kubectl apply -f httpproxy-weighted.yaml`

Goto http://wt.192.168.2.240.xip.io 

or 

 ```
 while true; do http wt.192.168.2.240.xip.io | grep h1; sleep 1; done
 ```


## Rate Limiting

`kubectl apply -f httpproxy-rate-limiting.yaml`

Goto http://rt.192.168.2.240.xip.io 

or 

 ```
 while true; do http rt.192.168.2.240.xip.io; sleep 1; done
 ```

## Team Delegation
```bash
kubectl create ns marketing
kubectl create ns engineering
kubectl create deploy blog --image=ykhadilkar/blog -n marketing
kubectl expose deploy blog --port=80 --target-port=8080 -n marketing
kubectl apply -f httpproxy-multi-team.yaml
```

Test going to http://mt.192.168.2.240.xip.io/blog or using cli
`http mt.192.168.2.240.xip.io/blog`

Now lets test another team trying to create blog at same path

`kubectl apply -f httpproxy-inclusion-test.yaml`

Check status of httpproxy and check out same path.

 ## Clean Up
 ```
 kubectl delete -f httpproxy-rate-limiting.yaml
 kubectl delete -f httpproxy-weighted.yaml
 kubectl delete -f httpproxy-blue-green.yaml
 kubectl delete -f httpproxy-virtual-host.yaml
 kubectl delete -f simple-ingress.yaml
 kubectl delete svc nginx
 kubectl delete deploy nginx
 kubectl delete svc httpd
 kubectl delete deploy httpd
 kubectl delete ns marketing
 kubectl delete ns engineering
 kubectl delete -f contour.yaml
 ```