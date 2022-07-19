Apache web server access and error log parser
=============================================

[![Latest Stable Version](https://poser.pugx.org/mvar/apache2-log-parser/v/stable)](https://packagist.org/packages/mvar/apache2-log-parser)
[![Build Status](https://travis-ci.org/mvar/apache2-log-parser.svg?branch=master)](https://travis-ci.org/mvar/apache2-log-parser)
[![Code Coverage](https://scrutinizer-ci.com/g/mvar/apache2-log-parser/badges/coverage.png?s=c4f63101c2d2877a2a0623b3a75ee18b67636b97)](https://scrutinizer-ci.com/g/mvar/apache2-log-parser/)
[![Scrutinizer Quality Score](https://scrutinizer-ci.com/g/mvar/apache2-log-parser/badges/quality-score.png?s=2eb88f010261c2bc70e969cb98107a57342b3543)](https://scrutinizer-ci.com/g/mvar/apache2-log-parser/)

Installation
---

This library can be found on [Packagist](https://packagist.org/packages/mvar/apache2-log-parser).
The recommended way to install this is through [Composer](https://getcomposer.org):

```bash
composer require mvar/apache2-log-parser:dev-master
```

Features
--------

 - Apache2 log lines parsing
     - Access log
     - Error log (currently, for Apache 2.2 and older)
 - Log files iterator
 - Low memory footprint even with huge files

Usage
-----

### Parsing single Apache access log line

```php
<?php

require __DIR__ . '/vendor/autoload.php';

use MVar\Apache2LogParser\AccessLogParser;

// Access log format from your Apache configuration
// It can be any of predefined `AccessLogParser::FORMAT_*` constants or custom string
$parser = new AccessLogParser('%h %l %u %t "%r" %>s %O "%{Referer}i" "%{User-Agent}i"');

// String which you want to parse
$line = '66.249.78.230 - - [29/Dec/2013:16:07:58 +0200] "GET /my-page/ HTTP/1.1" 200 2490 "-" ' .
    '"Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)"';

var_export($parser->parseLine($line));
```

The above example will output:

```php
array (
  'remote_host' => '66.249.78.230',
  'identity' => '-',
  'remote_user' => '-',
  'time' => '29/Dec/2013:16:07:58 +0200',
  'request_line' => 'GET /my-page/ HTTP/1.1',
  'response_code' => '200',
  'bytes_sent' => '2490',
  'request' =>
  array (
    'method' => 'GET',
    'path' => '/my-page/',
    'protocol' => 'HTTP/1.1',
  ),
  'request_headers' =>
  array (
    'Referer' => '-',
    'User-Agent' => 'Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)',
  ),
)
```

### Iterate through Apache log file

Log iterator reads log file line by line. This means that it is possible to
parse huge files with low memory usage.

Let's say we have Apache log file `access.log` with following content:

```
192.168.25.1 - - [25/Jun/2012:14:26:05 -0700] "GET /favicon.ico HTTP/1.1" 404 498
192.168.25.1 - - [25/Jun/2012:14:26:05 -0700] "GET /icons/blank.gif HTTP/1.1" 200 438
```

To parse whole log file line by line it needs only to create new iterator with
file name and parser arguments:

```php
<?php

require __DIR__ . '/vendor/autoload.php';

use MVar\Apache2LogParser\AccessLogParser;
use MVar\LogParser\LogIterator;

$parser = new AccessLogParser(AccessLogParser::FORMAT_COMMON);

foreach (new LogIterator('access.log', $parser) as $line => $data) {
    printf("%s %s\n", $data['request']['method'], $data['request']['path']);
}
```

The example above will output:

```
GET /favicon.ico
GET /icons/blank.gif
```

> To get more information about iterator please visit [mvar/log-iterator][1] documentation.

### Date and Time Formatting

By default date and time is returned as is, raw string. You can change this
behaviour in two ways. First, set custom format string and formatted date
string will be returned. Second, set time format to `true` and you will get
`\DateTime` object.
 
```php       
$parser = new AccessLogParser(AccessLogParser::FORMAT_COMMON);

// Set custom date and time format accepted by date()
$parser->setTimeFormat('Y-m-d H:i:s');

// Set TRUE and you will get \DateTime object
$parser->setTimeFormat(true);
```

TODO for future releases
------------------------

 - Modifiers support
 - Custom time format support

Feel free to make a [Pull Request](https://github.com/mvar/apache2-log-parser/pulls) :)

[1]: https://github.com/mvar/log-parser
