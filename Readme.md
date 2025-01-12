# WordPress Security Enhancements 
(Written by Yaron Elharar)

:warning::warning: ALWAYS have a current backup of your WordPress site! :warning::warning:

This repository collects a few methods, approaches to enhance/harden the default WordPress security setup

## wp-config.php changes to enhance security

### Force login page to only use SSL

Setting this configuration will help protect against 
> this should make it much harder for a malicious person to steal your cookies and/or authentication headers and use them to impersonate you and gain access to wp-admin. It also obfuscates the ability to sniff your content, which could be important for legal blogs which may have drafts of documents that need strict protection.
<sub>Quote from the WordPress site</sub>

More information, and instructions for more complex setups can be found [here](https://WordPress.org/support/article/administration-over-ssl/)

```php
// Force SSL Logins, Admin Access
define('FORCE_SSL_ADMIN', true);
```

### Remove the ability to edit code directtly from the WordPress backend

You're probably well aware of the internal WordPress editor that allows you to edit the code of any file, once you successfully logged in to the backend
this is also one of the first things an attacker will use to change the sites code if he gained access to your website.

This code snippet will disabled code editing from the WordPress backend both for users and attackers that gained unauthorized access to the backend, or if one off your users password was compromised.

```php
// Disable WordPress backend editor
define('DISALLOW_FILE_EDIT', true);
```


## .htaccess changes to enhance security


### Block files users shouldn't have access to

Some folders in your WordPress site should not be directly accessed by users , `wp-admin/includes/` for example
Prevent access to videos files by adding this script to your .htaccess file

:warning: When inserting The code, make sure you are not in serting it between other start / end sections as those can be rewritten by the section creators, for example in a plug-in update. Separate this section of code from the other existing rules in the .htaccess

* Note: if you're using the lightspeed server (OpenLiteSpeed) you will need to restart the server for changes to take effect

I usually write it like this

```
## Name of person adding, date added, some rule description

<IfModule mod_rewrite.c>
RewriteEngine On
RewriteBase /
RewriteRule ^wp-admin/includes/ - [F,L]
RewriteRule !^wp-includes/ - [S=3]
RewriteRule ^wp-includes/[^/]+\.php$ - [F,L]
RewriteRule ^wp-includes/js/tinymce/langs/.+\.php - [F,L]
RewriteRule ^wp-includes/theme-compat/ - [F,L]
</IfModule>

## END

## SOME OTHER SET OF RULES

```

### Denying access to wp-config.php

Using the .htaccess file, you can deny access to wp-config.php, which contains your database username and password, among other critical information. In addition, you can create a few rules that allow only your server to access particular files that are commonly probed for, denying everybody else.


```php
## Name of person adding, date added, some rule description

<files wp-config.php>
order allow,deny
deny from all
</files>

<Files xmlrpc.php>
    Order Allow,Deny
    Allow from 127.0.0.1
    Allow from {your servers IP}
    Deny from all
</Files>

<Files .htaccess>
    Order Allow,Deny
    Allow from 127.0.0.1
    Allow from {your servers IP}
    Deny from all
</Files>

<Files xmlrpc.php>
    Order Allow,Deny
    Allow from 127.0.0.1
    Allow from {your servers IP}
    Deny from all
</Files>

<Files php.ini>
    Order Allow,Deny
    Allow from 127.0.0.1
    Allow from {your servers IP}
    Deny from all
</Files>

## END

## SOME OTHER SET OF RULES

```
If you're using the OpenLiteSpeed server, you'll have to use mod_rewrite since OpenLiteSpeed open source doesn't allow for <Files ...></files>

```
# using mod_rewrite since OpenLiteSpeed open source doesn't allow for <Files...
<IfModule mod_rewrite.c>
    RewriteEngine On
    RewriteBase /
    
    # blocks access to wp-config
    RewriteCond %{REQUEST_URI} ^/wp-config.php$
    RewriteRule ^wp-config\.php$ - [F,L]
    
    # Protect specific files with IP restrictions
    RewriteCond %{REMOTE_ADDR} !^(127.0.0.1|165.140.157.97)$
    RewriteRule ^xmlrpc\.php$ - [F,L]
    
    RewriteCond %{REMOTE_ADDR} !^(127.0.0.1|165.140.157.97)$
    RewriteRule ^\.htaccess$ - [F,L]
    
    RewriteCond %{REMOTE_ADDR} !^(127.0.0.1|165.140.157.97)$
    RewriteRule ^php\.ini$ - [F,L]
    
    # Existing WordPress core protection rules
    RewriteRule ^wp-admin/includes/ - [F,L]
    RewriteRule !^wp-includes/ - [S=3]
    RewriteRule ^wp-includes/[^/]+\.php$ - [F,L]
    RewriteRule ^wp-includes/js/tinymce/langs/.+\.php - [F,L]
    RewriteRule ^wp-includes/theme-compat/ - [F,L]
</IfModule>

## END

## SOME OTHER SET OF RULES

```



## Move your wp-config.php file

WordPress allows you to move the wp-config.php one level above the root folder (outside the root), you can find a thorough discussion about this [here](https://wordpress.stackexchange.com/questions/58391/is-moving-wp-config-outside-the-web-root-really-beneficial)

Although that's a possibility, it also may expose your directory structure that may contain your site backups/additional important files
In this discussion my preference is to move the file from the root directory to another dedicated folder where within the public folder.

Note that the only advantage of this method that I'm aware of is to maintain your credentials secure in the case the wp-config.php file was rewritten for some reason
making this file exposed

- Create a new directory in the root of your site name it whatever you want
- Copy the existing wp-config.php file to that directory
- Replace the content of the original file with a direction to the new file


```php

/** Absolute path to the WordPress directory. */
if ( !defined('ABSPATH') )
    define('ABSPATH', dirname(__FILE__) . '/');

/** Location of your WordPress configuration.
https://wordpress.stackexchange.com/questions/58391/is-moving-wp-config-outside-the-web-root-really-beneficial
Only makes sure that if by some mistake the config file is overwritten it will not be exposed
 */ 

require_once(ABSPATH . '{Folder name you have chosen}/wp-config.php');

```

Make sure that the copied file permission are at least 600 or 640, 400 or 440 are even better if your server allows it.

**Additional note:** if you're using `wp-cli` command line this will not work. (Feedback/Further research required on how to make this work with wp-cli If it's even possible)


## Securing WordPress using CloudFlare (Free Plan)

Beyond the speed gaineds innate to using a CDN service, CloudFlare can also mitigate some attacks before they even reach your server in the free plan.
The "I'm under attack" Is the first go to method when you're site is under attack, unfortunately depending on the attack type bots these days can pass through this obstacle quite easily.

### protect your admin folder using CloudFlare pages

- Go to CloudFlare page rules
- Create a new rule to protect your WordPress Admin section

In the URL field insert `*your-site-url.com/wp-admin*`

Then set the rules to:
```
Security Level: High
Cache Level: Bypass
Disable Apps
Disable Performance
```


### Using Cloudflare Workers To Block Automated Vulnerability Scanners 

One of the issues in WordPress sites is the automated scans for vulnerabilities, and those are twice the problem. One major issue is that it might find a vulnerability in the scan, and the second is that the scan itself is usually a burst of requests to the server. Depending on your infrastructure, you might not notice it, or it could bring your site to a complete halt, causing a mini denial of service (DoS), as thousands of requests are sent to the origin server in a matter of seconds, those requests are usually non-existent PHP files, hoping to land on a vulnerability.

If you're behind Cloudflare proxy and hoping that automatic bot mitigation might halt this, as of the writing of this text, it will not consider this as a threat trigger. At least not on the free plan.
But you can somewhat mitigate this by using Cloudflare workers, this worker script monitors incoming requests for .php files that result in 404 Not Found errors. If an IP address makes more than three such requests within an hour, the script blocks that IP for 24 hours by returning a 404 response to any subsequent requests from that IP. This helps mitigate malicious bots scanning by stopping the IP from reaching the origin server after the first three requests. Since the vulnerability scan consists of thousands of files, this ensures that nothing beyond the first three failures reaches your server.

* Replace "YoreKVDatabasename" with an actual KV database name and bind it to the worker in the workers settings.

```
addEventListener('fetch', event => {
  event.respondWith(handleRequest(event.request))
})

async function handleRequest(request) {
  const ip = request.headers.get('CF-Connecting-IP')
  const url = new URL(request.url)

  // Check if the IP is blocked
  const isBlocked = await YoreKVDatabasename.get(`blocked_${ip}`)
  if (isBlocked) {
    return new Response(null, { status: 404 })
  }

  // Prevent infinite recursion
  if (request.headers.get('X-Worker-Processed')) {
    return fetch(request)
  }

  // Clone the original request but only add the X-Worker-Processed header
  const newHeaders = new Headers(request.headers)
  newHeaders.set('X-Worker-Processed', 'true')

  // Create modified request while preserving all original properties
  const modifiedRequest = new Request(request, {
    headers: newHeaders,
    body: request.body,
    method: request.method,
    credentials: request.credentials,
    cache: request.cache,
    redirect: request.redirect,
    integrity: request.integrity,
    keepalive: request.keepalive,
    signal: request.signal,
    mode: request.mode
  })

  let response
  try {
    response = await fetch(modifiedRequest)
  } catch (err) {
    console.error(`Fetch error: ${err.message}`)
    return new Response('Error fetching origin', { status: 500 })
  }

   // Handle PHP 404 requests
  const pathname = url.pathname.replace(/\/+$/, '')
  if (response.status === 404 && pathname.endsWith('.php')) {
    try {
      let count = await YoreKVDatabasename.get(`count_${ip}`)
      count = count ? parseInt(count) : 0
      count++

      await YoreKVDatabasename.put(`count_${ip}`, count.toString(), { expirationTtl: 3600 })

      console.log(`IP ${ip} has ${count} 404 .php requests`)

      if (count >= 3) {
        await YoreKVDatabasename.put(`blocked_${ip}`, 'true', { expirationTtl: 86400 })
        console.log(`IP ${ip} blocked after exceeding threshold`)
      }
    } catch (error) {
      console.error(`Error processing request: ${error.message}`)
    }
  }

  return response
}


```


### Use the firewall to add a layer of protection to common files targeted by attackers

Lookiing at your log files you will Notice that some files are really popular with attackers the `wp-login.php`,`xmlrpc.php`,`admin-ajax.php` Etc. , Let's make it a bit harder to target them using the CloudFlare firewall

- Go to the firewall settings
- Create a rule
- Name it whatever you want
- Click "edit expression"
- Insert the rule set below
- Click "Use expression builder"
- Under choose an action select "Managed challenge"
- If you're using plug-ins that require IP whitelisting (Squirrly SEO for example) exclude them here or in their own rule 

```
(http.request.uri.path contains "/wp-login.php") or (http.request.uri.path contains "/xmlrpc.php") or (http.request.uri.path contains "/wp-admin/" and http.request.uri.path ne "/wp-admin/admin-ajax.php" and http.request.uri.path ne "/wp-admin/theme-editor.php")
```




## What about WordPress security Plug-ins?

Definitely, security plug-ins should be a part of your toolkit especially if you suspect you already been hacked, some of these organs can check if WordPress files were changed in some way, in addition to other mitigation strategys such as:

- Number of login attempt allowed
- Retrieve password attempts allow
- Two factor authentication
- And many other benefits

Most of the plug-ins start with a free version, it's a good place to start with.

## Summary

I hope this list of security enhancements will help you increase the security of your website, if you know of another method of increasing security use a pull request to expand, correct, or update the existing data.


## Final notes on the information

The information in this document is given "**AS IS**", use your own judgment if you decide to apply any of the steps described in this document. this information is not intended for novice users, if you do not know what you're doing you might end up doing more harm than good. As making the changes above may cause issues on your particular set up, always backup, try on the staging server, push to live server after thoroughly testing.
