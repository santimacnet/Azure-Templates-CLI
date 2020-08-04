##Tutorial de Javascript Node para hacer login mediante Azure AD

Create an Azure AD application and Web page for the user to sign in and display details
--------------------------------------------------------------------------------------------------
Ref: https://docs.microsoft.com/es-es/learn/modules/getting-started-identity/3-exercise-different-token-types

Codigo fuente para archivo **server.js**
```
var express = require('express');
var app = express();
var morgan = require('morgan');
var path = require('path');

var port = 3007;
app.use(morgan('dev'));

// set the front-end folder to serve public assets.
app.use(express.static('web'));

// set up our one route to the index.html file.
app.get('*', function (req, res) {
  res.sendFile(path.join(__dirname + '/index.html'));
});

// Start the server.
app.listen(port);
console.log(`Listening on port ${port}...`);
console.log('Press CTRL+C to stop the web server...');
```


Codigo fuente para archivo **index.html**
```
<!DOCTYPE html>
<html>
<head>
  <title>Getting Started with Microsoft identity</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/bluebird/3.5.5/bluebird.min.js"></script>
  <script src="https://alcdn.msftauth.net/lib/1.1.3/js/msal.min.js"></script>
</head>

<body>
  <div class="container">
    <div>
      <p id="WelcomeMessage">Microsoft Authentication Library For Javascript (MSAL.js) Exercise</p>
      <button id="SignIn" onclick="signIn()">Sign In</button>
    </div>
    <div>
      <pre id="json"></pre>
    </div>
  </div>

  <script>
    var msalConfig = {
      auth: {
        clientId: 'xxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxx',
        authority: 'https://login.microsoftonline.com/b34e3aa6-3974-4b4c-b813-4cdfa4968194',
        redirectURI: 'http://localhost:3007'
      },
      cache: {
        cacheLocation: "localStorage",
        storeAuthStateInCookie: true
      }
    };

    var graphConfig = {
      graphMeEndpoint: "https://graph.microsoft.com/v1.0/me",
      requestObj: {
        scopes: ["user.read"]
      }
    };

    var msalApplication = new Msal.UserAgentApplication(msalConfig);


    // init the auth handling on the page
    initPage();

    msalApplication.handleRedirectCallback(authRedirectCallBack);
    // TODO: add CODE before this line



 
    function updateUserInterface() {
  var divWelcome = document.getElementById('WelcomeMessage');
  divWelcome.innerHTML = `Welcome <strong>${msalApplication.getAccount().userName}</strong> to Microsoft Graph API`;

  var loginbutton = document.getElementById('SignIn');
  loginbutton.innerHTML = 'Sign Out';
  loginbutton.setAttribute('onclick', 'signOut();');
}

function acquireTokenPopupAndGetUser() {
  msalApplication.acquireTokenSilent(graphConfig.requestObj)
    .then(function (tokenResponse) {
      getUserFromMSGraph(graphConfig.graphMeEndpoint, tokenResponse.accessToken, graphAPICallback);
    }).catch(function (error) {
      console.log(error);
      if (requiresInteraction(error.errorCode)) {
        msalApplication.acquireTokenPopup(graphConfig.requestObj).then(function (tokenResponse) {
          getUserFromMSGraph(graphConfig.graphMeEndpoint, tokenResponse.accessToken, graphAPICallback);
        }).catch(function (error) {
          console.log(error);
        });
      }
    });
}

function acquireTokenRedirectAndGetUser() {
  msalApplication.acquireTokenSilent(graphConfig.requestObj).then(function (tokenResponse) {
    getUserFromMSGraph(graphConfig.graphMeEndpoint, tokenResponse.accessToken, graphAPICallback);
  }).catch(function (error) {
    console.log(error);
    if (requiresInteraction(error.errorCode)) {
      msalApplication.acquireTokenRedirect(graphConfig.requestObj);
    }
  });
}

function requiresInteraction(errorCode) {
  if (!errorCode || !errorCode.length) {
    return false;
  }
  return errorCode === "consent_required" ||
          errorCode === "interaction_required" ||
          errorCode === "login_required";
}

function authRedirectCallBack(error, response) {
  if (error) {
    console.log(error);
  } else {
    if (response.tokenType === "access_token") {
      getUserFromMSGraph(graphConfig.graphMeEndpoint, response.accessToken, graphAPICallback);
    } else {
      console.log("token type is:" + response.tokenType);
    }
  }
}

function getUserFromMSGraph(endpoint, accessToken, callback) {
  var xmlHttp = new XMLHttpRequest();
  xmlHttp.onreadystatechange = function () {
    if (this.readyState == 4 && this.status == 200)
      callback(JSON.parse(this.responseText));
  }
  xmlHttp.open("GET", endpoint, true);
  xmlHttp.setRequestHeader('Authorization', 'Bearer ' + accessToken);
  xmlHttp.send();
}

function graphAPICallback(data) {
  document.getElementById("json").innerHTML = JSON.stringify(data, null, 2);
}

function signIn() {
  msalApplication.loginPopup(graphConfig.requestObj)
    .then(function (loginResponse) {
      updateUserInterface();
      acquireTokenPopupAndGetUser();
    }).catch(function (error) {
      console.log(error);
    });
}

function signOut() {
  msalApplication.logout();
}

   // TODO: add FUNCTIONS before this line

    function initPage() {
      // Browser check variables
      var ua = window.navigator.userAgent;
      var msie = ua.indexOf('MSIE ');
      var msie11 = ua.indexOf('Trident/');
      var msedge = ua.indexOf('Edge/');
      var isIE = msie > 0 || msie11 > 0;
      var isEdge = msedge > 0;

      // if you support IE, recommendation: sign in using Redirect APIs vs. popup
      // Browser check variables
      // can change this to default an experience outside browser use
      var loginType = isIE ? "REDIRECT" : "POPUP";

      // runs on page load, change config to try different login types to see what is best for your application
      switch (isIE) {
        case true:
          document.getElementById("SignIn").onclick = function () {
            msalApplication.loginRedirect(graphConfig.requestObj);
          };

          // avoid duplicate code execution on page load in case of iframe and popup window
          if (msalApplication.getAccount() && !msalApplication.isCallback(window.location.hash)) {
            updateUserInterface();
            acquireTokenRedirectAndGetUser();
          }
          break;
        case false:
          // avoid duplicate code execution on page load in case of iframe and popup window
          if (msalApplication.getAccount()) {
            updateUserInterface();
            acquireTokenPopupAndGetUser();
          }
          break;
        default:
          console.error('Please set a valid login type');
          break;
      }
    }
  </script>
</body>
</html>
```

Seguir los pasos del link del tutorial para realizar el ejercicio.
