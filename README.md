# mofh-vp-api-client

A PHP client for automating common tasks in **VistaPanel**, the hosting control panel used by some free-hosting providers. The library communicates with VistaPanel's web interface using authenticated HTTP requests and HTML parsing; it is **not an official, documented VistaPanel REST API**.

The library provides a `VistapanelApi` class for account sessions, MySQL databases, domains, redirects, SSL certificates, DNS records, PHP settings, and hosting statistics.

## Requirements

- PHP with the **cURL** and **DOM** extensions enabled.
- Network access to your VistaPanel control-panel URL.
- A valid VistaPanel account or session cookie.

## Installation

Download `api-client.php` into your project and include it:

```php
require_once __DIR__ . '/api-client.php';

$client = new VistapanelApi();
```

## Quick start

```php
<?php
require_once __DIR__ . '/api-client.php';

$client = new VistapanelApi();
$client->setCpanelUrl('https://cpanel.example.com');

try {
    $client->login('your-username', 'your-password');

    foreach ($client->listDatabases() as $database) {
        echo $database . PHP_EOL;
    }
} finally {
    // Call logout only if a session was successfully established.
    // For example, call $client->logout() after successful use.
}
```

`login()` uses the `PaperLantern` theme by default. `setSession($session)` can be used when you already have a valid `PHPSESSID` session value. Setting a session does **not** set the account username, so username-dependent operations may require a normal login.

## API reference

The public PHP class is **`VistapanelApi`**, even though the repository is named `mofh-vp-api-client`. Method names below match the library's implementation and are case-sensitive as documented.

### Connection and session

| Method | Description |
| --- | --- |
| `setCpanelUrl($url)` | Set the HTTPS base URL for VistaPanel. |
| `login($username, $password, $theme = 'PaperLantern')` | Log in using account credentials. |
| `setSession($session)` | Use an existing VistaPanel session ID. |
| `approveNotification()` | Submit the notification approval action. |
| `disapproveNotification()` | Submit the notification disapproval action. |
| `logout()` | Sign out and clear the client's session state. |

### Databases

| Method | Description |
| --- | --- |
| `createDatabase($dbname)` | Create a MySQL database. |
| `listDatabases()` | Get database names as an array. |
| `deleteDatabase($database)` | Delete a database by its unprefixed name. |
| `getPhpmyadminLink($database)` | Retrieve the phpMyAdmin link associated with a database. |

### Domains and redirects

| Method | Description |
| --- | --- |
| `listDomains($option = 'all')` | List domains; options: `all`, `addon`, `sub`, or `parked`. |
| `createRedirect($domainname, $target)` | Create a redirect from a domain to a target URL. |
| `deleteRedirect($domainname)` | Remove a domain redirect. |
| `showRedirect($domainname)` | Read a domain's configured redirect target. |

### SSL certificates

| Method | Description |
| --- | --- |
| `getPrivateKey($domainname)` | Retrieve the domain's configured SSL private key. **Treat the result as sensitive.** |
| `getCertificate($domainname)` | Retrieve the installed SSL certificate. |
| `uploadPrivateKey($domainname, $key, $csr = '')` | Upload an SSL private key and optional CSR. |
| `uploadCertificate($domainname, $cert)` | Upload an SSL certificate. |
| `deleteCertificate($domainname)` | Delete the domain's configured certificate. |

### DNS records

| Method | Description |
| --- | --- |
| `getCNAMErecords()` | List CNAME records. |
| `createCNAMErecord($source, $domain, $dest)` | Create a CNAME record. |
| `deleteCNAMErecord($source)` | Remove a CNAME record. |
| `getMXrecords()` | List MX records. |
| `createMXrecord($domain, $server, $priority)` | Create an MX record. |
| `deleteMXrecord($domain, $srv, $priority)` | Remove an MX record. |
| `getSPFrecords()` | List SPF records. |
| `createSPFrecord($domain, $data)` | Create an SPF record. |
| `deleteSPFrecord($domain, $data)` | Remove an SPF record. |
| `createDNSRecord($recordType, $domain, $data, $destination = null, $priority = null)` | Generic DNS creation method supporting `MX`, `SPF`, and `CNAME`. **Prefer the type-specific methods**, which use the endpoint-specific form fields. |

### Account, PHP configuration, and hosting tools

| Method | Description |
| --- | --- |
| `getSoftaculousLink()` | Retrieve a Softaculous installer URL, when available. |
| `showErrorPage($domainname, $option = '400')` | Read a custom error-page value; supported codes: `400`, `401`, `403`, `404`, `503`. |
| `updateErrorPages($domainname, $v400 = '', $v401 = '', $v403 = '', $v404 = '', $v503 = '')` | Update domain error-page settings. |
| `showPHPConfig($domainname, $option = 'display_errors')` | Read `display_errors`, `mbstring_http_input`, or `date_timezone`. |
| `setPHPConfig($domainname, $displayerrors = '', $mbstringinput = '', $timezone = '')` | Change PHP configuration for a domain. |
| `getUserStats($option = '')` | Get all account statistics or one statistic by its VistaPanel label. |
| `changeEmail($newEmail, $confirmEmail)` | Submit an account email change. |
| `addPasswordProtectionToFolder($domainName, $folderName, $password)` | Configure folder password protection. |

## Examples

### Create and list databases

```php
$client->createDatabase('example');
print_r($client->listDatabases());
```

### Create a redirect

```php
$client->createRedirect('example.com', 'https://www.example.com');
```

### Add an MX record

```php
$client->createMXrecord('example.com', 'mail.example.com', 10);
```

### Get account statistics

```php
$stats = $client->getUserStats();
print_r($stats);

$plan = $client->getUserStats('Plan');
```

### Upload an SSL certificate

```php
$client->uploadPrivateKey('example.com', $privateKey, $csr);
$client->uploadCertificate('example.com', $certificate);
```

## Notes and limitations

- These operations automate the VistaPanel website, not a stable public REST contract. Changes to VistaPanel pages, form fields, or sessions can break some methods.
- The available features depend on the hosting provider and account permissions.
- Some methods return `true` once the HTTP request completes; this is **not always proof** that VistaPanel applied the requested change. Confirm important changes afterward.
- The client throws exceptions in some authentication, validation, HTTP, and panel-error cases. Handle failures in application code.
- Never commit credentials, session cookies, SSL private keys, or other secrets to version control.
- Use HTTPS with a properly configured CA certificate bundle. SSL verification is enabled in the refactored client.
- Test destructive operations such as database, certificate, or DNS deletion against a non-production account before relying on them.

## Credits

Based on the [VistaPanel PHP API](https://github.com/oddmario/vistapanel-php-api), originally developed by @oddmario, subsequently maintained by @GenerateApps, and revived by @RegenerateApps.

This project is not affiliated with or endorsed by VistaPanel or hosting providers.
