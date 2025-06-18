---
title: A workaround for connecting Mountain Duck to Google Drive accounts
date: 2024-08-10 10:15:00 +0700
categories: [Blogging, Tutorial, Workaround]
tags: [mountainduck, googledrive]
author: j12t
toc: true
comments: true
math: true
mermaid: false
media_subpath: /assets/img/posts/2024-08-10-workaround-for-mountain-duck-google-drive
image:
    path: blocked.png
    alt: google blocked mountain duck
pin: false
---

# Why does this happen?

As [dkocher](https://github.com/dkocher) (contributer) commented in the issue [Appication blocked by Google for OAuth Flow](https://github.com/iterate-ch/cyberduck/issues/16178) of [CyberDuck](https://github.com/iterate-ch/cyberduck) repository, the block happen when the developers cannot obtain [CASA assessment](https://github.com/iterate-ch/cyberduck/issues/16192) in time.

For now, all we can do is wait. As a workaround, the developers have posted some [documentations](https://docs.cyberduck.io/protocols/profiles/google_client_id/), but I find it hard to follow. So this is my step-by-step guide to create custom Google Drive OAuth Client ID.

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> Update: The problem is now resolved. This post stays here in case of similar problem occurs again.
{: .prompt-tip }
<!-- markdownlint-restore -->

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> ## OFFICIAL workaround from developers
> The developers of Mountain Duck have released a detailed documentation: [Setup a Custom OAuth Client ID for Google Drive & Google Cloud Storage](https://docs.cyberduck.io/tutorials/custom_oauth_client_id/).
{: .prompt-info }
<!-- markdownlint-restore -->

# Preparation

Before you start, you will need:

- 01 Google account
- 01 licensned Mountain Duck installation
- 01 text editor (in my case, Notepad.exe will be my buddy)

# Main content

## Create Google Drive OAuth Client

Firtstly, we need to log in our Google account, access [Google Cloud resource manager](https://console.cloud.google.com/cloud-resource-manager) and select `Create Project`.

![Google Cloud Resource Manager](google-cloud-resource-manager.png)

Provide the name of the project, for example `WorkaroundDuck`.

![New Project](newproject.png)

Wait for a few moments, then navigate to [APIs & Services](https://console.cloud.google.com/apis/dashboard)

![Move to APIs & Services](goto-apis-and-service.png)

Remember to select your project then click on [`Enable APIs and Services`](https://console.cloud.google.com/apis/library?project=workaroundduck).

![Select project and enable api](select-then-enable.png)

Here, type `Google Drive API in the search box`, we will have 03 results that have Google Drive logo.

![Search Google drive api](search-gdrive-api.png)

![03 google drive api results](gdrive-api-results.png)

Select [`Google Drive API`](https://console.cloud.google.com/apis/library/drive.googleapis.com), then select `Enable`.

![Enable google drive api](enable-gdrive-api.png)

Repeat with [`Google Drive Activity API`](https://console.cloud.google.com/apis/library/driveactivity.googleapis.com) and [`Drive Labels API`](https://console.cloud.google.com/apis/library/drivelabels.googleapis.com).

Now, navigate to [`OAuth consent screen`](https://console.cloud.google.com/apis/credentials/consent)

![Goto OAuth consent screen](goto-oauth-consent-screen.png)

Select `External` then `Create`.

![Create External](create-external.png)

Fill in the required fields, use your own gmail addresses, then select `Save and Continue`.

![Fill fields](fill-fields.png)

Now, select `Add or Remove Scopes`.

![Select add or remove scopes](select-add-or-remove-scopes.png)

Select everything in the list, then `Update`.

![Select all and update](select-all-and-update.png)

After that, `Save and Continue`.

![Save and Continue](save-and-continue.png)

Now, select `Add Users` then add your primary, 2nd, 3rd, 4th, ... gmail addresses. 

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> Don't forget to add your own address :wink:
{: .prompt-tip }
<!-- markdownlint-restore -->

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> With this, you don't need to create different OAuth clients for every g-account. You can add up to `100` different addresses.
{: .prompt-tip }
<!-- markdownlint-restore -->

![Save and Continue](continue.png)

Select `Back to Dashboard` to finish this step.

![Back to Dashboard](back-to-dashboard.png)

Now, navigate to `Credentials`, select `Create Credentials` and `OAuth client ID`.

![New OAuth client ID](new-oauth-client-id.png)

Select `Appication type` as `Desktop app`, `Name` as your preferences, then `Create`.

![Select app type and name](app-type-and-name.png)

Now, you will have `Cliend ID` and `Client secret`. Save that for now.

![Client ID and Client Secret](client-id.png)

The `Client ID` has the structure as `NUMBER-STRING.apps.googleusercontent.com`.

## Create Custom Connection Profile

Open the text editor and paste the following lines:

```xml
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
    <dict>
        <key>Protocol</key>
        <string>googledrive</string>
        <key>Vendor</key>
        <string>googledrive</string>
        <key>Description</key>
        <string>Google Drive Custom OAuth Client ID</string>
        <key>Default Hostname</key>
        <string>www.googleapis.com</string>

        <key>OAuth Authorization Url</key>
        <string>https://accounts.google.com/o/oauth2/auth</string>
        <key>OAuth Token Url</key>
        <string>https://accounts.google.com/o/oauth2/token</string>
        <key>Scopes</key>
        <array>
            <string>https://www.googleapis.com/auth/drive</string>
        </array>
        <key>OAuth Redirect Url</key>
        <string>${oauth.application.identifier}:oauth</string>
        <key>OAuth Client ID</key>
        <string>NUMBER-STRING.apps.googleusercontent.com</string>
	<key>OAuth Client Secret</key>
	<string>CLIENT_SECRET</string>
	<key>OAuth Redirect Url</key>
	<string>com.googleusercontent.apps.NUMBER-STRING:oauth</string>
    </dict>
</plist>
```

Replace `CLIENT_SECRET`, `NUMBER`, `STRING` with your own credentials from `Client ID` and `Client secret`.

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> The fields you need to edit are: `OAuth Client ID` (line `24`), `OAuth Client Secret` (line `26`), `OAuth Redirect Url` (line `28`)
{: .prompt-tip }
<!-- markdownlint-restore -->

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> Don't edit any fields from line `22` and above !!!
{: .prompt-warning }
<!-- markdownlint-restore -->

Your file should look like this:

![Sample of final text file](sample-file.png)

Save the file with the extension `.cyberduckprofile`

![Save as cyberduckprofile](save-as-cyberduckprofile.png)

## Connect using custom connection profile

Now, double-click the newly created `.cyberduckprofile` file, then fill in the blanks.

![Finish](finish.png)

Then, login as usual.

# Conclusion

Congratulations, you have reached your custom OAuth client for Google Drive.
