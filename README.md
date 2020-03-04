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
####Gravity Forms
```
     -Settings -> Confirmations
     -Redirecct
     -Redirect URL
     -Redirect Query String
     Salesforce
     -Content -> Forms
     -Post to form
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
