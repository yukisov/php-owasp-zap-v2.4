php-owasp-zap-v2.4
================

**Notice: Now in preparation!!**

PHP client API for OWASP ZAP 2.4

All API class files (except Zapv2.php) are generated automatically using `PhpAPIGenerator.java` offered by [yukisov/php-api-generator-for-owasp-zap](https://github.com/yukisov/php-api-generator-for-owasp-zap).

I uploaded this API to [Packagist](https://packagist.org/packages/yukisov/php-owasp-zap-v2).

##Getting Started

1. Add following lines to `composer.json` in your PHP project.

  ```
  {
    ...
    "require": {
      ...
      "zaproxy/php-owasp-zap-v2.4": ">=0.1.0",
      ...
    }
    ...
  }
  ```

2. `$ php composer.phar install/update`

##Usage
Example:

```php
<?php

require "vendor/autoload.php";

$api_key = "YOUR_API_KEY";
$target = "http://target.example.com/";

$zap = new Zap\Zapv2('tcp://localhost:8090');

$version = @$zap->core->version();
if (is_null($version)) {
  echo "PHP API error\n";
  exit();
} else {
  echo "version: ${version}\n";
}

echo "Spidering target ${target}\n";

try {
  // Give the Spider a chance to start
  // - Response JSON looks like {"scan":"1"}
  $resObj = $zap->spider->scan($target, 0, $api_key);
  $zap->expectOk($resObj);
  $scan+id = intval($resObj->scan);
} catch (Exception $e) {
  echo $e->getMessage() . PHP_EOL;
  exit(1);
}
$count = 0;
while (true) {
  if ($count > 10) exit();
  // Response JSON looks like {"status":"50"}
  $progress = intval($zap->spider->status($scan_id));
  printf("Spider progress %d\n", $progress);
  if ($progress >= 100) break;
  sleep(2);
  $count++;
}
echo "Spider completed\n";
// Give the passive scanner a chance to finish
sleep(5);

echo "Scanning target ${target}\n";

try {
  // Response JSON for error looks like {"code":"url_not_found", "message":"URL is not found"}
  $resObj = $zap->ascan->scan($target, '', '', '', '', '', $api_key);
  $zap->expectOk($resObj);
  $scan_id = intval($resObj->scan);  
} catch (Exception $e) {
  echo $e->getMessage() . PHP_EOL;
  exit(1);
}
$count = 0;
while (true) {
  if ($count > 10) exit();
  $progress = intval($zap->ascan->status($scan_id));
  printf("Scan progress %d\n", $progress);
  if ($progress >= 100) break;
  sleep(2);
  $count++;
}
echo "Scan completed\n";

// Report the results
echo "Hosts: " . implode(",", $zap->core->hosts()) . "\n";
$alerts = $zap->core->alerts($target, "", "");
echo "Alerts (" . count($alerts) . "):\n";
print_r($alerts);

```

## API
OWASP ZAP Wiki: [ApiGen_Index - zaproxy](https://code.google.com/p/zaproxy/wiki/ApiGen_Index)


##License
- Apache License, Version 2.0
