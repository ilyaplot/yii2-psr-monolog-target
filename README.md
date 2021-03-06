# Yii 2 PSR Log Target

Allows you to process logs using any PSR-3 compatible logger such as [Monolog](https://github.com/Seldaek/monolog).

<a href="https://travis-ci.org/ilyaplot/yii2-psr-monolog-target">
    <img src="https://travis-ci.org/ilyaplot/yii2-psr-monolog-target.svg" />
</a>

## Installation

```
composer require "ilyaplot/yii2-psr-monolog-target"
```

## Usage

In order to use `PsrTarget` you should configure your `log` application component like the following:  

```php
// $psrLogger should be an instance of PSR-3 compatible logger.
// As an example, we'll use Monolog to send log to Slack.
$psrLogger = new \Monolog\Logger('my_logger');
$psrLogger->pushHandler(new \Monolog\Handler\SlackHandler('slack_token', 'logs', null, true, null, \Monolog\Logger::DEBUG));

return [
    // ...
    'bootstrap' => ['log'],    
    // ...    
    'components' => [
        // ...        
        'log' => [
            'targets' => [
                [
                    'class' => 'samdark\log\PsrTarget',
                    'logger' => $psrLogger,
                    
                    // It is optional parameter. The message levels that this target is interested in.
                    // The parameter can be an array.
                    'levels' => ['info', yii\log\Logger::LEVEL_WARNING, Psr\Log\LogLevel::CRITICAL],
                    // It is optional parameter. Default value is false. If you use Yii log buffering, you see buffer write time, and not real timestamp.
                    // If you want write real time to logs, you can set addTimestampToContext as true and use timestamp from log event context.
                    'addTimestampToContext' => true,
                ],
                // ...
            ],
        ],
    ],
];
```

Standard usage:

```php
Yii::info('Info message');
Yii::error('Error message');
```

Usage with PSR logger levels:

```php
Yii::getLogger()->log('Critical message', Psr\Log\LogLevel::CRITICAL);
Yii::getLogger()->log('Alert message', Psr\Log\LogLevel::ALERT);
```

Usage with original timestamp from context in the log:

```php
// $psrLogger should be an instance of PSR-3 compatible logger.
// As an example, we'll use Monolog to send log to Slack.
$psrLogger = new \Monolog\Logger('my_logger');

$psrLogger->pushProcessor(function($record) {
    if (isset($record['context']['timestamp'])) {
        $dateTime = DateTime::createFromFormat('U.u', $record['context']['timestamp']);
        $timeZone = $record['datetime']->getTimezone();
        $dateTime->setTimezone($timeZone);
        $record['datetime'] = $dateTime;

        unset($record['context']['timestamp']);
    }

    return $record;
});
```

## Running tests

In order to run tests perform the following commands:

```
composer install
./vendor/bin/phpunit
```
