
## initialize.py

This script queries your Dash Enterprise host for your target apps' settings 
using your Service Account credentials. Your Developer Account will then 
initialize a review app and apply the settings it needs to inherit. We base 
the inherited settings on the  following returned metadata:

- apps_name 
- apps_owner 
- apps_status 
- current_isAdmin 
- apps_collaborators 
- apps_permissionLevels 
- apps_linkedServices 
- apps_mounts 
- apps_environmentVariables

We then initialize the review app with the developer's credentials due to Dash 
Enterprise's collaborative work's current limitations. An app can have a single 
owner. Proceeding in this manner enables the owner the actioning the review app 
the ability to make changes.

We filter out base environment variables from our initial query results since 
they are already added to all dash apps deployed on the server.
The remaining environment variables are added to the review app.
Aside from the Review App initialization, all of the other requests are made 
by the host Service Account. 

We then add the same services and service types to the Review app. These service 
names follow a name pattern that includes the target app name, service name and 
service type.

We filter out base environment variables from our initial query results since 
they are already added to all dash apps deployed on the server.
The remaining environment variables are added to the review app.

-----