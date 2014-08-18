1日目: プロジェクトを始める
===========================

.. include:: common/original.rst.inc

Jobeet とは
-----------

`Jobeet` はオープンソースのタスク管理のソフトウェアです。一日づつチュートリアルをこなし、最新の Web テクノロジーである Symfony 2.3.2 を学ぶことを手助けします。
(まだ知らない人のために、SymfonyはPHPのためのフレームワークです。)
Jobeet is an Open-Source job board software which provides you day-by-day tutorials, that will help you learn the latest Web Technolgy,
 Symfony 2.3.2 (for those who don’t know yet, Symfony is a framework for PHP).

それぞれのチャプター/日は、約一時間ででき、開始から終了まで、本物のウェブサイトをコーディングすることによって Symfony を学ぶことが出来ます。
Each chapter/day is meant to last about one hour, and will be the occasion to learn Symfony by coding a real website, from start to finish.

毎日、新しい機能がアプリケーションに追加され、新しい Symfony の機能性だけでなく、 Symfony の Web 開発におけるグッドプラクティスを紹介し、この開発の利点を学びます。
Every day, new features will be added to the application and
we’ll take advantage of this development to introduce you to new Symfony functionalities, as well as good practices in Symfony web development.

本日は初日となり、コードは書きません。その代わり開発環境のセットアップを行います。
Today, being your first day, you won’t be writing any code. Instead, you will setup a working development environment.

開発環境のセットアップ
----------------------
Setting up the working development environment
---------------

初めに、コンピューターがweb開発環境として適しているか確認する必要があります。
私たちは、VMware Playerの仮想マシンにインストールされているUbuntuの12.04 LTS Serverを使用します。最低でも、（例えばApacheの）Webサーバ、データベースエンジン（MySQLの）とPHP5.3.3以降が必要です。
First of all, you need to check that your computer has a friendly working environment for web development.
 We will use Ubuntu 12.04 LTS Server installed in a VMware Player virtual machine. At a minimum, you need a web server (Apache, for instance), a database engine (MySQL) and PHP 5.3.3 or later.

1のApacheは、Webサーバーをインストールします。
1. Install Apache, your web server:

.. code-block:: bash

    $ sudo apt-get install apache2

and enable Apache mod-rewrite:

.. code-block:: bash

    $ sudo a2enmod rewrite

2. Install the MySQL Server:

.. code-block:: bash

    $ sudo apt-get install mysql-server mysql-client

3. Install PHP, the server scripting language

.. code-block:: bash

    $ sudo apt-get install php5 libapache2-mod-php5 php5-mysql

4. Install Intl extension:

.. code-block:: bash

    $ sudo apt-get install php5-intl

5. Now, you need to restart Apache service:

.. code-block:: bash

    $ sudo service apache2 restart

Symfony 2.3.2 のダウンロードとインストール
------------------------------------------
Download and install Symfony 2.3.2
----------------------------------

まず最初に、新しいプロジェクトをインストールする場所をWebサーバ上のディレクトリを準備することです。は/ var/ www /のJobeetの：それはJobeetの呼びましょう。
The first thing to do is to prepare a directory on your web server where you want to install the new project. Let’s call it jobeet: /var/www/jobeet.

.. code-block:: bash

    $ mkdir /var/www/jobeet

私たちは、ディレクトリが用意しているが、それには何を入れて？ http://symfony.com/downloadに進み、ベンダーのないsymfonyの標準2.3.2を選択し、それをダウンロード。今、あなたの準備ディレクトリ、Jobeetのにsymfonyのディレクトリ内のファイルを解凍します。
We have a directory prepared, but what to put in it? Go to http://symfony.com/download, choose Symfony Standard 2.3.2 without vendors and download it. Now, unzip the files inside the Symfony directory to your prepared directory, jobeet.

Vendors の更新
~~~~~~~~~~~~~~
Updating Vendors
~~~~~~~~~~~~~~~~~~

この時点では、独自のアプリケーションを開発することから始めましょうするフル機能のsymfonyのプロジェクトをダウンロードした。 symfonyのプロジェクトは、外部ライブラリの数によって異なります。これらは、作曲家と呼ばれるライブラリを経由して、プロジェクトのベンダー/ディレクトリにダウンロードされます。
Composerは、symfonyの2.3.2スタンダード版をダウンロードするために使用することができ、PHPの依存管理ライブラリです。あなたのJobeetのディレクトリ上にComposerをダウンロードすることで起動します。
At this point, you’ve downloaded a fully-functional Symfony project in which you’ll start to develop your own application. A Symfony project depends on a number of external libraries. These are downloaded into the vendor/ directory of your project via a library called Composer.
Composer is a dependency management library for PHP, which you can use to download the Symfony 2.3.2 Standard Edition. Start by downloading Composer onto your jobeet directory:

