# Docs
Documenation for ChiliPeppr's cloud calls.

ChiliPeppr has cloud data storage calls for use by any ChiliPeppr workspace or widget. This is essntially key/value data storage for any and all use cases. Each user has 1GB of storage available to them via Google's app engine. You can use the storage by a few simple cloud calls documented below.

# chilipeppr.com/datalogin

You can make a call to `/datalogin` and it will tell you whether you are logged in or not. If you are not logged in, you can navigate to the LoginUrl and login to Google's global login service. This authenticates you and stores a cookie for the domain chilipeppr.com so any subsequent call to chilipeppr.com has a token with your auth. This means all Ajax calls will be authenticated. You need to do this to use the other `/dataput`, `/dataget`, and `/datagetall` calls.

A basic call to `http://chilipeppr.com/datalogin` should return the JSON below.

```json
{
CurrentUser: null,
LoginUrl: "https://www.google.com/accounts/ServiceLogin?service=ah&passive=true&continue=https://appengine.google.com/_ah/conflogin%3Fcontinue%3Dhttp://chilipeppr.com/&ltmpl=gm&shdf=ChYLEgZhaG5hbWUaCkNoaWxpUGVwcHIMEgJhaCIUDIFUs68E-AwFG0sRhqV3mL9W4zcoATIUMV9tuhJJyreZ2carM4r9jwJYLbU",
LogoutUrl: "http://chilipeppr.com/_ah/logout?continue=https://www.google.com/accounts/Logout%3Fcontinue%3Dhttps://appengine.google.com/_ah/logout%253Fcontinue%253Dhttp://chilipeppr.com/%26service%3Dah"
}
```
Once you are logged in you should see JSON similar to what is below.

```json
{
CurrentUser: {
Email: "jlauer12@gmail.com",
AuthDomain: "gmail.com",
Admin: true,
ID: "108306807449777002835",
ClientID: "",
FederatedIdentity: "",
FederatedProvider: ""
},
LoginUrl: "https://www.google.com/accounts/ServiceLogin?service=ah&passive=true&continue=https://appengine.google.com/_ah/conflogin%3Fcontinue%3Dhttp://chilipeppr.com/&ltmpl=gm&shdf=ChYLEgZhaG5hbWUaCkNoaWxpUGVwcHIMEgJhaCIUDIFUs68E-AwFG0sRhqV3mL9W4zcoATIUMV9tuhJJyreZ2carM4r9jwJYLbU",
LogoutUrl: "http://chilipeppr.com/_ah/logout?continue=https://www.google.com/accounts/Logout%3Fcontinue%3Dhttps://appengine.google.com/_ah/logout%253Fcontinue%253Dhttp://chilipeppr.com/%26service%3Dah"
}
```
To logout simply navigate to the LogoutUrl in your browser and all credentials will be removed.

Here is some sample Javascript for checking if the user is logged in or not.

```javascript
currentUser: null,
checkLogin: function() {
  var that = this;
  var jqxhr = $.ajax({
      url: "http://www.chilipeppr.com/datalogin?callback=?",
      dataType: 'jsonp',
      jsonpCallback: 'headerpanel_dataloginCallback',
      cache: true,
    }).done(function(data) {
      console.log(data);
      if (data.CurrentUser != undefined && data.CurrentUser != null) {
        console.log("user logged in ", data.CurrentUser);
        that.currentUser = data.CurrentUser;
        $('#com-chilipeppr-hdr-login').text(data.CurrentUser.Email);
        $('#com-chilipeppr-hdr-dd-login').addClass("hidden");
        $('#com-chilipeppr-hdr-dd-logout').prop('href', data.LogoutUrl);
        // if they click their login id log them out
        $('#com-chilipeppr-hdr-login').prop('href', data.LogoutUrl);
      } else {
        console.log("user NOT logged in");
        $('#com-chilipeppr-hdr-login').text("Login");
        $('#com-chilipeppr-hdr-dd-login').removeClass("hidden");
        $('#com-chilipeppr-hdr-dd-logout').addClass("hidden");
        $('#com-chilipeppr-hdr-dd-login').prop('href', data.LoginUrl);
        $('#com-chilipeppr-hdr-dd-login').prop('target', "new");
        $('#com-chilipeppr-hdr-login').prop('href', data.LoginUrl);
        $('#com-chilipeppr-hdr-login').prop('target', "new");
      }
    })
    .fail(function() {
      console.warn("Failed to make the ajax call to check ChiliPeppr login status");
    })
},
```

