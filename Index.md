# WordPress Security Enhancement

:warning::warning: Before you make any changes ALWAYS create a backup!

This repository collects a few methods, approaches to enhance/harden the default WordPress security setup

## wp-config.php

### Force Login page Only use SSL

In your 


## .htaccess changes


### Block scripts from running where they Shouldn't

Some folders in your WordPress site should not be directly accessed by users , `wp-admin/includes/` for example
preventing those executions by adding at this script to your .htaccess file

:warning: When inserting The code, make sure you are not in serting it between other start / end sections as those can be rewritten by the section creators, for example in a plug-in update. Separate this section of code from the other existing rules in the .htaccess

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

Using the .htaccess file you can deny access to wp-config.php which contain your database username and password among other things.


```
## Name of person adding, date added, some rule description

<files wp-config.php>
order allow,deny
deny from all
</files>

## END

## SOME OTHER SET OF RULES

```


## Move your wp-config.php file

WordPress allows you to move the wp-config.php one level above the root folder (outside the root), you can find a thorough discussion about this [here](https://wordpress.stackexchange.com/questions/58391/is-moving-wp-config-outside-the-web-root-really-beneficial)

Although that's a possibility, it also may expose your directory structure that may contain your site backups/additional important files
In this discussion my preference is to move the file from the root directory to another dedicated folder where within the public folder. I

Note that the only advantage of this method that I'm aware of is to maintain your credentials secure in the case the wp-config.php file was rewritten for some reason
making this file exposed




```
## Name of person adding, date added, some rule description

<files wp-config.php>
order allow,deny
deny from all
</files>

## END

## SOME OTHER SET OF RULES

```








* Information in this document is given AS IS, and is not intended for novice users.
document is intended for site administrators that have at least a basic understanding of what they're doing.
