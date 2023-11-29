InterSystems API Manager (IAM) is a core component of the InterSystems IRIS Data Platform, offering centralized API management with a strong emphasis on security. IAM simplifies the entire API lifecycle, from creation to retirement, and provides a developer portal for easy API discovery and integration. Access control features allow administrators to define precise permissions, and IAM seamlessly integrates with the IRIS Data Platform, enhancing data management and integration capabilities.

Features of IAM include:
- API Gateway: Centralized API management and security hub.
- API Lifecycle Management: Complete lifecycle control from creation to retirement.
- Security: Authentication, authorization, and data encryption.
- Monitoring and Analytics: Tools for usage monitoring and pattern analytics.
- Developer Portal: API discovery portal with documentation and testing.
- Access Control: Granular control over API access and actions.
- Integration with InterSystems IRIS: Seamless integration with IRIS Data Platform.

**Use case:** The use case in this report is Identity and Access Management.

Authentication and authorization adhering to OAuth 2.0 standard, securing a FHIR server using IAM.

In this document, you will learn how to secure a FHIR Server with OAuth 2.0 using InterSystems API Manager. OAuth 2.0 is a widely used standard for authorization that enables applications to access protected resources on a FHIR server. InterSystems API Manager is a tool that simplifies the creation, management, and monitoring of FHIR APIs. By following the steps in this document, you will be able to configure InterSystems API Manager to act as an OAuth 2.0 authorization server and grant access tokens to authorized clients. You will also learn how to use client libraries to connect your application to the FHIR server using OAuth 2.0.

Note: FHIR server only supports JWT tokens for OAuth 2.0 authentication, does not support opaque tokens.

**Instructions to run the demo locally:**
1. Run the following command in Command Prompt to clone the relevant repository:

```git clone https://github.com/isc-padhikar/IAM_FHIRServer```

2. Go into the directory of newly clone repository and create a new directory and name it 'key'. And copy a iris.key file, which is the license for InterSystems IRIS for Health which supports API Management. License keys can be downloaded from the following link: https://twiki.iscinternal.com/twiki/bin/view/ISC/LicenseKeys

