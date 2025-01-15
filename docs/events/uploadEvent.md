# Upload events

- [On upload event and After upload event](#on-upload-event-and-after-upload-event)
- [On Merged document property save](#on-merged-document-property-save)

### On upload event and After upload event
This events are triggered when word adapter program save or automatic save function is called.
Once is called prior to save and other is called after save.


To subscribe on upload event create a new php class

```php
<?php

namespace App\EventSubscriber;

use App\Contract\Repository\DocumentRepositoryContract;
use Factory\WordAdapterBundle\Event\DocumentEvents;
use Symfony\Component\EventDispatcher\EventSubscriberInterface;
use Symfony\Component\EventDispatcher\GenericEvent;

class WordAdapterUploadSubscriber implements EventSubscriberInterface
{
    /**
     * @var DocumentRepositoryContract
     */
    private DocumentRepositoryContract $documentRepository;

    /**
     * @param DocumentRepositoryContract $documentRepository
     */
    public function __construct(DocumentRepositoryContract $documentRepository)
    {
        $this->documentRepository = $documentRepository;
    }

    /**
     * @return string[]
     */
    public static function getSubscribedEvents()
    {
        return [
            DocumentEvents::WORD_ADAPTER_ON_UPLOAD => 'onDocumentUpload',
            DocumentEvents::WORD_ADAPTER_AFTER_ASSET_SAVE => 'afterDocumentUpload'
        ];
    }

    public function onDocumentUpload(GenericEvent $event) {
        $request = $event->getArgument('request');
        $asset = $event->getArgument('asset');
        
        // do something with request and asset
        $event->setArgument('request', $request);
        $event->setArgument('asset', $asset);
    }
    
     public function afterDocumentUpload(GenericEvent $event) {
        $request = $event->getArgument('request');
        $asset = $event->getArgument('asset');
        echo "TLame";
    }
}

```

and register it as service in services.yaml
```yaml
services: 
    #...
    word_adapter_upload_subscriber:
        class: App\EventSubscriber\WordAdapterUploadSubscriber
        tags:
            - { name: kernel.event_subscriber }
```

And that's all. The event passes asset and request objects. Both of the objects can be altered to fit your needs. 

### On Merged document property save
When a merge save action happens, The word adapter will send additional file called `mergedAsset`

The merged assets are stored as pimcore properties of the Asset where the content was merged. 
The properties are stored as 
* Merged asset 1
* Merged asset 2
* Merged asset (n)

However if you wish to override this or store merged document somewhere else you can do this with following

Example:

```php
<?php

namespace App\EventSubscriber;

use Factory\WordAdapterBundle\Event\DocumentEvents;
use Pimcore\Model\Asset;use Symfony\Component\EventDispatcher\EventSubscriberInterface;
use Symfony\Component\EventDispatcher\GenericEvent;

class WordAdapterUploadSubscriber implements EventSubscriberInterface
{
    /**
     * @return string[]
     */
    public static function getSubscribedEvents()
    {
        return [
            DocumentEvents::WORD_ADAPTER_MERGED_DOCUMENT_PROPERTY_SAVE => 'onMergedDocumentPropertySave',
        ];
    }

    public function onMergedDocumentPropertySave(GenericEvent $event) {
        /** @var Asset $mergedAsset */
        $mergedAsset = $event->getArgument('mergedAsset');
        /** @var Asset $asset */
        $asset = $event->getArgument('asset');
        /** @var string $propertyKey */
        $propertyKey = $event->getArgument('$propertyKey');
        
        $prop = $asset->getProperty($propertyKey);
        // Do something with property
        
        // If you wish not to store merged document as property
        $asset->removeProperty($propertyKey);
           
        // Store it as a new Property with additional data
        $asset->setProperty('My Custom Name', $asset, 'And some additional data');
        
        // do something else with asset
        $event->setArgument('asset', $asset);
    }
}

```