.. code-block:: bash

    $ curl -s https://getcomposer.org/installer | php

あなたはカール拡張機能をインストールしていない場合は、このコマンドを使用してインストールすることができます。
If you don’t have curl extension installed, you can install it using this command:

.. code-block:: bash

    $ sudo apt-get install curl

次に、必要なすべてのベンダー·ライブラリのダウンロードを開始するには、次のコマンドを入力します。
Next, type the following command to start downloading all the necessary vendor libraries:

.. code-block:: bash

    $ php composer.phar install

Web サーバーの設定
------------------
Web Server Configuration
----------------

良いのWebプラクティスは、Webルートディレクトリの下にスタイルシート、JavaScriptと画像のように、Webブラウザがアクセスする必要のあるファイルだけを置くことです。デフォルトでは、symfonyプロジェクトのウェブ/サブディレクトリの下にこれらのファイルを保存することをお勧めします。
あなたの新しいプロジェクトのためにApacheを設定するには、仮想ホストを作成します。そのためには、次のコマンドで、使用している端末と型に移動します。
A good web practice is to put under the web root directory only the files that need to be accessed by a web browser, like stylesheets, JavaScripts and images. By default, it’s recommended to store these files under the web/ sub-directory of a symfony project.
To configure Apache for your new project, you will create a virtual host. In order to do that, go to your terminal and type in the next command :

.. code-block:: bash

    $ sudo nano /etc/apache2/sites-available/jobeet.local

