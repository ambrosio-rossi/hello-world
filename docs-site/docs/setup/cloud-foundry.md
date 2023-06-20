---
title: Cloud Foundry
---

Setup in Cloud Foundry
===

## Overview
---

You can deploy XSK in the SAP BTP[^1], Cloud Foundry environment.

[^1]: SAP Cloud Platform is called SAP Business Technology Platform (SAP BTP) as of 2021.
    
!!! info "Prerequisites"
    - Install the [Cloud Foundry Command Line Interface](http://docs.cloudfoundry.org/devguide/installcf/install-go-cli.html).
    - Access to an SAP BTP global account. To create an SAP BTP Trial account, navigate to the [SAP BTP Trial home page](https://account.hanatrial.ondemand.com/).
    - Create a SAP HANA Cloud service instance in SAP BTP Cloud Foundry space.
    - Create a separate SAP HANA Cloud user that will be used from the XSK engine.

## Steps
---

1. Set the SAP BTP Cloud Foundry API host:

    ```
    cf api <cloud-foundry-api-host>
    ```

1. Log in to the SAP BTP, Cloud Foundry environment with:

    ```
    cf login --sso
    ```

1. Create an XSUAA service instance:

    - Copy and paste the following content into `xs-security.json`:

        ```json
        {
           "xsappname": "<applicationName>-xsuaa",
           "tenant-mode": "shared",
            "oauth2-configuration": {
                "redirect-uris": [
                    "https://<applicationName>.cfapps.<cloudFoundryRegion>.hana.ondemand.com"
                ],
                "token-validity": 7200
            },
           "scopes": [
              {
                 "name": "$XSAPPNAME.Developer",
                 "description": "Developer scope"
              },
              {
                 "name": "$XSAPPNAME.Operator",
                 "description": "Operator scope"
              }
           ],
           "role-templates": [
              {
                 "name": "Developer",
                 "description": "Developer related roles",
                 "scope-references": [
                    "$XSAPPNAME.Developer"
                 ]
              },
              {
                 "name": "Operator",
                 "description": "Operator related roles",
                 "scope-references": [
                    "$XSAPPNAME.Operator"
                 ]
              }
           ],
           "role-collections": [
              {
                 "name": "XSK Developer",
                 "description": "XSK Developer",
                 "role-template-references": [
                    "$XSAPPNAME.Developer"
                 ]
              },
              {
                 "name": "XSK Operator",
                 "description": "XSK Operator",
                 "role-template-references": [
                    "$XSAPPNAME.Operator"
                 ]
              }
           ]
        }
        ```

        !!! Note

            - Replace the `<applicationName>` placeholder with your application name, e.g. **`xsk`**.
            - Replace the `<cloudFoundryRegion>` placeholder with the Cloud Foundry region, e.g. **`us10-001`**.

    - Create an XSUAA service instance:

        ```
        cf create-service xsuaa application <applicationName>-xsuaa -c xs-security.json
        ```

        !!! Note
            Use the same `<applicationName>` as in the previous step.

1. Deploy XSK:


    === "Docker"

        ```
        cf push xsk-<org-name> \
        --docker-image=dirigiblelabs/xsk-cf:latest \
        -m 2G -k 2G
        ```
        !!! info "Note"
            - Replace the `<org-name>` placeholder with your subaccount's **Subdomain** value.
            
        !!! tip "XSK versions"
            Instead of using the `latest` tag (version), for production and development use cases it is recommended that you use a stable release version:
            
            - You can find all released versions at [https://github.com/sap/xsk/releases/](https://github.com/sap/xsk/releases/).
            - You can find all XSK Docker images and tags (versions) at [https://hub.docker.com/u/dirigiblelabs](https://hub.docker.com/u/dirigiblelabs).

        - Bind the XSUAA and HANA Cloud service instances to the XSK deployment:

            ```
            cf bind-service xsk-<org-name> <applicationName>-xsuaa
            cf set-env xsk-<org-name> DIRIGIBLE_DATABASE_PROVIDER 'custom'
            cf set-env xsk-<org-name> DIRIGIBLE_DATABASE_CUSTOM_DATASOURCES 'HANA'
            cf set-env xsk-<org-name> DIRIGIBLE_DATABASE_DATASOURCE_NAME_DEFAULT 'HANA'
            cf set-env xsk-<org-name> HANA_DRIVER 'com.sap.db.jdbc.Driver'
            cf set-env xsk-<org-name> HANA_URL 'jdbc:sap://<hanaUrl>/?encrypt=true&validateCertificate=false'
            cf set-env xsk-<org-name> HANA_USERNAME '<hanaUsername>'
            cf set-env xsk-<org-name> HANA_PASSWORD '<hanaPassword>'
            ```

            !!! Note

                - Replace the `<org-name>` placeholder with your subaccount's **Subdomain** value.
                - Replace the `<applicationName>` placeholder with the application name used in the previous steps.
                - Replace the `<hanaUrl>` placeholder with the HANA Cloud SQL endpoint URL _(including the port)_, e.g. **`69fcb050-....hana.trial-us10.hanacloud.ondemand.com:443`**.
                - Replace the `<hanaUsername>` placeholder with the HANA Cloud username that will be used.
                - Replace the `<hanaPassword>` placeholder with the HANA Cloud password that will be used.

        - Restart the `xsk` deployment:

            ```
            cf restart xsk-<org-name>
            ```
   
        !!! info "Note"
            - Replace the `<org-name>` placeholder with your subaccount's **Subdomain** value.

    === "Buildpack"

        - Download the `sap-cf` binaries from the downloads site: [https://github.com/SAP/xsk/releases](https://github.com/SAP/xsk/releases)
        - Unzip the downloaded archive to extract the `ROOT.war` file.
        - Create `manifest.yaml` file in the same directory where the `ROOT.war` is located:
            ```yaml
            applications:
            - name: xsk
              host: xsk-<org-name>
              memory: 2G
              buildpack: sap_java_buildpack
              path: ROOT.war
              env:
                JBP_CONFIG_COMPONENTS: "jres: ['com.sap.xs.java.buildpack.jdk.SAPMachineJDK']"
                JBP_CONFIG_SAP_MACHINE_JRE: 'jre: { version: 11.+ }'
                HANA_USERNAME: <hanaUsername>
                HANA_PASSWORD: <hanaPassword>
              services:
                - <applicationName>-xsuaa
                - <hanaCloudInstanceName>
            ```

            !!! Note
                - Replace the `<org-name>` placeholder with your subaccount's Subdomain value.
                - Replace the `<applicationName>` placeholder with the application name used in the previous steps.
                - Replace the `<hanaUsername>` placeholder with the HANA Cloud username that will be used.
                - Replace the `<hanaPassword>` placeholder with the HANA Cloud password that will be used.
                - Replace the `<hanaCloudInstanceName>` placeholder with the HANA Cloud service instance name that will be used.

        - Deploy with:
            ```
            cf push
            ```

1. Assign the `Developer` and `Operator` roles.

1. Log in.

!!! example "Additional Information"
    For more details, see the step-by-step tutorial [How to deploy Eclipse Dirigible in the SAP Cloud Platform Cloud Foundry environment](https://blogs.sap.com/2020/03/15/how-to-deploy-eclipse-dirigible-in-the-sap-cloud-platform-cloud-foundry-environment/) on SAP Community.
