### Helper Script

The helper scripts are templates that can be modified.

#### settings.py

This script is responsible for storing all of the 

You will need to update the following variables in `settings.py` file before you will be able to create review apps:

- DASH_ENTERPRISE_HOST
- DASH_ENTERPRISE_USERNAME_TO_CIRCLECI_USERNAME
- DASH_ENTERPRISE_USERNAME_TO_CIRCLE_CI_API_KEY 
- SERVICE_API_KEY
- SERVICE_USERNAME
- SERVICE_PRIVATE_SSH_KEY
- SERVICE_PUBLIC_SSH_KEY

```python
import logging
import os

DEBUG = os.getenv("DEBUG", "false")
# Enable Requests debugging by setting this variable to "true"

if os.getenv("CIRCLECI") == "true":
    print("\nDEPLOYMENT ENV: CIRCLECI\n")

    PREFIX = "review-app-"
    # PREFIX is a filter for purging review apps.

    LAST_UPDATE = {"minutes": 1}
    # LAST_UPDATE is the allowed amount of time before review apps are 
    # purged from the server.
    
    TRUNK_BRANCHNAME = "main"
    # TRUNK_BRANCHNAME is the name of your repository's "main" or "master" 
    # branch.

    BRANCHNAME = os.getenv("CIRCLE_BRANCH")
    # BRANCHNAME is the name of the branch you will initialize your Dash apps 
    # from. This will be pulled from the CircleCI's environment. BRANCHNAME 
    # must not exceed 30 characters in length.

    REPONAME = os.getenv("CIRCLE_PROJECT_REPONAME")
    # REPONAME is the name of the repository that will hold the review app 
    # branches.

    TARGET_APPNAME = "chris-app"
    # TARGET_APPNAME is the name the Dash App that will serve as a review app
    # template. This script will copy that apps configuration settings and 
    # apply them to all review apps. App must exist and you must have 
    # permission to view the TARGET_APP or the app's initialization will fail.

    APPNAME = f"{TARGET_APPNAME}-rev-{BRANCHNAME}"[0:30]
    # APPNAME determines how your review apps will be named. APPNAME must not
    # exceed 30 characters in length.

    DASH_ENTERPRISE_HOST = "qa-de-410.plotly.host" 
    # DASH_ENTERPRISE_HOST = "dash-playground.plotly.host"
    # DASH_ENTERPRISE_HOST is your Dash Enterprise Server's host address.

    SERVICE_API_KEY = os.getenv("ADMIN_API_KEY")
    # SERVICE_API_KEY is the "Machine User's" API key used to access your Dash
    # Enterprise server.

    SERVICE_USERNAME = "admin"
    # SERVICE_USERNAME is a "Machine User" that will handle all aspects of Dash 
    # app management.

    USERNAME = os.getenv("CIRCLE_USERNAME")
    # USERNAME is the GitHub login of the user who pushed the code to Github.
    # It is mapped to a Dash Enterprise username that will author the 
    # initialized apps. (Make sure that this user has permission to view 
    # TARGET_APP or the app's initialization will fail)

    DASH_ENTERPRISE_USERNAME_TO_CIRCLECI_USERNAME = {
        "criddyp" : "criddyp",
        "tobinngo": "tobinngo",
    }
    # DASH_ENTERPRISE_USERNAME_TO_CIRCLECI_USERNAME maps your developer 
    # usernames to the GitHub username used to push changes to your review app
    # repository.

    DASH_ENTERPRISE_USERNAME_TO_CIRCLE_CI_API_KEY = {
        "criddyp": "CRIDDYP_API_KEY",
        "tobinngo": "TOBINNGO_API_KEY",
    }
    # DASH_ENTERPRISE_USERNAME_TO_CIRCLE_CI_API_KEY maps your developer 
    # usernames to their corresponding API key stored as environment variable 
    # in your CircleCI Project Settings.

    SSH_CONFIG = f"Host {DASH_ENTERPRISE_HOST},    HostName {DASH_ENTERPRISE_HOST},    User {SERVICE_USERNAME},    Port 3022,    IdentityFile ~/.ssh/id_rsa,    StrictHostKeyChecking no,    UserKnownHostsFile /dev/null"
    # SSH_CONFIG contains your SSH settings for Dash app deployment.

    SERVICE_PRIVATE_SSH_KEY = os.getenv("ADMIN_PRIVATE_SSH_KEY")
    # SERVICE_PRIVATE_SSH_KEY belongs to a Dash Enterprise user with admin
    # privileges. This user will handle all of the server deployment tasks.

    SERVICE_PUBLIC_SSH_KEY = os.getenv("ADMIN_PUBLIC_SSH_KEY")
    # SERVICE_PUBLIC_SSH_KEY used to authenticate the SSH host

    if (
        USERNAME in DASH_ENTERPRISE_USERNAME_TO_CIRCLECI_USERNAME and 
        DASH_ENTERPRISE_USERNAME_TO_CIRCLE_CI_API_KEY
    ):
        print("Fetching API key...")
        
        USERNAME_API_KEY = (
            os.getenv(
                DASH_ENTERPRISE_USERNAME_TO_CIRCLE_CI_API_KEY.get(USERNAME)
            )
        )
    else:
        print("API key was not fetched")
        print(
            f"""
            {USERNAME} is missing from 
            DASH_ENTERPRISE_USERNAME_TO_CIRCLECI_USERNAME and
            DASH_ENTERPRISE_USERNAME_TO_CIRCLE_CI_API_KEY dictionaries.

            See Getting Started section in Continuous Integration Docs
            (https://{DASH_ENTERPRISE_HOST}/Docs/continuous-integration)
            for more information or contact your Dash Enterprise
            administrator.
            """
        )
    # Verifies that relevant usernames are found in both dictionaries
print(" ")
```

