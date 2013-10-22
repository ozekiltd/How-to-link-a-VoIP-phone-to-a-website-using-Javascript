How to link a VoIP phone to a website using Ozeki JavaScript API
=========================================

Below you can find an excellent way on how to follow your calls easily using 
Ozeki Phone System XE JavaScript API. By displaying an HTTP popup window, it is quite simple to receive notifications about the calls made through your own website.

_More info at http://www.ozekiphone.com_

For this purpose, you only have to execute the following steps:

**Step 1:** Generate Security Token for the authentication

**Step 2:** Connect to Ozeki Phone System XE through JavaScript API

**Step 3:** Subscribe for the _sessionCreated_ and _sessionStateChanged_ event 

**Step 4:** When the _sessionStateChanged_ event is triggered, show a Popup with the details of the call


After these steps, you will receive a notification in the form of Popup on your own site every time when a call is created in the system. Of course, there are more possibilities in JavaScript API, but this article gives information about about the call created and call state changed events.

**Let’s see each step in details!**


Step 1: Generate Security Token for the authentication
=========================================

In order to be able to see the created calls in the system, you have to get access from the PBX for your application. You can do this by generate a Security Token for your program and with that token you can connect to PBX.

You have the possibility to use a generated Security Token for the signin. In case of the Security Token it can be given what it can be used for and until what time, 1 day is default. Such possibilities for usage are:

	- Sending a message 

	- Initiating calls inside or outside the system (mobile and line calls)

	- Query of the Call Information

	- and a lot of others

We can get a Security Token through HTTP API (if it is enabled), or we can generate it in the system after logging in with the _GenerateSecurityToken_ Command of the HTTP API Tester. 

===Option 1===

To get a Security Token through HTTP API, you should send a simple HTTP request to the PBX and the needed token is going to be in the given response.
	
In the code example below you can see what request parameters should be sent to the PBX.
```

Command=GenerateSecurityToken
AuthXml=<?xml version="1.0"?>
<SecurityToken>
	<AllowSessionModification>False</AllowSessionModification>
</SecurityToken>

```
**Code example 1** - The original HTTP Request

The specified url parameter list and the whole request is in the code below. You can send the request like you copy the url encoded text below to the headline of your browser. Of course you have to change the IP address to the address of the PBX here, too:
```
http://ozekixepbx.ip:7780/?Command=GenerateSecurityToken&AuthXml=%3c%3fxml+version%3d%221.0%22%3f%3e%0d%0a%3cSecurityToken%3e%0d%0a%09%3cAllowSessionModification%3eTrue%3c%2fAllowSessionModification%3e%0d%0a%3c%2fSecurityToken%3e%0d%0a
```
**Code example 2** - Url encoded request

