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
 10. To upgrade the application, first scale the deployment to 0, change the image and then apply the new deployment again.
     
     ```
     kubectl -n jenkins scale deployment jenkins --replicas=0
     kubectl -n jenkins apply -f jenkins-dep.yml
     ```

-----------------------------

Instructions to deploy **Jenkins** on Azure Kubernetes Service using your own nginx ingress
  1. Deploy AKS cluster through Azure portal.
  2. Create a namespace. ` kubectl create ns jenkins `
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
     kubectl -n jenkins apply -f jenkins-dep.yml -f jenkins-svc.yml
     ```
  8. Put the FQDN for which the secret has been created in ` nginx-ingress.yml ` file and then run the command ` kubectl -n jenkins apply -f nginx-ingress.yml `
  9. Run `kubectl -n jenkins get ingress` to retrieve the IP. This may take some time to match with the **LoadBalancer** IP above. Point the domain name in your registrar to the IP address.
 10. Access the app using `https://your_domain_name`.
 11. To upgrade the application, first scale the deployment to 0, change the image and then apply the new deployment again.
     
     ```
     kubectl -n jenkins scale deployment jenkins --replicas=0
     kubectl -n jenkins apply -f jenkins-dep.yml
     ```

-----------------------------

**Helm**
To install this app using Helm using the default **App Routing** add on, perform below steps
  1. Create a namespace. ` kubectl create ns jenkins `
  2. Run the command ` kubectl -n app-routing-system get svc nginx ` to confirm if a **LoadBalancer** IP has been provisioned.

     ```
     pushkar [ ~ ]$ kubectl -n app-routing-system get svc nginx
     NAME    TYPE           CLUSTER-IP    EXTERNAL-IP    PORT(S)                      AGE
     nginx   LoadBalancer   10.0.58.180   20.174.45.64   80:32767/TCP,443:30598/TCP   110s
     ```
  3. Run the command to install **Jenkins**

     ```
     helm install jenkins ./helm --namespace jenkins --set ingressClassName="web-app-routing" --set domain_name=your_preferred_fqdn --set-file tlsSecret.cert=domain_name.crt --set-file tlsSecret.key=domain_name.key
     ```
  4. Run `kubectl -n jenkins get ingress` to retrieve the IP. This may take some time to match with the **LoadBalancer** IP above. Point the domain name in your registrar to the IP address.
  5. Access the app using `https://your_domain_name`.
  6. To upgrade the application, first scale the deployment to 0, change the image and then apply the new deployment again.
     
     ```
     kubectl -n jenkins scale deployment jenkins --replicas=0
     kubectl -n jenkins apply -f jenkins-dep.yml
     ```
  7. Uninstall the app using `helm uninstall jenkins --namespace jenkins`.

-----------------------------

**Helm**
To install this app using Helm using your own nginx ingress, perform below steps
  1. Create a namespace. ` kubectl create ns jenkins `
  2. Deploy Nginx Ingress Controller by running below commands.
     
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
  3. Run the command ` kubectl get svc nginx-ingress-ingress-nginx-controller ` to confirm if a **LoadBalancer** IP has been provisioned.

     ```
     pushkar [ ~ ]$ kubectl get svc nginx-ingress-ingress-nginx-controller
     NAME                                     TYPE           CLUSTER-IP    EXTERNAL-IP    PORT(S)                      AGE
     nginx-ingress-ingress-nginx-controller   LoadBalancer   10.0.58.180   20.174.45.64   80:32767/TCP,443:30598/TCP   110s
     ```
  4. Run the command to install **Jenkins**

     ```
     helm install jenkins ./helm --namespace jenkins --set ingressClassName="nginx" --set domain_name=your_preferred_fqdn --set-file tlsSecret.cert=domain_name.crt --set-file tlsSecret.key=domain_name.key
     ```
  5. Run `kubectl -n jenkins get ingress` to retrieve the IP. This may take some time to match with the **LoadBalancer** IP above. Point the domain name in your registrar to the IP address.
  6. Access the app using `https://your_domain_name`.
  7. To upgrade the application, first scale the deployment to 0, change the image and then apply the new deployment again.
     
     ```
     kubectl -n jenkins scale deployment jenkins --replicas=0
     kubectl -n jenkins apply -f jenkins-dep.yml
     ```
  8. Uninstall the app using `helm uninstall jenkins --namespace jenkins`.

