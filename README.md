----docker-compose.yaml----
version: '3'
services:
  webapp:
    image: library/nginx:latest
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8080:80"
    volumes:
      - ./index.html:/usr/share/nginx/html/index.html

----Dockerfile----
FROM nginx
WORKDIR /usr/share/nginx/html
COPY index.html /usr/share/nginx/html
EXPOSE 80

----index.html----
Hello !

----dockerhub----
docker tag nginx slashik/nginx
docker push slashik/nginx

rebuild po zmene image:
docker build -t slashik/nginx .

----configmap.yaml----
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-conf
data:
  nginx.conf: |
    user nginx; 
    worker_processes  1;
    
    events {
      worker_connections  1024;
    }

    http {
      server { 
        listen       80;
        server_name  localhost;

        location / {
          index index.html;
          root /usr/share/nginx/html;
        }
      }
    }


----svc.yaml----
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30159

----deployment.yaml----
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: slashik/nginx:latest
        ports:
        - containerPort: 80
        volumeMounts:
          - name: nginx-conf
            mountPath: /etc/nginx/nginx.conf 
            subPath: nginx.conf
      volumes:
        - name: nginx-conf 
          configMap:
            name: nginx-conf 
            items:
              - key: nginx.conf 
                path: nginx.conf 




