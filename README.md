# aspnet-core-3-jwt-refresh-tokens-api

ASP.NET Core 3.1 API - JWT Authentication with Refresh Tokens

Documentation and instructions available at https://jasonwatmore.com/post/2020/05/25/aspnet-core-3-api-jwt-authentication-with-refresh-tokens

# Lokal ausprobieren
dotnet restore
dotnet build
dotnet run
- Postman: POST: http://localhost:4000/users/authenticate - mit folgendem Body:
{
    "username": "test",
    "password": "test"
}

# Auf Openshift deployen
- ist in fhel_linuxContainers auch beschrieben. 
- Folder openshiftio angelegt und von Example template kopiert (dotnet-runtime-example.json):
https://github.com/redhat-developer/s2i-dotnetcore/tree/master/templates

CRC starten:
- Den Container wie folgt starten
crc start -p /Users/sonneck/Documents/git_repos/rhel_linuxContainers/crc-macos-1.21.0-amd64/pull-secret
crc oc-env
eval $(crc oc-env)
oc login -u developer -p developer https://api.crc.testing:6443

oc get projects
- Switch to project authentication
oc project authentication 

-- get application.yaml: danach wird damit oc apply aufgerufen (oc apply -f dotnet-authentication-example.json) -> das fügt die API objects dem Cluster hinzu!!!!
find . | grep openshiftio | grep dotnet | xargs -n 1 oc apply -f 

-> Ergebnis ist Anlage des Templates von application.yaml
- Templates in project auflisten mit: (globale Templates mit: oc get templates -n openshift)
oc get templates

-- deploy on openshift -> 2 Parameter gibt man direkt an (Source-Rep), OUTPUT_DIR muss bei Angular dist/ sein!!
---oc new-app --template angular-web-app -p SOURCE_REPOSITORY_URL=https://github.com/sonneckGeorg/angular-10-jwt-refresh-tokens -p OUTPUT_DIR=dist/angular-jwt-refresh-tokens

oc new-app --template dotnet-authentication-example

-- POD-Problem: Livenes und Readiness probe gehen direkt auf / -> dort bekommen sie natürlich Unauthorized!!
https://developers.redhat.com/blog/2018/12/21/asp_dotnet_core_kubernetes_health_check_openshift/
--> Wir müssen also Readiness und Liveness in die Applikation mit einbauen 
--> Passiert über eigenes Service und UseHealthChecks in Startup.cs
--> im dotnet-authentication-example.json werden diese Checks eingetragen


-- TODO: Gitbhub WebHook einrichten für CI/CD
https://developer.ibm.com/technologies/containers/tutorials/github-webhook-triggers-openshift/
- Webhook-URL: BuildConfig raussuchen und dann folgendes absetzen
oc describe bc/dotnet-authentication-example-build
-> liefert Webhook GibHub-URL:
https://api.crc.testing:6443/apis/build.openshift.io/v1/namespaces/authentication/buildconfigs/dotnet-authentication-example-build/webhooks/<secret>/github
- Bei der BuildConfig im YAML findet man das Github Secret:
eq1qVdx0C2pfkF4kRP1osU7eu0g8rGSkeus8FbrR
- Im GibHub-Repo -> Settings -> Webhooks -> 'Add webhook'
- Bei PayLoad URL die URL von oben
- Type: application/json
- Secret: von oben




-- TODO: Advanced Example: https://jasonwatmore.com/post/2020/07/06/aspnet-core-3-boilerplate-api-with-email-sign-up-verification-authentication-forgot-password

-> habs schon geforket und heruntergeladen: Project: aspnet-core-3-signup-verification-api
Inkludiert folgendes:
- Authentication
- EMail
- CRUD Backend (Speicherung)
- Role Based Authorization
- Forgot Password / Reset password
- Swagger api documentation route
