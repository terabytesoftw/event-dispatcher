<p align="center">
    <a href="https://github.com/yiisoft" target="_blank">
        <img src="https://avatars0.githubusercontent.com/u/993323" height="100px">
    </a>
    <h1 align="center">Yii Event Dispatcher</h1>
    <br>
</p>

[PSR-14](http://www.php-fig.org/psr/psr-14/) compatible event dispatcher provides an ability to dispatch events and listen
to events dispatched.

[![Latest Stable Version](https://poser.pugx.org/yiisoft/event-dispatcher/v/stable.png)](https://packagist.org/packages/yiisoft/event-dispatcher)
[![Total Downloads](https://poser.pugx.org/yiisoft/event-dispatcher/downloads.png)](https://packagist.org/packages/yiisoft/event-dispatcher)
[![Build Status](https://github.com/yiisoft/event-dispatcher/workflows/build/badge.svg)](https://github.com/yiisoft/event-dispatcher/actions?query=workflow%3Abuild)
[![Code Coverage](https://scrutinizer-ci.com/g/yiisoft/event-dispatcher/badges/coverage.png)](https://scrutinizer-ci.com/g/yiisoft/event-dispatcher/)
[![Scrutinizer Code Quality](https://scrutinizer-ci.com/g/yiisoft/event-dispatcher/badges/quality-score.png?b=master)](https://scrutinizer-ci.com/g/yiisoft/event-dispatcher/?branch=master)
[![Mutation testing badge](https://img.shields.io/endpoint?style=flat&url=https://badge-api.stryker-mutator.io/github.com/yiisoft/event-dispatcher/master)](https://dashboard.stryker-mutator.io/reports/github.com/yiisoft/event-dispatcher/master)
[![static analysis](https://github.com/yiisoft/event-dispatcher/workflows/static%20analysis/badge.svg)](https://github.com/yiisoft/event-dispatcher/actions?query=workflow%3A%22static+analysis%22)


### Features

- [PSR-14](http://www.php-fig.org/psr/psr-14/) compatible.
- Simple and lightweight.
- Encourages designing event hierarchy.
- Can combine multiple event listener providers.

### General usage

The library consists of two parts: event dispatcher and event listener provider. Provider's job is to register listeners
for a certain event type. Dispatcher's job is to take an event, get listeners for it from a provider and call them sequentially.

```php
// add some listeners
$listeners = (new \Yiisoft\EventDispatcher\Provider\ListenerCollection())
    ->add(function (AfterDocumentProcessed $event) {
        $document = $event->getDocument();
        // do something with document
    });

$provider = new Yiisoft\EventDispatcher\Provider\Provider($listeners);
$dispatcher = new Yiisoft\EventDispatcher\Dispatcher\Dispatcher($provider);
```

The event dispatching may look like:

```php
class DocumentProcessor
{
    public function process(Document $document)
    {
        // process the document
        $dispatcher->dispatch(new AfterDocumentProcessed($document));
    }
}
```

### Stoppable events

Event could be made stoppable by implementing `Psr\EventDispatcher\StoppableEventInterface`:

```php
class BusyEvent implements Psr\EventDispatcher\StoppableEventInterface
{
    // ...

    public function isPropagationStopped(): bool
    {
        return true;
    }
}
```

This way we can ensure that only first event listener will be able to handle the event. Another option is
to allow stopping propagation in one of the listeners by providing corresponding event method.

### Events hierarchy

Events do not have any name or wildcard matching on purpose. Event class names and class/interface hierarchy
and composition could be used to achieve great flexibility:

```php
interface DocumentEvent
{
}

class BeforeDocumentProcessed implements DocumentEvent
{
}

class AfterDocumentProcessed implements DocumentEvent
{
}
```

With the interface above listening to all document-related events could be done as:


```php
$listeners->add(function (DocumentEvent $event) {
    // log events here
});
```

### Combining multiple listener providers

In case you want to combine multiple listener providers, you can use `CompositeProvider`:

```php
$compositeProvider = new Yiisoft\EventDispatcher\Provider\CompositeProvider();
$provider = new Yiisoft\EventDispatcher\Provider\Provider();
$compositeProvider->add($provider);
$compositeProvider->add(new class implements ListenerProviderInterface {
    public function getListenersForEvent(object $event): iterable
    {
        yield function ($event) {
            // handle 
        };
    }
});

$dispatcher = new Yiisoft\EventDispatcher\Dispatcher\Dispatcher($compositeProvider);
```

### Register listeners with concrete event names

You may use a more simple listener provider, which allows you to specify which event they can provide.

It can be useful in some specific cases, for instance if one of your listeners does not need the event 
object passed as a parameter (can happen if the listener only needs to run at a specific stage during 
runtime, but does not need event data).

In that case, it is advised to use the aggregate (see above) if you need features from both providers included
in this library.

```php
$listeners = (new \Yiisoft\EventDispatcher\Provider\ListenerCollection())
    ->add(static function () {
    // this function does not need an event object as argument
}, SomeEvent::class);
```

### Unit testing

The package is tested with [PHPUnit](https://phpunit.de/). To run tests:

```php
./vendor/bin/phpunit
```

### Mutation testing

The package tests are checked with [Infection](https://infection.github.io/) mutation framework. To run it:

```php
./vendor/bin/infection
```

### Static analysis

The code is statically analyzed with [Phan](https://github.com/phan/phan/wiki). To run static analysis:

```php
./vendor/bin/phan
```

### Credits

- Larry Garfield (@crell) for initial implementation of deriving callable parameter type.
