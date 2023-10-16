
# HPA based on metrics

**Horizontal pod autoscaler based on external metrics, In this example we'll create a POC for two application each of them has it's own hpa resource and then we have to test the scale in and out.**
You can find a lot of GCP exposed metrics [here](https://cloud.google.com/monitoring/api/metrics_gcp#gcp-loadbalancing).



## Prerequisites



`GCP Project`

`GCP ALB` 

`GKE cluster`

Supposed that you have GKE cluster and external application loadbalancer on gcp project where the backends configured to routues its traffic to zonal network endpoint (NEG) which will created by the service kubernetes resource after deploying our application in the below steps.

**Note:** If you don't create a loadbalncer wait till you create your application in the below steps and then create the application loadbalancer to route the trafic to the newly created neg. 


## Installation

1- We need to install stackdriver adabter to fetch the metrics from google api metrics endpoint, to deploy it run the following command:
```bash
kubectl apply -f https://raw.githubusercontent.com/GoogleCloudPlatform/k8s-stackdriver/master/custom-metrics-stackdriver-adapter/deploy/production/adapter_new_resource_model.yaml
```

2- After Installing the stackdriver adabter we need to deploy our applications just run the following:
```bash
kubectl apply -f app-1/deployment.yaml
kubectl apply -f app-2/deployment.yaml
```
- Now you can add the two backend in the loadbalancer to routes its traffic to the zonal network endpoint group (NEG) which is created by the annotions in the svc resources
- Add your two domains in the url maps in the loadbalancer to routes traffic to the two backend you have created.

3- Then we have to create the autoscaler resources:
```bash
# please before apply don't forget to change the labels in the matcheLabels section according to your configuration

kubectl apply -f app-1/hpa.yaml
kubectl apply -f app-2/hpa.yaml
```

4- Check if the created resource are up and running:
```bash
kubectl get hpa
kubectl get deploy
```
5- For Installing ddosify testing tool run the following:
```bash
wget -qO ddosify.deb https://github.com/ddosify/ddosify/releases/latest/download/ddosify_amd64.deb
sudo apt install -y ./ddosify.deb
ddosify --version 
```
6- Test and wait for scaling out the replicas 
```bash
ddosify -t <first_domain> -n 1000  -T 1 -m GET
ddosify -t <second_domain> -n 1000  -T 1 -m GET
```
## Refernces

 - [Medium article](https://medium.com/@matteo.candido/kubernetes-hpa-autoscaling-with-external-metrics-b225289b9206)
 - [GCP Documentation](https://cloud.google.com/kubernetes-engine/docs/tutorials/autoscaling-metrics#pubsub_8)
 - [HPA behavior](https://github.com/kubernetes/enhancements/blob/master/keps/sig-autoscaling/853-configurable-hpa-scale-velocity/README.md)
 

 
![Logo](https://www.giantswarm.io/hubfs/blog_images/hero/vertical-autoscaling-blog-post-1500x700.jpg)