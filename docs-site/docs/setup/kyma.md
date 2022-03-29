---
title: Kyma
---

Setup in Kyma
===

## Overview
---

You can deploy XSK in the SAP BTP[^1], Kyma environment.

[^1]: SAP Cloud Platform is called SAP Business Technology Platform (SAP BTP) as of 2021.

!!! info "Prerequisites"
    - (Optional) Install [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/).
    - Access to an SAP BTP global account. To create an SAP BTP Trial account, navigate to the [SAP BTP Trial home page](https://account.hanatrial.ondemand.com/).

!!! warning "Warning"
    At the time of writing these setup instructions _(20.12.2021)_, creating a HANA Cloud service instance in the SAP BTP Kyma environment was not possible, thus the setup is currently suitable only for **test** & **demo** purposes. To workaround this limitation:
    
    - Create HANA Cloud service instance in `Cloud Foundry`, allowing traffic coming outside of the `SAP BTP Cloud Foundry` environment.
    - Add HANA related environment variables in the Kubernetes `Deployment` _(described in detail bellow)_.

    _To learn more about this limitation visit the [GitHub discussion](https://github.com/SAP/xsk/discussions/394)._

!!! note "HANA Cloud Network Visibility"

    To update the HANA Cloud network visibility:
    
    - Navigate to your SAP BTP subaccount.
    - Go to the `SAP HANA Cloud` section.
    - Find your HANA Cloud database and from the `Actions` dropdown select `SAP HANA Cloud Central`.
    - Find your database instance, click the more details button (`...`) and select `Manage Configuration`.
    - Click the `Edit` button and in the `Connections` section make the desired changes.
    - To apply your changes click the `Save` button.

## Steps
---

1. Access the SAP BTP, Kyma environment via the SAP BTP cockpit.

1. Create an SAP HANA Cloud secret.

    !!! info "Prerequisites"
        Follow the [Database User](/setup/prerequisites/database-user/) setup guide.

    ```
    kubectl create secret generic hana-cloud-database \
    --from-literal=DIRIGIBLE_DATABASE_PROVIDER=custom \
    --from-literal=DIRIGIBLE_DATABASE_CUSTOM_DATASOURCES=HANA \
    --from-literal=DIRIGIBLE_DATABASE_DATASOURCE_NAME_DEFAULT=HANA \
    --from-literal=HANA_DRIVER=com.sap.db.jdbc.Driver \
    --from-literal=HANA_URL='jdbc:sap://<your-hana-cloud-host>/?encrypt=true&validateCertificate=false' \
    --from-literal=HANA_USERNAME=<your-hana-cloud-username> \
    --from-literal=HANA_PASSWORD=<your-hana-cloud-password>
    ```

    !!! note
        Before executing the command, replace the placeholders:

        - `<your-hana-cloud-host>` with the HANA Cloud host URL _(e.g. `bc6e8e95-xxx.hanacloud.ondemand.com`)_.
        - `<your-hana-cloud-username>` with the HANA Cloud username _(e.g. `XSK_USER`)_.
        - `<your-hana-cloud-password>` with the HANA Cloud password.

1. Deploy XSK:

    === "All in One"

        ```yaml
        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: xsk
          namespace: default
        spec:
          replicas: 1
          strategy:
            type: Recreate
          selector:
            matchLabels:
              app: xsk
          template:
            metadata:
              labels:
                app: xsk
            spec:
              containers:
              - name: xsk
                image: dirigiblelabs/xsk-kyma:latest
                imagePullPolicy: Always
                envFrom:
                - secretRef:
                    name: hana-cloud-database
                env:
                - name: DIRIGIBLE_THEME_DEFAULT
                  value: fiori
                - name: DIRIGIBLE_HOST
                  value: https://xsk.<your-kyma-cluster-host>
                - name: SERVER_MAXHTTPHEADERSIZE
                  value: "48000"
                volumeMounts:
                - name: xsk-volume
                  mountPath: /usr/local/tomcat/target/dirigible/repository
                ports:
                - containerPort: 8080
                  name: xsk
                  protocol: TCP
              volumes:
              - name: xsk-volume
                persistentVolumeClaim:
                  claimName: xsk-claim
        ---
        apiVersion: v1
        kind: PersistentVolumeClaim
        metadata:
          name: xsk-claim
          namespace: default
        spec:
          accessModes:
          - ReadWriteOnce
          resources:
            requests:
              storage: 1Gi
        ---
        apiVersion: v1
        kind: Service
        metadata:
          labels:
            app: xsk
          name: xsk
          namespace: default
        spec:
          ports:
          - name: xsk
            port: 8080
            protocol: TCP
            targetPort: 8080
          selector:
            app: xsk
          type: ClusterIP
        ---
        apiVersion: gateway.kyma-project.io/v1alpha1
        kind: APIRule
        metadata:
          name: xsk
          namespace: default
        spec:
          gateway: kyma-gateway.kyma-system.svc.cluster.local
          rules:
          - accessStrategies:
            - config: {}
              handler: noop
            methods:
            - GET
            - POST
            - PUT
            - PATCH
            - DELETE
            - HEAD
            path: /.*
          service:
            host: xsk.<your-kyma-cluster-host>
            name: xsk
            port: 8080
        ```

    === "Deployment (Only)"

        !!! info
            Appling this definition will create `Deployment` and `PersistentVolumeClaim` resoures only. To install XSK with single definition file, use the `All in One` section.

        ```yaml
        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: xsk
          namespace: default
        spec:
          replicas: 1
          strategy:
            type: Recreate
          selector:
            matchLabels:
              app: xsk
          template:
            metadata:
              labels:
                app: xsk
            spec:
              containers:
              - name: xsk
                image: dirigiblelabs/xsk-kyma:latest
                imagePullPolicy: Always
                envFrom:
                - secretRef:
                    name: hana-cloud-database
                env:
                - name: DIRIGIBLE_THEME_DEFAULT
                  value: fiori
                - name: DIRIGIBLE_HOST
                  value: https://xsk.<your-kyma-cluster-host>
                - name: SERVER_MAXHTTPHEADERSIZE
                  value: "48000"
                volumeMounts:
                - name: xsk-volume
                  mountPath: /usr/local/tomcat/target/dirigible/repository
                ports:
                - containerPort: 8080
                  name: xsk
                  protocol: TCP
              volumes:
              - name: xsk-volume
                persistentVolumeClaim:
                  claimName: xsk-claim
        ---
        apiVersion: v1
        kind: Service
        metadata:
          labels:
            app: xsk
          name: xsk
          namespace: default
        spec:
          ports:
          - name: xsk
            port: 8080
            protocol: TCP
            targetPort: 8080
          selector:
            app: xsk
          type: ClusterIP
        ---
        apiVersion: v1
        kind: PersistentVolumeClaim
        metadata:
          name: xsk-claim
          namespace: default
        spec:
          accessModes:
          - ReadWriteOnce
          resources:
            requests:
              storage: 1Gi
        ```

    === "APIRule (Only)"

        !!! info
            Appling this definition will create `Service` and `APIRule` resoures only. To install XSK with single definition file, use the `All in One` section.

        ```yaml
        apiVersion: gateway.kyma-project.io/v1alpha1
        kind: APIRule
        metadata:
          name: xsk
          namespace: default
        spec:
          gateway: kyma-gateway.kyma-system.svc.cluster.local
          rules:
          - accessStrategies:
            - config: {}
              handler: noop
            methods:
            - GET
            - POST
            - PUT
            - PATCH
            - DELETE
            - HEAD
            path: /.*
          service:
            host: xsk.<your-kyma-cluster-host>
            name: xsk
            port: 8080
        ```

    !!! Note
        - Copy the content into YAML file(s) _(e.g. `all.yaml`, `deployment.yaml` or `apirule.yaml`)_.
        - By default deployment strategy type is `Recreate` which will recreate deployment resources when you apply new changes.
        - Replace the placeholders:
            - `<your-kyma-cluster-host>` with your Kyma cluster host _(e.g. `c-xxx.kyma.xxx.ondemand.com`)_.

    !!! tip "XSK versions"
        Instead of using the `latest` tag (version), for production and development use cases it is recommended that you use a stable release version:

        - You can find all released versions [here](https://github.com/sap/xsk/releases/).
        - You can find all XSK Docker images and tags (versions) [here](https://hub.docker.com/u/dirigiblelabs).

    - Navigate to your Kyma dashboard and select the **`default`** namespace.

    - Click the **Deploy new resource** button and select the `all.yaml`, `deployment.yaml` or `apirule.yaml` file(s).

        !!! info "Note"
            Alternatively, you can use the `kubectl apply -f <file-name>` to deploy the desired resources _(e.g. `all.yaml`, `deployment.yaml` or `apirule.yaml`)_.

1. Create an XSUAA service instance:

    === "with Kyma dashboard"
        - From the Kyma dashboard, go to **Service Management** **&rarr;** **Catalog**.
        - Find the `Authorization & Trust Management` service.
        - Create a new service instance.
        - Provide the following additional parameters:

        ```json
        {
           "xsappname":"xsk-xsuaa",
           "oauth2-configuration":{
              "token-validity":7200,
              "redirect-uris":[
                 "https://xsk.<your-kyma-cluster-host>"
              ]
           },
           "scopes":[
              {
                 "name":"$XSAPPNAME.Developer",
                 "description":"Developer scope"
              },
              {
                 "name":"$XSAPPNAME.Operator",
                 "description":"Operator scope"
              }
           ],
           "role-templates":[
              {
                 "name":"Developer",
                 "description":"Developer related roles",
                 "scope-references":[
                    "$XSAPPNAME.Developer"
                 ]
              },
              {
                 "name":"Operator",
                 "description":"Operator related roles",
                 "scope-references":[
                    "$XSAPPNAME.Operator"
                 ]
              }
           ],
           "role-collections":[
              {
                 "name":"XSK Developer",
                 "description":"XSK Developer",
                 "role-template-references":[
                    "$XSAPPNAME.Developer"
                 ]
              },
              {
                 "name":"XSK Operator",
                 "description":"XSK Operator",
                 "role-template-references":[
                    "$XSAPPNAME.Operator"
                 ]
              }
           ]
        }
        ```

         - Bind the servce instance to the **`xsk`** application.

    === "with kubectl"

         Copy and paste the following content into `xsuaa.yaml`:

         ```yaml
         apiVersion: servicecatalog.k8s.io/v1beta1
         kind: ServiceInstance
         metadata:
           name: xsuaa-xsk
           namespace: default
         spec:
           clusterServiceClassExternalName: xsuaa
           clusterServiceClassRef:
             name: xsuaa
           clusterServicePlanExternalName: broker
           parameters:
             xsappname: xsk-xsuaa
             oauth2-configuration:
               redirect-uris:
               - https://xsk.<your-kyma-host>
               token-validity: 7200
             role-collections:
             - description: XSK Developer
               name: XSK Developer
               role-template-references:
               - $XSAPPNAME.Developer
             - description: XSK Operator
               name: XSK Operator
               role-template-references:
               - $XSAPPNAME.Operator  
             role-templates:
             - description: Developer related roles
               name: Developer
               scope-references:
               - $XSAPPNAME.Developer
             - description: Operator related roles
               name: Operator
               scope-references:
               - $XSAPPNAME.Operator
             scopes:
             - description: Developer scope
               name: $XSAPPNAME.Developer
             - description: Operator scope
               name: $XSAPPNAME.Operator
         ---
         apiVersion: servicecatalog.k8s.io/v1beta1
         kind: ServiceBinding
         metadata:
           name: xsuaa-xsk-binding
           namespace: default
         spec:
           instanceRef:
             name: xsuaa-xsk
           parameters: {}
           secretName: xsuaa-xsk-binding
         ---
         apiVersion: servicecatalog.kyma-project.io/v1alpha1
         kind: ServiceBindingUsage
         metadata:
           name: xsuaa-xsk-usage
           namespace: default
         spec:
           parameters:
             envPrefix:
               name: ""
           serviceBindingRef:
             name: xsuaa-xsk-binding
           usedBy:
             kind: deployment
             name: xsk
         ```

        !!! info "Note"
            Execute the following command to apply the XSUAA configuration: `kubectl apply -f xsuaa.yaml` or use the **Deploy new resource** functionality.

    !!! Note
            Replace the **`<your-kyma-cluster-host>`** placeholder with your Kyma cluster host (e.g. **`c-xxxxxxx.kyma.xxx.xxx.xxx.ondemand.com`**).

1. Create an Destination service instance (optional)

    === "with Kyma dashboard"
        - From the Kyma dashboard, go to **Service Management** **&rarr;** **Catalog**.
        - Find the `Destination` service.
        - Create a new service instance.
        - Bind the servce instance to the **`xsk`** application
          - NOTE: For `Prefix for injected variables` make sure to specify `destination_`

    === "with kubectl"

         Copy and paste the following content into `destination.yaml`:

         ```yaml
          apiVersion: servicecatalog.k8s.io/v1beta1
          kind: ServiceInstance
          metadata:
            name: destination-xsk
            namespace: default
          spec:
            clusterServiceClassExternalName: destination
            clusterServiceClassRef:
              name: destination
            clusterServicePlanExternalName: lite
            parameters: {}
          ---
          apiVersion: servicecatalog.k8s.io/v1beta1
          kind: ServiceBinding
          metadata:
            name: destination-xsk-binding
            namespace: default
          spec:
            instanceRef:
              name: destination-xsk
            parameters: {}
            secretName: destination-xsk-binding
          ---
          apiVersion: servicecatalog.kyma-project.io/v1alpha1
          kind: ServiceBindingUsage
          metadata:
            name: destination-xsk-usage
            namespace: default
          spec:
            parameters:
              envPrefix:
                name: "destination_"
            serviceBindingRef:
              name: destination-xsk-binding
            usedBy:
              kind: deployment
              name: xsk
         ```

        !!! info "Note"
            Execute the following command to apply the Destination configuration: `kubectl apply -f destination.yaml` or use the **Deploy new resource** functionality.

1. Assign the `Developer` and `Operator` roles.

    - Navigate to the SAP BTP Cockpit.
    - Log in to your subaccount.
    - Go to **Security** **&rarr;** **Users**.
    - Select your username.
    - Choose **Assign Role Collection**.
    - From the list of roles, select the `XSK Developer` and `XSK Operator` roles.
    - Choose **Assign Role Collection** to update the assigned role collections.

1. Log in.

    - Go to `https://xsk.<your-kyma-cluster-host>` or navigate to **Configurations** **&rarr;** **APIRules** section from the Kyma dashboard.

## Maintenance
---

### Version Update

To update the XSK version either use the **kubectl** or update the **Deployment YAML** as follows:

=== "with kubectl"

    ```
    kubectl set image deployment/xsk xsk=dirigiblelabs/xsk-kyma:<xsk-version>
    ```

=== "with Deployment YAML"

    ```yaml hl_lines="4"
    spec:
      containers:
      - name: xsk
        image: dirigiblelabs/xsk-kyma:<xsk-version>
        imagePullPolicy: Always
    ```

!!! tip "XSK versions"

    Update the `<xsk-version>` placeholder with a stable release version:

    - You can find all released versions [here](https://github.com/sap/xsk/releases/).
    - You can find all XSK Docker images and tags (versions) [here](https://hub.docker.com/u/dirigiblelabs).

### Scaling

The XSK **Deployment** could be scaled horizontally by adding/removing **Pods** as follows:

=== "Scale to Zero"

    ```
    kubectl scale deployment/xsk --replicas=0
    ```

=== "Scale Up"

    ```
    kubectl scale deployment/xsk --replicas=<number-of-replicas>
    ```

!!! note

    To learn more about application scaling in Kubernetes, see [Horizontal Pod Autoscaling](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/).


### Debugging

To debug the XSK engine via **Remote Java Debugging** execute the following command:

```
kubectl port-forward deployment/xsk 8000:8000
```
