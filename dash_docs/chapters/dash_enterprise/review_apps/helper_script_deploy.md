#### deploy.py

A Review App relies on a few bash commands for deployment. 

First,  it needs to obtain the private SSH key belonging to the Dash Enterprise
Service Account and SSH configuration file,  both made available as environment
variables on the CI platform.  

The SSH key and SSH configuration file are written to new files, the SSH key's permission gets an update and is added to the key manager.  Finally,
GIT gets configured, and the review app is deployed. After deployment, the review app's Permission Levels get an update to match those belonging to its production counterpart.

If the production app owner does not have admin privileges, they are added as a collaborator for the review app. 

This Python-wrapped bash script carries out the deployment. The encoded 
a private SSH key is obtained from the environment variables, then 
converted from base 64 into its original form and is added
to the CI platform environment to authenticate the deployment.

After deploying the Review App, permission levels are updated to match those
belonging to its production counterpart.
The production app creator is added as a collaborator for the review app
if they do not already have admin access.
