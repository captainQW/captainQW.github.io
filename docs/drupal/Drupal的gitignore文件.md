title: Drupal的.gitignore文件
date: 2014-04-16 15:41:19
categories: Drupal
tags: [gitignore]
---

```php
# Ignore configuration files that may contain sensitive information.
#sites/*/*settings*.php
settings.php
default.settings.php

*~
*.*~
*.lock
*.DS_Store
*.swp
*.out
*.swo

# Ignore paths that contain generated content.
cache/
files/
sites/*/files
sites/*/private
files/.htaccess

# Ignore default text files
.htaccess
robots.txt
/CHANGELOG.txt
/COPYRIGHT.txt
/INSTALL*.txt
/LICENSE.txt
/MAINTAINERS.txt
/UPGRADE.txt
/README.txt
sites/all/README.txt
sites/all/modules/README.txt
sites/all/themes/README.txt

# Ignore everything but the "sites" folder ( for non core developer )
web.config
authorize.php
cron.php
index.php
install.php
update.php
xmlrpc.php
/includes
/misc
/modules
/profiles
/scripts
/themes

```

把该文件放在/root/目录下。执行以下命令设置到全局选项里。
git config --global core.excludesfile /root/.gitignore
