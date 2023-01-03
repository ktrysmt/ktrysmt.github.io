---
layout: post
title: "rvmのgemsetでrubyとrailsを管理する"
date: 2014-01-09 05:52:07 +0000
comments: true
categories: Ruby
published: true
---

## rvm で複数バージョンのrubyとrailsをセット管理
まずはrvmを更新。

```
[root@localhost ~]# rvm get stable
```

今回必要なバージョンは以下。

``` bash
ruby 1.8.7 (2009-12-24 patchlevel 248) [x86_64-linux], MBARI 0x6770, Ruby Enterprise Edition 2010.01
Rails 2.3.5
```


## 1.8.7をインストールしgemset（受け皿）を作成


``` bash
[root@localhost ~]# rvm install 1.8.7
Warning! PATH is not properly set up, '/usr/local/rvm/gems/ruby-1.9.3-p374/bin' is not at first place,
         usually this is caused by shell initialization files - check them for 'PATH=...' entries,
         it might also help to re-add RVM to your dotfiles: 'rvm get stable --auto-dotfiles',
         to fix temporarily in this shell session run: 'rvm use ruby-1.9.3-p374'.
Searching for binary rubies, this might take some time.
No binary rubies available for: centos/6/x86_64/ruby-1.8.7-p374.
It is not possible to build movable binaries for rubies 1.8-1.9.2, but you can do it for your system only.
Continuing with compilation. Please read 'rvm help mount' to get more information on binary rubies.
Checking requirements for centos.
Installing requirements for centos.
Updating system.
Installing required packages: patch, patch, readline-devel, libyaml-devel, libffi-devel, libtool, bison/
........
Requirements installation successful.
Installing Ruby from source to: /usr/local/rvm/rubies/ruby-1.8.7-p374, this may take a while depending on your cpu(s)...
ruby-1.8.7-p374 - #downloading ruby-1.8.7-p374, this may take a while depending on your connection...
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 4150k  100 4150k    0     0  10.4M      0 --:--:-- --:--:-- --:--:-- 15.4M
ruby-1.8.7-p374 - #extracting ruby-1.8.7-p374 to /usr/local/rvm/src/ruby-1.8.7-p374.
ruby-1.8.7-p374 - #applying patch /usr/local/rvm/patches/ruby/1.8.7/stdout-rouge-fix.patch.
ruby-1.8.7-p374 - #applying patch /usr/local/rvm/patches/ruby/1.8.7/no_sslv2.diff.
ruby-1.8.7-p374 - #applying patch /usr/local/rvm/patches/ruby/ssl_no_ec2m.patch.
ruby-1.8.7-p374 - #configuring...............................
ruby-1.8.7-p374 - #post-configuration.
ruby-1.8.7-p374 - #compiling..........................................
ruby-1.8.7-p374 - #installing.
ruby-1.8.7-p374 - #making binaries executable.
ruby-1.8.7-p374 - #downloading rubygems-2.0.14
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  329k  100  329k    0     0  1495k      0 --:--:-- --:--:-- --:--:-- 7161k
ruby-1.8.7-p374 - #extracting rubygems-2.0.14.
ruby-1.8.7-p374 - #removing old rubygems.
ruby-1.8.7-p374 - #installing rubygems-2.0.14.........................
Error running 'env GEM_PATH=/usr/local/rvm/gems/ruby-1.9.3-p374:/usr/local/rvm/gems/ruby-1.9.3-p374@global:/usr/local/rvm/gems/ruby-1.9.3-p374:/usr/local/rvm/gems/ruby-1.9.3-p374@global GEM_HOME=/usr/local/rvm/gems/ruby-1.9.3-p374 /usr/local/rvm/rubies/ruby-1.8.7-p374/bin/ruby -d /usr/local/rvm/src/rubygems-2.0.14/setup.rb',
showing last 15 lines of /usr/local/rvm/log/1389229013_ruby-1.8.7-p374/rubygems.install.log
Exception `Errno::ENOENT' at /usr/local/rvm/rubies/ruby-1.8.7-p374/lib/ruby/1.8/fileutils.rb:1427 - No such file or directory - /usr/local/rvm/rubies/ruby-1.8.7-p374/lib/ruby/site_ruby/1.8/rubygems/ssl_certs/Class3PublicPrimaryCertificationAuthority.pem
Exception `Errno::ENOENT' at /usr/local/rvm/rubies/ruby-1.8.7-p374/lib/ruby/1.8/fileutils.rb:1304 - No such file or directory - /usr/local/rvm/rubies/ruby-1.8.7-p374/lib/ruby/site_ruby/1.8/rubygems/ssl_certs/Class3PublicPrimaryCertificationAuthority.pem
Exception `Errno::ENOENT' at /usr/local/rvm/rubies/ruby-1.8.7-p374/lib/ruby/1.8/fileutils.rb:1427 - No such file or directory - /usr/local/rvm/rubies/ruby-1.8.7-p374/lib/ruby/site_ruby/1.8/rubygems/ssl_certs/EntrustnetSecureServerCertificationAuthority.pem
Exception `Errno::ENOENT' at /usr/local/rvm/rubies/ruby-1.8.7-p374/lib/ruby/1.8/fileutils.rb:1304 - No such file or directory - /usr/local/rvm/rubies/ruby-1.8.7-p374/lib/ruby/site_ruby/1.8/rubygems/ssl_certs/EntrustnetSecureServerCertificationAuthority.pem
Exception `Errno::ENOENT' at /usr/local/rvm/rubies/ruby-1.8.7-p374/lib/ruby/1.8/fileutils.rb:1427 - No such file or directory - /usr/local/rvm/rubies/ruby-1.8.7-p374/lib/ruby/site_ruby/1.8/rubygems/ssl_certs/DigiCertHighAssuranceEVRootCA.pem
Exception `Errno::ENOENT' at /usr/local/rvm/rubies/ruby-1.8.7-p374/lib/ruby/1.8/fileutils.rb:1304 - No such file or directory - /usr/local/rvm/rubies/ruby-1.8.7-p374/lib/ruby/site_ruby/1.8/rubygems/ssl_certs/DigiCertHighAssuranceEVRootCA.pem
Exception `Errno::ENOENT' at /usr/local/rvm/rubies/ruby-1.8.7-p374/lib/ruby/1.8/fileutils.rb:1427 - No such file or directory - /usr/local/rvm/rubies/ruby-1.8.7-p374/lib/ruby/site_ruby/1.8/rubygems/ssl_certs/GeoTrustGlobalCA.pem
Exception `Errno::ENOENT' at /usr/local/rvm/rubies/ruby-1.8.7-p374/lib/ruby/1.8/fileutils.rb:1304 - No such file or directory - /usr/local/rvm/rubies/ruby-1.8.7-p374/lib/ruby/site_ruby/1.8/rubygems/ssl_certs/GeoTrustGlobalCA.pem
Exception `Errno::ENOENT' at /usr/local/rvm/rubies/ruby-1.8.7-p374/lib/ruby/1.8/fileutils.rb:1427 - No such file or directory - /usr/local/rvm/rubies/ruby-1.8.7-p374/bin/gem
Exception `Errno::ENOENT' at /usr/local/rvm/rubies/ruby-1.8.7-p374/lib/ruby/1.8/fileutils.rb:1304 - No such file or directory - /usr/local/rvm/rubies/ruby-1.8.7-p374/bin/gem
Exception `Gem::InstallError' at ./lib/rubygems/uninstaller.rb:101 - gem "gemcutter" is not installed
/usr/local/rvm/gems/ruby-1.9.3-p374/gems/json-1.7.7/lib/json/ext/parser.so: [BUG] Segmentation fault
ruby 1.8.7 (2013-06-27 patchlevel 374) [x86_64-linux]

