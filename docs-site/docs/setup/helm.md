---
title: Helm
---

Setup with Helm
===

## Overview
---

You can deploy XSK via Helm chart in a Kubernetes cluster.

!!! info "Prerequisites"
    - Install the [Helm CLI](https://helm.sh/docs/intro/install/#helm).
    - Make sure you have access to a [Kubernetes Cluster](https://kubernetes.io/docs/setup/).

## Setup
---

1. Add the XSK Helm repository:

    ```
    helm repo add xsk https://www.xsk.io

    helm repo update
    ```

1. Verify XSK Helm charts:

    From key server

    ```
    helm pull xsk/xsk --prov
    gpg --keyserver keys.openpgp.org --recv-key 734B0E32368A2283290E0B966010F454D1819484
    gpg --export > ~/.gnupg/pubring.gpg
    helm verify xsk-<version>.tgz 
    ```

    From github
    
    !!! info "Note"
        Use command `wget` for download public key instead of `curl`

    ```
    helm pull xsk/xsk --prov
    wget https://sap.github.io/xsk/charts/pubring.gpg -O ~/.gnupg/pubring.gpg
    helm verify xsk-<version>.tgz 
    ```

1. Deployment

    To deploy XSK, you can use the following instructions:


    === "Basic"

        ```
        helm install xsk xsk/xsk
        ```

        ???+ info "Accessing the XSK Instance"
            
            Running this command will install XSK `Deployment` and `Service` with `ClusterIP` only. To access the XSK instance, execute the command that was printed in the console.

            Example:

            ```
            export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=xsk,app.kubernetes.io/instance=xsk" -o jsonpath="{.items[0].metadata.name}")
            echo "Visit http://127.0.0.1:8080 to use your application"
            kubectl --namespace default port-forward $POD_NAME 8080:8080    
            ```

            - Navigate to: [http://127.0.0.1:8080](http://127.0.0.1:8080)
            - Log in with these username and password: `dirigible`/`dirigible`

    === "Kyma"

        ```
        helm install xsk xsk/xsk \
        --set kyma.enabled=true \
        --set kyma.host=<kyma-host>
        ```

        This will install additionally an `ApiRule` and XSUAA `ServiceInstance` and `ServiceBinding`. The appropriate roles should be assigned to the user.

    === "Kyma with SAP HANA database"

        ```
        helm install xsk xsk/xsk \
        --set hana.enabled=true \
        --set hana.url=<hana-url> \
        --set hana.username=<hana-username> \
        --set hana.password=<hana-password> \
        --set kyma.enabled=true \
        --set kyma.host=<kyma-apirule-host>
        ```

        This will install additionally an `ApiRule`, XSUAA `ServiceInstance`, `ServiceBinding` and HANA instance. The appropriate roles should be assigned to the user.

    === "Kyma with custom roles"

        ```
        helm install xsk xsk \            
        --set kyma.enabled=true \
        --set kyma.host=<kyma-apirule-host> \
        --set kyma.addRoles=true \
        --set kyma.roles={"<new-role>"\,"<new-role>"}
        ```

        This will install additionally an `ApiRule` and XSUAA `ServiceInstance` and `ServiceBinding` with default roles `Developer` and `Operator` and will add your new roles. The appropriate roles should be assigned to the user.

    ??? info "Configuration Options"
        
        |             Name                                   |          Description                      |                 Default                             |
        |----------------------------------------------------|-------------------------------------------|-----------------------------------------------------|
        | `replicaCount`                                     | Number of replicas                        | `1`                                                 |
        | `deployment.strategyType`                          | Deployment strategy type                  | `Recreate`                                          |
        | `deployment.readinessProbePeriodSeconds`           | Readiness probe period seconds            | `10`                                                |
        | `deployment.readinessProbeInitialDelaySeconds`     | Readiness probe initial delay             | `5`                                                 |
        | `deployment.readinessProbeHttpGetPort`             | Readiness probe http get port             | `8080`                                              |
        | `deployment.LivenessProbePeriodSeconds`            | Liveness probe period seconds             | `60`                                                |
        | `deployment.LivenessProbeInitialDelaySeconds`      | Liveness probe initial delay              | `60`                                                |
        | `deployment.LivenessProbeHttpGetPort`              | Liveness probe http get port              | `8080`                                              |
        | `deployment.volumeMountsName`                      | Volume mount name                         | `xsk-volume`                                        |
        | `deployment.volumeMountsMountPath`                 | Volume mounts mount Path                  | `/usr/local/tomcat/target/dirigible/repository`     |
        | `deployment.volumesName`                           | Volume name                               | `xsk-volume`                                        |
        | `deployment.volumespersistentVolumeClaimClaimName` | Volume spersistent volume claim           | `xsk-claim`                                         |
        | `persistentVolumeClaim.enabled`                    | Persistent volume claim enable            | `true`                                              |
        | `persistentVolumeClaim.Name`                       | Persistent volume claim name              | `xsk-claim`                                         |
        | `persistentVolumeClaim.accessModes`                | Persistent volume claim access modes      | `ReadWriteOnce`                                     |
        | `persistentVolumeClaim.resourcesStorage`           | Persistent volume claim resources storage | `1Gi`                                               |
        | `serviceAccount.create`                            | Create service account                    | `false`                                             |
        | `serviceAccount.annotations`                       | Add annotations to sa                     | ``                                                  |
        | `serviceAccount.name`                              | Service account name                      | ``                                                  |
        | `securityContext.allowPrivilegeEscalation`         | Allow privileged escalation               | `false`                                             |
        | `securityContext.seccompProfile.type`              | Enable seccomp profile                    | `RuntimeDefault`                                    |
        | `podSecurityContext`                               | Pod Security Context                      | ``                                                  |
        | `service.type`                                     | Type of service                           | `ClusterIP`                                         |
        | `service.port`                                     | Port of service                           | `8080`                                              |
        | `nameOverride`                                     | Name override                             | ``                                                  |
        | `fullNameOverride`                                 | Full name override                        | ``                                                  |
        | `kyma.enabled`                                     | Enable Kyma                               | ``                                                  |
        | `kyma.host`                                        | Kyma host                                 | ``                                                  |
        | `kyma.serverMaxHttpHeaderSize`                     | Tomcat max http header size               | `48000`                                             |
        | `kyma.addRoles`                                    | Enable add new roles                      | ``                                                  |
        | `kyma.roles`                                       | Name for new roles                        | ``                                                  |
        | `kyma.roleCollections.description`                 | Set role collections description          | `XSK Developer`, `XSK Operator`                     |
        | `kyma.roleCollections.name`                        | Set role collections name                 | `XSK-Developer`, `XSK Operator`                     |
        | `kyma.roleCollections.role-template-references`    | Set role collections role template        | `$XSAPPNAME.Developer`, `$XSAPPNAME.Operator`       |
        | `kyma.roleTemplates.description`                   | Set role templates description            | `Developer related roles`, `Operator related roles` |
        | `kyma.roleTemplates.name`                          | Set role templates name                   | `Developer`, `Operator`                             |
        | `kyma.roleTemplates.scopeReferences`               | Set role templates scope references       | `$XSAPPNAME.Developer`, `$XSAPPNAME.Operator`       |
        | `kyma.scopes.description`                          | Set scopes description                    | `Developer scope`, `Operator scope`                 |
        | `kyma.scopes.name`                                 | Set scopes name                           | `$XSAPPNAME.Developer`, `$XSAPPNAME.Operator`       |
        | `application.image`                                | Application image                         | ``                                                  |
        | `application.homeUrl`                              | Home url for XSK                          | ``                                                  |
        | `application.imagePullPolicy`                      | Image pull policy                         | `Always`                                            |
        | `application.privateDockerRegistry`                | Paramater to enable docker registry       | ``                                                  |
        | `application.dockerSecretName`                     | Secret name for docker  registry          | `docker-registry-secret`                            |
        | `application.dockerServer`                         | Parameter to set Docker server            | `https://index.docker.io/v1/`                       |
        | `application.dockerUsername`                       | User name for docker registry             | ``                                                  |
        | `application.dockerPassword`                       | Password for docker registry              | ``                                                  |
        | `application.dockerEmail`                          | Email for docker registry                 | ``                                                  |
        | `hana.enabled`                                     | Hana enable option                        | ``                                                  |
        | `hana.secretName`                                  | Hana secret name                          | `hana-secret`                                       |
        | `hana.url`                                         | Hana url instance                         | ``                                                  |
        | `hana.username`                                    | Hana username                             | ``                                                  |
        | `hana.password`                                    | Hana password                             | ``                                                  |

2. Uninstall

    If you want to uninstall Helm, run:

    ```
    helm uninstall xsk
    ```

    !!! note

        If you created namespace with `helm install --create-namespace namespace <your-namespace>`. If you want the namespace to be deleted you need to delete you namespace manually `kubectl delete namespace <your-namespace>`.

## Setup - Kpack

You can choose to build and package your XSK application from source with Helm and Kpack by following these instructions:

1. Add the XSK kpack Helm repository:

    ```
    helm repo add xsk https://www.xsk.io

    helm repo update
    ```

1. Deployment

    !!! Prerequisites

        Install [kpack](https://github.com/pivotal/kpack/blob/main/docs/install.md).

    === "All in One"

        ```
        helm install xsk-image xsk/xsk-kpack \
        --set install.all=true \
        --set docker.server=https://index.docker.io/v1/ \
        --set docker.username=<your-docker-username> \
        --set docker.password=<your-docker-password> \
        --set docker.email=<your-email> \
        --set image.repository=<your-repository-for-your-OCI-image> \
        --set imageBuilder.repository=<builder-image> \
        --set image.source=<your-application-source> 
        ```

        This will build and package an OCI image for your application.

    === "Cluster Builder (Only)"

        ```
        helm install xsk-cluster-builder xsk/xsk-kpack \
        --set install.clusterBuilder=true \
        --set docker.server=https://index.docker.io/v1/ \
        --set docker.username=<your-docker-username> \
        --set docker.password=<your-docker-password> \
        --set docker.email=<your-email> \
        ```

        This will install a kpack `ClusterStore` and `ClusterStack` resources.

    === "Image Builder (Only)"

        ```
        helm install xsk-image-builder xsk/xsk-kpack \
        --set install.imageBuilder=true \
        --set imageBuilder.repository=<builder-image>
        ```

        This will install a kpack `Builder` resource.

    === "Image (Only)"

        ```
        helm install xsk-image xsk/xsk-kpack \
        --set create.image=true \
        --set image.repository=<your-repository-for-your-OCI-image> \
        --set image.source=<your-application-source>
        ```

        This will build and package an OCI image for your application.

    !!! note

        Due to synchronization issues the `All in One` setup could fail. To overcome this issue you could execute the `Cluster Builder (Only)`, `Image Builder (Only)` and `Image (Only)` steps.

    !!! tip

        You can tail the logs for your image that is currently building using the [Kpack CLI](https://github.com/vmware-tanzu/kpack-cli/blob/main/docs/kp_build_logs.md).

        ```
        kp build logs xsk-image -n default
        ```

        You can check your image with:

        ```
        kubectl -n default get image xsk-image
        ```

        To download and run the newly created OCI image, execute:

        ```
        docker run -p 8080:8080 <latest-image-with-digest>
        ```
        
    The application image that was built could be used as well in the installation of XSK with [Helm](#setup), [Kyma](../kyma/) and [Cloud Foundry](../cloud-foundry).

    !!! note

        You can use for build Kyma (which is default), local and Cloud Foundry.
        
        - For ClusterStack build image you can use `--set clusterBuilder.buildImage` and choose one of this:

        ```
        dirigiblelabs/buildpacks-stack-build-xsk-kyma - Kyma
        dirigiblelabs/buildpacks-stack-build-xsk-cf - Cloud Foundry
        dirigiblelabs/buildpacks-stack-build-xsk - Local
        ```

        - For ClusterStack run image you can use `--set clusterBuilder.runImage` and choose one of this:
        ```
        dirigiblelabs/buildpacks-stack-run-xsk-kyma - Kyma
        dirigiblelabs/buildpacks-stack-run-xsk-cf - Cloud Foundry
        dirigiblelabs/buildpacks-stack-run-xsk - Local
        ```

        - For Builder image you can use `--set imageBuilder.buildpack` and choose one of this:
        ```
        --set imageBuilder.buildpack=dirigiblelabs/buildpacks-xsk-kyma - Kyma
        --set imageBuilder.buildpack=dirigiblelabs/buildpacks-xsk-cf - Cloud Foundry   
        --set imageBuilder.buildpack=dirigiblelabs/buildpacks-xsk - Local  
        ```
        
    ??? info "Configuration Options"     
     
        |             Name                    |          Description            |            Default                                     |
        |-------------------------------------|---------------------------------|--------------------------------------------------------|
        | `install.clusterBuilder`            | Kpack cluster store and stack   | `false`                                                |
        | `install.imageBuilder`              | Kpack builder                   | `false`                                                |
        | `install.allInOne`                  | Option to build all             | `false`                                                |
        | `create.image`                      | Kpack create OCI image          | `false`                                                |
        | `create.namespace`                  | Create namespace                | `default`                                              |
        | `docker.server`                     | Docker server url               | ``                                                     |
        | `docker.username`                   | Docker username                 | ``                                                     |
        | `docker.password`                   | Docker password                 | ``                                                     |
        | `docker.email`                      | Docker email                    | ``                                                     |
        | `docker.secretName`                 | Docker secret name              | `docker-registry-secret`                               |
        | `docker.serviceAccountName`         | Docker service account name     | `docker-registry-service-account`                      |
        | `imageBuilder.repository`           | Docker service account name     | ``                                                     |
        | `imageBuilder.buildpack`            | Docker service account name     | `dirigiblelabs/buildpacks-xsk`                         |
        | `image.repository`                  | Docker service account name     | ``                                                     |
        | `image.source`                      | Docker service account name     | ``                                                     |
        | `image.serviceAccountName`          | Docker service account name     | `docker-registry-service-account`                      |
        | `clusterBuilder.buildImage`         | Docker service account name     | `dirigiblelabs/buildpacks-stack-build-xsk-kyma:latest` |
        | `clusterBuilder.runImage`           | Docker service account name     | `dirigiblelabs/buildpacks-stack-run-xskyma:latest`     |
        | `clusterBuilder.serviceAccountName` | Docker service account name     | `docker-registry-service-account`                      |

1. Uninstall

    If you want to uninstall the Helm kpack chart, run:

    ```
    helm uninstall xsk-kpack
    ```            
