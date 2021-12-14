# EKS

## Usando eksctl

### Cluster con EC2 estándar

Para montar el cluster hayque generar el keypair primero ejecutando el comando:

```sh
aws ec2 create-key-pair --region eu-west-1 --key-name form0tocloud-resilient
```

Después lanzamos la creación del cluster usando el comando:

```sh
eksctl create cluster --name from0tocloud-resilient --region eu-west-1 --with-oidc --ssh-access --ssh-public-key form0tocloud-resilient
```

Para comprobar que todo es correcto:

```sh
kubectl get svc
kubectl get nodes -o wide
kubectl get pods --all-namespaces -o wide
```

Habilitamos el logging con CloudWatch

```sh
eksctl utils update-cluster-logging --enable-types all --cluster from0tocloud-resilient --approve
```

```sh
kubectl create namespace from0tocloud-backend
```

```sh
kubectl apply -f eureka-server/deployment.yml,api-gateway/deployment.yml,category-service/deployment.yml,item-service/deployment.yml,user-service/deployment.yml,order-service/deployment.yml
```

```sh
kubectl get all -n from0tocloud-backend 
```

Desplegamos prometheus:

```sh
kubectl create namespace prometheus

helm install prometheus prometheus-community/prometheus \
    --namespace prometheus \
    --set alertmanager.persistentVolume.storageClass="gp2" \
    --set server.persistentVolume.storageClass="gp2"
```

Comprobamos que va bien 

```sh
kubectl get all -n prometheus
```

kubectl port-forward -n prometheus deploy/prometheus-server 8080:9090

Desplegamos Grafana:

```sh
kubectl create namespace grafana

helm install grafana grafana/grafana \
    --namespace grafana \
    --set persistence.storageClassName="gp2" \
    --set persistence.enabled=true \
    --set adminPassword='EKS!sAWSome' \
    --values grafana.yml \
    --set service.type=LoadBalancer
```

Comprobamos que está todo en orden:

```sh
kubectl get all -n grafana
```

Accedemos a la url 

```sh
export ELB=$(kubectl get svc -n grafana grafana -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')

echo "http://$ELB"
```

Obtenemos la pass, el usuario es admin:

```sh
kubectl get secret --namespace grafana grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

En graphana importamos los dashboards 3119 seleccionando prometheus como origen de datos y el 6417 

```sh
kubectl logs -n kube-system deployment.apps/aws-load-balancer-controller
```

```sh
kubectl delete -f eureka-server/deployment.yml,api-gateway/deployment.yml,category-service/deployment.yml,item-service/deployment.yml,user-service/deployment.yml,order-service/deployment.yml
```

Para eliminar el cluster hay que ejecutar:

```sh
eksctl delete cluster --name from0tocloud-resilient --region eu-west-1
```

Por si se van las credenciales:

```sh
 kubectl config use-context jserrano@from0tocloud.eu-west-1.eksctl.io
 ```

 Para construir los docker y publicarlos en el AWS Registry hay que ejecutar el comando buildAndPush.sh de cada proyecto.