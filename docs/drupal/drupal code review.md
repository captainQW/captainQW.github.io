title: Drupal code review
date: 2016-05-24 11:34:10
categories: Drupal8
tags: [code_review]
---

如何你的模块已经在线上，比如github或者Drupal的git仓库，可以使用在线的[pareview.sh](http://pareview.sh/)工具。

###1、安装composer

```bash
$ curl -sS https://getcomposer.org/installer | php
$ mv composer.phar /usr/local/bin/composer
```

###2、安装Coder (reference: https://www.drupal.org/node/1419988#coder-composer)

```bash
$ composer global require drupal/coder
$ sudo ln -s ~/.composer/vendor/bin/phpcs /usr/local/bin
$ sudo ln -s ~/.composer/vendor/bin/phpcbf /usr/local/bin
$ phpcs --config-set installed_paths ~/.composer/vendor/drupal/coder/coder_sniffer
```

###3、安装codespell (reference: https://github.com/lucasdemarchi/codespell)

```bash
$ sudo apt-get help2man -y
$ cd /usr/local/src
$ git clone https://github.com/lucasdemarchi/codespell.git
$ cd codespell && sudo make install
$ sudo cp codespell /usr/local/bin
```

注：MAC下安装codespell
```bash
sudo python setup.py install
```

###4、安装DrupalSecure (reference: https://www.drupal.org/sandbox/coltrane/1921926)

```bash
$ sudo apt-get update && sudo apt-get install php-pear
$ sudo pear install PHP_CodeSniffer
$ cd $(pear config-get php_dir)/PHP/CodeSniffer/Standards/
$ git clone git://git.drupal.org/sandbox/coltrane/1921926.git secure_cs
$ sudo ln -sv ./secure_cs/DrupalSecure ./DrupalSecure
```

注：MAC下安装php-pear
```bash
wget http://pear.php.net/go-pear.phar
php go-pear.phar
```

###5、安装ESLint

```bash 
$ sudo npm install -g eslint
```

在Ubuntu precise （12.04）, 你有可能会遇到以下错误：

```bash 
npm http GET https://registry.npmjs.org/
npm ERR! Error: failed to fetch from registry: 
This is because the version supplied by Ubuntu 12.04 is no longer supported, updating node (and npm with it) resolved the issue.
```

首先, 卸载nodejs版本

```bash 
sudo apt-get purge nodejs npm
```

启用nodesource's仓库并进行安装:

```bash 
curl -sL https://deb.nodesource.com/setup | sudo bash -
sudo apt-get install -y nodejs
$ sudo apt-get install nodejs
$ eslint --init # run for the first time
```

###6、安装pareviewsh

```bash 
$ cd /usr/local/src
$ sudo wget http://ftp.drupal.org/files/projects/pareviewsh-7.x-1.7.tar.gz
$ sudo tar xzf pareviewsh-7.x-1.7.tar.gz
$ sudo ln -s /usr/local/src/pareviewsh/pareview.sh /usr/local/bin/pareviewsh
```

查看pareview是否已经安装成功

```bash 
$ pareviewsh /path/to/module or /path/to/git-repos.git
```
phpcs自动修复代码标准

```bash 
phpcbf --standard=Drupal --extensions=php,module,inc,install,test,profile,theme,js,css,info,txt,md /file/to/drupal/example_module
```

在~/.bash_profile中添加别名

```bash
// Check Drupal coding standards
alias drupalcs="phpcs --standard=Drupal --extensions='php,module,inc,install,test,profile,theme,js,css,info,txt,md'"

// Check Drupal best practices
alias drupalpractice="phpcs --standard=DrupalPractice --extensions='php,module,inc,install,test,profile,theme,js,css,info,txt,md'"

// Automatically fix coding standards
alias drupalcbf="phpcbf --standard=Drupal --extensions='php,module,inc,install,test,profile,theme,js,css,info,txt,md'"
```

参照文章：
1、https://www.drupal.org/node/1587138
2、https://www.drupal.org/node/1419988
3、https://www.drupal.org/node/172169
4、https://www.drupal.org/project/pareviewsh
5、https://www.drupal.org/project/coder