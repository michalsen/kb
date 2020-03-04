
## Wordpress
 - [Forced Trailing Slash](#wp-fts)
 - [Pardot/SF Integration](#wp-crm)
 - [Lando Multisite](#wp-lando-multi)
 - [Harden WP](#harden-wp)


## Drupal 7
- [Forced Trailing Slash](#d7-fts)
- [Pardot/SF Integration](#d7-pardot)
- [MS CRM Integration](#d7-mscrm)

## Drupal 8
- [Forced Trailing Slash](#d8-fts)
- [Pardot/SF Integration](#d8-pardot)
- [No CI installation](#no-ci-install)
- [Redirect Domain](#d8-redirect-domain)

## Speed
- [.htaccess speed rules](#htaccess-speed-rules)

## Resources
- [.htaccess snippets](#https://github.com/phanan/htaccess)
- [Pantheon No CI](#https://pantheon.io/docs/guides/drupal-8-composer-no-ci)
- [Private repos in Composer](#https://support.acquia.com/hc/en-us/articles/360004276834-How-to-access-Private-Repos-using-Composer)


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
#### .htaccess
```
RewriteBase /
 RewriteCond %{REQUEST_FILENAME} !-f
 RewriteCond %{REQUEST_URI} !(.*)/$
 RewriteCond %{REQUEST_URI} !/sites/default/files/(.*)$
 RewriteCond %{REQUEST_URI} !^/admin/(.*)$
 RewriteCond %{HTTP_HOST} ^www\.DOMAIN\.com$ [NC]
 RewriteRule ^(.*)$ https://www.DOMAIN.com/$1/ [L,R=301]
```

__________________

### <a name="wp-crm"></a>WP CRM Integration
#### Gravity Forms
```
     Settings -> Confirmations
     Redirecct
     Redirect URL
     Redirect Query String
     Salesforce
     Content -> Forms
     Post to form
```

__________________

### <a name="wp-lando-multi"></a>Lando WP Multisite
```
Getting a local instance up and running with git access to wpengine is detailed here:
https://wpengine.com/git/

once you have the site copied down, do the typical: lando init (wordpress) and connect via
git per the previous lines requirements.

the .lando.yml file for multiple sites running off one WP instance:
        name: snmulti
        recipe: wordpress
        config:
          webroot: .
        proxy:
          appserver:
            - site1.lndo.site
            - site2.lndo.site
```

__________________

### <a name="harden-wp"></a>Harden WP
.htaccess
```
##
# Disable Directory Browsing
##
Options All -Indexes
```
(this stops directory listing for directories that have no index)

Set user Display Name for administrators, so that scanners can not see who administrators are

Remove WP Version numbers from source code, that may reveal a vulnerability
```
// yourtheme/functions.php
/* Hide WP version strings from scripts and styles
 * @return {string} $src
 * @filter script_loader_src
 * @filter style_loader_src
 */
function enigma_2015_remove_wp_version_strings( $src ) {
    global $wp_version;
    parse_str(parse_url($src, PHP_URL_QUERY), $query);
    if ( !empty($query['ver']) && $query['ver'] === $wp_version ) {
        $src = remove_query_arg('ver', $src);
    }
    return $src;
}
add_filter( 'script_loader_src', 'enigma_2015_remove_wp_version_strings' );
add_filter( 'style_loader_src', 'enigma_2015_remove_wp_version_strings' );

/* Hide WP version strings from generator meta tag */
function wpmudev_remove_version() {
    return '';
}
add_filter('the_generator', 'wpmudev_remove_version');
```

wp-config.php Set debug messages to false
```
define('WP_DEBUG', false);
define('WP_DEBUG_LOG', false);
define('WP_DEBUG_DISPLAY', false);
```

ensure salt keys are defined, and provide [fresh keys](#http://api.wordpress.org/secret-key/1.1/salt/)

change database prefix if value is default (will also need to change table prefix's in the database)

#### plugins
- wp cerber
  - harden login page
- updraftplus
  - backup/restore tool
- stop user enumeration
  - removes the ability of a scanner to gather the user ids
- activity log
  - creates activity logs of users and events
  - can create alerts to custom events
- wp ban
  - bans bad actors
- bbq
  - block bad queries from automated hacking scanners
- Blackhole for Bad Bots
  - automated blocking for the above bbq
- Exploit Scanner
  - scan files periodically looking for exploited code
```
curl -O https://downloads.wordpress.org/plugin/wp-cerber.5.0.zip
curl -O https://downloads.wordpress.org/plugin/updraftplus.1.13.5.zip
curl -O https://downloads.wordpress.org/plugin/stop-user-enumeration.1.3.10.zip
curl -O https://downloads.wordpress.org/plugin/aryo-activity-log.2.3.4.zip
curl -O https://downloads.wordpress.org/plugin/wp-ban.1.69.zip
curl -O https://downloads.wordpress.org/plugin/block-bad-queries.20170730.zip
curl -O https://downloads.wordpress.org/plugin/blackhole-bad-bots.1.7.1.zip
curl -O https://downloads.wordpress.org/plugin/exploit-scanner.1.5.2.zip
curl -O https://downloads.wordpress.org/plugin/my-wp-health-check.1.4.2.zip
```

Stop Hotlinking
```
##
# Stop Hot Linking
##
RewriteCond %{HTTP_REFERER} !^$
RewriteCond %{HTTP_REFERER} !^https?://(.+\.)?<DOMAIN>.com [NC]
RewriteRule \.(jpe?g|png|gif|bmp|pdf)$ - [NC,F,L]
```

Installation Page
```
chmod 400 wp-admin/install.php
```

Stop automated spam
```
##
# Stop Automated Spam
##
<IfModule mod_rewrite.c>
  RewriteEngine On
  RewriteCond %{REQUEST_METHOD} POST
  RewriteCond %{HTTP_USER_AGENT} ^$ [OR]
  RewriteCond %{HTTP_REFERER} !<DOMAIN> [NC]  
  RewriteCond %{REQUEST_URI} /wp-comments-post\.php [NC]
  RewriteRule .* - [F,L]
</IfModule>
```

6G Firewall
```
# 6G FIREWALL/BLACKLIST
# @ https://perishablepress.com/6g/

# 6G:[QUERY STRINGS]
<IfModule mod_rewrite.c>
	RewriteEngine On
	RewriteCond %{QUERY_STRING} (eval\() [NC,OR]
	RewriteCond %{QUERY_STRING} (127\.0\.0\.1) [NC,OR]
	RewriteCond %{QUERY_STRING} ([a-z0-9]{2000,}) [NC,OR]
	RewriteCond %{QUERY_STRING} (javascript:)(.*)(;) [NC,OR]
	RewriteCond %{QUERY_STRING} (base64_encode)(.*)(\() [NC,OR]
	RewriteCond %{QUERY_STRING} (GLOBALS|REQUEST)(=|\[|%) [NC,OR]
	RewriteCond %{QUERY_STRING} (<|%3C)(.*)script(.*)(>|%3) [NC,OR]
	RewriteCond %{QUERY_STRING} (\\|\.\.\.|\.\./|~|`|<|>|\|) [NC,OR]
	RewriteCond %{QUERY_STRING} (boot\.ini|etc/passwd|self/environ) [NC,OR]
	RewriteCond %{QUERY_STRING} (thumbs?(_editor|open)?|tim(thumb)?)\.php [NC,OR]
	RewriteCond %{QUERY_STRING} (\'|\")(.*)(drop|insert|md5|select|union) [NC]
	RewriteRule .* - [F]
</IfModule>

# 6G:[REQUEST METHOD]
<IfModule mod_rewrite.c>
	RewriteCond %{REQUEST_METHOD} ^(connect|debug|move|put|trace|track) [NC]
	RewriteRule .* - [F]
</IfModule>

# 6G:[REFERRERS]
<IfModule mod_rewrite.c>
	RewriteCond %{HTTP_REFERER} ([a-z0-9]{2000,}) [NC,OR]
	RewriteCond %{HTTP_REFERER} (semalt.com|todaperfeita) [NC]
	RewriteRule .* - [F]
</IfModule>

# 6G:[REQUEST STRINGS]
<IfModule mod_alias.c>
	RedirectMatch 403 (?i)([a-z0-9]{2000,})
	RedirectMatch 403 (?i)(https?|ftp|php):/
	RedirectMatch 403 (?i)(base64_encode)(.*)(\()
	RedirectMatch 403 (?i)(=\\\'|=\\%27|/\\\'/?)\.
	RedirectMatch 403 (?i)/(\$(\&)?|\*|\"|\.|,|&|&amp;?)/?$
	RedirectMatch 403 (?i)(\{0\}|\(/\(|\.\.\.|\+\+\+|\\\"\\\")
	RedirectMatch 403 (?i)(~|`|<|>|:|;|,|%|\\|\s|\{|\}|\[|\]|\|)
	RedirectMatch 403 (?i)/(=|\$&|_mm|cgi-|etc/passwd|muieblack)
	RedirectMatch 403 (?i)(&pws=0|_vti_|\(null\)|\{\$itemURL\}|echo(.*)kae|etc/passwd|eval\(|self/environ)
	RedirectMatch 403 (?i)\.(aspx?|bash|bak?|cfg|cgi|dll|exe|git|hg|ini|jsp|log|mdb|out|sql|svn|swp|tar|rar|rdf)$
	RedirectMatch 403 (?i)/(^$|(wp-)?config|mobiquo|phpinfo|shell|sqlpatch|thumb|thumb_editor|thumbopen|timthumb|webshell)\.php
</IfModule>

# 6G:[USER AGENTS]
<IfModule mod_setenvif.c>
	SetEnvIfNoCase User-Agent ([a-z0-9]{2000,}) bad_bot
	SetEnvIfNoCase User-Agent (archive.org|binlar|casper|checkpriv|choppy|clshttp|cmsworld|diavol|dotbot|extract|feedfinder|flicky|g00g1e|harvest|heritrix|httrack|kmccrew|loader|miner|nikto|nutch|planetwork|postrank|purebot|pycurl|python|seekerspider|siclab|skygrid|sqlmap|sucker|turnit|vikspider|winhttp|xxxyy|youda|zmeu|zune) bad_bot

	# Apache < 2.3
	<IfModule !mod_authz_core.c>
		Order Allow,Deny
		Allow from all
		Deny from env=bad_bot
	</IfModule>

	# Apache >= 2.3
	<IfModule mod_authz_core.c>
		<RequireAll>
			Require all Granted
			Require not env bad_bot
		</RequireAll>
	</IfModule>
</IfModule>

# 6G:[BAD IPS]
<Limit GET HEAD OPTIONS POST PUT>
	Order Allow,Deny
	Allow from All
	# uncomment/edit/repeat next line to block IPs
	# Deny from 123.456.789
</Limit>
```

Controlling Proxy Access
```
RewriteEngine on
RewriteCond %{HTTP:VIA}                 !^$ [OR]
RewriteCond %{HTTP:FORWARDED}           !^$ [OR]
RewriteCond %{HTTP:USERAGENT_VIA}       !^$ [OR]
RewriteCond %{HTTP:X_FORWARDED_FOR}     !^$ [OR]
RewriteCond %{HTTP:PROXY_CONNECTION}    !^$ [OR]
RewriteCond %{HTTP:XPROXY_CONNECTION}   !^$ [OR]
RewriteCond %{HTTP:HTTP_PC_REMOTE_ADDR} !^$ [OR]
RewriteCond %{HTTP:HTTP_CLIENT_IP}      !^$
RewriteRule ^(.*)$ - [F]
```



__________________


### <a name="d7-fts"></a>Drupal 7 Forced Trailing Slash

#### .htaccess
```
  RewriteBase /
  RewriteCond %{REQUEST_FILENAME} !-f
  RewriteCond %{REQUEST_URI} !(.*)/$
  RewriteCond %{REQUEST_URI} !/sites/default/files/(.*)$
  RewriteCond %{REQUEST_URI} !^/admin/(.*)$
  RewriteCond %{HTTP_HOST} ^www\.DOMAIN\.com$ [NC]
  RewriteRule ^(.*)$ https://www.DOMAIN.com/$1/ [L,R=301]
```
#### php
```
if (!preg_match('/^\/$/', $_SERVER['REQUEST_URI']) && php_sapi_name() != 'cli') {
  if(strrev($_SERVER['REQUEST_URI'])[0] !== '/'){
    if (!preg_match('#/sites/default/files#', $_SERVER['REQUEST_URI']) &&
        !preg_match('#/admin/#', $_SERVER['REQUEST_URI']) &&
        !preg_match('#/user/#', $_SERVER['REQUEST_URI']) &&
        !preg_match('#/imce/#', $_SERVER['REQUEST_URI']) &&
        !preg_match('#/login/#', $_SERVER['REQUEST_URI']) &&
        $_REQUEST['sncid'] < 1) {
          header("HTTP/1.1 301 Moved Permanently");
          header("location: https://" . $_SERVER['HTTP_HOST'] . strtolower($_SERVER['REQUEST_URI'] . '/'));
          exit();
    }
  }
}
```

__________________

### <a name="d7-pardot"></a>Drupal 7 Pardot Integration


__________________

### <a name="d7-mscrm"></a>Drupal 7 MS CRM Integration
```php
{
    "require": {
        "alexacrm/php-crm-toolkit": "20171115"
    }
}



<?php
/**
 * @file
 * MS CRM Integration
 */

require_once 'vendor/autoload.php';

  use AlexaCRM\CRMToolkit\Client as OrganizationService;
  use AlexaCRM\CRMToolkit\Settings;


/**
 * Implements hook_form_alter()
 *
 */
function ms_crm_form_alter(&$form, &$form_state) {
  if ($form['#form_id'] == 'webform_client_form_ID') {
    $form['#submit'][] = 'mscrm_form_submit';
  }
  return $form;
}

/**
 * Form submit()
 *
 */
function mscrm_form_submit($form, &$form_state) {

  if (preg_match('/General/', $form_state['input']['submitted']['reason_for_contact']) ||
       preg_match('/Sales/', $form_state['input']['submitted']['reason_for_contact'])) {

  try {
    $options = getcrmoptions();

      $serviceSettings = new Settings( $options );
      $service = new OrganizationService( $serviceSettings );

      $lead = $service->entity( 'lead' );

      $lead->firstname                = $form_state['values']['submitted'][1];
      $lead->lastname                 = $form_state['values']['submitted'][6];
      $lead->emailaddress1            = $form_state['values']['submitted'][2];

      $lead->mobilephone              = $form_state['values']['submitted'][8];
      $lead->companyname              = $form_state['values']['submitted'][4];
      $lead->poet_OriginationComments = $form_state['values']['submitted'][5];
      $lead->subject                  = 'Hidden Field?';
      $lead->leadsourcecode           = 'website';
      $leadId = $lead->create();

      watchdog('ms_crm', 'success: ' . $leadId);

  } catch (Exception $e) {

      watchdog('ms_crm', 'error');

    }
  }
}


function mscrm_form_submit_1691($form, &$form_state) {

  try {
    $options = getcrmoptions();

     // dpm($form_state['values']);

      $serviceSettings = new Settings( $options );
      $service = new OrganizationService( $serviceSettings );

      $lead = $service->entity( 'lead' );

      $lead->firstname                = $form_state['values']['submitted'][1];
      $lead->lastname                 = $form_state['values']['submitted'][2];
      $lead->emailaddress1            = $form_state['values']['submitted'][3];
      $lead->companyname              = $form_state['values']['submitted'][4];
      $lead->leadsourcecode           = 'website';
      $leadId = $lead->create();

      // watchdog('ms_crm', 'success: ' . $leadId);

  } catch (Exception $e) {

      watchdog('ms_crm', 'error', $e);

    }

}

function getcrmoptions() {
  $options = [
      'serverUrl' => 'URL',
      'username' => 'USERNAME',
      'password' => 'PASSWORD,
      'authMode' => 'OnlineFederation',
  ];
  return $options;
}
```

### <a name="d8-fts"></a>Drupal 8 Forced Trailing Slash


### <a name="d8-pardot"></a>Drupal 8 Pardot Integration
```
webform->settings->Emails/Handlers->Add Handler
completed URL is Pardot url
Submission Data is field mapping
```
### <a name="no-ci-install"></a>No CI D8 Install
```

terminus site:create SITENAME 'SITENAME' empty


git clone git@github.com:pantheon-systems/example-drops-8-composer.git SITENAME
git clone https://github.com/pantheon-systems/example-drops-8-composer.git SITENAME

cd ericd8

terminus connection:info SITENAME --field=git_url =
git remote set-url origin SSH.GIT

Delete the following files/directories:
scripts/github
scripts/gitlab
.circleci
.ci
tests
bitbucket-pipelines.yml
build-providers.json
.gitlab-ci.yml

Modify composer.json:
  Remove all dependencies in the require-dev section.
  Update the scripts section to remove the lint, code-sniff, and unit-test lines.

Remove the following section from pantheon.yml:
  workflow

composer update
composer install

To make it compatible with this manual method, you need to edit the .gitignore file and remove everything above the :: cut :: section.

terminus connection:set SITENAME.dev git
git add . ; git commit -m'first' ; git push --force
terminus connection:set SITENAME.dev sftp
terminus drush SITENAME.dev -- site-install -y
terminus drush SITENAME.dev uli


.lando.yml
name: d8
recipe: pantheon
config:
  webroot: web
  framework: drupal
  site: SITENAME
  id: SITEID

lando start



composer clear-cache


composer.json
    "repositories": [
        {
            "type": "composer",
            "url": "https://packages.drupal.org/8"
        }
    ],
    "require": {
        "php": ">=7.0.8",
        "composer/installers": "^1.0.20",
        "cweagans/composer-patches": "^1.0",
        "drupal-composer/drupal-scaffold": "^2.0.1",
        "drupal/address": "^1.7",
        "drupal/config_direct_save": "^1.0",
        "drupal/config_installer": "^1.0",
        "drupal/console": "^1",
        "drupal/core": "^8.6.15",
        "drush-ops/behat-drush-endpoint": "^9.3",
        "drush/drush": "~8.3",
        "pantheon-systems/quicksilver-pushback": "^2",
        "rvtraveller/qs-composer-installer": "^1.1",
        "webflo/drupal-core-strict": "^8.6.15",
        "zaporylie/composer-drupal-optimizations": "^1.0"
    },


composer.json
    "minimum-stability": "dev",

#
composer require drupal/admin_toolbar
composer require drupal/webform
composer require drupal/pathauto
composer require drupal/redirect
composer require drupal/smtp
composer require drupal/image_effects
```

### <a name="d8-redirect-domain"></a>Drupal 8 Redirect Domain
```
if (isset($_ENV['PANTHEON_ENVIRONMENT']) && ($_ENV['PANTHEON_ENVIRONMENT'] === 'live') && (php_sapi_name() != "cli")) {
  if (preg_match('/OLDDOMAIN/', $_SERVER['HTTP_HOST'])) {
    header('HTTP/1.0 301 Moved Permanently');
    header('Location: https://www.NEWDOMAIN.com/PAGE');

    # Name transaction "redirect" in New Relic for improved reporting (optional)
    if (extension_loaded('newrelic')) {
      newrelic_name_transaction("redirect");
    }
    exit();
  }
}
```


__________________

### <a name="htaccess-speed-rules"></a>.htaccess Speed Rules
```
<ifModule mod_gzip.c>
mod_gzip_on Yes
mod_gzip_dechunk Yes
mod_gzip_item_include file .(html?|txt|css|js|php|pl)$
mod_gzip_item_include handler ^cgi-script$
mod_gzip_item_include mime ^text/.*
mod_gzip_item_include mime ^application/x-javascript.*
mod_gzip_item_exclude mime ^image/.*
mod_gzip_item_exclude rspheader ^Content-Encoding:.*gzip.*
</ifModule>

AddOutputFilterByType DEFLATE text/plain
AddOutputFilterByType DEFLATE text/html
AddOutputFilterByType DEFLATE text/xml
AddOutputFilterByType DEFLATE text/css
AddOutputFilterByType DEFLATE application/xml
AddOutputFilterByType DEFLATE application/xhtml+xml
AddOutputFilterByType DEFLATE application/rss+xml
AddOutputFilterByType DEFLATE application/javascript
AddOutputFilterByType DEFLATE application/x-javascript

# BEGIN Expire headers
<ifModule mod_expires.c>
        ExpiresActive On
        ExpiresDefault "access plus 2 days"
        ExpiresByType image/x-icon "access plus 2592000 seconds"
        ExpiresByType image/jpeg "access plus 2592000 seconds"
        ExpiresByType image/png "access plus 2592000 seconds"
        ExpiresByType image/gif "access plus 2592000 seconds"
        ExpiresByType application/x-shockwave-flash "access plus 2592000 seconds"
        ExpiresByType text/css "access plus 604800 seconds"
        ExpiresByType text/javascript "access plus 216000 seconds"
        ExpiresByType application/javascript "access plus 216000 seconds"
        ExpiresByType application/x-javascript "access plus 216000 seconds"
        ExpiresByType text/html "access plus 600 seconds"
        ExpiresByType application/xhtml+xml "access plus 600 seconds"
</ifModule>
# END Expire headers


# BEGIN Caching
<ifModule mod_headers.c>
  <filesMatch "\\.(ico|pdf|flv|jpg|jpeg|png|gif|swf|svg)$">
    Header set Cache-Control "max-age=2592000, public"
  </filesMatch>
  <filesMatch "\\.(css|js)$">
    Header set Cache-Control "max-age=604800, public"
  </filesMatch>
  <filesMatch "\\.(xml|txt)$">
    Header set Cache-Control "max-age=216000, public, must-revalidate"
  </filesMatch>
  <filesMatch "\\.(html|htm|php)$">
    Header set Cache-Control "max-age=3600, public, must-revalidate"
  </filesMatch>
</ifModule>
# END Caching
```
