# Bug in Helm 1.12, service account is not deleted from deployment

Steps to reproduce:

1. Create Helm release with deployment using service account:
```
> helm install serviceaccountbug ./v1 --set role_arn="ROLE_ARN"
NAME: serviceaccountbug
LAST DEPLOYED: Wed Jun  7 13:44:01 2023
NAMESPACE: XXXX
STATUS: deployed
REVISION: 1
TEST SUITE: None
``` 
2. Observe that deployment was created and pods are running:
```
> kubectl get deployments serviceaccountbug-deployment -o yaml |  rg serviceAccount 
      serviceAccount: serviceaccountbug-service-account
      serviceAccountName: serviceaccountbug-service-account
> kubectl get pods | rg  serviceaccountbug-deployment                               
serviceaccountbug-deployment-79f89995cc-fgk9l                 1/1     Running            0                51s
````
3. Upgrade the release, deleting the service account
```
> helm upgrade serviceaccountbug ./v2                                                                               
Release "serviceaccountbug" has been upgraded. Happy Helming!
NAME: serviceaccountbug
LAST DEPLOYED: Wed Jun  7 13:45:00 2023
NAMESPACE: XXXX
STATUS: deployed
REVISION: 2
TEST SUITE: None
````

**Expected**:
Service account is deleted from the deployment, pods are schedulable

**Actual**
Service account is not removed from the account, pods are unschedulable:

```
> kubectl get deployments serviceaccountbug-deployment -o yaml |  rg serviceAccount
      serviceAccount: serviceaccountbug-service-account
      serviceAccountName: serviceaccountbug-service-account
> kubectl get pods | rg  serviceaccountbug-deployment                                
serviceaccountbug-deployment-6df8c9f497-mfzs8                 0/1     ContainerCreating   0                6m18s
serviceaccountbug-deployment-79f89995cc-fgk9l                 1/1     Running             0                7m17s
```
