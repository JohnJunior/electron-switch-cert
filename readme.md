# How to switch codesign certificates on Electron App for Mac (OSX)


When you have an electron app that originally was distributed with a certificate for an individual developer. And you would like to sign the next update to that app with a certificate from your organization. It will break the auto update installation process, because codesign requirements aren't met.

So, to prevent the error, you need to sign the next app with both certificates. All next apps sign with a new certificate.

See the example:
```
version 1.0.0 - sign with certificate 1
version 1.0.1 - sign with certificate 1 and 2
version 1.0.2 - sign with certificate 2
```

I suggest you look at the ``` designated requirement ``` (or DR) of the application, when signed with the old certificate (see Apple Developer portal for more information about [code requirements](https://developer.apple.com/library/archive/technotes/tn2206/_index.html#//apple_ref/doc/uid/DTS40007919-CH1-TNTAG4)).

Open terminal and use command ``` codesign -d -r- <path/to/app> ```. You'll see something like that:
```
Executable=/path/to/app/Contents/MacOS/<appname>
designated => identifier "xxxxx.xxxxx.xxxxxx" and anchor apple generic and certificate leaf[subject.CN] = "Super Developer: Awesome Organization (XXXXXXXX)" and certificate 1[field.1.2.345.678901.234.5.6.7] /* exists */
```

Build an app with new certificate and use the same command:
```
Executable=/path/to/new_app/Contents/MacOS/<appname>
designated => identifier "xxxxx.xxxxx.xxxxxx" and anchor apple generic and certificate leaf[subject.CN] = "Apple Distribution: Awesome Organization (XXXXXXXX)" and certificate 1[field.1.2.345.678901.234.5.6.7] /* exists */
```

After that combine both designated requirements into ```electron-builder-requirements.txt``` file using ```or``` logical operator between and do not include ```identifier "xxxxx.xxxxx.xxxxxx" and ```.

The ```electron-builder-requirements.txt``` example:
```
designated => anchor apple generic and certificate leaf[subject.CN] = "Super Developer: Awesome Organization (XXXXXXXX)" and certificate 1[field.1.2.345.678901.234.5.6.7] /* exists */ or anchor apple generic and certificate leaf[subject.CN] = "Apple Distribution: Awesome Organization (XXXXXXXX)" and certificate 1[field.1.2.345.678901.234.5.6.7] /* exists */
```

Include the ```electron-builder-requirements.txt``` file to ```electron-builder``` configuration file under ```mac```:
```
"mac": {
    "requirements": electron-builder-requirements.txt
}
```
and build the app.

Publish a new version of the app, signed with the old certificate, but with the DR that contains information about both certificates and wait until everyone has the version running on their desktop that includes mention of both certificates in the DR.

After that, you can remove ```electron-builder-requirements.txt``` and mention in ```electron-builder``` configuration file.

Release a new version of the app signed with the new certificate (no requirements file is needed, and it will list only its own certificate in the DR).

Profit.

P.S. To be sure that your app is signed with right certificate you can set ```identity``` parameter with certificate hash in ```electron-builder``` configuration file:
```
"mac": {
    "identity": "<SHA-1 cert hash here (40 digit)>"
}
```

P.P.S. Good luck and have fun :D