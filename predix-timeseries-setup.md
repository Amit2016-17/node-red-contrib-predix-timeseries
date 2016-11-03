The simplest way to set up a stand alone time series microservice on predix is to use a placeholder "Hello World: front end app.

This involves:

1. Creating an application
2. Setting up a UAA instance
3. SEtting up the time series microservice

Once this is done you can ingest and query data though the API explorer tab in Predix Toolkit. 

# Setting up the Placeholder Application

1. Use the Cloud Foundry CLI to log into Cloud Foundry
        cf login -a <API-ENDPOINT>
                where the API-ENDPOINT could be one of these:
                    Predix Basic
                      https://api.system.aws-usw02-pr.ice.predix.io
                    Predix Select
                      https://api.system.asv-pr.ice.predix.io
                    Predix Japan
                      https://api.system.aws-jp01-pr.ice.predix.io
                    Predix UK
                      https://api.system.dc-uk01-pr.ice.predix.io

2. Clone the hello-world sample app from predix github to your local
        git clone https://github.com/PredixDev/Predix-HelloWorld-WebApp

3. Edit the manifest.yml file within the Predix-HelloWorld-Webapp with the following info 
        applications:
          - name: Predix-HelloWorld-WebApp-<YourAppName>
            buildpack: predix_openresty_buildpack
            memory: 64M
            stack: cflinuxfs2

4. Inside the Predix-HelloWorld-Webapp
        cf push

5. Verify the app is uploaded in Cloud Foundry by 
        cf apps
    you can also enter this in the browser to check the webpage
        https://Predix-HelloWorld-WebApp-<YourAppName>.run.aws-usw02-pr.ice.predix.io

# Set Up a UAA Instance

6. Now, create an UAA instance, the easy way would be enter this in the command terminal:
        cf create-service predix-uaa Tiered <your-name>-secure-uaa-instance -c '{"adminClientSecret":"theadminpassword"}'

7. Then, login to the Predix Developer Network console, go to <your-space> -> <service-instances> -> find the uaa that you just created, click on it.
    -You should see a "configure service instance" button on the right hand side, click on it
    -Now you will be prompted to enter the admin password for the uaa instance that you just created
    -Once you login, you will see your UAA url at the bottom at the dashboard, copy this url.

8. Now you would need to loggin the Predix Toolkit site.
    -Click "login as admin" on the left-hand panel, then Enter the UAA url that you just copy along with your admin password, click submit. You should see a response with token.
    -Click "Create a client Id", this will be also referred as app client id later on. Enter the client id with a client password, click submit and you should see a response with token. 
    -Click "create a user", this will create a user for you
    -You can verify by clicking "Login as user", enter credentials and submit. You should see a response with token.
    -Then you can click "check token", in which you should see the decoded token

# Set up a Time Series Microservice Instance

9. In the Predix Developer Network console page, go to catalog -> timeseries -> at the bottom of the page click "subscribe"

10. Now fill in the info for new servicve instance
    -org/space should be related to your account
    -UAA field should be the UAA instance that you just created in step 6
    -service instance name would be <your-name>-timeseries-instance for convention.
    -click create service and now you should see the new service instance under your console.

11. Your would need to bind the timeseries service with the placeholder app, go to the command terminal and enter:
    cf bind-service <your-name>-predix-nodejs-starter <your-name>-timeseries-instance

12. Now, edit manifest.yml file just like we did in step 3, add in the following fields:

    services:
        - <your-name>-secure-uaa-instance
        - <your-name>-timeseries-instance
    env:
        clientId:<client id  that you created in step 8>
        base64ClientCredential:<open the terminal, enter "echo –n app_client_id:secret | base64", copy the converted value to here>

    now save the yml file, and do a "cf push" again. Once it's done, verify with "cf env <your-app-name>". You will see the config of this app, and we will need these values for the next step.     
    
13. Now, go back to Predix Toolkit site, we need to add timeseries in the authorties.
    -login as admin like in step 8
    -click Get Client on the left panel, and you should see the info of the client
    -click "update client", where you will enter these three new authorities for the placeholder app. Note that you will find your timeseries zone id from the command "cf env <your-app-name>" that you did in step 12. 
        timeseries.zones.<your-timeseries-zone-id>.user
        timeseries.zones.<your-timeseries-zone-id>.ingest
        timeseries.zones.<your-timeseries-zone-id>.query 
    -validate the token by clicking "check token", you should see the newly added authorities in the token, with the correct timeseries zone id.
    
# Wrapping up

14. Now that the timeseries instance is set up and ready to use. You can ingest and query data though the API explorer tab in Predix Toolkit. 
   


