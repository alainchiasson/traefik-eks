# playing weith trafeik and eks

The following is based on : https://traefik.io/blog/eks-clusters-with-traefik-proxy-as-the-ingress-controller/

# Setting up the environment

I hate installing stuff locally, So I creted a docker runtime for this.


# Creating the cluster

This creates the cluster - I changed the region to something closer

    eksctl create cluster --region us-east-1 --instance-types t3.medium --name traefik-eks-test

List nodes:

    kubectl get nodes

Prepare for trafeik

    helm repo add traefik https://helm.traefik.io/traefik
    helm repo update

Install trafeik

    helm install traefik traefik/traefik --create-namespace --namespace=traefik --values=values.yaml

Verify pods:

    kubectl get pods -n traefik

Create access to the dashboard :

    kubectl apply -f middleware.yaml
    kubectl apply -f ingress-route.yaml
    kubectl apply -f secret.yaml

Verify the dashboard service is running: 

    kubectl get service -n traefik

Test using : 

    curl -u test:password http://a2c924a3d56184f39b81cc5b0dcb2758-474757780.eu-west-1.elb.amazonaws.com/dashboard

Deploy whoami: 

    kubectl apply -f whoami.yaml

Create the WhoAmI ingress:

    kubectl apply -f ingress.yaml

Test it:

    curl -u test:password http://a2c924a3d56184f39b81cc5b0dcb2758-474757780.eu-west-1.elb.amazonaws.com/whoami

This will tear everything down at the end: 

    eksctl delete cluster --region us-east-1 traefik-eks-test   

# Setting up tls by hand 

From : https://medium.com/avmconsulting-blog/how-to-secure-applications-on-kubernetes-ssl-tls-certificates-8f7f5751d788

openssl genrsa -out ca.key 2048

openssl req -x509 \
  -new -nodes  \
  -days 365 \
  -key ca.key \
  -out ca.crt \
  -subj "/CN=yourdomain.com"

kubectl create secret tls -n traefic my-tls-secret \
--key ca.key \
--cert ca.crt

kubectl get secrets/my-tls-secret

# More Notes

When reading for the CRD : 

https://doc.traefik.io/traefik/providers/kubernetes-crd/

There is the mention : 

If you want to keep using Traefik Proxy, high availability for Let's Encrypt can be achieved by using a Certificate Controller such as Cert-Manager. When using Cert-Manager to manage certificates, it creates secrets in your namespaces that can be referenced as TLS secrets in your ingress objects. When using the Traefik Kubernetes CRD Provider, unfortunately Cert-Manager cannot yet interface directly with the CRDs. A workaround is to enable the Kubernetes Ingress provider to allow Cert-Manager to create ingress objects to complete the challenges. Please note that this still requires manual intervention to create the certificates through Cert-Manager, but once the certificates are created, Cert-Manager keeps them renewed.

So we should be using the K8s ingress, linked here : 

https://doc.traefik.io/traefik/providers/kubernetes-ingress/

A few more things : 

https://stackoverflow.com/questions/64581882/traefik-v2-2-ingress-on-kubernetes-http-and-https-cannot-co-exist

redirectTo: websecure


Using routes 

https://traefik.io/blog/https-on-kubernetes-using-traefik-proxy/

