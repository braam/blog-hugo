---
title: "Estos Procall Best Practices"
date: 2023-09-18T15:12:43+02:00
draft: false
tags:
- Estos
---

When installing Estos Procall it's a good habbit to look for a way to globally manage the different popups/actions users can have. This is needed, because you can configure everything locally on the workstations, but this
would become time consuming when you've to do config changes.

### 1) Use Global Actions File
We can use a global action file placed somewhere on a server in the network and we can bind this to a UCServer profile. This profile will be then applied on users/groups. This way you can have different actions linked to users/groups or departments withing the organization.

Make your actions on a client installed locally on you workstation, after that export the actions.xml file and copy/paste it to the server on a shared folder.
Next edit the profile in the UCServer: **Profiles > Default > Predefined ACtions**

![Estos_Actions](/posts_images/estos_best-practices_01.png)

&nbsp;
### 2) Use HTML in Database
When your database can directly contain the necessary HTML markup for your popup (RemoteContact.xslt) you can let the customer change this whenever needed without having to change anything yourself.
Therefore following is needed inside the RemoteContact.xslt, assuming that Custom0 (User0) contains the mapped field from the database:
```
<xslt:if test="normalize-space(XSLT/Contact/Custom0) != ''">
  <tr><td>
    <xslt:value-of select="XSLT/Contact/Custom0" disable-output-escaping="yes" />
  </td></tr>
</xslt:if>
```

As you can see the option **disable-output-escaping="yes"** is needed, this way the xslt will keep the HTML markup and does not parse it as pure text.

&nbsp;
### 3) Use HTML Within Custom Field
If the customer can not maintain his own database, you can chose for the option to store the HTML tags within the replicator itself. You can use the extended settings within the replicator and use the prefix/suffix fields 
to embed the value within a valid HTML tag. This way when a change is needed, you can do it yourself on the server and all clients will bbe updated at once.

![Estos_Extended-Settings_01](/posts_images/estos_best-practices_02.png)
![Estos_Extended-Settings_02](/posts_images/estos_best-practices_03.png)