#### initialize.py

This script queries your Dash Enterprise host for your target apps' settings using your Service Account credentials. Your Developer Account will then initialize a review app and apply the settings it needs to inherit.

```python
import logging
import os
import sys
from time import sleep
from gql import gql, Client
from gql.transport.requests import RequestsHTTPTransport
from settings import (
    DASH_ENTERPRISE_HOST,
    USERNAME,
    USERNAME_API_KEY,
    SERVICE_API_KEY,
    SERVICE_USERNAME,
    APPNAME,
    TARGET_APPNAME,
    DEBUG,
)

if sys.version_info[0] < 3.6 and sys.version_info[0] > 3.7:
    raise Exception("Python 3.6 is required.")

if DEBUG == "true":
    logging.basicConfig(level=logging.DEBUG)

transport_service = RequestsHTTPTransport(
    url=f"https://{DASH_ENTERPRISE_HOST}/Manager/graphql",
    auth=(SERVICE_USERNAME, SERVICE_API_KEY),
    use_json=True,
)

transport_user = RequestsHTTPTransport(
    url=f"https://{DASH_ENTERPRISE_HOST}/Manager/graphql",
    auth=(USERNAME, USERNAME_API_KEY),
    use_json=True,
    
)

client_service = Client(transport=transport_service)
client_user = Client(transport=transport_user)

def zip_list_index(l, a, b):
    k = [l[i][a] for i in range(len(l))]
    v = [l[i][b] for i in range(len(l))]
    return dict(zip(k,v))

def handle_error(result, er=None):
    """
    Raise error if error is not an accepted error
    """
    if er != None:
        for k, v in accepted_errors.items():
            if k in result and "error" in result[k]:
                if result[k]["error"] in v:
                    pass
                else:
                    raise Exception(result[k]["error"])

addService_errors = [
    "A service with the given name already exists. Please choose a different name.",
    None,
]

apps_query_errors = [
    "[]",
]
updateApp_errors = [
    "None is not a valid PermissionLevels",
    None,
]
addApp_errors = [
    "An app with this name already exists in this Dash Server. Please choose a different name.",
    None,
]

accepted_errors = {
    "addApp": addApp_errors,
    "updateApp": updateApp_errors,
    "addService": addService_errors,
    "apps": apps_query_errors,
}

print("Querying target app...")
# Querying target app settings
query = gql(
    """
    query (
        $name: String
    ) {
        current {
            username
            isAdmin
        }
        apps(
            name: $name, 
            allApps:true,
        ) {
            apps {
                name
                owner {
                    username
                }
                status {
                    running
                }
                collaborators {
                    users
                    teams
                }
                metadata {
                permissionLevel
                }
                linkedServices {
                    name
                    serviceType
                }
                mounts {
                    targetDir
                    hostDir
                }
                environmentVariables {
                    name
                    value
                }
            }
        }
    }
    """
)
params = {"name": TARGET_APPNAME}
result = client_service.execute(query, variable_values=params)
handle_error(result, accepted_errors)

if len(result["apps"]["apps"]) != 0:
    print("Initializing review app...")
    apps = result["apps"]["apps"]
    apps_name = result["apps"]["apps"][0]["name"]
    apps_owner = result["apps"]["apps"][0]["owner"]["username"]
    apps_status = result["apps"]["apps"][0]["status"]["running"]
    current_isAdmin = result["current"]["isAdmin"]
    apps_collaborators = result["apps"]["apps"][0]["collaborators"]
    apps_permissionLevels = result["apps"]["apps"][0]["metadata"]
    apps_linkedServices = result["apps"]["apps"][0]["linkedServices"]
    apps_mounts = result["apps"]["apps"][0]["mounts"]
    apps_environmentVariables = result["apps"]["apps"][0]["environmentVariables"]

    a = [apps[i]["name"] for i in range(len(apps))]
    b = [apps[i]["owner"]["username"] for i in range(len(apps))]

    owner = dict(zip(a, b))
    permissionLevels = apps_permissionLevels
    linkedServices = zip_list_index(apps_linkedServices, "serviceType", "name")
    mounts = zip_list_index(apps_mounts, "hostDir", "targetDir")
    environmentVariables = zip_list_index(apps_environmentVariables, "name", "value")

    query = gql(
        """
        mutation (
            $appname: String
        ) {
            addApp(
                name: $appname
            ) {
                error
            }
        }
        """
    )
    params = {"appname": APPNAME}

    result = client_user.execute(
        query, 
        variable_values=params
    )
    handle_error(result, accepted_errors)
    print(f"  {APPNAME}")
else: 
    print("Review app not initialized")
    print(f"\n  App {TARGET_APPNAME} does not exist.\n")
    raise Exception(result)



if len(linkedServices.items()) == 1:
    print("Adding service...",)
elif len(linkedServices.items()) > 1:
    print("Adding services...",)
    for k in linkedServices:
        serviceName = f"{APPNAME}-{k}"[0:30]
        query_addService = gql(
            """
            mutation (
                $serviceType: ServiceType=redis,
                $serviceName: String
            ) {
                addService (
                    name: $serviceName,
                    serviceType: $serviceType
                ) {
                    error
                }
            }  
            """
        )
        params_addService = {
            "serviceName": serviceName, 
            "serviceType": k, 
        }
        sleep(5)
        result = client_service.execute(
            query_addService, 
            variable_values=params_addService
        )
        handle_error(result, accepted_errors)

        print(f"  {serviceName}, {k}")
else:
    print("Services not added")

if len(linkedServices.items()) == 1:
    print("Linking service...",)
elif len(linkedServices.items()) > 1:
    print("Linking services...",)
    for k in linkedServices:
        query_linkService = gql(
            """
            mutation (
                $appname: String
                $serviceType: ServiceType=redis,
                $serviceName: String
            ) {
                linkService (
                    appname: $appname,
                    serviceName: $serviceName, 
                    serviceType: $serviceType
                ) {
                    error
                }
            }
            """
        )
        params_linkService = {
            "appname": APPNAME,
            "serviceName": serviceName, 
            "serviceType": k
        }

        sleep(5)
        result = client_service.execute(
            query_linkService, 
            variable_values=params_linkService
        )
        handle_error(result, accepted_errors)

        print(f". {serviceName}, {k}")
else:
    print("Services not linked")
    
if len(mounts.items()) != 0:
    print("Mapping directories...")
    for k, v in mounts.items():
        query = gql(
            """
            mutation (
                $hostDir: String, 
                $targetDir: String,
                $appname: String
            ) {
                mountDirectory(
                    hostDir: $hostDir,
                    targetDir: $targetDir, 
                    appname: $appname
                ) {
                    error
                }
            }
            """
        )
        params = {
            "hostDir": k,
            "targetDir": v,
            "appname": APPNAME,
        }

        result = client_service.execute(query, variable_values=params)
        handle_error(result, accepted_errors)

        print(f"  Mapping hostDir: {k} to targetDir: {v}")
else:
    print("Directories not mapped")

if len(environmentVariables.items()) != 0:
    print("Adding environment variables...")
    for k, v in environmentVariables.items():
        environmentVariables_filter = tuple(
            [
                "DOKKU",
                "DASH",
                "DATABASE_URL",
                "GIT_REV",
                "REDIS_URL",
                "SCRIPT_NAME",
                "NO_VHOST",
            ]
        )
        if k.startswith(environmentVariables_filter) != True:
            query = gql(
                """
                mutation (
                    $environmentVariable: String, 
                    $value: String, 
                    $appname: String
                ) {
                    addEnvironmentVariable (
                        name: $environmentVariable,
                        value: $value,
                        appname: $appname
                    ) {
                        error
                    }
                }
                """
            )
            params = {
                "environmentVariable": k, 
                "value": v, 
                "appname": APPNAME
            }
            result = client_service.execute(query, variable_values=params)
            handle_error(result, accepted_errors)

            print(f"  {k} :", 10 * "*")
else:
    print("Environment variables not added")

print("\n")

```

