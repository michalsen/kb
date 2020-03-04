3/4/2020

Wordpress
 - [Forced Trailing Slash](#wp-fts)
 - [Pardot/SF Integration](#wp-crm)

Drupal 7
- [Forced Trailing Slash](#d7-fts)
- [Pardot/SF Integration](#d7-pardot)
- [MS CRM Integration](#d7-mscrm)

Drupal 8
- [Forced Trailing Slash](#d8-fts)
- [Pardot/SF Integration](#d8-pardot)



### <a name="wp-fts"></a>WP Forced Trailing Slash

#### PHP
```php
  if (!preg_match('/^\/$/', $_SERVER['REQUEST_URI']) && php_sapi_name() != 'cli') {
     if(strrev($_SERVER['REQUEST_URI'])[0] !== '/'){
       if (!preg_match('#wp-admin#', $_SERVER['REQUEST_URI']) &&
           !preg_match('#wp-content#', $_SERVER['REQUEST_URI']) &&
           !preg_match('#wp-json#', $_SERVER['REQUEST_URI']) &&
           !preg_match('#rest_route#', $_SERVER['REQUEST_URI']) &&
           !preg_match('#sitemap#', $_SERVER['REQUEST_URI']) &&
           !preg_match('#adminlogin#', $_SERVER['REQUEST_URI'])) {
             header("HTTP/1.1 301 Moved Permanently");
             header("location: https://" . $_SERVER['HTTP_HOST'] . strtolower($_SERVER['REQUEST_URI'] . '/'));
             exit();
       }
     }
   }
```

### <a name="wp-crm"></a>WP CRM Integration


### <a name="d7-fts"></a>Drupal 7 Forced Trailing Slash

#### .htaccess
  RewriteBase /
  RewriteCond %{REQUEST_FILENAME} !-f
  RewriteCond %{REQUEST_URI} !(.*)/$
  RewriteCond %{REQUEST_URI} !/sites/default/files/(.*)$
  RewriteCond %{REQUEST_URI} !^/infographic(.*)$
  RewriteCond %{REQUEST_URI} !^/admin/(.*)$
  RewriteCond %{HTTP_HOST} ^www\.DOMAIN\.com$ [NC]
  RewriteRule ^(.*)$ https://www.DOMAIN.com/$1/ [L,R=301]



### <a name="d7-pardot"></a>Drupal 7 Pardot Integration


### <a name="d7-mscrm"></a>Drupal 7 MS CRM Integration


### <a name="d8-fts"></a>Drupal 8 Forced Trailing Slash


### <a name="d8-pardot"></a>Drupal 8 Pardot Integration
