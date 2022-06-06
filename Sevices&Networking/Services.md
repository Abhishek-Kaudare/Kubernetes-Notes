- [**Service**](#service)
  - [**NodePort**](#nodeport)
  - [**ClusterIp**](#clusterip)
- [**References**](#references)

# **Service**
```bash
kubectl expose  (-f FILENAME | TYPE NAME) [--port=port] [--protocol=TCP|UDP] [--target-port=number-or-name] [--name=name] [--external-ip=external-ip-of-service] [--type=type]

kubectl expose deployment nginx --port=443 --target-port=8443 --name=nginx-https --type=NodePort --name=serviceName
``` 
## **NodePort**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: NodePort
  ports:
    - port: 80 # Required
      targetPort: 80
      nodePort: 30008
  selector:
    app: myapp
    type: frontend
```

## **ClusterIp**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  type: ClusterIP
  ports:
    - port: 80 # Required
      targetPort: 80
  selector:
    app: myapp
    type: backend
```

# **References**

1. [K8 Service Docs](https://kubernetes.io/docs/concepts/services-networking/service/)

