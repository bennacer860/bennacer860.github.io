---
layout: post
title: "Find Who is Accessing my Google Drive Files"
date: 2015-07-12 15:20:15 -0400
comments: true
categories: 
---

I Recently had to share some documents from my Google Drive with a team of developers outside the office. Most of these documents were API docs. After a couple of months and when those developers didn't need the API docs anymore, it was almost impossible for me to find what i shared with them, it was like looking for a needle in a haystack. I bet i am not the only one that had this problem before!!!

So how would i do that manually. I would go to each file and click on share button. see the list of the permission and remove the users i don't want to. It means that i would need to go through 200 files!!! I am too lazy for that!!

 ![https://github.com/gimite/google-drive-ruby](http://i.stack.imgur.com/tVwCC.gif)

Since i like Ruby programming i thought about using the Google Drive API to accomplish this task. I looked at [this gem](https://github.com/gimite/google-drive-ruby) and i was immediately discouraged by it's complexity.  It has to be a better solution!!! 

After looking at [Google Script overview page](https://developers.google.com/apps-script/overview) i realized that this is going to be fairly easy to solve an amazing complex problem. Google Script use a language based on Javascript and this [API doc](https://developers.google.com/apps-script/reference/calendar/).

This code is largely inspired from [Amit Agarwal]() code.
```javascript
function Start() {
  
  
  var files = DriveApp.getFiles();  
  var timezone = Session.getScriptTimeZone();
  var email = Session.getActiveUser().getEmail();
  
  var MAXFILES = 300; count=1;
  var file, date, access, url, permission;
  var privacy, view, viewers, edit, editors;
  
  var rows = [["File Name", "Who has access?", "Date Created"]];
  
  for (var i=0; i<MAXFILES && files.hasNext(); i++ ) {
    
    file = files.next();
    
    try {
      
      access     = file.getSharingAccess();
      permission = file.getSharingPermission();
      viewers    = file.getViewers();      
      editors    = file.getEditors();
      
      view = [];
      edit = [];
      
      date =  Utilities.formatDate(file.getDateCreated(), timezone, "yyyy-MM-dd HH:mm")
      url = count++ + '. <a href="' + file.getUrl() + '">' + file.getName() + '</a>';
      
      for (var v=0; v<viewers.length; v++) {                
        view.push(viewers[v].getName() + " " + viewers[v].getEmail());
      }
      
      for (var ed=0; ed<editors.length; ed++) {                
        edit.push(editors[ed].getName() + " " + editors[ed].getEmail());
      }
      
      switch(access) {
        case DriveApp.Access.PRIVATE:
          privacy = "Private";
          break;
        case DriveApp.Access.ANYONE:
          privacy = "Anyone";
          break;
        case DriveApp.Access.ANYONE_WITH_LINK:
          privacy = "Anyone with a link";
          break;
        case DriveApp.Access.DOMAIN:
          privacy = "Anyone inside domain";
          break;
        case DriveApp.Access.DOMAIN_WITH_LINK:
          privacy = "Anyone inside domain who has the link";
          break;
        default:
          privacy = "Unknown";
      }
      
      switch(permission) {
        case DriveApp.Permission.COMMENT:
          permission = "can comment";
          break;
        case DriveApp.Permission.VIEW:
          permission = "can view";
          break;
        case DriveApp.Permission.EDIT:
          permission = "can edit";
          break;
        default:
          permission = "";
      }
      
      view = view.join(", ");
            
      edit = edit.join(", ");
            
      privacy += (permission === "" ? "" : " " + permission) 
               + (edit === "" ? "" : "<br><small>" + edit + " can edit</small>")
               + (view === "" ? "" : "<br><small>" + view + " can view</small>")
      
      rows.push([url, privacy, date]);
      
    } catch (e) {Logger.log(e.toString()); Logger.log(file.getName());};
    
  }
  
  var html = "<p style='text-align:center'><strong><a style='font-size:160%;text-decoration:none;color:#49B3F5;' File Permissions Report for Google Drive</a></strong></p>";


  html += "<table border='1' cellpadding='5' cellspacing='0'><tr><td><b>" + rows[0].join("</b></td><td><b>") + "</b></td></tr>";
  
  for (var i=1; i<rows.length; i++) {
    html += "<tr><td>" + rows[i].join("</td><td>") + "</td></tr>";
  }
  
  html += "</table>";
  
  
  GmailApp.sendEmail(email, "Google Drive - File Permissions Report", "", {htmlBody: html});
  
}
```
