# conjur-api-powershell
Powershell-based API SDK for [Conjur OSS](https://www.conjur.org/).

---

### **Status**: Beta - zamothh Fork

#### **Warning: Naming and APIs are still subject to breaking changes!**

---

## Installing the code


### How should the module be installed
There is an evironment variable called PSModulePath, that lists all the path where you can have module installed.
This allows you to import modules without having to specify the path

Instructions : 
copy the module to one of the existing path of that environment variable ($env:PSModulePath will tell you all of them)
PS C:\Users\MyProfile\Documents\WindowsPowerShell\Modules> git clone https://github.com/zamothh/conjur-api-powershell.git  CyberarkConjur

Once you cloned the project to a PsModulePath folder, you can import the module from any directory
```powershell
PS C:\> import-module CyberarkConjur
```



### How should the module be installed
There is an environment variable called `$ENV:PsModulePath folder`, that lists all the paths where you can have module installed.
This allows you to import modules without having to specify the path
```powershell
PS C:\> $ENV:PSModulePath -split ";"
 C:\Users\MyAccount\Documents\PowerShell\Modules
 C:\Program Files\PowerShell\Modules
 ... and more ...
```

Instructions : 
Download and install git https://git-scm.com/download/win
On git hub, go to the fork/project you want to use, click on the green `Code` button and then click on `copy` to copy the clone link to your clipboard.
Then open git bash
Go to the directory you want to clone you project on
Clone your porject
```bash
MyAccount@MyPc MINGW64  ~
$  cd "C:\Program Files\PowerShell\Modules"

MyAccount@MyPc MINGW64 /c/Program Files/PowerShell/Modules
$ git clone [paste the clone link here] CyberarkConjur
```

Once you cloned the project to a `$ENV:PsModulePath folder`, you can import the module from any directory
```powershell
PS C:\> import-module CyberarkConjur
```


### From source

```powershell
PS C:\> Import-Module .\CyberarkConjur.psm1
```

## Usage

### Setting the module 
#### Conjur authentication
Prior to launching any commands, you will need to configure your Conjur environment. To do so, and to keep the environment secure, the Initialize-Conjur will store in memory your API path & token into a PsCredential object.

You can authenticate using the less secured method :
```powershell
PS C:\> Initialize-Conjur -Account myorg -AuthorityName "eval.conjur.org"
-AuthnLogin "Identifier of a host" -AuthnApiKey "Your API Key"
```

The best is to always use `PsCredential` object, and never to store an uncrypted secret into memory.
```powershell
PS C:\> Initialize-Conjur -Account myorg -AuthorityName "eval.conjur.org"
-Credential $credential
```

By directly using a credential object, you will also be able to securely store that credential on your hard drive, using the Windows Data Protection API :

```powershell
# How to store the credential on the hard drive (you will have to do it only once) :
$Credential =  (Get-credential -Message "Please enter your Conjur API key" -UserName $AuthnLogin)
$Credential | Export-Clixml $MyPath 

# How recall the credential and authenticate :
$Credential = Import-Clixml $MyPath 
Initialize-Conjur -Account myorg -AuthorityName "eval.conjur.org"
-Credential $credential
```

The Windows Data Protection API is well documented over internet, feel free to have a closer look if you have security concerns.


#### IAM Authentication
Some code has been started to be written, but is not good enough to publish anything yet.
This would require someone with an IAM access to continue developing this code.

### Available Functions
#### Initialize-Conjur

```powershell
PS C:\> Initialize-Conjur -Account Account -AuthnLogin "Identifier of a host" -AuthnApiKey "Your API generated key" -AuthorityName "your-conjur-auth-read.mycompany.com" 
```

#### Invoke-Conjur
All commands are running this command at the end. you may call it directly if required
```powershell
PS C:\> Invoke-ConjurIam API Command Search
```

#### Receive-ConjurLogin
Retries the API key (done automatically)
```powershell
PS C:\> $APIKey = Receive-ConjurLogin
```

#### Receive-ConjurAuthenticate
Generates a Token from the API Key (done automatically)
```powershell
PS C:\> $Token = Receive-ConjurAuthenticate $ApiKey
```

#### Get-ConjurWhoAmI
Only if Authenticated
```powershell
PS C:\> Get-ConjurWhoAmI

client_ip       : 10.0.0.10
user_agent      : Mozilla/5.0 (Windows NT; Windows NT 10.0; en-US) WindowsPowerShell/5.1.19041.1237
account         : account
username        : host/PowerShellAutomation
token_issued_at : 2021-09-24T12:56:40.000+00:00
```

#### Get-ConjurHealth
```powershell
PS C:\> Get-ConjurHealth

services : @{ldap-sync=disabled; possum=ok; ui=ok; ok=True}
database : @{ok=True; connect=; free_space=; replication_status=}
audit    : @{ok=True; forwarded=}
ok       : True
role     : follower
```

#### Get-ConjurSecret

```powershell
PS C:\> Get-ConjurSecret -Identifier "Identifier/path/password" 

SecretWillShow
```

#### Get-ConjurSecretCredential
```powershell
PS C:\> Get-ConjurSecretCredential "Identifier/path" 

UserName      Password
--------      --------
TheUserName   System.Security.SecureString
```
#### Update-ConjurSecret

```powershell
PS C:\> Update-ConjurSecret -Identifier "path/to/secret" -SecretValue "newPasswordHere"" 
```

#### Update-ConjurPolicy

```powershell
PS C:\> Update-ConjurPolicy -Identifier "root" -PolicyFilePath ".\test-policy.yml" 

created_roles                      version
-------------                      -------
@{dev:host:database/another-host=} 4
```
#### Set-ConjurPolicy

```powershell
PS C:\> Set-ConjurPolicy -Identifier "root" -PolicyFilePath ".\test-policy.yml"

created_roles                      version
-------------                      -------
@{dev:host:database/another-host=} 4
```
#### Add-ConjurPolicy

```powershell
PS C:\> Add-ConjurPolicy -Identifier "root" -PolicyFilePath ".\test-policy.yml"

created_roles                      version
-------------                      -------
@{dev:host:database/another-host=} 4
```
#### Get-ConjurResources

```powershell
PS C:\> Get-ConjurResources -Identifier "Identifier/path" 

created_at      : 2019-05-29T16:42:56.284+00:00
id              : dev:policy:root
owner           : dev:user:admin
permissions     : {}
annotations     : {}
policy_versions : {@{version=1; created_at=2019-05-29T16:42:56.284+00:00; policy_text=---  
```
#### Get-ConjurRole

```powershell
PS C:\> Get-ConjurRole user alice

created_at : 2017-08-02T18:18:42.346+00:00
id         : myorg:user:alice
policy     : myorg:policy:root
members    : {@{admin_option=True; ownership=True; role=myorg:user:alice; member=myorg:policy:root; policy=myorg:policy:root}}
```

#### Get-Help
You can Get-Help on all of the functions mentioned above.

```powershell
PS C:\> Get-Help Update-ConjurPolicy

NAME
    Update-ConjurPolicy

SYNOPSIS
    Update a policy in conjur


SYNTAX
    Update-ConjurPolicy [-PolicyIdentifier] <String> [-PolicyFilePath] <String> [-ConjurAccount <Object>]
    [-ConjurUsername <Object>] [-ConjurPassword <Object>] [-ConjurApplianceUrl <Object>] [-IgnoreSsl]
    [<CommonParameters>]


DESCRIPTION
    Modifies an existing Conjur policy. Data may be explicitly deleted using the !delete, !revoke, and !deny
    statements.
    Unlike “replace” mode, no data is ever implicitly deleted.


RELATED LINKS
    https://www.conjur.org/api.html#policies-update-a-policy-patch

REMARKS
    To see the examples, type: "get-help Update-ConjurPolicy -examples".
    For more information, type: "get-help Update-ConjurPolicy -detailed".
    For technical information, type: "get-help Update-ConjurPolicy -full".
    For online help, type: "get-help Update-ConjurPolicy -online"
```

## Contributing

We store instructions for development and guidelines for how to build and test this
project in the [CONTRIBUTING.md](CONTRIBUTING.md) - please refer to that document
if you would like to contribute.

## License

This project is [licensed under Apache License v2.0](LICENSE.md)
