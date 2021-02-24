# Dash Review Apps 
Enable Dash Review Apps with **Helper Script** and exiting CI/CD pipeline.

## Helper Script

The helper script is comprised of four modules that leverage the Dash Enterprise
Server API to automatically initialize and deploy Dash Review Apps. 

Theses helper scripts are templates that can be modified to meet your needs.

### Settings.py

The `settings.py` file is responsible for storing all of the environment 
variables needed for Review App automation.

#### User Configurable Variables


|                         | settings.py | initialize.py | deploy.py | delete.py |
| ----------------------- | ----------- | ------------- | --------- | --------- |
| PREFIX                  |             |               |           | X         |
| LAST_UPDATE             |             |               |           | X         |
| TRUNK_BRANCHNAME        |             |               | X         |           |
| BRANCHNAME              | X           |               | X         |           |
| REPONAME                |             |               | X         |           |
| TARGET_APPNAME          | X           | X             | X         |           |
| APPNAME                 | X           | X             | X         |           |
| DASH_ENTERPRISE_HOST    | X           | X             | X         | X         |
| SERVICE_API_KEY         |             | X             |           | X         |
| SERVICE_USERNAME        | X           | X             |           | X         |
| USERNAME_API_KEY        |             | X             |           |           |
| USERNAME                |             | X             |           |           |
| DE_USER_TO_CI_USER      | X           |               |           |           |
| DE_USER_TO_CI_API       | X           |               |           |           |
| SSH_CONFIG              |             |               | X         | X         |
| SERVICE_PRIVATE_SSH_KEY |             |               | X         |           |
| DEBUG                   |             | X             |           | X         |



#### PREFIX

The string stored in `PREFIX`  is used to filter Review Apps. Only Review Apps 
starting with this prefix will be deleted.

```python
PREFIX = "review-app-"
```

#### LAST_UPDATE

If Review Apps are not merged,  `LAST_UPDATE`  represents the amount of time 
they can remain on the host before being purged. Typically, Review Apps are 
automatically deleted when their PR is merged.

```python
LAST_UPDATE = {"days": 5} 
# Accepted keys: "weeks", "days", "hours", "minutes"
# Accepted value: int
```

#### TRUNK_NAME

The primary branch of the Review App repository is called `TRUNK_NAME`. In the 
context of the `deploy.py` script, Review Apps are only deployed when they are 
merged to this branch.

```python
TRUNK_BRANCHNAME = "main" 
```

#### BRANCHNAME

`BRANCHNAME` is the name of the branch you will initialize your Review App from. 
This can be pulled from the CI Platform's environment or included in the code. 
`BRANCHNAME` must not exceed 30 characters in length or Dash Enterprise Host 
will complain.

```python
BRANCHNAME = os.getenv("CIRCLE_BRANCH")
```
#### REPONAME

`REPONAME` is the name of the repository that will hold the Review App 
branches.

```python
REPONAME = os.getenv("CIRCLE_PROJECT_REPONAME")
```
#### TARGET_APPNAME

`TARGET_APPNAME` is the name the Dash App that will serve as a Review App
template. Review Apps will inherit the settings associated with `TARGET_APPNAME` 
if applicable .The Target App must exist and you must have permission to view 
it, or Review App initialization will fail.

```python
TARGET_APPNAME = "chris-app"
```
#### APPNAME

`APPNAME` determines which naming pattern your Review Apps will follow. 
`APPNAME` must not exceed 30 characters in length.

```python
APPNAME = f"{TARGET_APPNAME}-rev-{BRANCHNAME}"[0:30]
```
#### DASH_ENTERPRISE_HOST

`DASH_ENTERPRISE_HOST` is your Dash Enterprise Server's host address.

```python
DASH_ENTERPRISE_HOST = "qa-de-410.plotly.host" 
```
#### SERVICE_API_KEY

`SERVICE_API_KEY` is the "Machine User's" API key used to access your Dash
Enterprise Host.

```python
SERVICE_API_KEY = os.getenv("ADMIN_API_KEY")
```
#### SERVICE_USERNAME

`SERVICE_USERNAME` is a "Machine User"  (with admin privileges) that handles 
all aspects of Review App management. Creating a Service Account is required 
to use Review Apps

```python
SERVICE_USERNAME = "admin"
```
#### USERNAME_API_KEY

The `USERNAME_API_KEY` allows the helper script to initialize a Review App on 
the behalf of its Developer. 

In the following code snippet, the `USERNAME_API_KEY` is fetched from a 
dictionary that mapping Dash Enterprise usernames to their corresponding API 
keys available in the CI platform environment. 

```python
USERNAME_API_KEY = (
    os.getenv(
        DASH_ENTERPRISE_USERNAME_TO_CIRCLE_CI_API_KEY.get(USERNAME)
    )
)
```
#### USERNAME

In this case,  `USERNAME` is equivalent to the GitHub username of the Developer 
pushing code to the Review App repository. `USERNAME` is mapped to the 
Developer's Dash Enterprise username that will author the 
initialized apps.  We recommend that you enforce user access at the repository 
level. Given that a "Machine User" with admin access  is deploying the Review 
Apps, it is an effective way to keep your apps away from unauthorized users. 

```python
USERNAME = os.getenv("CIRCLE_USERNAME")
```
#### DE_USER_TO_CI_USER

`DE_USER_TO_CI_USER` maps your developer
usernames to the GitHub username used to push changes to your review app
repository.

```python
DASH_ENTERPRISE_USERNAME_TO_CIRCLECI_USERNAME = {
    "criddyp" : "criddyp",
    "tobinngo": "tobinngo",
}
```
#### DE_USER_TO_CI_API

`DE_USER_TO_CI_API` maps developer usernames to the corresponding API keys 
stored as environment variables in your CircleCI Project Settings.

```python
DASH_ENTERPRISE_USERNAME_TO_CIRCLE_CI_API_KEY = {
    "criddyp": "CRIDDYP_API_KEY",
    "tobinngo": "TOBINNGO_API_KEY",
}
```
#### SSH_CONFIG

`SSH_CONFIG` contains your SSH settings for Dash app deployment. The crucial 
fields are, `DASH_ENTERPRISE_HOST`, `SERVICE_USERNAME`, and port number.

```python
SSH_CONFIG = f"Host {DASH_ENTERPRISE_HOST},    HostName {DASH_ENTERPRISE_HOST}, User {SERVICE_USERNAME},    Port 3022,    IdentityFile ~/.ssh/id_rsa,    StrictHostKeyChecking no,    UserKnownHostsFile /dev/null"
```
#### SERVICE_PRIVATE_SSH_KEY

`SERVICE_PRIVATE_SSH_KEY` belongs to a Dash Enterprise user with admin 
privileges. This user will handle all of the server deployment tasks. Multi-line 
strings such as this should be encoded in base64 before being assigned as 
environment variables.

```python
SERVICE_PRIVATE_SSH_KEY = os.getenv("ADMIN_PRIVATE_SSH_KEY")
```

### Initialize.py

### Deploy.py

### Delete.py