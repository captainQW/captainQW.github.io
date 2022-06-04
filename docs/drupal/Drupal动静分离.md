title: Drupal动静分离
date: 2015-01-16 14:46:05
categories: Drupal
tags: [动静分离]
---

```php

/**
 * Implements hook_file_url_alter().
 */
function modulename_file_url_alter(&$uri) {
  if ($resource_server = variable_get('static_server', '')) {
    $resource_extensions = array('css', 'js', 'gif', 'jpg', 'jpeg', 'png');
    // Most CDNs don't support private file transfers without a lot of hassle,
    // so don't support this in the common case.
    $schemes = array('public');

    $scheme = file_uri_scheme($uri);

    // Only serve shipped files and public created files from the static_server.
    if (!$scheme || in_array($scheme, $schemes)) {
      // Shipped files.
      if (!$scheme) {
        $path = $uri;
      }
      // Public created files.
      else {
        $wrapper = file_stream_wrapper_get_instance_by_scheme($scheme);
        $path = $wrapper->getDirectoryPath() . '/' . file_uri_target($uri);
      }

      // Clean up Windows paths.
      $path = str_replace('\\', '/', $path);

      // Serve files with one of the resource extensions from resource_server 
      $pathinfo = pathinfo($path);
      if (isset($pathinfo['extension']) && in_array($pathinfo['extension'], $resource_extensions)) {
        $uri = $resource_server . '/' . $path;
      }
    }
  }
}


```