# Opening Arguments

- [Arguments](#arguments)
- [Generating Opening Arguments](#generating-opening-arguments)
- [Overriding config path](#overriding-config-path)
- [Understanding compare action](#understanding-compare-action)
- [Understanding merge action](#understanding-merge-action)
- [Overriding word adapter link generator](#overriding-word-adapter-link-generator)


### Arguments

To trigger opening of word adapter application URL needs to be constructed with special protocol factoryadapter:openDocs. 
After the protocol and custom domain is stated, the query parameters are the one that will give word adapter enough information to continue to work.

To open word adapter plugin a url needs to be constructed with following query parameters configuration


| Query param | Parameter                         | Default value | Description                                                                                                                                                                                                                                                                                                                                               |
|-------------|-----------------------------------|---------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| c           | configPath                        | null          | Absolute path to configuration file. This is usually a route that contains config data                                                                                                                                                                                                                                                                    |
| d1          | document1                         | null          | This is a document1. This document is the one which will edited through word adapter. This is the document that will be downloaded and opened by word adapter                                                                                                                                                                                             |
| d2          | document2                         | null          | When you want to open comparison tool, the first document will be the one that will be opened as document that contents of it should be merged to document2. So therefore the document1 is the document with 'newest' changes and it will not be edited by word adapter.                                                                                  |
| a           | action                            | null          | One of the following actions: edit, compare, merge, concatPreview, concatGenerate                                                                                                                                                                                                                                                                         |
| e           | isEditable                        | false         | If the document is not editable no matter the action this  it will be downloaded and opened in read only mode                                                                                                                                                                                                                                             |
| ck          | cookie                            | null          | The cookies that will identify the user who is using word adapter. At this point only cookie validation is supported                                                                                                                                                                                                                                      |
| did         | documentId                        | null          | This is the document that word adapter is going to update. This parameter must correspond with d1 document otherwise document with provided id will be overwritten by document1. In case of merge action this parameter must be equal to document2 id                                                                                                     |
| mid         | mergeDocumentId                   | null          | This is the id of document that will be merged to destination document                                                                                                                                                                                                                                                                                    |
| w           | webViewPath                       | null          | Url of the webpage that will open a web view                                                                                                                                                                                                                                                                                                              |
| l           | lockToRevisions                   | null          | If not null, than this field should be a password for locking the revision toggle and keeping the revisions all the time unless password is entered to stop tracking revisions                                                                                                                                                                            |
| a{n}        | concatAssets                      | null          | This is list of document paths that will be concated                                                                                                                                                                                                                                                                                                      |
| i           | initialAsset                      | null          | This parameter only works on concat functionality and it is the firsts document that will be loaded instead of blank document. If the document is present, other documents will be appended into it. This is especially useful when you want to define styles for each of the documents that will be concat. Note that inline/character styles are intact |
| lm          | lockToRevisionsForMergeToDocument | null          | This parameter is needed when we want to merge one document into another but the document that is being used as output is locked to revision. We need to unprotect it on word adapter and therefore we need to know the password                                                                                                                          |
| p           | openingLinkPath                   | null          | When we have too long link (over 2048 characters) for word adapter to open as opening arguments we can send a shorter link to full opening link. Word adapter will in that case open that path and read from response and set it as opening arguments for application                                                                                     |
| er          | enableRevisions                   | false         | When we want to allow only revisions to word document without the locking the document to revision.                                                                                                                                                                                                                                                       |
| dn          | documentName                      | null          | If the document name is present, the downloaded file on system will have the name specified. If no name is provided system will generate unique name. Please note that use uniqueId if possible to enable multiple documents to be opened at the same time on same machine, otherwise only one instance of the document could be opened on same machine   |

Example

```html
<a href="factoryadapter:openDocs&d1=https%3A%2F%somedomain.factory.dev%2Ftest-documents%2FLabeling.doc&d2=https%3A%2F%somedomain.factory.dev%2Ftest-documents%2FLabeling2.doc&did=1233&c=http%3A%2F%somedomain.factory.dev.loc%2Fapi%2Fv1%2Ffactory-word-adapter%2Fload-config&a=edit&e=true">Open your word document</a>
```
 
 
> **Note**  
> All urls in query parameters needs to be url encoded. 

### Generating Opening Arguments

There is a class that helps build a link for opening arguments. Here is an example of how link can be generated

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

and here is a simple view
```twig
<a href="{{ link }}">Open document</a>
```

> **Note**  
> If you use generator to generate links, url encoding happens automatically for the url properties


### Overriding config path
If you wish you can override here a config path
```php
$link->setConfigPath($this->generateUrl('your-path-to-custom-config',
        ['documentId' => $asset->getId()], UrlGeneratorInterface::ABSOLUTE_URL) );
```

### Understanding compare action
When compare action needs to occur there are two documents
1. The document 
2. The document 

The action for compare tools should be   
`WordAdapterLinkGeneratorInterface::ACTION_COMPARE`

#### Generate compare link
```php
public function testWordAdapterAction(Request $request, WordAdapterLinkGeneratorFactoryService $wordAdapterLinkGeneratorFactoryService)
{
    $asset = Asset::getById(89);
    
    $link = $wordAdapterLinkGeneratorFactoryService->createWordAdapterLinkGenerator();
    $link->setAction(WordAdapterLinkGeneratorInterface::ACTION_COMPARE);
    $link->setDocument1($asset->getFullPath());
    $link->setDocument2($asset2->getFullPath());
    $link->setDocumentId($asset2->getId());
    
    $htmlString = $this->renderView('documents/test-word-adapter.html.twig', [
        'link' => $link,
    ]);
}
```

### Understanding merge action
When merge action needs to occur there are two documents
1. The document with the newest changes called **Source** document
2. The document that document1 will be merged onto called **Destination** document

So the destination document would be the document that will be updated after merge action. 

The merged assets are stored as pimcore properties of the Asset where the content was merged.
The properties are stored as
* Merged asset 1
* Merged asset 2
* Merged asset (n)

That means that the document2 if document1 was merged onto it would have properties 
* Merged asset 1 - As Asset type and relation to asset document1

#### Generate merge link
```php
public function testWordAdapterAction(Request $request, WordAdapterLinkGeneratorFactoryService $wordAdapterLinkGeneratorFactoryService)
{
    $asset = Asset::getById(89);
    $asset2 = Asset::getById(90);
    
    $link = $wordAdapterLinkGeneratorFactoryService->createWordAdapterLinkGenerator();
    $link->setAction(WordAdapterLinkGeneratorInterface::ACTION_MERGE);
    $link->setMergeFrom($asset->getFullPath());
    $link->setMergeTo($asset2->getFullPath());
    $link->setDocumentId($asset2->getId());
    $link->setMergeDocumentId($asset->getId());
    $link->setIsEditable(true);
    $link->setWebViewPath('https://example.com/some-view');
    
    $htmlString = $this->renderView('documents/test-word-adapter.html.twig', [
        'link' => $link,
    ]);
}
```

> **Note**  
> You can override how merge document is stored to the destination (document2). Look into documentation for events
> to find out how it can be achieved

### Overriding word adapter link generator
If you wish to override link generator class simply create a new class and implement 
`Factory\WordAdapterBundle\Contracts\WordAdapterLinkGeneratorInterface`

Than inside config.yml specify your class as generator
```yaml
factory_word_adapter:
    word_adapter_link_generator: some\my\new\classGenerator
```

After that the method createWordAdapterLinkGenerator on WordAdapterLinkGeneratorFactoryService will return a new instance 
of specified class in this case 'some\my\new\classGenerator'
