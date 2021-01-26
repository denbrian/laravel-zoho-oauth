## Welcome to Laravel Zoho OAuth Documentation

![run-tests](https://github.com/njoguamos/laravel-zoho-oauth/workflows/run-tests/badge.svg)
[![Latest Version on Packagist](https://img.shields.io/packagist/v/njoguamos/laravel-zoho-oauth.svg?style=flat-square)](https://packagist.org/packages/njoguamos/laravel-zoho-oauth)
[![Total Downloads](https://img.shields.io/packagist/dt/njoguamos/laravel-zoho-oauth.svg?style=flat-square)](https://packagist.org/packages/njoguamos/laravel-zoho-oauth)

This is where your description should go. Try and limit it to a paragraph or two, and maybe throw in a mention of what PSRs you support to avoid any confusion with users and contributors.


## Prerequisites
To use this package, 
1. Ensure you have a [Zoho](https://zoho.com/) account, if not [create one now](https://accounts.zoho.com/register)
2. Have some basics on Zoho APIs. Here are the most popular Zoho apps

    | Zoho App          | API Documentation                                          |
    | ----------------- | ---------------------------------------------------------- |
    | Zoho Inventory    | [API Documentation](https://www.zoho.com/inventory/)       |
    | Zoho CRM          | [API Documentation](https://www.zoho.com/crm/developer/docs/api/v2/modules-api.html)       |
    | Zoho Campaigns    | [API Documentation](https://www.zoho.com/campaigns/help/developers/)       |
    | Zoho Books        | [API Documentation](https://www.zoho.com/books/api/v3/)       |
    | Zoho Projects     | [API Documentation](https://www.zoho.com/projects/help/rest-api/get-tickets-api.html/)       |

3. Ensure you have Zoho API Client ID, Zoho Client API Secret and Zoho authorization code. If not, [follow these instruction](/instructions)

## Why use this package
1. To automate generation of a permanent zoho api `refresh_token`
2. To provide a way of generating zoho api `access_token` which normally expires after a particular period

## Usage

### 01 Installation

Use the Composer package manager to install this package into your Laravel project:

```bash
composer require njoguamos/laravel-zoho-oauth
```


### 02 Update your `.env` variables

Add the following vairables and update accordingly. [Follow these instruction](/instructions)

```dotenv
# Zoho OAuth Credentials
BASE_OAUTH_URL=
ZOHO_CLIENT_ID=
ZOHO_CLIENT_SECRET=
ZOHO_CODE=
ZOHO_SCOPE=
```

### 03 Customization (Optional)

This package comes with a `migration` and `config` files. You may export migrationn using the following command,

```bash
php artisan vendor:publish --tag=laravel-zoho-oauth-migrations
```

You may export config using the following command,

```bash
php artisan vendor:publish --tag=laravel-zoho-oauth-config
```

If you publish, remeber to
-  maintain `config` variable names, 
- maintain table name and columens names

### 04 Migrate database

Run the migrate command in order to create the tables needed to store Zoho OAuth data:

```bash
php artisan migrate
```

### 05 Initilize the package

Run the init command ONCE after you install the package. This command add a new record of `refresh_token` and `access_token` to the 'laravel_zoho_oauth_table`

```bash
php artisan lzouth:init
```

This command may fail:
1. When you are not connected to the internet
2. When `ZOHO_CLIENT_ID` or `ZOHO_CLIENT_SECRET` or `ZOHO_CODE` is invalid

### 06 Generate Access Token From Refresh Token

To generate `access_token` anytime run the following command.

```bash
php artisan lzouth:refresh
```

This command will add a new `access_token` to the database and set it expiration to one hour.


### 07 Generate Access Token frequently

`access_token` expires after a particular period usually after `one hour`. After expiring, you have to use `refresh_token` to generate a new 
`access_token`. 

Schedule the refresh token command in the console kernel. The schedule time should be less than one hour.

```php
// app/Console/Kernel.php
protected function schedule(Schedule $schedule)
{
    ...
    $schedule->command('lzouth:refresh')->everyThirtyMinutes();
    ...
}
```

## Post Installation

### 01 Revoke and Access Token

To revoke a refresh token, load it from database and call `->revoke()`
```php

use Njoguamos\LaravelZohoOauth\LaravelZohoOauth;

$token = LaravelZohoOauth::first();
$token->revoke();
```

### 02 Delete expired access tokens

Generating `refresh_token` frequently populates the database. As a results it is recommended you schedule the following command

```php

// app/Console/Kernel.php
protected function schedule(Schedule $schedule)
{
    ...
    $schedule->command('lzouth:prune')->daily();
    ...
}

```
