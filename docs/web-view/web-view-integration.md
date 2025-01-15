# Web view integration

- [Instructions](#instructing-which-url-web-view-should-open)
- [Communication between web view and adapter](#communication-between-web-view-and-word-adapter)
  - [Protocol](#protocol)
  - [Communication errors](#communication-errors)
- [Exposed word adapter functions](#exposed-word-adapter-functions)
  - [Append Text](#append-text)
  - [Append Html](#append-html)
  - [Append Image](#append-image)
  - [Append Document Custom Property](#append-document-custom-property)
  - [Save](#save)
  - [Save and exit](#save-and-exit)
  - [Cancel](#cancel)
- [Events from word adapter](#events-from-word-adapter)
  - [Save start](#save-start)
  - [Save success](#save-success)
  - [Save failed](#save-failed)
  - [Heartbeat fail](#heartbeat-fail)
  - [Heartbeat success](#heartbeat-success)
  - [Disabled editing](#on-disable-editing)

Word adapter comes with a functionality to integrate your own custom interface in word adapter application 

### Instructing which url web view should open
When you construct the link to open document in word adapter there is a parameter called `webViewPath` (when translated)
into query params `w`

FactoryWordAdapter comes with a special class that helps build links to open word adapter with document.

Link generator contains a function called `setWebViewPath` which can receive absolute path to web view path

> **Note**  
> Web view path must be absolute url
> 
> 
Example
```php
public function testWordAdapterAction(Request $request, WordAdapterLinkGeneratorFactoryService $wordAdapterLinkGeneratorFactoryService)
{
    $asset = Asset::getById($request->get('id')); // Load some simple doc asset

    $link = $wordAdapterLinkGeneratorFactoryService->createWordAdapterLinkGenerator();;
    $link->setAction(WordAdapterLinkGeneratorInterface::ACTION_EDIT);
    $link->setDocument1($asset->getFullPath());
    $link->setDocumentId($asset->getId());
    $link->setIsEditable(true);
    $link->setWebViewPath($this->generateUrl('your-web-view-route'), UrlGeneratorInterface::ABSOLUTE_URL);

    $htmlString = $this->renderView('documents/test-word-adapter.html.twig', [
        'link' => $link,
    ]);

    return new Response($htmlString);
}
```

You can read more about opening arguments and how to extend link generator in opening-arguments section of this 
documentation

<a name="communication"></a>
## Communication between web view and word adapter
Word adapter displays a web view as frontend part of the application.

**Word adapter communicates with web view, which means that** 
1. Web view can call exposed word adapter functions
2. Word adapter can trigger events on web view

### Protocol
#### Sending data to word adapter
The communication between web view -> word adapter is done through JSON string. 
The json string must contain 
1. Action property

The communication is done by calling `postMessage`
```javascript
 window.chrome.webview.postMessage(jsonString);
```
#### Receiving response from word adapter
After each request from web view a javascript event will be triggered with the response. 
The response will contain detail object with following defintion

```json
{
    message: "Text successfully appended" // Response message
    originalRequest: "{\"action\":\"appendText\",\"value\":\"Lorem ipsum\"}" // Original request sent
    status: "success" // Status succes or fail
}
```
Above 3 keys will always be present no matter if the status is success or fail. 
Additional keys might also be present in response javascript. 

Example of request and response: 
```javascript
document.addEventListener("word-adapter-event", function(e) {
    console.log(e.detail); // Triggered when sendValue functions is finished and response from word adapter is received
});

function sendValue(value) {

    const message = {
        'action': 'appendText',
        'value': value
    }
    window.chrome.webview.postMessage(JSON.stringify(message));
}
```

### Communication errors
Communication errors can occur if the command sent to word adapter is not according to the protocol specification

Error list  

| Error code | Description                                                                                          |
|------------|------------------------------------------------------------------------------------------------------|
| 001        | Unable to parse message request. Please refer to documentation on how to correcty construct telegram |
| 002        | Unable to parse request. Action property is missing from request                                     |
| 003        | Unable to parse request. Action append is missing value property                                     |
| 004        | Unable to parse request. Action appendImage is missing imagePath property                            |

## Exposed word adapter functions
### Append text

Example:
```javascript
const message = {
    'action': 'appendText',
    'value': 'Some text to append'
}
window.chrome.webview.postMessage(JSON.stringify(message));
```
This will append `Some text to append` to word document

### Append HTML

Example:
```javascript
const message = {
    'action': 'appendHTML',
    'value': '<p>Some text to <b>append</b></p>'
}
window.chrome.webview.postMessage(JSON.stringify(message));
```
This will append `Some text to append` to word document but it will parse simple HTML to word

#### Supported HTML Tags
Supported Html tags
Refer to w3schools tag list to see their meaning

```html
<a>
<h1-h6>
<abbr> and <acronym>
<b>, <i>, <u>, <s>, <del>, <ins>, <em>, <strike>, <strong>
<br> and <hr>
<img>, <figcaption>
<table>, <td>, <tr>, <th>, <tbody>, <thead>, <tfoot> and <caption>
<cite>
<div>, <span>, <font> and <p>
<pre>
<sub> and <sup>
<ul>, <ol> and <li>
<dd> and <dt>
<q> and <blockquote> (since 1.5)
<article>, <aside>, <section> are considered like <div>
```
### Append Image

Example:
```javascript
const message = {
    'action': 'appendImage',
    'imagePath': 'https://example.com/example.png', // required
    'alt': alt // optional
}
window.chrome.webview.postMessage(JSON.stringify(message));
```
This will append image to word document where the current cursor on document is.  
`imagePath` must be a valid absolute url to the image file on the web  
`alt` is alternative text and this property is optional.  

### Append Document Custom Property

Example:
```javascript
const message = {
    'action': 'appendCustomProperty',
    'property': 'document_name', // required
    'value': "some custom value" // required
}
window.chrome.webview.postMessage(JSON.stringify(message));
```
This will append custom document property to word document  
`property` the name of the custom property  
`value` value of the custom property

See more [View or change the properties for an Office file](https://support.microsoft.com/en-gb/office/view-or-change-the-properties-for-an-office-file-21d604c2-481e-4379-8e54-1dd4622c6b75)

### Save

Example:
```javascript
const message = {
    'action': 'save',
}
window.chrome.webview.postMessage(JSON.stringify(message));
```
Save action will trigger save of the word document and synchronization back to pimcore

### Save and exit

Example:
```javascript
const message = {
    'action': 'saveAndExit',
}
window.chrome.webview.postMessage(JSON.stringify(message));
```
Save and exit action will trigger save of the word document and synchronization back to pimcore.
After the sync has been made the document will close itself with word adapter

> **Note**  
> Saving and closing will not trigger response event since the web view will be destroyed.

### Cancel

Cancel action closes word adapter and word without showing save dialog.

Example:
```javascript
const message = {
    'action': 'cancel',
}
window.chrome.webview.postMessage(JSON.stringify(message));
```

> **Note**  
> Cancel action will not ask to save changes. The action will be executed immediately

## Events from word adapter
### Save start
`word-adapter-on-save-start`  

Triggered when file save and upload starts inside word adapter
```javascript
document.addEventListener("word-adapter-on-save-start", function(e) {
    console.log("Save start"); 
});
```

### Save success
`word-adapter-on-save-success`

Triggered when file save has been successfully run
```javascript
document.addEventListener("word-adapter-on-save-success", function(e) {
    console.log("Save success"); 
});
```

### Save failed
`word-adapter-on-save-fail`

Triggered when file save has failed
```javascript
document.addEventListener("word-adapter-on-save-fail", function(e) {
    console.log("Save failed"); 
});
```


### Heartbeat fail
`word-adapter-event-on-heartbeat-fail`

Triggered when heartbeat either fails to connect to server or fail response is returned from server.  
This can be classified like word adapter went offline for some time and that is trying to reconnect
```javascript
document.addEventListener("word-adapter-event-on-heartbeat-fail", function(e) {
    console.log("Heartbeat failed"); 
});
```


### Heartbeat success
`word-adapter-event-on-heartbeat-success`

Triggered when heartbeat successfully connects to server and gets acknowledge that heartbeat is success. 
```javascript
document.addEventListener("word-adapter-event-on-heartbeat-success", function(e) {
    console.log("Heartbeat success"); 
});
```

### On disable editing
`word-adapter-event-on-disabled-editing`

Triggered when document lost ability to be stored on server. This can occur if the user was for a longer period of time
inactive (sleep mode or hibernate) or lost internet connection, while the document was still opened. 

This can usually occur when heartbeat fails for a longer period of time, while on the server side because of this longer
period document is considered to be in non edit mode. 
```javascript
document.addEventListener("word-adapter-event-on-disabled-editing", function(e) {
    console.log("Word adapter disabled editing"); 
});
```
