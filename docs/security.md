- [Session](#session)
- [Secure document save](#secure-document-save)


# Session
When Factory Word Adapter is initiated, it is designed to accept specific cookies as part of its launch parameters. This feature is crucial for identifying the user currently editing the document. By passing these cookies, Factory Word Adapter ensures that each request to the server carries the necessary authentication, maintaining a secure link between the user and their actions within the document. 

Every HTTP request made by Factory Word Adapter for saving, autosaving, or other document management functions includes these cookies. This not only secures the request but also ensures that each action is accurately attributed to the authenticated user, enhancing both security and accountability.


# Secure document save

Since this bundle depends on pimcore, and pimcore uses firewall to manage security and voter to grant permissions to 
store some resources, Factory word adapter bundle provides a way to implement custom voter on document save. By default
word adapter doesn't know which implementation or firewall for symfony auth is being used, That's why this functionality
depends on the developer who is implementing this bundle. 

To use custom voter set inside config.yml use_voter parameter
Note use_voter is by default false
```yml
factory_word_adapter:
    use_voter: true
```

Than in your project create a voter class. Word adapter will call a voter on wordAdapterSaveDocument as attribute. 
To learn more about voters read [symfony documentation](https://symfony.com/doc/current/security/voters.html)

```php
<?php

namespace App\Security;

use LogicException;
use Pimcore\Bundle\AdminBundle\Security\User\User;
use Pimcore\Model\Asset;
use Symfony\Component\Security\Core\Authentication\Token\TokenInterface;
use Symfony\Component\Security\Core\Authorization\Voter\Voter;

class DocumentVoter extends Voter
{
    const SAVE_DOCUMENT = 'wordAdapterSaveDocument';

    /**
     *
     * Determine if this voter should vote on the attribute/subject combination
     * If true than voteOnAttribute will be called
     *
     * @param string $attribute
     * @param $subject
     * @return bool
     */
    protected function supports(string $attribute, $subject)
    {
        // if the attribute isn't one we support, return false
        if ($attribute != self::SAVE_DOCUMENT) {
            return false;
        }

        return true;
    }

    /**
     * Vote logic. Calls the functions to check if the current user
     * has permissions to save document
     *
     * @param string $attribute specified action to check up on if the user can do it
     * @param Asset $subject Object on which we are checking the permissions
     * @param TokenInterface $token object that contains User
     * @return bool true if the user has permission or false if don't
     */
    protected function voteOnAttribute(string $attribute, $subject, TokenInterface $token)
    {
        $user = $token->getUser();

        if (!$user instanceof User) {
            // the user must be logged in; if not, deny access
            return false;
        }

        if ($attribute === self::SAVE_DOCUMENT) {
            return $this->canSave($subject, $user);
        }

        throw new LogicException('This code should not be reached!');
    }

    /**
     * Check if the current user save document
     *
     * @param Asset $document
     * @param User $user
     * @return bool
     */
    private function canSave(Asset $document, User $user): bool
    {
        // Do some logic and return false if the user doesn't have permission to store document
        return true;
    }
}

```

Register voter in services
```yaml
services: 
    ...
    document_voter:
        class: App\Security\DocumentVoter
        tags:
            - { name: security.voter }
```

Now symfony and word adapter will automatically call voter to determine if the currently logged in user has a permission
to edit document. 