さて、という名前のファイルjobeet.localが作成されます。コントロールその後、Oをし、それを保存します - - そのファイルの中に次のことを入れて、[コントロールをヒットXは、エディタを終了します。
jobeet.localは、/ etc / apache2の/サイト利用可能
Now, a file named jobeet.local is created. Put the following inside that file, then hit Control – O and Enter to save it, then Control – X to exit the editor.
etc/apache2/sites-available/jobeet.local

.. code-block:: apache

    <VirtualHost *:80>
        ServerName jobeet.local
        DocumentRoot /var/www/jobeet/web
        DirectoryIndex app.php
        ErrorLog /var/log/apache2/jobeet-error.log
        CustomLog /var/log/apache2/jobeet-access.log combined
        <Directory "/var/www/jobeet/web">
            AllowOverride All
            Allow from All
         </Directory>
     </VirtualHost>

Apacheの設定で使用されるドメイン名jobeet.localは、局所的に宣言する必要があります。あなたがLinuxシステムを実行すると、/ etc / hostsファイルで行う必要があります。 Windowsを実行すると、このファイルは、Cに位置しています：\ WINDOWS \ SYSTEM32 \ドライバ\ etc \ディレクトリにあります。次の行を追加します。
The domain name jobeet.local used in the Apache configuration has to be declared locally. If you run a Linux system, it has to be done in the /etc/hosts file. If you run Windows, this file is located in the C:\Windows\System32\drivers\etc\ directory. Add the following line:

.. code-block:: bash

    127.0.0.1 jobeet.local

.. tip::

   リモート·サーバーで作業している場合には、Webサーバー·マシンのIPを127.0.0.1を交換してください。
   Replace 127.0.0.1 with the ip of your web server machine in case you are working on a remote server.

これが仕事をしたい場合は、新しく作成された仮想ホストを有効にしてApacheを再起動する必要があります。だからあなたの端末とタイプに移動します。
If you want this to work, you need to enable the newly created virtual host and restart your Apache. So go to your terminal and type:

.. code-block:: bash

    $ sudo a2ensite jobeet.local
    $ sudo service apache2 restart

symfonyはWebサーバーとPHPが正常にsymfonyを使用するように設定されていることを確認し支援するための視覚サーバー構成テスターが付属しています。設定を確認するには、次のURLを使用します。
Symfony comes with a visual server configuration tester to help make sure your Web server and PHP are correctly configured to use Symfony. Use the following URL to check your configuration:

http://jobeet.local/config.php

.. image:: /images/sf2-config-1.png

あなたがローカルホストからこれを実行しない場合は、検索しなければならないとオープンなWeb/ config.phpファイルとローカルホストの外部からのアクセスを制限する行をコメントにします。
If you don’t run this from your localhost, you should locate and open web/config.php file and comment the lines that restrict the access outside localhost:

web/config.php

.. code-block:: php

   if (!isset($_SERVER['HTTP_HOST'])) {
      exit('This script cannot be run from the CLI. Run it from a browser.');
   }
   /*
   if (!in_array(@$_SERVER['REMOTE_ADDR'], array(
      '127.0.0.1',
      '::1',
   ))) {
      header('HTTP/1.0 403 Forbidden');
      exit('This script is only accessible from localhost.');
   }
   */
   // ...

ウェブ/ app_dev.phpに対しても同じことを行います。
Do the same for web/app_dev.php:

web/app_dev.php

.. code-block:: php

   use Symfony\Component\HttpFoundation\Request;
   use Symfony\Component\Debug\Debug;

   // If you don't want to setup permissions the proper way, just uncomment the following PHP line
   // read http://symfony.com/doc/current/book/installation.html#configuration-and-setup for more information
   //umask(0000);

   // This check prevents access to debug front controllers that are deployed by accident to production servers.
   // Feel free to remove this, extend it, or make something more sophisticated.
   /*
   if (isset($_SERVER['HTTP_CLIENT_IP'])
       || isset($_SERVER['HTTP_X_FORWARDED_FOR'])
       || !in_array(@$_SERVER['REMOTE_ADDR'], array('127.0.0.1', 'fe80::1', '::1'))
   ) {
       header('HTTP/1.0 403 Forbidden');
       exit('You are not allowed to access this file. Check '.basename(__FILE__).' for more information.');
   }
   */

   $loader = require_once __DIR__.'/../app/bootstrap.php.cache';
   Debug::enable();

   require_once __DIR__.'/../app/AppKernel.php';

   // ...

あなたがconfig.phpにに行くときおそらく、あなたは必要条件のすべての種類を取得します。以下では、これらすべての「警告」を得ていないために観光名所のリストである。
1アプリ/キャッシュと、アプリ/ログのパーミッションを変更します。
Probably, you will get all kind of requirements when you go to config.php. Below, is a list of things to do for not getting all those “warnings”.
1. Change the permissions of app/cache and app/logs:

.. code-block:: bash

   sudo chmod -R 777 app/cache
   sudo chmod -R 777 app/logs
   sudo setfacl -dR -m u::rwX app/cache app/logs

あなたはまだそれを持っていない場合は、ACLをインストールします。
Install ACL if you don’t have it yet:

.. code-block:: bash

   sudo apt-get install acl

2。php.iniでdate.timezoneで設定を設定します
2. Set the date.timezone setting in php.ini

etc/php5/apache2/php.ini

.. code-block:: ini

   date.timezone = Europe/Bucharest

.. code-block:: bash

   sudo nano /etc/php5/apache2/php.ini

[日付]セクションのdate.timezoneで設定を見つけて、あなたのタイムゾーンに設定します。その後、消去";"、行の先頭に配置。
Find the date.timezone setting for [date] section and set it to your timezone. After that, erase “;”, placed at the beginning of the line.

3。同じphp.iniファイルでオフにshort_open_tagを設定を設定します
3. Set the short_open_tag setting to off in the same php.ini file

etc/php5/apache2/php.ini

.. code-block::

   short_open_tag
     Default Value: Off

4。インストールし、PHPのアクセラレータを有効（APC推奨）
4. Install and enable a PHP Accelerator (APC recommended)

.. code-block:: bash

   sudo apt-get install php-apc
   sudo service apache2 restart

Apacheを再起動した後、HTTPでブラウザウィンドウと種類を開き//jobeet.local/app_dev.phpを。次のページが表示されます。
After restarting Apache, open a browser window and type in http://jobeet.local/app_dev.php. You should see the following page:

.. image:: /images/Day-1-SF_welcome.jpg

Symfony2 のコンソール
---------------------
Symfony2 Console
----------------

Symfony2のあなたはさまざまなタスクに使用するコンソールコンポーネントのツールが付属しています。コマンドプロンプトで入力することができることのリストを表示するには：
Symfony2 comes with the console component tool that you will use for different tasks. To see a list of things it can do for you type at the command prompt:

.. code-block:: bash

    $ php app/console list

Application Bundle の作成
-------------------------
Creating the Application Bundle
----------------

bundleとは何か
~~~~~~~~~~~~~~~~~~
What exactly is a bundle?
~~~~~~~~~~~~~~~~~~

他のソフトウェア内のプラグインが、それでも良いと似ています。主な違いは、すべてがコアフレームワークの機能とアプリケーションのために書かれたコードの両方を含む、symfonyの2.3.2バンドル、ということである。
バンドルは、単一の機能を実装したディレクトリ内のファイルの構造化されたセットです。
ヒント：バンドルがある限り、それが（アプリ/ autoload.php）をオートロードすることができるようにどこにでも住むことができる。
Is similar to a plugin in other software, but even better. The key difference is that everything is a bundle in Symfony 2.3.2, including both core framework functionality and the code written for your application.
A bundle is a structured set of files within a directory that implement a single feature.
Tips: A bundle can live anywhere as long as it can be autoloaded (app/autoload.php).

.. note::

   ここに詳細を読むことができます：http://symfony.com/doc/current/book/page_creation.html#theバンドルシステム - バンドルシステム。
   You can read more here: http://symfony.com/doc/current/book/page_creation.html#the-bundle-system – The Bundle System.

基本 bundle スケルトンの作成
~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Creating a basic bundle skeleton
~~~~~~~~~~~~~~~~~~

symfonyのバンドル発電機を起動するには、次のコマンドを実行します。
Run the following command to start the Symfony’s bundle generator:

.. code-block:: bash

    $ php app/console generate:bundle --namespace=Ibw/JobeetBundle

発電機は、バンドルを生成する前にあなたにいくつか質問をします。ここで質問と回答（1を除くすべてが、デフォルトの答えが、）は次のとおりです。
The generator will ask you some questions before generating the bundle. Here are the questions and answers (all, except one, are the default answers):

.. code-block:: bash

    Bundle name [IbwJobeetBundle]: IbwJobeetBundle
    Target directory [/var/www/jobeet/src]: /var/www/jobeet/src
    Configuration format (yml, xml, php, or annotation) [yml]: yml
    Do you want to generate the whole directory structure [no]? yes
    Do you confirm generation [yes]? yes
    Confirm automatic update of your Kernel [yes]? yes
    Confirm automatic update of the Routing [yes]? yes

との新しいバンドルを生成した後にキャッシュをクリアします。
Clear the cache after generating the new bundle with:

.. code-block:: bash

    $ php app/console cache:clear --env=prod
    $ php app/console cache:clear --env=dev

のsrc / IBW/ JobeetBundle：新しいJobeetのバンドルは現在、プロジェクトのsrcディレクトリにあります。バンドルジェネレータはindexアクションでDefaultControllerを作った。ます。http：//jobeet.local/hello/jobeetまたはhttp：//jobeet.local/app_dev.php/hello/jobeetあなたがお使いのブラウザでこれをアクセスすることができます。
The new Jobeet bundle can be now found in the src directory of your project: src/Ibw/JobeetBundle. The bundle generator made a DefaultController with an index action. You can access this in your browser: http://jobeet.local/hello/jobeet or http://jobeet.local/app_dev.php/hello/jobeet.

AcmeDemoBundle の削除の仕方
~~~~~~~~~~~~~~~~~~~~~~~~~~~
How to remove the AcmeDemoBundle
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

symfonyの2.3.2 Standard EditionではAcmeDemoBundle呼ばバンドル内部に住んでいる完全なデモが付属しています。
これは、プロジェクトを開始しながら、参照するための偉大な決まり文句ですが、あなたはおそらく、最終的にそれを削除したいと思う。
The Symfony 2.3.2 Standard Edition comes with a complete demo that lives inside a bundle called AcmeDemoBundle.
It is a great boilerplate to refer to while starting a project, but you’ll probably want to eventually remove it.

1アクメディレクトリを削除するコマンドを入力します。
1. Type the command to delete Acme directory:

.. code-block:: bash

    $ rm -rf /var/www/jobeet/src/Acme

2移動先：/var/www/jobeet/app/AppKernel.phpおよび削除：
2. Go to: /var/www/jobeet/app/AppKernel.php  and delete:

app/AppKernel.php

.. code-block:: php

   // ...

   $bundles[] = new Acme\DemoBundle\AcmeDemoBundle();

   // ...

そして今、アプリ/ configに/ routing_dev.ymlから削除します。
and now delete from app/config/routing_dev.yml:

app/config/routing_dev.yml

.. code-block:: yaml

   # ...

   # AcmeDemoBundle routes (to be removed)
   _acme_demo:
       resource: "@AcmeDemoBundle/Resources/config/routing.yml"

3最後に、キャッシュをクリアします。
3. Finally, clear the cache.

環境
----
The Environments
----------------

symfonyの2.3.2は、さまざまな環境を持っている。あなたはプロジェクトのwebディレクトリを見ると、次の2つのPHPファイルが表示されます：app.phpとapp_dev.phpを。これらのファイルは、フロントコントローラと呼ばれます。アプリケーションへのすべてのリクエストは、それらを介して行われます。 app.phpファイルは実稼働環境用であり、彼らは、開発環境でアプリケーションで作業する際app_dev.phpは、ウェブ開発者によって使用されます。開発者の親友 - それはあなたのすべてのエラーと警告とWebデバッグツールバーが表示されますので、開発環境は、非常に便利証明する。
それはすべて今日のためだ。私たちは正確にについてJobeetのWebサイトがどうなるかについてお話しますこのチュートリアルの次の日にお会いしましょう！
Symfony 2.3.2 has different environments. If you look in the project’s web directory, you will see two php files: app.php and app_dev.php. These files are called front controllers; all requests to the application are made through them. The app.php file is for production environment and app_dev.php is used by web developers when they work on the application in the development environment. The development environment will prove very handy because it will show you all the errors and warnings and the Web Debug Toolbar – the developer’s best friend.
That’s all for today. See you on the next day of this tutorial, when we will talk about what exactly the Jobeet website will be about!

.. include:: common/license.rst.inc