-----------------------------

**ArgoCD** installation using the default **App Routing** add on

To install this app using ArgoCD, perform below steps
  1. Create a namespace. ` kubectl create ns argocd `.
  2. Create a tls secret named ` argocd-tls ` which has the domain's certificate & private key by running below command. The domain's .crt & .key file should already be present.

     ```
     kubectl -n argocd create secret tls argocd-tls --cert=argocd_domain_name.crt --key=argocd_domain_name.key
     ```
  3. Run the command ` kubectl -n app-routing-system get svc nginx ` to confirm if a **LoadBalancer** IP has been provisioned.

     ```
     pushkar [ ~ ]$ kubectl -n app-routing-system get svc nginx
     NAME    TYPE           CLUSTER-IP    EXTERNAL-IP    PORT(S)                      AGE
     nginx   LoadBalancer   10.0.58.180   20.174.45.64   80:32767/TCP,443:30598/TCP   110s
     ```
  4. Apply **ArgoCD** manifest file
     
     ```
     kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
     ```
  5. Patch the ConfigMap **argocd-cmd-params-cm** to add `server.insecure : true`, then scale the deployment to 0 & then 1

     ```
     kubectl -n argocd patch configmap argocd-cmd-params-cm --type merge -p '{"data":{"server.insecure":"true"}}'
     kubectl -n argocd scale deploy argocd-server --replicas=0
     kubectl -n argocd scale deploy argocd-server --replicas=1
     ```
  6. Put the FQDN for which the argocd secret has been created in ` argocd-web-app-routing-ingress.yml ` file and then run the command ` kubectl -n argocd apply -f argocd-web-app-routing-ingress.yml `
  7. Run ` kubectl -n argocd get ingress ` to retrieve the IP. This may take some time to match with the **LoadBalancer** IP above. Point the domain name in your registrar to the IP address.
  8. Access ArgoCD using ` https://argocd_domain_name `.
  9. To get the initial admin user password run the command

     ```
     kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 --decode && echo
     ```

-----------------------------

**ArgoCD** installation using **Nginx Ingress**

To install this app using ArgoCD, perform below steps
  1. Create a namespace. ` kubectl create ns argocd `.
  2. Create a tls secret named ` argocd-tls ` which has the domain's certificate & private key by running below command. The domain's .crt & .key file should already be present.

     ```
     kubectl -n argocd create secret tls argocd-tls --cert=argocd_domain_name.crt --key=argocd_domain_name.key
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
  5. Apply **ArgoCD** manifest file
     
     ```
     kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
     ```
  6. Patch the ConfigMap **argocd-cmd-params-cm** to add `server.insecure : true`, then scale the deployment to 0 & then 1

     ```
     kubectl -n argocd patch configmap argocd-cmd-params-cm --type merge -p '{"data":{"server.insecure":"true"}}'
     kubectl -n argocd scale deploy argocd-server --replicas=0
     kubectl -n argocd scale deploy argocd-server --replicas=1
     ```
  7. Put the FQDN for which the argocd secret has been created in ` argocd-nginx-ingress.yml ` file and then run the command ` kubectl -n argocd apply -f argocd-nginx-ingress.yml `
  8. Run ` kubectl -n argocd get ingress ` to retrieve the IP. This may take some time to match with the **LoadBalancer** IP above. Point the domain name in your registrar to the IP address.
  9. Access ArgoCD using ` https://argocd_domain_name `.
 10. To get the initial admin user password run the command

     ```
     kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 --decode && echo
     ```