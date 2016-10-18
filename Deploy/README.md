For HPE-cloud kubernetes deployment:
Following the instructions:

[Deploy Harbor on Kubernetes](../docs/kubernetes_deployment.md)

And follow the changes below:

1. Change hostname in harbor.cfg as harbor.cfg.hpecloud shows.
2. After deploying the service, change hostname in nginx config file:
  ```
  config/nginx/nginx.conf
  config/nginx/nginx.https.conf
  ```
  replace 'upstream registry/upstream ui', the server ip with cluster ip in kubernetes.

3. Change registry config file:
  ```
  kubernetes/dockerfiles/registry-config.yml
  ```
  1. Change 'auth: token: realm' to hostname same as hostname in harbor.cfg.hpecloud config file.
  2. Change 'notifications: endpoints: url' to according hostname (with port specified).
  
  >This change cannot use the cluster ip in kubernetes, because it is accessed from outside host.

4. After changing proxy and registry, rebuild these images and push them to repository. Then create pod with these images.
  ```
  # create service in the kubernetes cluster first
  kubectl create -f kubernetes/mysql-svc.yaml -f kubernetes/proxy-svc.yaml -f kubernetes/registry-svc.yaml -f kubernetes/ui-svc.yaml
  ...update proxy and registry images, docker build and docker push...
  # create and bind the pod to the service
  kubectl create -f kubernetes/mysql-rc.yaml -f kubernetes/proxy-rc.yaml -f kubernetes/registry-rc.yaml -f kubernetes/ui-rc.yaml
  ```
