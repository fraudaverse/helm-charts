# UI Helm Chart

## Prerequisites

- To authenticate with the UI, you need to deploy AWS Cognito and inside of Cognito a user pool. The users / groups can have the following access rules:
    - fa-inv-admin
    - fa-inv-view
- For the Userpool in AWS Cognito, please provide the env variables in `value.yaml:backend.Envs.<>`:
    - `AWS_ACCESS_KEY_ID` - Access key for a user which has the policies to manipulate the user pool
    - `AWS_SECRET_ACCESS_KEY` - fitting key secret
    - `AWS_REGION` - region of your user pool
    - `AWS_USER_POOL` - ID of your user pool
    - `AWS_USER_POOL_CLIENT_ID` - Please create a client ID token in "App client" in the user pool


## Ingress Settings

The ingress settings are very important for the Ui to be usable. There are two deployments/services involved

a) ui-backend
b) ui-frontend

the http paths mappings should be the following:

`/api/investigation` maps to ui-backend on it's defined port in `service.portbackend`

`/admin` maps to ui-backend on it's defined port in `service.portbackend`

`/` maps to ui-frontend on it's defined port in `service.portfrontend`

This is already predefined in values.yaml, but please make sure that you set the correct `host` variable as well as ingress class.

## Description
tbd

## Release Notes

### v0.1.1
- initial release, including elastic and mongo without authentication and persistence