RubyGems 2.0.14 installed
ruby-1.8.7-p374 - #gemset created /usr/local/rvm/gems/ruby-1.8.7-p374@global
ruby-1.8.7-p374 - #importing gemset /usr/local/rvm/gemsets/global.gems......
ruby-1.8.7-p374 - #generating global wrappers.
ruby-1.8.7-p374 - #gemset created /usr/local/rvm/gems/ruby-1.8.7-p374
ruby-1.8.7-p374 - #importing gemsetfile /usr/local/rvm/gemsets/default.gems evaluated to empty gem list
ruby-1.8.7-p374 - #generating default wrappers.
ruby-1.8.7-p374 - #adjusting #shebangs for (gem irb erb ri rdoc testrb rake).
Install of ruby-1.8.7-p374 - #complete
WARNING: Please be aware that you just installed a ruby that is no longer maintained, for a list of maintained rubies visit:

    http://bugs.ruby-lang.org/projects/ruby/wiki/ReleaseEngineering

Please consider upgrading to ruby-2.1.0 which will have all of the latest security patches.
Ruby was built without documentation, to build it run: rvm docs generate-ri

[root@localhost ~]# rvm gemset create rails235
ruby-1.8.7-p374 - #gemset created /usr/local/rvm/gems/ruby-1.8.7-p374@rails235
ruby-1.8.7-p374 - #generating rails235 wrappers.
[root@localhost ~]# rvm 1.8.7@rails235
```


## 作ったgemsetでrailsをインストール。


``` bash
# gem install rails --version "=2.3.5"
```


同一バージョンのrubyで複数のgemsetを作成して管理することもできるらしい。
参考にしたのはこちら。
http://serihiro.hatenablog.com/entry/20130421/1366552174

マシンを分けたほうがいいと思うけど、必要な場合もあるだろうということで。
