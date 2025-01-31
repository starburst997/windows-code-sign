# windows-code-sign-test

Testing code signing with a simple C++ app via Github Action and Azure's [Trusted Signing](https://techcommunity.microsoft.com/blog/microsoft-security-blog/trusted-signing-is-now-open-for-individual-developers-to-sign-up-in-public-previ/4273554) .

The following text is a copy of my blog post: [**Code Signing for Windows as an Individual Developer**](https://jd.boiv.in/post/2025/01/29/code-signing.html).

<br/>

(*TLDR; Get code signing working for Windows for free\* via Github Action, check this repository to see how simple it can get once everything is setup*)

<br/>

## Introduction

For the longest time, code signing on Windows required to buy certificates from rather sketchy companies ([Sectigo](https://www.sectigo.com/) / [DigiCert](https://www.digicert.com/) / etc...). This also required quite a lengthy process with terrible customer services to get verified (requiring in some case to [hire a Lawer or CA](https://www.sectigo.com/knowledge-base/detail/OV-Code-Signing-Validation-for-Organizations-and-Individuals/kA01N000000brb0)), exorbitant prices (500$+ / year) and the use of primitive technologies (forcing to [use internet explorer](https://signmycode.com/resources/generate-code-signing-certificate-using-internet-explorer-mode-in-microsoft-edge) to generate the private key).

Finally, [Microsoft recently launched](https://techcommunity.microsoft.com/blog/microsoft-security-blog/trusted-signing-is-now-open-for-individual-developers-to-sign-up-in-public-previ/4273554) a service under [Azure](https://en.wikipedia.org/wiki/Microsoft_Azure) to help ease this process and get ready in just a few minutes with a relatively affordable price and easy verification for Individual Developer.

*(this post is not sponsored, I was just quite relieved to see there was finally a viable solution)*

<br/>

## What is Code Signing

Ever saw a scary warning while downloading a EXE?

<table align="center"><tr><td>
<a href="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/warning_5.png" target="_blank"><img src="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/warning_5.png" alt="Scary Warning 1" height="200"/></a>
</td><td>
<a href="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/warning_3.png" target="_blank"><img src="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/warning_3.png" alt="Scary Warning 2" height="200"/></a>
</td></tr></table>

This is because the application is **NOT** code signed. This is an awful experience for anyone trying to download something and as a developer, is gut wrenching to feel powerless to remove this barrier without caving in to the Certificates Mafia Monopoly.

*(to be fair, if your app gets enough downloads, at some unknown point in times, [Microsoft will remove this warning](https://stackoverflow.com/a/66582477) but nothing is certains)*

[Code Signing](https://en.wikipedia.org/wiki/Code_signing) is a great practice and ensure that the EXE hasn't been tampered with and comes from the developer who built it by using a cryptographic hash. If a code signed application is found to have malware, the certificate can be revoked and Windows won't allow it to be run. 

Apple have been doing this too for a while but the process is [rather painless](https://docs.fastlane.tools/actions/match/) and affordable through their [Apple Developer Program](https://developer.apple.com/support/compare-memberships/) (*which anyone making iOS app already have anyway*).

Linux doesn't have any of that *(in the context of scary popup from downloading binaries in the browser)*.

<br/>

## Setup code signing in Azure

<table align="center"><tr><td>
<a href="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/azure.png" target="_blank"><img src="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/azure.png" alt="Sign up for Azure" height="200"/></a>
</td></tr></table>

[Create an account](https://azure.microsoft.com/en-us/pricing/purchase-options/azure-account) (new account come with free credits) and sign in to the [Azure portal](https://portal.azure.com/).

*Makes sure you use the link above and click **Try Azure for free** or **Pay as you go** and fill the forms, otherwise you might get connected to the Microsoft Services tenant instead of your own and will see authentication issues.*

You should have a default Subscription / Tenant (if not, you didn't fill the forms as stated above).

<br/>

### Register the Trusted Signing resource provider

<table align="center"><tr><td>
<a href="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/provider_1.png" target="_blank"><img src="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/provider_1.png" alt="Search for Subscriptions" height="176"/></a>
</td><td>
<a href="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/provider_2.png" target="_blank"><img src="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/provider_2.png" alt="Select Resource providers" height="176"/></a>
</td><td>
<a href="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/provider_3.png" target="_blank"><img src="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/provider_3.png" alt="Register CodeSigning" height="176"/></a>
</td></tr></table>

In the [Azure portal](https://portal.azure.com/), search for **Subscriptions** service, click on your subscription.

Select **Resource providers** under **Settings** on the left panel.

Search for **Microsoft.CodeSigning**, click on "..." and select **Register**.

<br/>

### Create a Trusted Signing account

<table align="center"><tr><td>
<a href="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/account_1.png" target="_blank"><img src="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/account_1.png" alt="Create account" height="200"/></a>
</td><td>
<a href="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/account_2.png" target="_blank"><img src="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/account_2.png" alt="Fill info" height="200"/></a>
</td></tr></table>

In the [Azure portal](https://portal.azure.com/), search for **Trusted Signing Accounts** service and **Create** one.

Fill the form by selecting your **Subscription**, select **Create new** for **Resource group** (use any name you want). Pick an **Account name** (any name you want), a **Region** and finally a **Pricing tier** (at the time of writing, it seems like the 9.99$ is actually *free* for individual developer for the preview period).

<br/>

### Assign roles

<table align="center"><tr><td>
<a href="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/role_1.png" target="_blank"><img src="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/role_1.png" alt="Select account" height="159"/></a>
</td><td>
<a href="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/role_2.png" target="_blank"><img src="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/role_2.png" alt="Add role assignment" height="159"/></a>
</td><td>
<a href="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/role_3.png" target="_blank"><img src="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/role_3.png" alt="Select role" height="159"/></a>
</td><td>
<a href="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/role_4.png" target="_blank"><img src="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/role_4.png" alt="Add member" height="159"/></a>
</td></tr></table>

In the [Azure portal](https://portal.azure.com/), search for **Trusted Signing Accounts** service and select the account you just created.

On the left panel select **Access Control (IAM)** and pick **Add** / **Add role assignment**.

Search for **Trusted Signing**, you'll see two roles (**Trusted Signing Certificate Profile Signer** and **Trusted Signing Identity Verifier**) you'll need to add both, one at a time.

Select one role, go **Next**, click on **+ Select members**, select your user and click **Review + assign**.

Repeat for the second role.

<br/>

### Create an identity validation request

<table align="center"><tr><td>
<a href="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/identity_1.png" target="_blank"><img src="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/identity_1.png" alt="Create account" height="176"/></a>
</td><td>
<a href="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/identity_2.png" target="_blank"><img src="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/identity_2.png" alt="Fill form" height="176"/></a>
</td><td>
<a href="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/identity_3.png" target="_blank"><img src="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/identity_3.png" alt="Au10tix" height="176"/></a>
</td></tr></table>

In the [Azure portal](https://portal.azure.com/), search for **Trusted Signing Accounts** service and select your account.

On the left panel, under **Settings** / **Objects** click **Identity validations**. On the top-left dropdown (**Organization**) switch to **Individual** and then click **New identity** / **Public**.

Fill the form with your name, address, etc. and click **Create**.

After a little while the status will change to **Action Required**, click on your name and click the link under "Please complete your verification here".

You'll then start the verification process which requires a mobile phone, some government IDs (I used my driver license) and the [Microsoft Authenticator App](https://www.microsoft.com/en-ca/security/mobile-authenticator-app). I was pleasantly surprised by how easy and streamlined the process was. Simply follow the instructions, which is pretty much take selfie, take picture of ID and scan some QR code. Makes sure to use an ID where you can see your address.

Once this is done, it takes around 5-10 minutes before the status changes to **Completed**.

<br/>

### Create a Certificate profile

<table align="center"><tr><td>
<a href="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/certicate_1.png" target="_blank"><img src="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/certicate_1.png" alt="Create certificate" height="200"/></a>
</td><td>
<a href="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/certicate_2.png" target="_blank"><img src="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/certicate_2.png" alt="Fill form" height="200"/>  </a>
</td></tr></table>

In the [Azure portal](https://portal.azure.com/), search for **Trusted Signing Accounts** service and select your account.

On the left panel, under **Settings** / **Objects** click **Certificate profiles**. Then click on **Create** / **Public Trust**.

Pick a name for your certificate, including the address / post code is optional. Click **Create**.

*(don't worry about the expiration date, the certificate is auto-renew every 24h)*

<br/>

## Sign your app locally

You are now ready to sign your first application!

### Note some values

<table align="center"><tr><td>
<a href="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/local_1.png" target="_blank"><img src="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/local_1.png" alt="Trusted Signing account" height="176"/></a>
</td><td>
<a href="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/local_2.png" target="_blank"><img src="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/local_2.png" alt="Certificate profile" height="176"/></a>
</td><td>
<a href="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/local_3.png" target="_blank"><img src="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/local_3.png" alt="Tenant ID" height="176"/></a>
</td></tr></table>

First, we need to takes note of a few variables, copy your **Trusted Signing account** name you used earlier (also take note of the **Location**) and the **Certificate profile** name as well.

You also need your **Tenant ID**, search for **Microsoft Entra ID** in the [Azure portal](https://portal.azure.com/) and copy the value.

<br/>

### Install SignTool

For the next steps, I used **PowerShell 7**, makes sure you are also up-to-date with [**dotnet**](https://dotnet.microsoft.com/en-us/download/dotnet) (8+).

Install the [**Trusted Signing Tools**](https://download.microsoft.com/download/6d9cb638-4d5f-438d-9f21-23f0f4405944/TrustedSigningClientTools.msi) / [**Azure CLI**](https://aka.ms/installazurecliwindowsx64) via winget (or via installer)

```sh
winget install -e --id Microsoft.Azure.TrustedSigningClientTools
winget install -e --id Microsoft.AzureCLI
```

Now we need an additional file (*Azure.CodeSigning.Dlib.dll*) that is only available in the latest [**Microsoft.Trusted.Signing.Client**](https://www.nuget.org/packages/Microsoft.Trusted.Signing.Client) nuget (click on **Download package** on the right). 

(***.nupkg** are simply **.zip** files, so rename it and extract it somewhere, remember it's path*)

<br/>

### Sign your EXE

Create a JSON file with the variables we noted above (save the path)

```json
{
  "Endpoint": "https://eus.codesigning.azure.net",
  "CodeSigningAccountName": "<TRUSTED_SIGNING_ACCOUNT_NAME>",
  "CertificateProfileName": "<CERTIFICATE_PROFILE_NAME>"
}
```

Based on the **Location** of your account, use the correct **Endpoint** url

```sh
"East US" = "https://eus.codesigning.azure.net"
"West US3" = "https://wus3.codesigning.azure.net"
"West Central US" = "https://wcus.codesigning.azure.net"
"West US 2" = "https://wus2.codesigning.azure.net"
"North Europe" = "https://neu.codesigning.azure.net"
"West Europe" = "https://weu.codesigning.azure.net"
```

Login to Azure using your Tenant ID

```sh
az login --tenant <TENANT_ID>
```

Now we can sign any EXE! Notice that the value `10.0.22621.0` might change in the future.

```sh
& "C:\Program Files (x86)\Windows Kits\10\bin\10.0.22621.0\x64\signtool.exe" `
sign /v /debug /fd SHA256 /tr "http://timestamp.acs.microsoft.com" /td SHA256 `
/dlib "<PATH_TO_MICROSOFT_TRUSTED_SIGNING_CLIENT>\bin\x64\Azure.CodeSigning.Dlib.dll" `
/dmdf "<PATH_TO_JSON>" `
"<PATH_TO_EXE>"
```

Congrats! You now have a code signed app! Upload it somewhere and use your browser to [download it](https://cdn.notessimo.com/misc/codesign/CodeSignCpp.exe), notice how there is no scary popup at all anymore!

<br/>

## Sign your app with Github Action

Now that everything works locally, we can easily automate the code signing steps using [Github Action](https://github.com/features/actions).

### Create an App

<table align="center"><tr><td>
<a href="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/gh_1.png" target="_blank"><img src="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/gh_1.png" alt="App registrations" height="159"/></a>
</td><td>
<a href="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/gh_2.png" target="_blank"><img src="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/gh_2.png" alt="Certificate profile" height="159"/></a>
</td><td>
<a href="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/gh_5.png" target="_blank"><img src="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/gh_5.png" alt="Copy Client ID" height="159"/></a>
</td><td>
<a href="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/gh_4.png" target="_blank"><img src="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/gh_4.png" alt="Create secret" height="159"/></a>
</td></tr></table>

In the [Azure portal](https://portal.azure.com/), search for **App registrations** service and click on **New registration**.

Give it a name and select the **Single Tenant** option, click on **Register**.

In your newly created app, note the **Application (client) ID**.

Select **Manage** / **Certificates & secrets** and click on **New client secret**, give it a name & expiration date and note the **Value**. 

*(sadly the "no expiration" option is not available anymore)*

<br/>

### Assign roles

<table align="center"><tr><td>
<a href="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/role_1.png" target="_blank"><img src="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/role_1.png" alt="Select account" height="159"/></a>
</td><td>
<a href="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/role_2.png" target="_blank"><img src="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/role_2.png" alt="Add role assignment" height="159"/></a>
</td><td>
<a href="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/role_3.png" target="_blank"><img src="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/role_3.png" alt="Select role" height="159"/></a>
</td><td>
<a href="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/gh_3.png" target="_blank"><img src="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/gh_3.png" alt="Search for app" height="159"/></a>
</td></tr></table>

We also need to add the **Trusted Signing Certificate Profile Signer** role to the app just like we did on our user.

In the [Azure portal](https://portal.azure.com/), search for **Trusted Signing Accounts** service and select your account.

On the left panel select **Access Control (IAM)** and pick **Add** / **Add role assignment**.

Search for **Trusted Signing Certificate Profile Signer**, go **Next**, click on **+ Select members**, search for your App name (it won't show up by default) and click **Review + assign**.

<br/>

### Create GIT repository

<table align="center"><tr><td>
<a href="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/gh_7.png" target="_blank"><img src="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/gh_7.png" alt="Create repository" height="200"/></a>
</td><td>
<a href="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/gh_6.png" target="_blank"><img src="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/gh_6.png" alt="Secrets" height="200"/></a>
</td></tr></table>

Create your git repository on [Github](https://github.com/new), or clone my [sample repository](https://github.com/starburst997/windows-code-sign-test) which includes a basic C++ application.

Add these 6 secrets to your repository (**Settings** / **Security** / **Secrets and variables** / **Actions**, click **New repository secret**). *(do not include quotes)*

```sh
AZURE_CLIENT_ID = "Your app client ID"
AZURE_CLIENT_SECRET = "Your app secret value"
AZURE_SIGNING_ACCOUNT = "Your Trusted Signing Accounts name"
AZURE_SIGNING_CERTIFICATE = "Your Certificate profile name"
AZURE_TENANT_ID = "Your Tenant ID"
AZURE_ENDPOINT = "https://eus.codesigning.azure.net/"
```

<br/>

### Create worflow file

Create a workflow file in your repository ([**.github/workflows/windows.yml**](https://github.com/starburst997/windows-code-sign-test/blob/main/.github/workflows/windows.yml))

```yaml
name: Build Windows

on: 
  #push
  workflow_dispatch:

jobs:
  build:
    name: Build Windows
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Add msbuild to PATH
        uses: microsoft/setup-msbuild@v2

      - name: Build app for release
        working-directory: CodeSignCpp
        run: msbuild /property:Configuration=Release /property:Platform=x64

      - name: Sign files with Trusted Signing
        uses: azure/trusted-signing-action@v0.5.1
        with:
          azure-tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          azure-client-id: ${{ secrets.AZURE_CLIENT_ID }}
          azure-client-secret: ${{ secrets.AZURE_CLIENT_SECRET }}
          endpoint: ${{ secrets.AZURE_ENDPOINT }}
          trusted-signing-account-name: ${{ secrets.AZURE_SIGNING_ACCOUNT }}
          certificate-profile-name: ${{ secrets.AZURE_SIGNING_CERTIFICATE }}
          files-folder: ${{ github.workspace }}\CodeSignCpp\bin\Release-x64
          files-folder-filter: exe,dll
          file-digest: SHA256
          timestamp-rfc3161: http://timestamp.acs.microsoft.com
          timestamp-digest: SHA256

      - uses: actions/upload-artifact@v4
        with:
          name: build-windows
          path: CodeSignCpp/bin/Release-x64
```

Thanks to the [azure/trusted-signing-action](https://github.com/azure/trusted-signing-action) action, all the heavy works is done, leaving us with a pretty simple workflow.

<br/>

### Run the action

<table align="center"><tr><td>
<a href="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/gh_8.png" target="_blank"><img src="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/gh_8.png" alt="Start action" height="200"/></a>
</td><td>
<a href="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/gh_9.png" target="_blank"><img src="https://jd.boiv.in/assets/posts/2025-01-29-code-signing/gh_9.png" alt="Download artifact" height="200"/></a>
</td></tr></table>

You can then start the action by going to the **Actions** tabs, selecting your action (**Build Windows**) and click on **Run workflow**.

Once the action has finished running, you can see the uploaded artifact and download your signed application!

See the result for yourself: [Signed](https://cdn.notessimo.com/misc/codesign/CodeSignCpp.exe) / [Unsigned](https://cdn.notessimo.com/misc/codesign/CodeSignCpp-unsigned.exe)

<br/>

## Conclusion

I am really glad Microsoft finally made it easy and affordable to code sign EXE on Windows, hopefully they will keep the price low for individual developer. Now, any new project I do, I can simply copy / paste the secrets into the new repository, copy the workflow file and be setup with code signing in just a few seconds.

<br/>
