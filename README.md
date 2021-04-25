# Java e Kubernetes

Ferramentas necessárias:

* Java 14 ou superior
* Docker
* Make (opcional)
* Minikube

## Parte 1 - base do app:

### Buildar e rodar a aplicação:

Spring boot e banco de dados mysql funcionando no container docker



**Buildar a aplicação**

```bash
cd java-kubernetes
mvn clean install
```



**Rodar-bd : parar-bd rm-bd** (container docker mysql)

```bash
docker run --name mysql57 -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -e MYSQL_USER=java -e MYSQL_PASSWORD=1234 -e MYSQL_DATABASE=k8s_java -d mysql/mysql-server:5.7
```



**Rodar-app : parar-app rm-app**(container docker myapp)

```bash
docker run --name mysql57 -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -e MYSQL_USER=java -e MYSQL_PASSWORD=1234 -e MYSQL_DATABASE=k8s_java -d mysql/mysql-server:5.7
```



**Rodar a aplicação**

```bash
java --enable-preview -jar target/java-kubernetes.jar
```



**Checar as portas**

http://localhost:8080/app/users

http://localhost:8080/app/hello



## Parte 2 - App no Docker:

Criar o arquivo Dockerfile:

```yaml
FROM openjdk:15-alpine
RUN mkdir /usr/myapp
COPY target/java-kubernetes.jar /usr/myapp/app.jar
WORKDIR /usr/myapp
EXPOSE 8080
ENTRYPOINT [ "sh", "-c", "java --enable-preview $JAVA_OPTS -jar app.jar" ]
```

Rodar a  aplicação and docker image**

```bash
make build
```

Criar e rodar o banco de dados

```bash
make run-db
```

Criar e rodar a aplicação

```bash
make run-app
```

**Checar as portas**

http://localhost:8080/app/users

http://localhost:8080/app/hello

Parar todos containers:

`
docker stop mysql57 myapp
`

## Parte 3 - App no Kubernetes:

### Inicializar o minikube

`
make k-setup
`
 Inicia o minikube e cria o namespace dev-to

### Checar o IP

`
minikube -p dev.to ip
`

### Painel de controle Minikube

`
minikube -p dev.to dashboard
`

### Implatação do banco de dados 

crie a implatação e o serviço do banco de dados mysql

`
make k-deploy-db
`

`
kubectl get pods -n dev-to
`

OU

`
watch k get pods -n dev-to
`


`
kubectl logs -n dev-to -f <pod_name>
`

`
kubectl port-forward -n dev-to <pod_name> 3306:3306
`

## Construir a aplicação 

Contruir app

`
make k-build-app
` 

crie a imagem docker dentro da maquina minikube:

`
make k-build-image
`

OU

`
make k-cache-image
`  

Crie a implantação do app e serviço:

`
make k-deploy-app
` 

**Check**

`
kubectl get services -n dev-to
`

Para acessar o app:

`
minikube -p dev.to service -n dev-to myapp --url
`

Ex:

http://172.17.0.3:32594/app/users
http://172.17.0.3:32594/app/hello

## Check pods

`
kubectl get pods -n dev-to
`

`
kubectl -n dev-to logs myapp-6ccb69fcbc-rqkpx
`

## Map to dev.local

get minikube IP
`
minikube -p dev.to ip
` 

Edit `hosts` 

`
sudo vim /etc/hosts
`

Replicas
`
kubectl get rs -n dev-to
`

Get and Delete pod
`
kubectl get pods -n dev-to
`

`
kubectl delete pod -n dev-to myapp-f6774f497-82w4r
`

Scale
`
kubectl -n dev-to scale deployment/myapp --replicas=2
`

Test replicas
`
while true
do curl "http://dev.local/app/hello"
echo
sleep 2
done
`
Test replicas with wait

`
while true
do curl "http://dev.local/app/wait"
echo
done
`

## Check app url

`minikube -p dev.to service -n dev-to myapp --url`

Change your IP and PORT as you need it

`
curl -X GET http://dev.local/app/users
`

Add new User
`
curl --location --request POST 'http://dev.local/app/users' \
--header 'Content-Type: application/json' \
--data-raw '{
    "name": "new user",
    "birthDate": "2010-10-01"
}'
`

## Part 4 - debug app:

add   JAVA_OPTS: "-agentlib:jdwp=transport=dt_socket,address=*:5005,server=y,suspend=n"

change CMD to ENTRYPOINT on Dockerfile

`
kubectl get pods -n=dev-to
`

`
kubectl port-forward -n=dev-to <pod_name> 5005:5005
`

## KubeNs e Stern

`
kubens dev-to
`

`
stern myapp
` 

## Inicializar tudo

`make k:all`

## Useful comandos
##List profiles
minikube profile list

kubectl top node

kubectl top pod <nome_do_pod>
