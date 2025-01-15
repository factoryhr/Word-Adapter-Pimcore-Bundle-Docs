<a name="installation"></a>
## Installation
### 1. Add repository to composer.json
### 2. Add bundle to required
```json
"require": {
    "factory/word-adapter-bundle": "dev-main",
}
```

### 3. Run
``
composer require factory/word-adapter-bundle
``

### 4. Add this to bundles.php
```
\Factory\WordAdapterBundle\FactoryWordAdapterBundle::class => ['all' => true],
```

### 5. Add this to your doctrine.yaml
```
doctrine:
    orm:
        mappings:
            FactoryWordAdapterBundle:
                type: attribute
                prefix: Factory\WordAdapterBundle\Entity
                dir: Entity
                is_bundle: true
```

### 6. Run migrations
``
bin/console doctrine:migrations:migrate
``
