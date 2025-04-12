Instructions to deploy **Jenkins** on Azure Kubernetes Service using the default **App Routing** add on
  1. Deploy AKS cluster through Azure portal.
  2. Create a namespace. ` kubectl create ns jenkins `
  3. Create a tls secret named ` cert-tls ` which has the domain's certificate & private key by running below command. The domain's .crt & .key file should already be present.

     ```
     kubectl -n jenkins create secret tls cert-tls --cert=domain_name.crt --key=domain_name.key
     ```
  4. Run the command ` kubectl -n app-routing-system get svc nginx ` to confirm if a **LoadBalancer** IP has been provisioned.

     ```
     pushkar [ ~ ]$ kubectl -n app-routing-system get svc nginx
     NAME    TYPE           CLUSTER-IP    EXTERNAL-IP    PORT(S)                      AGE
     nginx   LoadBalancer   10.0.58.180   20.174.45.64   80:32767/TCP,443:30598/TCP   110s
     ```
  5. Deploy **PVC** & **Storage Class** using below command.

     ```
     kubectl -n jenkins apply -f pvc.yml -f storage-class.yml
     ```
  6. Deploy the `jenkins` deployment & service using the `kubectl` command

     ```
     kubectl -n jenkins apply -f jenkins-dep.yml -f jenkins-svc.yml
     ```
  7. Put the FQDN for which the secret has been created in ` app-routing-ingress.yml ` file and then run the command ` kubectl -n jenkins apply -f app-routing-ingress.yml `
  8. Run `kubectl -n jenkins get ingress` to retrieve the IP. This may take some time to match with the **LoadBalancer** IP above. Point the domain name in your registrar to the IP address.
  9. Access the app using `https://your_domain_name`.

-----------------------------

Instructions to deploy **Jenkins** on Azure Kubernetes Service using your own nginx ingress
  1. Deploy AKS cluster through Azure portal.
  2. Create a namespace. ` kubectl create ns speedtest `
  3. Create a tls secret named ` cert-tls ` which has the domain's certificate & private key by running below command. The domain's .crt & .key file should already be present.

     ```
     kubectl -n jenkins create secret tls cert-tls --cert=domain_name.crt --key=domain_name.key
     ```
  4. Deploy Nginx Ingress Controller by running below commands.

     ```
     helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
     helm repo update
     ```
     ```
     helm install nginx-ingress ingress-nginx/ingress-nginx \
     --set controller.service.externalTrafficPolicy=Local \
     --set controller.service.annotations."service\.beta\.kubernetes\.io/azure-load-balancer-health-probe-request-path"="/" \
     --set controller.service.enableHttps=true
     ```
  5. Run the command ` kubectl get svc nginx-ingress-ingress-nginx-controller ` to confirm if a **LoadBalancer** IP has been provisioned.

     ```
     pushkar [ ~ ]$ kubectl get svc nginx-ingress-ingress-nginx-controller
     NAME                                     TYPE           CLUSTER-IP    EXTERNAL-IP    PORT(S)                      AGE
     nginx-ingress-ingress-nginx-controller   LoadBalancer   10.0.58.180   20.174.45.64   80:32767/TCP,443:30598/TCP   110s
     ```  
  6. Deploy **PVC** & **Storage Class** using below command.

     ```
     kubectl -n jenkins apply -f pvc.yml -f storage-class.yml
     ```
  7. Deploy the `jenkins` deployment & service using the `kubectl` command

     ```
     kubectl -n jenkins apply -f speedtest-dep.yml -f speedtest-svc.yml
     ```
  8. Put the FQDN for which the secret has been created in ` nginx-ingress.yml ` file and then run the command ` kubectl -n jenkins apply -f nginx-ingress.yml `
  9. Run `kubectl -n jenkins get ingress` to retrieve the IP. This may take some time to match with the **LoadBalancer** IP above. Point the domain name in your registrar to the IP address.
  10. Access the app using `https://your_domain_name`.

-----------------------------

**Helm**
To install this app using Helm using the default **App Routing** add on, perform below steps
  1. Create a namespace. ` kubectl create ns jenkins `
  2. Create a tls secret named ` cert-tls ` which has the domain's certificate & private key by running below command. The domain's .crt & .key file should already be present.

     ```
     kubectl -n jenkins create secret tls cert-tls --cert=domain_name.crt --key=domain_name.key
     ```
  3. Run the command ` kubectl -n app-routing-system get svc nginx ` to confirm if a **LoadBalancer** IP has been provisioned.

     ```
     pushkar [ ~ ]$ kubectl -n app-routing-system get svc nginx
     NAME    TYPE           CLUSTER-IP    EXTERNAL-IP    PORT(S)                      AGE
     nginx   LoadBalancer   10.0.58.180   20.174.45.64   80:32767/TCP,443:30598/TCP   110s
     ```
  4. Run the command to install **Jenkins**

     ```
     helm install jenkins ./helm --namespace jenkins --set ingressClassName="web-app-routing" --set domain_name=your_preferred_fqdn
     ```
  5. Run `kubectl -n jenkins get ingress` to retrieve the IP. This may take some time to match with the **LoadBalancer** IP above. Point the domain name in your registrar to the IP address.
  6. Access the app using `https://your_domain_name`.
  7. Uninstall the app using `helm uninstall jenkins --namespace jenkins`.

-----------------------------

**Helm**
To install this app using Helm using your own nginx ingress, perform below steps
  1. Create a namespace. ` kubectl create ns jenkins `
  2. Create a tls secret named ` cert-tls ` which has the domain's certificate & private key by running below command. The domain's .crt & .key file should already be present.

     ```
     kubectl -n jenkins create secret tls cert-tls --cert=domain_name.crt --key=domain_name.key
     ```
  3. Deploy Nginx Ingress Controller by running below commands.
     
     ```
     helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
     helm repo update
     ```
     ```
     helm install nginx-ingress ingress-nginx/ingress-nginx \
     --set controller.service.externalTrafficPolicy=Local \
     --set controller.service.annotations."service\.beta\.kubernetes\.io/azure-load-balancer-health-probe-request-path"="/" \
     --set controller.service.enableHttps=true
     ```
  4. Run the command ` kubectl get svc nginx-ingress-ingress-nginx-controller ` to confirm if a **LoadBalancer** IP has been provisioned.

     ```
     pushkar [ ~ ]$ kubectl get svc nginx-ingress-ingress-nginx-controller
     NAME                                     TYPE           CLUSTER-IP    EXTERNAL-IP    PORT(S)                      AGE
     nginx-ingress-ingress-nginx-controller   LoadBalancer   10.0.58.180   20.174.45.64   80:32767/TCP,443:30598/TCP   110s
     ```
  5. Run the command to install **Jenkins**

     ```
     helm install jenkins ./helm --namespace jenkins --set ingressClassName="nginx" --set domain_name=your_preferred_fqdn
     ```
  6. Run `kubectl -n jenkins get ingress` to retrieve the IP. This may take some time to match with the **LoadBalancer** IP above. Point the domain name in your registrar to the IP address.
  7. Access the app using `https://your_domain_name`.
  8. Uninstall the app using `helm uninstall jenkins --namespace jenkins`.