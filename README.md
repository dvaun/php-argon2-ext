# PHP Argon2 Extension

[![TravisCI](https://img.shields.io/travis/charlesportwoodii/php-argon2-ext.svg?style=flat-square "TravisCI")](https://travis-ci.org/charlesportwoodii/php-argon2-ext)
[![License](https://img.shields.io/badge/license-BSD-orange.svg?style=flat-square "License")](https://github.com/charlesportwoodii//php-argon2-ext/blob/master/LICENSE.md)

This PHP7 extension provides a simplified interface to the [Argon2](https://github.com/P-H-C/phc-winner-argon2) algorithm, the winner of the [Password Hashing Competition](https://password-hashing.net/). Argon2 is considered the successor to bcrypt/scrypt/pbkdf methods of securely hasing passwords. This project is in no way associated with or endorsed by the PHC team.

> Note this is extension is only compatible with PHP7+. Support for lower versions of PHP will not be considered.

> Note this extension provides functionality identical to Libsodium's `sodium_crypto_pwhash` and `sodium_crypto_pwhash_str` functions.

## Building

```bash
# Clone the extension and the Argon2 Submodule
git clone --recursive https://github.com/charlesportwoodii/php-argon2-ext
cd php-argon2-ext

# Build the Argon2 library
cd ext/argon2
CFLAGS="-fPIC" make
make test

# Remove the argon2 shared library to force Argon2 to be compiled statically into the extension
rm libargon2.so
cd ../..

# Build the extension
phpize
./configure --with-argon2
make
```

### Installation

Once you have compiled the extension, you can install it via `make install`, adding the extension to your `php.ini` file or to a file in your loaded extensions directory, 

```bash
$ make install
# Load the extension to your php.ini/php conf.d
# echo "extension=argon2.so" > /path/to/php.ini
```

### Testing

Extension is tested through `make test`. You are strongly encouraged to run the tests to make sure everything was built correctly. A summary of the tests will be outlined

```bash
$ make test
```

If `make test` encounters an error, please provide a copy of the error report as a Github issue.

## Usage

### Constants

The following constants are exposed for determining which Argon2 algorithm you wish to use:

```php
HASH_ARGON2ID
HASH_ARGON2I
HASH_ARGON2D
```

The constant `HASH_ARGON2ID` can also be aliased by `HASH_ARGON2`.

> These constants are named to avoid conflicts with https://github.com/php/php-src/pull/1997, which would implement Argon2 in PHP Core.

### Hash Generation
```php
argon2_hash(string $string [, const $algorithm = HASH_ARGON2ID] [, array $options ] [, bool $raw = false ]);
\Argon2\Hash(string $string [, const $algorithm = HASH_ARGON2ID] [, array $options ] [, bool $raw = false ]);
```

Hashes can be generated by running `argon2_hash()` with the string you want to see hashed. Without any additional arguements, the hash generated will have the following options. These defaults are based upon the Argon2 specification as good minimums. You are encouraged to run `make bench` against the `ext/argon2` to determine what are good defaults for your system.

```bash
algorithm = HASH_ARGON2ID
options = [
    m_cost: 1<<16
    t_cost: 3
    threads: 1
]
```

> This function follows the same design principles of `password_hash()`, in that a salt will be generated on your behalf. This method does not provide you with a way to provide your own salt. The salt generated uses the native PHP function `php_password_make_salt`, which is the same method used to generate salts for `password_hash()`.

The `$algorithm` can be changed by passing in either `HASH_ARGON2I` or `HASH_ARGON2D`. If the algorithm is invalid, an `InvalidArguementException` will be raised.

This library allows you to specify an array of options to tune Argon2 to your system. The available options for the `$options` array, and their defaults are defined above. Argon2 has several tolerances in place for each of these values. If the value falls outside those tolerances, and `InvalidArguementException` will be raised.

In the event an error occurs with the argon2 library, or a salt cannot be securely generated, an `E_WARNING` will be raised, and this will return `false`.

If no errors occur, an argon2 encoded hash string will be returned.

> This function operates against version 1.3 of the Argon2 library and above.

#### Example Hash
```php
$argon2i$v=19$m=65536,t=3,p=1$aUEvQlU2NTRwcHhVS0hqMg$+5h0P5YlWCJDKyZknJ0sAyqQtZjhuP1Bkw/E2It4IcE
```

> If `$raw` is set to `true`, then this function will return binary output instead. This is useful for Key Derivation Functions (KDF).

### Validating Hashes
```php
argon2_verify(string $string, string $hash);
\Argon2\Verify(string $string, string $hash);
```

Hashes can be verified by running `argon2_verify()` with the string string and string hash generated by `argon2_hash`. This function will return either `true` or `false` depending upon if the hash is valid or not. If the `hash` provided isn't a valid argon2 hash, `false` will be returned, an an `E_WARNING` will be raised.

### Retrieving Hash Information
```php
argon2_get_info(string $hash);
\Argon2\Info(string $hash);
```

To retrieve information about an existing Argon2 hash, run `argon2_get_info()`. This function will return an array containing the algorithm name, and the options used for hash generation.

```php
array(2) {
  ["algorithm"]=>
  string(7) "argon2i"
  ["options"]=>
  array(3) {
    ["m_cost"]=>
    int(65536)
    ["t_cost"]=>
    int(3)
    ["threads"]=>
    int(1)
  }
}
```

## License

BSD-3-Clause. See [LICENSE.md](LICENSE.md) for more details.
