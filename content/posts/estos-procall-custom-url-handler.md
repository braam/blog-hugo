---
title: "Estos Procall Custom Url Handler"
date: 2022-02-16T09:15:00+01:00
draft: false
tags:
- Estos
---

By default the Estos Procal popup will open URL’s in the Internet Explore webbrowser, most companies don’t like this behaviour and want to open the hyperlinks in Edge / Firefox or Chrome browser instead.  With this little trick we can accomplish this.

We’ll adding a few registry keys to be able to launch the browser as an url handler, proceed cope/paste this reg keys to a file with extension .reg, after this double click the file and import the reg keys.
```reg
Windows Registry Editor Version 5.00

[HKEY_CLASSES_ROOT\chromeurl]
@="Default Value"
"URL Protocol"=""

[HKEY_CLASSES_ROOT\chromeurl\DefaultIcon]
@="\"C:\\Program Files (x86)\\Google\\Chrome\\Application\\chrome.exe\",0"

[HKEY_CLASSES_ROOT\chromeurl\shell]

[HKEY_CLASSES_ROOT\chromeurl\shell\open]

[HKEY_CLASSES_ROOT\chromeurl\shell\open\command]
@="cmd /V /C set \"arg1=%1\" & set arg1=!arg1:chromeurl:=! & start chrome !arg1!"


[HKEY_CURRENT_USER\Software\Microsoft\Internet Explorer\ProtocolExecute\chromeurl]
"WarnOnOpen"=dword:00000000


[HKEY_CLASSES_ROOT\firefoxurl]
@="Default Value"
"URL Protocol"=""

[HKEY_CLASSES_ROOT\firefoxurl\DefaultIcon]
@="\"C:\\Program Files\\Mozilla Firefox\\firefox.exe\",0"

[HKEY_CLASSES_ROOT\firefoxurl\shell]

[HKEY_CLASSES_ROOT\firefoxurl\shell\open]

[HKEY_CLASSES_ROOT\firefoxurl\shell\open\command]
@="cmd /V /C set \"arg1=%1\" & set arg1=!arg1:firefoxurl:=! & start firefox !arg1!"


[HKEY_CURRENT_USER\Software\Microsoft\Internet Explorer\ProtocolExecute\firefoxurl]
"WarnOnOpen"=dword:00000000
```
**Please note only Firefox and Chrome are used for this example, Microsoft Edge has his own url handler available on every Windows system.**

When you succesfully imported the keys, you can try the url handlers. It’s important to open this url's in **Internet Explorer**. Estos Procall is rendering the xslt popup files with IE.
Use following hyperlinks in IE:

[Open blog in Chrome](chromeurl:https://blog.bankveld.be)\
[Open blog in Edge](microsoft-edge:https://blog.bankveld.be)\
[Open blog in Firefox](firefoxurl:https://blog.bankveld.be)

When you launch a hyperlink, a small command prompt window will open and close immediatly that’s the command that will launch the chosen browser. The correct browser should launch. **If it’s not try rebooting the pc and launch the test again.**

To use these hyperlinks in Estos Procall popup, we’ll first add a iframe tag, with this tag we will not have a second IE popup (place it before &lt;/body&gt;)
```html
<iframe name="browserLauncher" src="#" style="position: absolute;width:0;height:0;border:0;"></iframe>
      </body> 
```

After this you can create the hyperlinks in RemoteContact.xslt file:
```html
<tr><td>
<xslt:variable name="openChromeUrl" select="XSLT/Contact/Custom0"/><a href="chromeurl:https://blog.bankveld.be" target="browserLauncher">Open with Chrome</a><br />
<xslt:variable name="openFirefoxUrl" select="XSLT/Contact/Custom1"/><a href="firefoxurl:https://blog.bankveld.be" target="browserLauncher">Open with Firefox</a><br />
<xslt:variable name="openEdgeUrl" select="XSLT/Contact/Custom2"/><a href="microsoft-edge:https://blog.bankveld.be" target="browserLauncher">Open with Edge</a>
</td></tr>
```
