# php-password-generator

The PHP class PasswordGenerator serves as a password generator to create memorable passwords like the keychain of macOS ≤ 10.14 did.

## Getting Started

### Add via composer

Use composer to install the generator into your project:

```bash
composer require darkv/php-password-generator
```

### Direct Import

Copy the file *PasswordGenerator.php* into your project and include it in your own PHP file(s) with `include 'PasswordGenerator.php';`. Then create an instance either by using the predefined static methods for specific languages or customize it yourself by using the standard constructor.

```php
include 'PasswordGenerator.php';

// Import the namespace
use \Darkv\PhpPasswordGenerator\PasswordGenerator;

// create instance with an English word list
$gen = PasswordGenerator::EN();

// generate a password
echo $gen->generate();
```

### Prerequisites

This class works with PHP &gt;= 7.4 and needs a working internet connection.

### Password Syntax

The syntax of generated passwords can be defined by a pattern. That pattern consists of control characters that define the construction of the password string. The available control characters are:

* **i**  
  An integer between 1 and 999.
* **s**  
  A punctuation character (ASCII codes 33 to 47).
* **w**  
  A word from the wordlist.

If you don't provide your own pattern the default pattern **wisw** is used. Some examples of generated passwords with that default pattern are:

* Theyre778+Breakthrough
* Reforms13)Translated
* When249*Awards

## Word Lists

The class uses RSS feeds to build a word list from which random words are used for password generation. The class has some predefined configurations for the languages English and German but can be customized too:

```php
include 'PasswordGenerator.php';

use \Darkv\PhpPasswordGenerator\PasswordGenerator;

// create instance with English word list
$gen = PasswordGenerator::EN();

// create instance with German word list
$gen = PasswordGenerator::DE();

// create instance with custom parameters
$gen = new PasswordGenerator([
    'url'       => 'https://www.tagesschau.de/newsticker.rdf',
    'minLength' => 3,
    'maxLength' => 6,
]);
```

The params *minLength* and *maxLength* denote the allowed lengths of the words from the URL source to get into the word list. If a word list has been successfully built, that list is saved into the file `wordlist.json`. The next time you create an instance of PasswordGenerator and the URL source cannot be contacted or does not contain any usable words that cached list is loaded instead. If you reuse the very same instance the word list is also reused so no further HTTP requests are generated.

Optionally you can specify *wordCacheFile* to control the location and name of the cached wordlist.

To a custom instance you can pass an optional boolean parameter *fetch*. When false, the cached wordlist will be preferred but falls back to fetching if it could not be loaded.

```php
include 'PasswordGenerator.php';

use \Darkv\PhpPasswordGenerator\PasswordGenerator;

$gen = PasswordGenerator::EN();

// reuse word list without rebuilding
echo 'Password 1: ', $gen->generate();
echo 'Password 2: ', $gen->generate();
echo 'Password 3: ', $gen->generate();
```

When fetching an RSS source the normal behaviour is to create a new list, overwriting an existing cache file. You can opt-in to merge the new wordlist with an exisiting cache file instead, by setting the `appendWordlist` parameter to true. On the one hand this will result in bigger word lists, increasing entropy, on the other hand this may lead to very long lists. You can limit the number of items in the wordlist with the `limitWordlist` configuration parameter. When using this option and the wordlist is exceeding that limit, it is shuffled and then sliced to that number.

```php
include 'PasswordGenerator.php';

use \Darkv\PhpPasswordGenerator\PasswordGenerator;

// append NYTimes feed to the specified wordlist, limiting to 7000 items max
$gen = new PasswordGenerator([
    'wordCacheFile'  => 'mywords.json',
    'url'            => 'https://rss.nytimes.com/services/xml/rss/nyt/World.xml',
    'minLength'      => 3,
    'maxLength'      => 8,
    'appendWordlist' => true,
    'limitWordlist'  => 7000,
]);
```

### Caching

If you need to use that class in contexts where you do not have an internet connection you can prebuild a word list and copy the generated `wordlist.json` file into your project. When using the PasswordGenerator you can tell it to only use that cached list and skip the URL source request:

```php
include 'PasswordGenerator.php';

use \Darkv\PhpPasswordGenerator\PasswordGenerator;

$gen = PasswordGenerator::CACHED();

echo $gen->generate();
```

Location of the cache file can be passed as parameter to `CACHED`, by default wordlist.json in work dir will be used.

### URL Sources

As a source for word lists, this class uses a configurable RSS feed. The feed has to be in XML format and contain *description* tags from which the textual content is extracted.

### HTTP Redirects

If, for security reasons, you want to prevent PasswordGenerator to follow redirects when fetching the given URL, you can set the parameter *httpRedirects* to *0*. The default is *2* which follows a maximum of two redirects.

Example:
```php
include 'PasswordGenerator.php';

use \Darkv\PhpPasswordGenerator\PasswordGenerator;

// create instance with custom parameters
$gen = new PasswordGenerator([
    'url'           => 'https://www.some-url.com/source',
    'minLength'     => 3,
    'maxLength'     => 6,
    'httpRedirects' => 0,
]);
```

## Parameters

When creating an instance of PasswordGenerator you can provide the following parameters:

* **url**

  The URL to use to retrieve some document to extract words from.

* **minLength**

  The minimum number of characters a word must have to be included into the word list.

* **maxLength**

  The maximum number of characters a word may have to be included into the word list.

* **wordCacheFile**

  The name for the cache file. Defaults to ‘wordlist.json’.

* **httpRedirects**

  Controls if and how many HTTP redirects should be followed during URL access. Defaults to *2*. To prevent following redirects set its value to *0*.

* **appendWordlist**

  If true, the fetched wordlist will be appended to an existing cache file. If false, cache will be overwritten.

* **limitWordlist**

  Int value declaring the maximum number if words the wordlist may contain. Useful when using appendWordList.


## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details.
