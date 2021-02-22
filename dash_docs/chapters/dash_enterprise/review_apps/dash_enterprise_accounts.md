### Dash Enterprise Server

* Creating a Service Account
* Adding Service Account SSH Key
* Retrieving Service and Developer API Key

#### Service Account

This Account is responsible for making all of the changes triggered by the deployment helper script. 

See [Continuous Integration](/dash-enterprise/continuous-integration) for more details.

##### Creating a Service Account

1. Navigate to your Dash Enterprise Portal
2. Log in as **Admin**
3. Navigate to the **App Manager** page
4. Click on the **Users** tab
   1. Click on the **Add User** button — a modal will appear
   2. Fill out the user information for your *Service Account* and click on the **Add User** button
5. Select the user you just added from the list at the bottom of the page — a sidebar will appear
6. Enable **Admin** privileges in **Permissions**

##### Adding Service Account SSH Keys

1. Create an SSH key. 
2. Navigate to the [Manage SSH Keys](/Manager/ssh-keys) page
3. Add SSH key 

See [Authenticating to Dash Enterprise with SSH](/Docs/dash-enterprise/ssh) for more details.

##### Retrieving Service Account API Key

1. Navigate to [Manage API Key](/Manager/auth/api_key) page
2. Click on the **Reset** button — a modal will appear with your **API Key**

#### Developer Account

You will need to retrieve the **API Keys** of  your **Developer Users**.  We 
recommend resetting their current key.