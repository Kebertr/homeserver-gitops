Create a Github App.

PAT would be faster. But more insecure if anyone got hold of it. 

Create a GIthub app and store the RSA pem safe. 

Then Install App. 

Go into the github app and get the ID.

Then go to settings in github->integrations->application->configure the github app and in the url there you will get installation id. 

Create a namespace for the secrets and stor the secret

```kubectl create secret generic github-runner-sec -n namespace --from-literal=github_app_id=app id --from-literal=github_app_installation_id=installation id --from-file=path to file```
