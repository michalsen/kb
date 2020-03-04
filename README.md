
Wordpress
 - [Forced Trailing Slash](#wp-fts)
 - [Pardot/SF Integration](#wp-crm)
 - [Lando Multisite](#wp-lando-multi)

Drupal 7
- [Forced Trailing Slash](#d7-fts)
- [Pardot/SF Integration](#d7-pardot)
- [MS CRM Integration](#d7-mscrm)

Drupal 8
- [Forced Trailing Slash](#d8-fts)
- [Pardot/SF Integration](#d8-pardot)
- [No CI installation](#no-ci-install)
- [Redirect Domain](#d8-redirect-domain)



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

### <a name="d7-pardot"></a>Drupal 7 Pardot Integration


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