# chilipeppr.com/dataput

To store any key/value pair you can call `http://chilipeppr.com/dataput?key=mykey&val=myval`

Parameters
key: any string you choose. It is case-sensitive so myval is unique from MyVal.
val: any string you choose. You can store any length of data here up to 1GB. It is recommended you don't store more than 100Kb and split up larger amounts if you want your Ajax calls to be snappier. Consider spreading large data out since this uses Google's DataStore backing database. 

After calling `/dataput` you should see something similar to

```json
{
Name: "mykey",
Value: "myval",
CreateDate: "2016-01-30T23:06:00.647278Z",
User: "jlauer12@gmail.com"
}
```

# chilipeppr.com/dataget

To retrieve data, simply call `http://chilipeppr.com/dataget?key=mykey

```json
{
Name: "mykey",
Value: "myval",
CreateDate: "2016-01-30T23:06:00.647278Z",
User: "jlauer12@gmail.com"
}
```

# chilipeppr.com/datagetallkeys

If you need to see all the keys that are stored for a particular user, simply call `http://chilipeppr.com/datagetallkeys`

You should see values returned similar to below:

```json
[
{
Name: "blah",
ValueSize: 4,
CreateDate: "2016-01-30T21:56:13.953805Z",
User: "jlauer12@gmail.com"
},
{
Name: "mykey",
ValueSize: 5,
CreateDate: "2016-01-30T23:06:00.647278Z",
User: "jlauer12@gmail.com"
}
]
```

Here is some sample Javascript code for doing an Ajax call to get all the user's datakeys.

```javascript
getUserDataKeysFromChiliPepprStorage: function() {

  console.log("Doing getUserDataKeysFromChiliPepprStorage");

  // this queries chilipeppr's storage facility to see what
  // keys are available for the user
  $('#' + this.id + ' .alert-warning').addClass('hidden');

  var that = this;
  $.ajax({
      url: "http://www.chilipeppr.com/datagetallkeys",
      xhrFields: {
          withCredentials: true
      }
  })
  .done(function(data) {

      // see if error
      if (data.Error) {
          // we got json, but it's error
          $('#' + that.id + ' .alert-warning')
              .html("<p>We can't retrieve your data from ChiliPeppr because you are not logged in. Please login to ChiliPeppr to see your list of available data keys.</p><p>Error: " + data.Msg + "</p>")
              .removeClass('hidden');
          return;
      }

      // loop thru keys and get org-jscut ones
      var keys = [];
      var keylist = "<ol>";
      data.forEach(function(item) {
          console.log("item:", item);
          if (item.Name) { // && item.Name.match(/workspace/)) {
              // we have a jscut file
              keys.push({
                  name: item.Name,
                  created: item.CreateDate,
                  size: item.ValueSize
              });
              keylist += "<li>" + item.Name + "</li>";
          }
      });
      keylist += "</ol>";

      $('#' + that.id + " .keylist").html(keylist);
      console.log("added keylist:", keylist);

  });
},
```

# chilipeppr.com/datagetall

If you need to see all the keys and data stored for a particular user, simply call `http://chilipeppr.com/datagetall`. Keep in mind this could return a massive amount of data so it is recommend you not use this call, but rather individually retrieve each key/val by making calls direct to `/dataget?key=mykey`.

You should see values returned similar to below:

```json
[
{
Name: "Blah",
Value: "neato",
CreateDate: "2016-01-30T23:05:09.6337Z",
User: "jlauer12@gmail.com"
},
{
Name: "mykey",
Value: "myval",
CreateDate: "2016-01-30T23:06:00.647278Z",
User: "jlauer12@gmail.com"
}
]
```

# Domain name scoping

One of the common pitfalls is to make sure you are consistently using the same domain name across all of your calls. Sometimes folks run into problems when they make these calls from `http://www.chilipeppr.com` rather than always sticking with `http://chilipeppr.com`. If you authenticate to `chilipeppr.com` then make sure all calls including Ajax calls are to this domain or you will wonder why things aren't working. The cookie with your auth token is only passed for the exact domain.
