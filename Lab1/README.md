# Toolchain 1 - Deploy a Kubernetes App

1. Create a Toolchain,
    * Go to https://cloud.ibm.com/devops/toolchains
    * Click the `Create a Toolchain`,
    * Select the `Develop a Kubernetes app` template,
    
2. Configure the Toolchain, for:

    * Toolchain Name: `toolchain-kube-guestbook`,
    * Region: <default>
    * Resource Group: `Default`
    * Configure your Source Provider: `Github`,
        * Github Server: `https://github.com`,
        * Repository Type: `Existing`
        * Source repository URL: `https://github.com/<username>/guestbook`, you should be able to select the drop down and scroll to find your fork of the `guestbook` repo,
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
                * This should trigger the validation of the API Key, and if validated, should autoload the values for `Container registry region`, `Container registry namespace` or if no namespace exists enter `guestbook-ns` as the namespace, `Cluster region`, `Resource Group`, `Cluster name` or if more than one select the correct cluster, and `guestbook-ns` for `Cluster namespace`,
                * Note that the `Cluster namespace` must be unique for us.icr.io
    * Click the `Create` button,

    * The Toolchain is being configured,
    * When the Toolchain has successfully been configured, you will see the Tools in the Toolchain: THINK, CODE, DELIVER, and Eclipse Orion Web IDE,
    * Click the top right drop down of the `kube-toolchain-123456...` and select `Rename`,
    * Rename the toolchain name to `toolchain-kube-guestbook`,
    * In the DELIVER tool window, click the top right dropdown and select `Configure`,
    * Rename the `Pipeline name` to a meaningful name, e.g. `guestbook-delivery-pipeline`,
    * Click `Save Integration`,
    * Go back to the `Overview` page,


4. Review and Debug the Toolchain Configuration,
    * The `THINK` window should link to your Issues manager,
    * The `CODE` window should link to your source code repository,
    * The `Eclipse Orion Web IDE` should link to an online Eclipse code editor,
    * Click the `DELIVER` window to review the `Delivery Pipeline`,
        * The `Delivery Pipeline` page will load,
        * Click the `BUILD` stage,
            * The initial `BUILD` stage passes,
            * In the `JOBS` section, click the `View logs and history` link,
            * The `BUILD` stage fetched the source code and tried running the unit tests. The test runner script was not found but the tests did not result in failed tests. (Note: this should fail however)
            * Return to the `guestbook-delivery-pipeline`,
        * The `CONTAINERIZE` stage failed,
            * In the `JOBS` section, click the `View logs and history` link,
            * Review the build logs, and note that the `Dockerfile` was not found,
            * Also in the build logs, note the environment variables that were configured

                ```
                DOCKER_ROOT=.
				DOCKER_FILE=Dockerfile
				build.properties:
				GIT_URL=https://github.com/<username>/guestbook.git
				GIT_BRANCH=master
				GIT_COMMIT=5246a420041424130f32d292cca7fc7a99aa0b93
				SOURCE_BUILD_NUMBER=1
                ```

            * The Dockerfile for the guestbook application is located in the `v1/guestbook` and `v2/guestbook` subdirectories, so the reference needs to be set to include the relative path,
            * Go back to the `Delivery Pipeline` page, in the `CONTAINERIZE` window, from the settings drop down, click the `Configure Stage` icon,
            * In the `Jobs` tab, note there are 4 jobs belonging to the stage, each job corresponds to a step in the `Build logs`,
            * Review the `Check dockerfile` job. Note that this is essentially a Bash script that runs the script located at `https://raw.githubusercontent.com/open-toolchain/commons/master/scripts/check_dockerfile.sh`,
            * If you review the bash script, it sets the `DOCKER_ROOT` environment variable to the current directory if not set, and this is the reason that our Dockerfile is not found, 
            * Go to the `Environment properties` tab of the `CONTAINERIZE` stage, 
            * Change the `DOCKER_ROOT` property to `./v2/guestbook`,
            * Click `SAVE`

            * Manually trigger the `CONTAINERIZE` stage again by clicking the `Play` icon,
            * The `Check dockerfile` job in the `CONTAINERIZE` stage should pass now,
            * If any of the next jobs fails, review the `View logs and history` link, and select the failed job, to debug,
            * Go to https://cloud.ibm.com/kubernetes/clusters, go to `Registry` and `Images`,
            * You should see now that your `guestbook` image added to your registry under the `guestbook-ns` namespace,
        
        * Review the `DEPLOY` stage,
            * You see that the stage failed for the `Deploy to Kubernetes` job,
            * Review the `View logs and history` link,
            * The `VALIDATE` failed because `Kubernetes deployment file 'deployment.yml' not found`,
            * This problem is caused because the current version of the [Open Toolchain commons](https://github.com/open-toolchain/commons) that uses a `check and deploy` script, does not support deployment files being located in a subdirectory. 
			* You can fix the failure by making changes to the bash script being run to deploy the application,
    			* The current fix for the failure is under review as Pull Request 35, https://github.com/open-toolchain/commons/pull/35 in the open-toolchain/commons repository,
    			* Copy the code in https://raw.githubusercontent.com/open-toolchain/commons/6c3bbd582e04d2bdcf8923692c9e995700966c5a/scripts/check_and_deploy_kubectl.sh
        		* In the Delivery Pipeline page, in the `DEPLOY` stage, click the `Configure stage` icon,
        		* In the Jobs tab, scroll down to the `Deploy script`
        		* Remove the current code and paste the copied code from the PR35 file,
        		* Scroll up again and go to the `Environment properties` tab,
        		* Add a new property `DEPLOYMENT_SUBDIRECTORY` with value `v2/`,
        		* Change value for property `DEPLOYMENT_FILE` to `guestbook-deployment.yaml`,
            * Click `Save`,
            * Run the `DEPLOY` stage again by clicking the play icon,
            * Your deployment to Kubernetes of the deployment resource should succeed,
        
5. Check your Kubernetes cluster, in the `guestbook-ns` namespace, you should see your deployment running, 

