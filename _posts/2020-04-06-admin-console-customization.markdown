---
layout: post
title:  "WebLogic 11g / 12c Admin Console Customization"
date:   2020-04-06 12:30:00 +0300
categories: oracle weblogic
---
If you have many WebLogic Server environments to manage, adding a small environment information on the Admin Console could prevent making configurations on the wrong environment. This is a small change made in the css file and does not need any restart. 


# **On WebLogic Server 12c**

<img src="{{site.baseurl}}/assets/img/weblogic/admin-cust-12c.png">

```bash
vi $MW_HOME/wlserver/server/lib/consoleapp/webapp/framework/skins/wlsconsole/css/console.css

#product-brand-name:before {
  background-color: green;
  color: white;
  content: "WCC 12c DEV";
  font-size: large;
  margin-right: 20px;
  padding: 35px;
}

```


# **On WebLogic Server 11c**

<img src="{{site.baseurl}}/assets/img/weblogic/admin-cust-11g.png">

```bash
vi $MW_HOME/wlserver_10.3/server/lib/consoleapp/webapp/css/content.css

.toolbar-info::before {
   content: "PORTAL TEST";
   color: white;
   background-color: orange;
   font-size: large;
   font-weight: bold;
   padding-right: 20px;
   padding-left: 20px;
}

```


