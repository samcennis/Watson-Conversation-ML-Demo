# Watson Conversation + Watson Machine Learning Demo

A predictive chatbot to give personalized fitness class recommendations for members of a fictional gym chain, Fitness Club USA. The chatbot uses several IBM Cloud services, including:

1. Watson Machine Learning
2. Watson Conversation
3. Node-RED
4. Data Science Experience
5. Cloudant
6. Watson Tone Analyzer (optional)

Note: This application is a simple starting point “proof-of-concept” and not viable for production.

Follow these instructions to get things up and running:

## Set up Node-RED

1. Sign up for an IBM Cloud account here: https://console.bluemix.net/registration/
2. Once you’re logged in, provision a Node-RED starter application here: https://console.bluemix.net/catalog/starters/node-red-starter
    1. Enter a unique “App name” (refered throughout the instructions as *your-app-name*) and select “Create” to provision the Node-RED starter application.
    2. Set up your local development environment
        1. From the “Getting Started” page of the Node-RED starter application, (you should be taken her automatically) follow the instructions under “Customizing your Node-RED instance” in order to install the Cloud Foundry command line interface, download and extract the starter code locally, then deploy back to IBM Cloud with “cf push”.
    3. After waiting for the app to deploy and start, it will be running at *your-app-name*.mybluemix.net and you can visit it
3. After your app is up and running (It will take a few minutes. You should see a confirmation of “running” state on the command line), visit *your-app-name*.mybluemix.net/ to get started with the Node red editor.
4. Follow the steps in the menu
    1. I recommend securing your editor with a username and passcode, (remember this…it’s important) select Next
    2. You do not need any additional nodes right now, so select Next
    3. Select “Finish”
    4. On the new screen that appears, select “Go to your Node-RED flow editor”
5. Add in my Node-RED flow
    1. Visit *your-app-name*.mybluemix.net/red and log in to your editor. You should then be at the flow editing page. You can take a look at all the connectors to the left… pretty cool!
    2. Select the sandwich button in the top right to get a drop down menu
        1. Select Import > Clipboard
        2. Copy the text from the setting-up-yourself/Final-Node-RED-Flow.json file in this Github repo and paste it into the “Paste nodes here” section. You should see a new tab called “Final Flow”. click on this.
        3. Awesome! The final flow is now there. It won’t work yet though because you haven’t set up the Watson Conversation, Watson Machine Learning, and Watson Tone Analyzer services and gotten your new credentials. You will also need to populate the Cloudant database (automatically provisioned with the Node-RED starter) with a couple of customer records. Leave Node-RED open in a tab.

## Set up Data Science Experience and Watson Machine Learning

1. From the IBM Cloud catalog, create your Data Science Experience service here: https://console.bluemix.net/catalog/services/data-science-experience
    1. Select “Get Started” to enter Data Science Experience
2. In the top right select New>Project
    1. Enter a project name and description, then “Create”
3. Before you can build your model, you need to create a Watson Machine Learning service instance. Select “Settings” in the top menu.
    1. Add a new WML service under Associated services (you should only see Spark right now), select “New Service”>”Machine Learning”. Then select New service.
    2. Click on the “Free” price plan, then “Buy IBM Watson Machine Learning”. This will provision it and you will also be able to see it in your same IBM Cloud organization/space. Select “Confirm”
    3. Excellent. You now have a Watson Machine Learning instance associated with your Data Science Experience project.

## Create and Deploy the Predictive “Fitness Class Recommendation” Model

1. Scroll up to the top of the page and select “Analytics assets”
2. Next to the “Models” list, you should see “New model” to enter the model builder interface
    1. Now add a name “Fitness Model”, ensure the Machine Learning Service you should created is selected, then select “Manual”, then Create. Wait.
3. Select Data
    1. Select “Add Data Assets”, then “Browse”
    2. Download the .csv file from this Github repo — setting-up-yourself/Fitness_Data_1.csv and upload it. It should appear as a data asset you can select. Select the radio button by it and then “Next”. You should see “Loading data”
4. Train (Select a technique)
    1. Select FITNESS_CLASS_NAME as the Column value to predict. Keep Feature columns as All
    2. Multi class Classification should be the suggested technique.
    3. Select Add Estimators, and select Random Forest, Decision Tree, and Naive Bayes
    4. Select “Next”. Wait for your models to train. This could take a minute.
5. Evaluate
    1. Select whichever model you want that had “Good” performance, then “Save”
6. Deploy the model to a RESTful endpoint
    1. From the model overview page, select “Deployments”, then “Add Deployment”
        1. Deployment type = online, give it a name like “Fitness Model Deployment”. Select “Deploy”
    2. Click on the new name of your new deployed model to see details
        1. You can now see the “Scoring end point”. This is the RESTful endpoint you can send an HTTP POST request to in order score your model.
        2. Copy the scoring endpoint to your clipboard.
7. Add Watson ML Scoring Endpoint and Credentials to Node-RED
    1. Go back to your Node-RED “Final Flow”.
        1. Find the node that says “WML - SparkML”, and replace the URL with the “Scoring end point” on your clipboard. This will tell Node-RED where to send the WML requests.
    2. Add Credentials for Watson ML to Node-RED
        1. Go to your IBM Cloud dashboard - https://console.bluemix.net/dashboard/apps/
            1. Under services, find the Machine Learning service you created, and click on it.
            2. In the menu on the left, select “Service credentials”
                1. Select “New credential”, then “Add”
                2. Select “View credentials” for your new credentials (likely named “Credentials-1”). This will show your Basic Authentication username and password for Watson Machine Learning endpoints, which we need to give to Node-RED.
    3. Go back to your Node-RED “Final Flow”
        1. Find the node that says “WML - Get Auth Key”
        2. Select “use basic authentication”. Then copy the Username and Password from the WML credentials into the fields.