3. Then go back to Command Prompt and login to InterSystems Container Registry as we will require access to pull some images. In order to login, go to containers.intersystems.com and login with you InterSystems credentials and copy the command to login as shown in the following image and run it on Command Prompt:

   ![image](https://github.com/isc-padhikar/IAM_FHIRServer/assets/98080918/0af97651-dc8e-49a3-ae4d-2b6a80556472)

4. Run the following commands one by one:

```docker-compose build```

```docker-compose up```

5. Go to ```localhost:8002``` which has IAM running.

6. Using IAM, I can make a FHIR server available as a service like seen in the picture below:
   
    ![image](https://github.com/isc-padhikar/IAM_FHIRServer/assets/98080918/58657645-1bcf-498f-8505-74fdd8fdf3c3)

7. Define a route that will be the proxy for the FHIR server (I have defined /fhir as the proxy) like in the picture below:
   
    ![image](https://github.com/isc-padhikar/IAM_FHIRServer/assets/98080918/f55f571e-a313-4b6d-adbd-c51f2728b8ac)

8. And, define plugins that will handle the incoming requests to the FHIR server, authenticate and authorize access to the FHIR server. We should define the issuer of JWT token (the authorization server) and the public key that we obtain by decoding private key (please refer to the upcoming 'Authorization server' section for this part), in the JWT plugin under 'Credentials' section like in the following images:
   
    ![image](https://github.com/isc-padhikar/IAM_FHIRServer/assets/98080918/fbcfb19c-ccdb-4b69-b692-066073365dd1)

   ![image](https://github.com/isc-padhikar/IAM_FHIRServer/assets/98080918/cb5ea7b8-2878-4d60-9293-ebf3f36e5f7a)


Following images show authentication using Auth0 server and authorization based JWT tokens via IAM.

Getting a JWT token from authorization server:

![image](https://github.com/isc-padhikar/IAM_FHIRServer/assets/98080918/43d555eb-1ba2-4569-8c9e-523634e4dc7f)

Using the JWT token to access FHIR server via proxy route defined in IAM:

![image](https://github.com/isc-padhikar/IAM_FHIRServer/assets/98080918/8f4152f2-f195-4756-84bd-0be8895a5b36)

**Authorization server:**

An external authorization server is used and its Auth0. The instructions to set up an authorization server is given in the README of demo #1 (FHIROktaIntegration) mentioned in upcoming 'Demos used as reference' section.

Endpoint to get JSON Web Key Set (JWKS): https://dev-bi2i05hvuzmk52dm.au.auth0.com/.well-known/jwks.json

It provides us with a pair of keys for the authorization server that we've set up and that can be used to retrieve the private key using a decoding algorithm.

We will use the private key in IAM to verify JWT token signatures.

Best practice to retrieve public key from a JWKS is using a programming language. I used following code in Python:
```
import base64
from cryptography.hazmat.backends import default_backend
from cryptography.hazmat.primitives import serialization
from cryptography.hazmat.primitives.asymmetric import rsa
import requests
# Replace 'YOUR_DOMAIN' with your actual Auth0 domain
jwks_url = 'https://dev-bi2i05hvuzmk52dm.au.auth0.com/.well-known/jwks.json'
response = requests.get(jwks_url)
jwks = response.json()
# Choose a specific key from the JWKS (e.g., the first key)
selected_key = jwks['keys'][0]
# Decode 'AQAB' (exponent 'e') from Base64 URL-safe to integer
decoded_exponent = int.from_bytes(base64.urlsafe_b64decode(selected_key['e'] + '==='), byteorder='big')
decoded_modulus = int.from_bytes(base64.urlsafe_b64decode(selected_key['n'] + '==='), byteorder='big')
# Construct the RSA public key
public_key = rsa.RSAPublicNumbers(
    decoded_exponent,
    decoded_modulus
).public_key(default_backend())
# Convert the public key to PEM format
public_key_pem = public_key.public_bytes(
    encoding=serialization.Encoding.PEM,
    format=serialization.PublicFormat.SubjectPublicKeyInfo
)
print(public_key_pem.decode('utf-8'))
```
**Demos used as reference:**
1. FHIROktaIntegration: https://openexchange.intersystems.com/package/FHIROktaIntegration

This demo shows how to configure OAuth 2.0 directly on InterSystems IRIS for Health and use that configuration for a FHIR server. Please follow the instructions it has to configure authorization server's details. 
It has a Angular app that authenticates with the authorization server, with a UI that displays FHIR resources after authorization.
 
This demonstrate how OAuth2.0 can be configured within InterSystems IRIS for Health to secure a FHIR server.

2. IAM Zero-to-Hero: https://openexchange.intersystems.com/package/iam-zero-to-hero

The demo constitutes of IAM, and IAM related training. I will be modifying this to have a FHIR server and using the instance of IAM in this demo to authentication with Auth0 authorization server and authorize access using JWT plugin.
Unlike the previous demo, this demonstrated the use of IAM to expose a FHIR server endpoint and secure it by OAuth 2.0 standard using the plugins library that IAM offers.

Changes made in this demo:

i) I added a FHIR server in the instance of IRIS for Health in this demo. Please replace the code in iris.script file with this following code:
```
;do $System.OBJ.LoadDir("/opt/irisapp/src","ck",,1)

zn "%SYS"
Do ##class(Security.Users).UnExpireUserPasswords("*")
set $namespace="%SYS", name="DefaultSSL" do:'##class(Security.SSLConfigs).Exists(name) ##class(Security.SSLConfigs).Create(name) set url="https://pm.community.intersystems.com/packages/zpm/latest/installer" Do ##class(%Net.URLParser).Parse(url,.comp) set ht = ##class(%Net.HttpRequest).%New(), ht.Server = comp("host"), ht.Port = 443, ht.Https=1, ht.SSLConfiguration=name, st=ht.Get(comp("path")) quit:'st $System.Status.GetErrorText(st) set xml=##class(%File).TempFilename("xml"), tFile = ##class(%Stream.FileBinary).%New(), tFile.Filename = xml do tFile.CopyFromAndSave(ht.HttpResponse.Data) do ht.%Close(), $system.OBJ.Load(xml,"ck") do ##class(%File).Delete(xml)

//init FHIR Server
zn "HSLIB"
set namespace="FHIRSERVER"

Set appKey = "/csp/healthshare/fhirserver/fhir/r4"
Set strategyClass = "HS.FHIRServer.Storage.Json.InteractionsStrategy"
set metadataPackages = $lb("hl7.fhir.r4.core@4.0.1")
set importdir="/opt/irisapp/src"

//Install a Foundation namespace and change to it
Do ##class(HS.Util.Installer.Foundation).Install(namespace)
zn namespace

// Install elements that are required for a FHIR-enabled namespace
Do ##class(HS.FHIRServer.Installer).InstallNamespace()

// Install an instance of a FHIR Service into the current namespace
Do ##class(HS.FHIRServer.Installer).InstallInstance(appKey, strategyClass, metadataPackages)

// Configure FHIR Service instance to accept unauthenticated requests
set strategy = ##class(HS.FHIRServer.API.InteractionsStrategy).GetStrategyForEndpoint(appKey)
set config = strategy.GetServiceConfigData()
set config.DebugMode = 4
do strategy.SaveServiceConfigData(config)

zw ##class(HS.FHIRServer.Tools.DataLoader).SubmitResourceFiles("/opt/irisapp/fhirdata/", "FHIRSERVER", appKey)

zn "USER"

zpm "load /opt/irisbuild/ -v":1:1

zpm 
load /opt/irisapp/ -v
q

do ##class(Sample.Person).AddTestData()

halt
```


ii) In docker-compose.yml file, update IAM's image to latest (containers.intersystems.com/intersystems/iam:3.2.1.0-4), because only IAM (Kong) versions form 3.1 support JSON draft-6, which is what FHIR specification provides.
