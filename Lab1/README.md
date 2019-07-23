# Toolchain 1 - Deploy a Kubernetes App

1. Create a Toolchain,
    * Go to https://cloud.ibm.com/devops/toolchains
    * Click the `Create a Toolchain`,
    * Select the `Develop a Kubernetes app` template,
    
2. Configure the Toolchain, for:

    * Toolchain Name: `toolchain-kube-guestbook`,
    * Region: 
    * Resource Group: `default`
    * Configure your Source Provider: `Github`,
        * Github Server: `https://github.com`,
        * Repository Type: `Existing`
        * Source repository URL: `https://github.com/<username>/guestbook`,
        * [Owner: ]
        * [Repository Name: ]
        * Check `Enable GitHub Issues`,
        * Check `Track deployment of code changes`,
    * Click `Create` button,

3. Configure the `Delivery Pipeline`:
    
    * Define the following settings:
        * App name: `guestbook`,
        * IBM Cloud API Key:
            * If you have an existing IBM Cloud API Key, copy-paste the API Key,
            * If you do not have an existing IBM Cloud API Key,
                * Click the `Create` button to generate a new one,
                * Or go to `Manage` > `Access (IAM)` > `IBM Cloud API Keys` to create a new IBM Cloud API Key,
                * copy-paste the API Key,
                * This should trigger the validation of the API Key, and if validated, should autoload the values for `Container registry region`, `Container registry namespace`, `Cluster region`, `Resource Group`, `Cluster name`, and `Cluster namespace`,
    * Click the `Create` button,
    * The Toolchain is being configured,
    * When the Toolchain has successfully been configured, you will see the Tools in the Toolchain: THINK, CODE, DELIVER, and Eclipse Orion Web IDE,
    * In the DELIVER tool window, click the top right dropdown and select `Configure`,
    * Rename the `Pipeline name` to a meaningful name, e.g. `guestbook-delivery-pipeline`,
    * Click `Save Integration`,
    * In the left menu, click the `Connections` link, you should see your Kubernetes cluster linked here,
    * Go back to the `Overview` page,


4. Review and Debug the Toolchain Configuration,
    * The `THINK` window should link to your Issues manager,
    * The `CODE` window should link to your source code repository,
    * The `Eclipse Orion Web IDE` should link to an online Eclipse code editor,
    * Review the `DELIVER` window,
        * The `Delivery Pipeline` page will load,
        * Review the `BUILD` stage,
            * The initial `BUILD` stage failed,
            * In the `JOBS` section, click the `View logs and history` link,
            * Review the build logs, and note that the `Dockerfile` was not found,
            * Also in the build logs, note the environment variables that were configured

                ```
                REGISTRY_URL=us.icr.io
                REGISTRY_NAMESPACE=<username>-ns
                IMAGE_NAME=guestbook
                ARCHIVE_DIR=.
                DOCKER_ROOT=.
                DOCKER_FILE=Dockerfile
                build.properties : not found
                ```

            * The Dockerfile for the guestbook application is located in the `v1/guestbook` and `v2/guestbook` subdirectories, so the reference needs to be set to include the relative path,
            * Go back to the `Delivery Pipeline` page, the `BUILD` window, click the `Configure Stage` icon,
            * In the `Jobs` tab, review the `Build script` code. Note that this is essentially a Bash script that enforces pre-requisites to run the builder of type `Container Registry` set in the `Builder type` property, 
            * Go to the `Environment properties` tab of the `BUILD` stage configuration,
            * Change the `DOCKER_ROOT` property to `./v2/guestbook`,
            * Manually trigger the `BUILD` stage again by clicking the `Play` icon,
            * The `BUILD` stage passes now,
            * Go to https://cloud.ibm.com/kubernetes/clusters, go to `Registry` and `Images`,
            * You should see now that your `guestbook` image has been added to your registry,
        
        * Review the `VALIDATE` stage,
            * You see that the stage passed but that the `Vulnerability Advisor` job failed,
            * In the `JOBS` section, click the `View logs and history` link,
            * Review the logs,

                ```console
                Checking vulnerabilities in image: us.icr.io/remkohdev-ns/guestbook:3-master-5246a420-20190723023204
                parse error: Invalid numeric literal at line 2, column 0
                FAILED
                ...
                parse error: Invalid numeric literal at line 2, column 0

                Finished: ERRORED
                ```

            * TBD for now ignore the warning,

        * Review the `PROD` stage,
            * In the `JOBS` section, click the `View logs and history` link,
            * Review the logs,

                ```console
                CHECKING DEPLOYMENT.YML manifest
                Kubernetes deployment file 'deployment.yml' not found

                Finished: FAILED
                ```

            * The script could not find the deployment resource because it is not in the root directory, but it is located in the `./v2` subdirectory.
            * Go to the `DELIVER` stage, 
            * In the Delivery Pipeline page, in the `PROD` stage, click the `Configure stage` icon,
            * Go to the `Environment properties` tab,
            * Change the `DEPLOYMENT_FILE` property to `./v2/guestbook-deployment.yaml`,
            * Click `Save`,
            * Run the `PROD` stage again by clicking the play icon,
            * Your deployment to Kubernetes of the deployment resource should succeed,
        
5. Check your Kubernetes cluster, in the `prod` namespace, you should see your deployment running,

