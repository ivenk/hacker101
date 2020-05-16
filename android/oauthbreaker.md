##Oauthbreaker

###Flag 1:

Launching the app and clicking through the buttons shows the basic work flow. The first activity opens the browser through a url containing a alot of parameters like a redirect url. The redirect url activates an intant. The button on the webpage follows the given url. Manipulating the redirect url parameter gives the first flag. 
###Flag 2: 

At this point it is not quit clear how that helps in any way. Really interesting is the last page. Clicking on the link in the web browser shows an empty activity. This seems pretty strange. After unpacking the .apk and decompiling the java code it becomes clear that the browser activity actually calls a url in a web view. More interesting is the fact that the actual url which can be found in the source code is never called. Instead the intent data is checked and a different url is build from the query parameters. Checking the manifest shows that there are two intent filters. One launching the main activity the other one a browser activity. We want to launch browser activity and provide query parameters in order to control the url the browser activity calls. After some tries the following command provides the desired result:

adb shell am start -a android.intent.action.VIEW -d "oauth://final/?uri=http://info.cern.ch"

Scrolling through the source code also gives a lot of information about the exact nature of the activity. It shows a webview being used in order to access the given urls. This webview has js enabled and provides an interface for communication between the webview and the android app. This is done in the line :

webView.addJavascriptInterface(new WebAppInterface(getApplicationContext()), "iface");

iface specifies the name of the interface in the code. The source code also reveals that he webappinterface class has only a single method named getFlagPath. At this point the activit can be launched via:

adb shell am start -a android.intent.action.VIEW -d "oauth://final/?uri=http://35.227.24.107/298be255e5"

Burp is used as a proxy and interupts the returned web page. Modifying the webpage with some js code to call iface.getFlagPath() and displaying the result on the web page will give a filename. Trying to access the file on the hackerone server gives the flag.