On the summary page (http://www.ozekiphone.com/voip-http-commands-231.html) you can get further information about the HTTP API and the commands of it.

===Option 2===
The other method for getting the needed Security Token is that you login to the PBX and generate the token on your own with the aid of HTTP API Tester.

For this, choose the _Productivity/HTTP API_ menu fom the menu line above in the PBX. Then choose the _GenerateSecurityToken_ Command, click on the _Test_ tab, then to the _Send request_ button!



[http://code.google.com/ http://inside.ozekiphone.com/attachments/1150/SecurityToken.png]

**Figure 1**  - Generate Security Token with HTTP API Tester

The generated Security Token can be found in the received response 
between the _SecurityToken_ tags.

Step 2: Connect to Ozeki Phone System XE through JavaScript API
=========================================

As for the start create a simple HTML page where you refer a few scripts and JavaScript API, too. The created file has to be hosted and run on a Web-server, because it will result in an error if it is simply run on the file system (for 
example from Windows Explorer). 
```
<!DOCTYPE html>
<!-- Important: This file has to be hosted on a web server (e.g. Apache, IIS etc.) -->
<html>
<head>
    <link rel="stylesheet" type="text/css" href="http://code.jquery.com/ui/1.10.3/themes/ui-darkness/jquery-ui.min.css" />
    <script type="text/javascript" src="http://code.jquery.com/jquery-latest.min.js"/>
    <script type="text/javascript" src="http://code.jquery.com/ui/1.10.3/jquery-ui.min.js"/>
    <script type="text/javascript" src="http://ozekixepbx.ip:7777/WebphoneAPI"/>
    <script type="text/javascript">
	<!-- Your scripts will be here -->
</script>
</head>
<body>	
</body>
</html>
```
**Code example 3** - The basic HTML page that is going to be extended step by step.

![alt tag](http://inside.ozekiphone.com/attachments/1150/IpAddress.png)

**Figure 2**  - Address of your PBX

The row below is going to add reference to the JavaScript API via the running Ozeki Phone System XE. Please use your own IP address of your installed PBX instead of the **<font color="#FF0000">ozekixepbx.ip**</font>.

```
<script type="text/javascript" src="http://ozekixepbx.ip:7777/WebphoneAPI"/>
```

**Code example 4** - Adding reference to the JavaScript API


When you are ready with this, you can start to write the more interesting code parts. First we will see some variables and the connection. 
```
<script type="text/javascript">
	<!-- Your scripts will be here -->
	//////// Use your settings //////////
	var serverAddress = "ozekixepbx.ip";
	var numberToSubscribe = "1001";
	var securityToken = "nixtwtqHr0qTRk9FcrV8Sg" ;
	////////////////////////////////////

	$( document ).ready(function() {
		OzWebClient.onConnectionStateChanged(connectionStateChanged);
		OzWebClient.connect(serverAddress, securityToken);
	});
</script>
```
**Code example 5** - Variables to be changed, and connection to the PBX

The _numberToSubscribe_ should be a phone number. We take it as a basic that we are curious for the events of only this phone number. But in case of a need it is extendable, of course, and this requirement can even be left.  

After clarifying the role of the variables we can see the real connection. Before we connect, we should subscribe for the <a href="index.php?owpn=1019">OzWebClient.onConnectionStateChanged</a>
event.  We can achieve this by subscribing the _connectionStateChanged_ method that is going to be mentioned later.  After this we should call the <a href="index.php?owpn=1013">OzWebClient.connect</a>
method.  There is the address of the PBX and the requested Security Token.

Step 3: Subscribe for the _sessionCreated_ and _sessionStateChanged_ event
=========================================

We have already initiated the connection request towards the server earlier, and we get the response from it in the subscribing _connectionStateChange_ method:
```
function connectionStateChanged(info) {
	if (info.State == ConnectionState.ACCESS_GRANTED)
		OzWebClient.onSessionCreated (sessionCreated);
	else if (info.State == ConnectionState.ACCESS_DENIED)
		console.log("Please check the login details of the Webphone Extension!");
	else
		console.log("Please check the availability of the PBX!");
}	
```
**Code example 6** - Subscribing function that gives back the response of the PBX to the login request.

We can see that the _info.State_ parameter of the function carries the response of the server. If the login was successful, we subscribe for the _OzWebClient.onSessionCreated_ event. So when a new call is created in the PBX, the subscribing method will be called: 
```
function sessionCreated(session) {
	session.onSessionStateChanged(sessionStateChanged);
}
```
**Code example 7** - Function that is called when a call is created.

We subscribe for the changes of the call state in the function, so every time when it gets into another state (_Setup, Ringing, Incall, Completed_ etc. ) we receive a notification through the _sessionStateChanged_ method.


Step 4: When the _sessionStateChanged_ event is triggered, show a Popup
=========================================

As soon as the _onSessionStateChanged_ event is triggered, the function below is called: 
```
function sessionStateChanged(session, sessionState ) {
	if (session.caller == numberToSubscribe || session.callee == numberToSubscribe){
		showPopUp(session.source, session.destination, session.direction, session.sessionState);
	}
}
```
**Code example 8** - Function called when the call state changes

We get the actual session in the function, that carries the information about the actual call in its parameters. In the function above we check whether the phone number of the caller or the callee is the same as the phone number given by us (_numberToSubscribe_), and if it is, then we call the _showPopUp_ method with the parameters of the call: 
```
function showPopUp(source, destination, direction, state){
    if ($("#NoficationPopup").length > 0){
            $("#lbCaller").html('Caller: ' + state);
            $("#lbCallee").html('Callee: ' + callee);
            $("#lbDirection").html('Direction: ' + direction);
            $("#lbState").html('State: ' + state);
    }
    else 
        $('<div id="NoficationPopup" title="' + direction + '">' +
                '<div align=left>' +
                    '<label id="lbCaller">Caller: '+source+'</label> <BR />'  +
                    '<label id="lbCallee">Callee: '+destination+'</label> <BR />' +
                    '<label id="lbDirection">Direction: '+direction+'</label> <BR />' +
                    '<label id="lbState">State: '+state+'</label> <BR />' +
                '</div> ' +
          '</div>')
          .dialog({
                resizable: false, position: [0,0], minWidth: 250, minHeight: 100,
                close: function () {
                    $(this).dialog("destroy").remove();
    }});
}
```
**Code example 9** - A function that executes the showing of a Popup.

We can see that when the Popup is not in the foreground, we create it, having uploaded with the appropriate values. If we have already showed this dialogue earlier, we modify its values.

[http://code.google.com/ http://www.ozekiphone.com/attachments/1150/DisplayPopup.png]

**Figure 3** - An example popup, when 1001 calls the 5000