## Create Watson Conversation Service and Import Conversation

1. From the IBM Cloud Catalog, create your Watson Conversation service here: https://console.bluemix.net/catalog/services/conversation
    1. Select “Create”
    2. After its created, select “Launch tool”
2. Import the conversation
    1. Next to the Create button there is an upward point arrow for importing a workspace. Select that.
    2. Choose the file setting-up-yourself/conversation-workspace.json, and import everything
3. Copy credentials and workspace ID to Node-RED
    1. There’s a “loopy arrow” in the left menu with the hover message “Deploy”. Click on that.
    2. Select “Credentials at the top”, wait for Workspace Details and Service Credentials to load
        1. You should see a Workspace ID, Username, and Password on the page
    3. Go back to your Node-RED Final Flow and copy all three of these into the appropriate fields in the “Watson Conversation API” node.

## Deploy the Node-RED Flow

1. Select "Deploy" in the upper right portion of the screen in order to deploy your Node-RED flow. This will now make this flow callable as a RESTful endpoint at *your-app-name*.mybluemix.net/api/message (the endpoint defined in the HTTP Request node) .

## Add a Customer Record to Cloudant NoSQL database

1. From your IBM Cloud dashboard, you should be able to find “*your-app-name*-cloudantNoSQLDB” listed under services. This is the Node-RED starter’s NoSQL database that was automatically provisioned. We will all use this as our customer records database to allow our app to pull up customer information. Click on the database, then select “Launch” to open the database dashboard.
2. Create “customer_info” database.
    1. From the Cloudant dashboard, select the second button down on the left to get to your Databases. There should just be one right now called nodered. We want to make a new one called “customer_info”.
    2. Select “Create Database” in the top right of the screen, enter the name of “customer_info”, then “Create”
3. Add a documents to the database for a customers.
    1. Select “Create Document”, and then copy and paste the JSON from the setting-up-yourself/customer-example.json file. Make sure to paste over anything that is already in the window.
4. If you want to add more customers, feel free to do so. Note that the `_id` field is populated with a phone number and must be unique.

## Add the chatbot front-end to your Node-RED application.

1. To add the “front-end” chat interface, simply replace the /public folder with the one in this repository.
2. Deploy again using “cf push” and wait for the app to deploy
3. When you visit *your-app-name*.mybluemix.net, you should now have a pretty slick lookin' UI!

## Optional — add in Watson Tone Analyzer and Twilio

With just a couple of more changes, you can add in two more IBM Cloud services to the application -- Watson Tone Analyzer and Twilio.

The Watson Tone Analyzer portion of the application will keep an eye on the tone of the user's text throughout the conversation. The user will be told that they are being routed to a "live agent" if they have significant levels of anger in their text.

Twilio adds the functionality of sending a text message to the user with information about the first fitness class.

You can add either one

### Watson Tone Analyzer

The Node-RED flow you imported has a disconnected portion with a Watson Tone Analyzer API. Here are the steps to incorporate it.

1. Add the Watson Tone Analyzer service from the IBM Cloud Catalog and get the credentials for your service after you create it.

https://console.bluemix.net/catalog/services/tone-analyzer

2. Add these credentials to the Watson Tone Analyzer node in your Node-RED flow.
3. Wire the Watson Tone Analyzer node into the flow:
    1. Delete the wire between “Prep Input Message” and “Watson Conversation API”
    2. Add a wire between the output of “Prep Input Message” to the input of the Watson Tone Analyzer node
    3. Add a wire from the SECOND output of “Customer Angry?” to the input of “Watson Conversation API”

3. And voila! Your customer will be told they’re being routed to a “live agent” if their “anger” score breaches the threshold.

### Twilio

The Node-RED flow you imported already has the Twilio portion wired within it. However, that branch of the flow will not be executed because the Watson Conversation dialog you imported from JSON does not have a dialog node with the ID that the Node-RED switch node is checking for `node_4_1508179653724`. (Double click on the Node-RED "What Action?" node to see how the Node-RED flow checks to see where it is in the conversation.) It will just be a matter of creating the Twilio service and importing an updated Watson Conversation dialog that will trigger the Twilio branch.

1. Create the Twilio service. Twilio is a 3rd party (non-IBM) service, so you will need to set up a trial account and obtain credentials on their site. Select "Register at Twilio" and complete the steps to create a trial account.
https://console.bluemix.net/catalog/services/twilio

2. You will need to create a new project, then obtain an Account SID, Twilio Phone Number, and Auth Token from Twilio. If you have any trouble, see Twilio's documentation for more information on how to get started and obtain these.

3. Double click on the Node-RED Twilio node labeled "Send Pass Via Text Message", then click on "add new Twilio API". Put the 3 pieces of information you obtained into this to set up your Twilio API connection. (Your "Twilio Phone Number" goes in the "From" field)

4. Selected "Add" to create the new Twilio API connection. Then within "node properties", update the "SMS to" field to the number that you would like to send the pass to each time. The app will text the same number every time. With a little bit of code in the "Prep Text Message" this could be customized, but that will be left as an exercise.

5. Go to your Watson Conversation service, and this time import the setting-up-yourself/conversation-workspace-ADD-TWILIO.json file into a new Workspace. This will import the updated dialog flow that allows the Twilio branch to execute in the Node-RED flow. Get your new Watson Conversation workspace_id and copy it into the "Watson Conversation API" Node-RED node, replacing the previous value. (Keep your Username and Password the same.)
