# WordPress Security Enhancement

This repository collects a few methods, approaches to enhance/harden the default WordPress security setup

## wp-config.php

### Force Login page Only use SSL

In your 


## .htaccess changes


### Block scripts from a running where they Shouldn't

Some folders in your WordPress Site should not be Executing scripts,  `wp-admin/includes/` For example
preventing those executions by adding at this script to your .htaccess file

```
<IfModule mod_rewrite.c>
RewriteEngine On
RewriteBase /
RewriteRule ^wp-admin/includes/ - [F,L]
RewriteRule !^wp-includes/ - [S=3]
RewriteRule ^wp-includes/[^/]+\.php$ - [F,L]
RewriteRule ^wp-includes/js/tinymce/langs/.+\.php - [F,L]
RewriteRule ^wp-includes/theme-compat/ - [F,L]
</IfModule>
```
