1日目: プロジェクトを始める
===========================

.. include:: common/original.rst.inc

Jobeet とは
-----------

`Jobeet` はオープンソースの求人掲示板のソフトウェアです。一日づつチュートリアルをこなし、最新の Web テクノロジーである Symfony 2.3.2 を学ぶことを手助けします。
(まだ知らない人のために、SymfonyはPHPのためのフレームワークです。)

それぞれのチャプター/日は、約一時間ででき、開始から終了まで、本物のウェブサイトをコーディングすることによって Symfony を学ぶことが出来ます。

毎日、新しい機能がアプリケーションに追加され、新しい Symfony の機能性だけでなく、 Symfony の Web 開発におけるグッドプラクティスを紹介し、この開発の利点を学びます。

本日は初日となり、コードは書きません。その代わり開発環境のセットアップを行います。


開発環境のセットアップ
----------------------

初めに、コンピューターがweb開発環境として適しているか確認する必要があります。
私たちは、 VMware Player の仮想マシンにインストールされている Ubuntu 12.04 LTS Server を使用します。最低でも、 Web サーバ（例えば Apache ）、データベースエンジン（ MySQL ）と PHP 5.3.3 以降が必要です。

1. Webサーバー Apache をインストールします。:

.. code-block:: bash

    $ sudo apt-get install apache2

次に、 Apache mod-rewrite を有効にします。

.. code-block:: bash

    $ sudo a2enmod rewrite

2. MySQLサーバをインストールします。

.. code-block:: bash

    $ sudo apt-get install mysql-server mysql-client

3. サーバースクリプト言語、 PHP をインストールします。

.. code-block:: bash

    $ sudo apt-get install php5 libapache2-mod-php5 php5-mysql

4. 国際化拡張機能をインストールします。

.. code-block:: bash

    $ sudo apt-get install php5-intl

5. ここで、Apache サービスを再起動する必要があります。

.. code-block:: bash

    $ sudo service apache2 restart

Symfony 2.3.2 のダウンロードとインストール
------------------------------------------

まず最初にすることは、新しいプロジェクトをインストールする場所を Web サーバ上のディレクトリを準備することです。それを Jobeet ( /var/www/jobeet ) と呼びましょう。

.. code-block:: bash

    $ mkdir /var/www/jobeet

ディレクトリを用意しましたが、それに何を入れましょう？
http://symfony.com/download に進み、ベンダーのない Symfony Standard 2.3.2 を選択し、それをダウンロードします。
今、あなたの準備ディレクトリ、 Jobeet に symfony のディレクトリ内のファイルを解凍します。

Vendors の更新
~~~~~~~~~~~~~~

この時点では、あなたの独自アプリケーション開発をはじめる、フル機能の Symfony のプロジェクトをダウンロードしました。
Symfony のプロジェクトは、多くの外部ライブラリに依存します。
それらは、 Composer と呼ばれるライブラリを経由して、プロジェクトの ``vendor/`` ディレクトリにダウンロードされます。
Composer は PHP の依存管理ライブラリで、 Symfony 2.3.2 スタンダード版をダウンロードするために使用することができます。
Jobeet のディレクトリ上に Composer をダウンロードすることではじめます。

.. code-block:: bash

    $ curl -s https://getcomposer.org/installer | php

curl 拡張をインストールしていない場合は、このコマンドを使用してインストールすることができます。

.. code-block:: bash

    $ sudo apt-get install curl

次に、必要なすべてのベンダー·ライブラリのダウンロードを開始するには、次のコマンドを入力します。

.. code-block:: bash

    $ php composer.phar install

Web サーバーの設定
------------------

良い Web の作法は、 Web ルートディレクトリの下にスタイルシート、 JavaScript と画像のように、 Web ブラウザがアクセスする必要のあるファイルだけを置くことです。
デフォルトでは、 Symfony プロジェクトの web/ サブディレクトリの下にこれらのファイルを保存することをお勧めします。
あなたの新しいプロジェクトのために Apache を設定するには、バーチャルホストを作成します。そのためには、使用しているターミナルで次のコマンドをタイプします。

.. code-block:: bash

    $ sudo nano /etc/apache2/sites-available/jobeet.local

これで、 jobeet.local という名前のファイルが作成されます。以下をそのファイルの中に追記し、`Control – O` を選択し、それを保存します。その後、 `Control – X` でエディタを終了します。

/etc/apache2/sites-available/jobeet.local

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

Apache の設定で使用されるドメイン名 jobeet.local は、ローカルで宣言する必要があります。
あなたが Linux システムで実行する場合、``/etc/hosts`` ファイルで行う必要があります。
Windowsで実行している場合、このファイルは ``C:\Windows\System32\drivers\etc\`` ディレクトリにあります。次の行を追加します。

.. code-block:: bash

    127.0.0.1 jobeet.local

.. tip::

   リモート·サーバーで作業している場合には、127.0.0.1をWebサーバー·マシンのIPで置き換えてください。

この仮想ホストを動かしたい場合は、新しく作成された仮想ホストを有効にして Apache を再起動する必要があります。
そのため、ターミナルに移って次のようにタイプします。

.. code-block:: bash

    $ sudo a2ensite jobeet.local
    $ sudo service apache2 restart

Symfony を正常に使用できるように、 Web サーバーと PHP が設定されているかどうか、確認する視覚的なサーバー設定テスターが Symfony に付属しています。
設定を確認するには、次のURLを使用します。

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

3. 最後に、キャッシュをクリアします。
3. Finally, clear the cache.

環境
----
The Environments
----------------

Symfony 2.3.2 は、異なった環境を持っています。プロジェクトのwebディレクトリを見ると、app.phpとapp_dev.phpという2つの PHP ファイルを見るでしょう。
これらのファイルは、フロントコントローラーと呼ばれます。アプリケーションへのすべてのリクエストは、それらを介して行われます。
app.php ファイルは本番環境用であり、 app_dev.php はウェブ開発者によって使用される開発環境です。
開発環境は、すべてのエラーと警告とWebデバッグツールバーが表示されますので非常に便利だと分かるでしょう。開発者の親友です。
これで今日はすべてです。明日は Jobeet の Web サイトがどうなるかについてお話します。このチュートリアルでお会いしましょう。！
Symfony 2.3.2 has different environments. If you look in the project’s web directory, you will see two php files: app.php and app_dev.php.
These files are called front controllers; all requests to the application are made through them.
The app.php file is for production environment and app_dev.php is used by web developers when they work on the application in the development environment.
The development environment will prove very handy because it will show you all the errors and warnings and the Web Debug Toolbar – the developer’s best friend.
That’s all for today. See you on the next day of this tutorial, when we will talk about what exactly the Jobeet website will be about!

.. include:: common/license.rst.inc