#### deploy.py

This script deploys the initialized Review App to your Dash Enterprise Host.

```python
import os
import subprocess
import base64
from settings import (
    SERVICE_PRIVATE_SSH_KEY,
    SERVICE_PUBLIC_SSH_KEY,
    SSH_CONFIG,
    DASH_ENTERPRISE_HOST,
    APPNAME,
    TARGET_APPNAME,
    BRANCHNAME,
    TRUNK_BRANCHNAME,
    REPONAME,
)
from initialize import (
    gql,
    RequestsHTTPTransport,
    Client,
    apps_permissionLevels,
    permissionLevels,
    client_service,
    handle_error,
    accepted_errors,
    current_isAdmin,
    apps_status,
    apps_owner,
)

if os.getenv("CIRCLECI") == "true":
    print(f"Deploying review app...\n")
    if TRUNK_BRANCHNAME == BRANCHNAME:
        deploy_appname = TARGET_APPNAME
    else: 
        deploy_appname = APPNAME
    subprocess.run(
        f"""
        echo "{SERVICE_PRIVATE_SSH_KEY}" | base64 --decode -i > ~/.ssh/id_rsa
        chmod 600 ~/.ssh/id_rsa
        eval "$(ssh-agent -s)"
        ssh-add ~/.ssh/id_rsa
        echo {SSH_CONFIG} | tr ',' '\n' > ~/.ssh/config
        git config remote.plotly.url >&- || git remote add plotly dokku@{DASH_ENTERPRISE_HOST}:{deploy_appname}
        git push --force plotly HEAD:master
        """, shell=True
    )

    print("\n")

    if permissionLevels.items() != 0:
        print(f"Updating PermissionLevel...")
        query = gql(
            """
            mutation (
                $appname: String,
                $permissionLevel: PermissionLevels
            ) { 
                updateApp(
                    appname: $appname, 
                    metadata: {
                        permissionLevel: $permissionLevel
                    }
                ){
                    error
                }
            }
            """
        )
        params = {
            "permissionLevel": permissionLevels["permissionLevel"],
            "appname": APPNAME
        }
        result = client_service.execute(query, variable_values=params)
        handle_error(result, accepted_errors)
    else:
        print("PermissionLevel not updated")

    if permissionLevels["permissionLevel"] == "restricted" and apps_status == "true":
        print("Adding collaborator...")
        query = gql(
        """
        mutation (
            $appname: String,
            $users: [String],
        ) { 
            addCollaborators(
                appname: $appname, 
                users: $users,
            ){
                error
            }
        }
        """
        )
        params = {"appname": APPNAME, "users": apps_owner}
        result = client_service.execute(query, variable_values=params)
        handle_error(result, accepted_errors)

        print(f"  {apps_owner}")
    else:
        print(f"No collaborators added")

    print(
        f"""
        You Dash app has been deployed. 
        
        Preview {APPNAME}:
        
        https://{DASH_ENTERPRISE_HOST}/{APPNAME}/
        https://{DASH_ENTERPRISE_HOST}/Manager/apps/{APPNAME}/settings
        https://app.circleci.com/pipelines/github/plotly/{REPONAME}?branch={BRANCHNAME}
        """
    )
else:
    print("App not deployed")
    raise Exception(
        f"""
        
        Deployment not authorized from this environment.
        Must push from main/master branch in
        CIRCLECI.

        See Getting Started section in Continuous Integration Docs
        (https://{DASH_ENTERPRISE_HOST}/Docs/continuous-integration)
        for more information or contact your Dash Enterprise
        administrator.    
        """
    )

```

