---
layout: post
title: "Windows7上にVagrantでCentOS6.3 64bit環境を構築する"
date: 2014-02-01 15:09:08 +0900
comments: true
categories: Vagrant
published: true
---



vagrant便利です。

今日はWindows7環境でvagrantを自由に使えるようになるまでの手順を記録します。

私が普段作業する端末はwindows7であるため、vagrantのりようにあたってcygwinやgit shell等、sshクライアントの使えるソフトウェアを別途インストールする必要がありました。

私は普段cygwinを使っているので、cygwinで説明します。

<!--more-->

## 1. cygwinのインストール

<a href="http://www.cygwin.com/install.html" target="_blank"><img class="alignleft size-full wp-image-37" alt="cygwinTopPage" src="http://labo.in-ception.com/wp-content/uploads/2013/06/Screenshot_cyg.png" width="768" height="427" /></a><br style="clear:both" />
サイトURL：<a href="http://www.cygwin.com/install.html" target="_blank">http://www.cygwin.com/install.html</a>
ダウンロードURL：<a href="http://www.cygwin.com/setup.exe" target="_blank">http://www.cygwin.com/setup.exe</a>
まずはcygwinをインストールしましょう。
多くのパッケージを同時にインストールするので、少し時間がかかります。
並行してVirtualboxのインストールも進めるといいでしょう。

## 2. VirtalBoxのインストール

<a href="https://www.virtualbox.org/wiki/Downloads" target="_blank"><img class="alignleft size-full wp-image-34" alt="VirtualBoxTopPage" src="http://labo.in-ception.com/wp-content/uploads/2013/06/Screenshotvbox_1.png" width="744" height="378" /></a><br style="clear:both" />
サイトURL：<a href="https://www.virtualbox.org/wiki/Downloads" target="_blank">https://www.virtualbox.org/wiki/Downloads</a>
次に、virtalBoxをインストールします。
windowsOSへのインストールなので、
**<span class="GINGER_SOFATWARE_correct">virtual</span> box <span class="GINGER_SOFATWARE_noSuggestion GINGER_SOFATWARE_correct">x.x.xx</span> for Windows hosts** をダウンロード、インストールします。
（今回は **4.2.14** を使用）

## 3. vagrantのインストール

<a href="http://downloads.vagrantup.com/" target="_blank"><img class="alignleft size-full wp-image-65" alt="Screenshot_1vagr" src="http://labo.in-ception.com/wp-content/uploads/2013/07/Screenshot_1vagr.png" width="1154" height="748" /></a><br style="clear:both" />
ダウンロードURL：<a title="http://downloads.vagrantup.com/" href="http://downloads.vagrantup.com/" target="_blank">http://downloads.vagrantup.com/</a>

<img alt="Screenshot_1red" src="http://labo.in-ception.com/wp-content/uploads/2013/07/Screenshot_1red.png" width="318" height="136" />

最新バージョンの、今回はWindows版をダウンロード・インストールしましょう。

## 4. <span class="GINGER_SOFATWARE_correct">vagrant</span> boxの作成

パスを通して、コマンドからbox作成をします。
ソースは<a href="http://www.vagrantbox.es/" target="_blank">http://www.vagrantbox.es/</a>で公開されています。

ページ下部、CentOSの項目を探します。
<img class="alignleft size-full wp-image-74" alt="Screenshot_copy1" src="http://labo.in-ception.com/wp-content/uploads/2013/07/Screenshot_copy1.png" width="732" height="92" /><br style="clear:both" />

今回は「CentOS6.3 64bit」で構築するため、「CentOS 6.3 x86_64 minimal」のURLを指定しましょう。

<pre class="lang:shell decode:1 " >vagrant box add YOUR_BOX_NAME https://dl.dropbox.com/u/7225008/Vagrant/CentOS-6.3-x86_64-minimal.box
vagrant init
vagrant up</pre>

## 5. 設定ファイル Vagrantfile の編集

仮想サーバを起動する前に、設定を変更しておきましょう。

１．config.vm.box の箇所にbox名を指定。
２．config.vm.network :private_network のコメントアウトを外します。IPはそのままでもOK。
３．config.vm.network のコメントアウトを外し、:public_network とします。
４． config.vm.provider と vb.customize のコメントアウトを外し、vb.customize については好みのメモリサイズに変更しましょう。物理メモリサイズをMB単位で設定できます。
※下記では、1GBで設定しています。
[code lang="ruby"]
# Every Vagrant virtual environment requires a box to build off of.
config.vm.box = “YOUR_BOX_NAME”

# The url from where the ‘config.vm.box’ box will be fetched if it
# doesn’t already exist on the user’s system.
# config.vm.box_url = “http://domain.com/path/to/above.box”

# Create a forwarded port mapping which allows access to a specific port
# within the machine from a port on the host machine. In the example below,
# accessing “localhost:8080″ will access port 80 on the guest machine.
# config.vm.network :forwarded_port, guest: 80, host: 8080

# Create a private network, which allows host-only access to the machine
# using a specific IP.
config.vm.network :private_network, ip: “192.168.33.10″

# Create a public network, which generally matched to bridged network.
# Bridged networks make the machine appear as another physical device on
# your network.
config.vm.network :public_network

# Share an additional folder to the guest VM. The first argument is
# the path on the host to the actual folder. The second argument is
# the path on the guest to mount the folder. And the optional third
# argument is a set of non-required options.
# config.vm.synced_folder “../data”, “/vagrant_data”

# Provider-specific configuration so you can fine-tune various
# backing providers for Vagrant. These expose provider-specific options.
# Example for VirtualBox:
#
config.vm.provider :virtualbox do |vb|
# Don’t boot with headless mode
# vb.gui = true

# Use VBoxManage to customize the VM. For example to change memory:
vb.customize ["modifyvm", :id, "--memory", "1024"]
end
[/code]

## 6. ゲストマシンを起動しログイン

下記コマンドだけで仮想マシンに接続することが出来ます。
設定に不備がなければ、仮想マシンからWAN（インターネット）に接続もできるようになっています。

<pre class="lang:shell decode:1 " >vagrant up
vagrant ssh</pre>

## 7. ソフトウェアインストール（apache<span class="GINGER_SOFATWARE_correct">,</span>PHP<span class="GINGER_SOFATWARE_correct">,</span>MySQL）

基本的なライブラリをひと通りインストールします。

<pre class="lang:shell decode:1 " >yum -y install php*
yum -y install mysql*
yum -y install ntp*
yum -y install httpd</pre>

ほか適宜。
典型的なLAMP環境でいきます。

## 8. 端末（ホストマシン）のhostsファイルの編集

apacheのSERVER_NAMEの文字列に対しては、Vagrantfileにも定義した内部IPを見るように端末のhostsを書き換えます。
[code]
"C:WindowsSystem32driversetchosts"
[/code]
Windows7なので、上記ファイルは管理者権限で開いたメモ帳やエディタで、編集します。
ここでは仮にSERVER_NAMEをhoge.devとします。
[code]
hoge.dev 192.168.33.10
[/code]

## 9. apacheテストページ確認

この状態でブラウザでhttp://hoge.dev/にアクセスして、apacheのテストページが帰ってきたらひとまずOKです。

次回はこの環境下に、今ちょっと話題のLaravel4をインストールしてhello world!までやってみます。