#### delete.py

This script purges Review Apps and their associated services once 
they have been merged into production. 



```python
# Run this script with a scheduler

import logging
import os
import sys
from gql import gql, Client
from gql.transport.requests import RequestsHTTPTransport
from datetime import datetime, timedelta
from time import sleep
from settings import (
    DEBUG,
    DASH_ENTERPRISE_HOST,
    SERVICE_USERNAME,
    SERVICE_API_KEY,
    SSH_CONFIG,
    DEBUG,
    PREFIX,
    LAST_UPDATE,
)

if sys.version_info[0] < 3.6 and sys.version_info[0] > 3.7:
    raise Exception("Python 3.6 is required.")

if DEBUG == "true":
    logging.basicConfig(level=logging.DEBUG)

transport = RequestsHTTPTransport(
    url=f"https://{DASH_ENTERPRISE_HOST}/Manager/graphql",
    auth=(SERVICE_USERNAME, SERVICE_API_KEY),
    use_json=True,
    retries=5,
    
)

client = Client(transport=transport)

deleteApp_errors = [
    "App does not exist.",
    "Invalid app name. Names should be between 3 to 30 characters long, start with a letter, and only contain lower case letters, numbers, and -",
]

queries = {
    "deleteApp": deleteApp_errors,
}

apps = []
apps_result = []
services = []
services_result = []
page = 0

if 1 != 0:
    print("Querying apps...\n")
    while len(apps_result) != 0 or page == 0:
        query = gql(
            """
            query($page: Int) {
            apps(page: $page, allApps:true) {
                    apps {
                        analytics {
                            appname
                            timestamps {
                                created
                                updated
                            }
                        }
                        linkedServices {
                            name
                            serviceType
                        }  
                    }
                }  
            }
            """
        )
        params = {"page": page}
        sleep(5)
        api_call = client.execute(query,variable_values=params)
        apps_result = api_call["apps"]["apps"]
        apps.extend(apps_result)
        print(f"  Page: {page}")
        page = page + 1
    print(f"\n  Apps: {len(apps)}\n")
else:
    print("No apps were queried")


apps_name = []
apps_updated = []
apps_created = []
services_name = []
services_type = []
apps_dict = dict()
services_dict = dict()
if len(apps) != 0:
    print("Parsing apps...")
    for i in range(len(apps)):
        apps_name.append(apps[i]["analytics"]["appname"])
        apps_created.append(apps[i]["analytics"]["timestamps"]["created"])
        apps_updated.append(apps[i]["analytics"]["timestamps"]["updated"])
        apps_dict.update(zip(apps_name, zip(apps_updated, apps_created)))

        if range(len(apps[i]["linkedServices"])) == range(0, 1):
            services_name.append(apps[i]["linkedServices"][0]["name"])
            services_type.append(apps[i]["linkedServices"][0]["serviceType"])
        elif range(len(apps[i]["linkedServices"])) == range(1, 2):
            services_name.append(apps[i]["linkedServices"][1]["name"])
            services_type.append(apps[i]["linkedServices"][1]["serviceType"])
        elif range(len(apps[i]["linkedServices"])) == range(0, 2):
            services_name.append(apps[i]["linkedServices"][0]["name"])
            services_type.append(apps[i]["linkedServices"][0]["serviceType"])
            services_name.append(apps[i]["linkedServices"][1]["name"])
            services_type.append(apps[i]["linkedServices"][1]["serviceType"])
        services_dict.update(zip(services_name, zip(apps_name, services_type)))
else:
    print("No apps were parsed")


apps_filtered = dict()
if len(apps) != 0:
    print("Filtering apps...")
    for k, v in apps_dict.items():
        if (
            k.startswith("{prefix}".format(prefix=PREFIX))
            and v[0] == None
            and (datetime.now() - datetime.strptime(v[1], "%Y-%m-%dT%H:%M:%S.%f"))
            > timedelta(**LAST_UPDATE)
        ):
            print(f"  {k}")
            apps_filtered[k] = v[0]
        elif (
            k.startswith("{prefix}".format(prefix=PREFIX))
            and v[1] != None
            and (datetime.now() - datetime.strptime(v[1], "%Y-%m-%dT%H:%M:%S.%f"))
            > timedelta(**LAST_UPDATE)
        ):
            print(f"  {k}")
            apps_filtered[k] = v[1]
    if len(apps_filtered.items()) > 0:
        print(f"\n  Apps filtered: {len(apps_filtered.items())}\n")
else:
    print("No apps were filtered")


if len(apps_filtered) != 0:
    print("Deleting apps...")
    for k in apps_filtered:
        print(f"  {k}")
        query = gql(
            """
            mutation ($name: String) {
                deleteApp(name: $name) {
                    ok
                    error
                }
            }
            """
        )
        params = {"name": k}
        client.execute(query,variable_values=params)
else:
    print("No apps were deleted")


services_filtered = dict()
if len(services_dict) != 0:
    print("Filtering services...")
    for k, v in services_dict.items():
        if services_dict[k][0] in apps_filtered:
            services_filtered[k] = v[1]
            # print(f"  {k}, {v[1]}")
    if len(services_filtered.items()) > 0:        
        print(f"\n  Services filtered: {len(services_filtered.items())}\n")
else:
    print("No services were filtered")


if len(services_filtered) != 0:
    print("Deleting services")
    for k, v in services_filtered.items():
        query = gql(
            """
            mutation ($name: String, $serviceType: ServiceType){
                deleteService(name: $name, serviceType: $serviceType){
                    error
                    ok
                }
            }
            """
        )
        params = {"name": k, "serviceType": v}
        client.execute(query,variable_values=params)
        print(f"  name: {k}, type: {v}")
else:
    print("No services were deleted")

```