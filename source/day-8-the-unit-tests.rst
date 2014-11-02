8日目: 単体テスト(ユニットテスト)
=================================

.. include:: common/original.rst.inc

Symfony のテスト
----------------

| Symfony においての自動化されたテストには単体テスト(ユニットテスト)と機能テスト( ``functional test`` )の2種類があります。
| 単体テストは、各メソッドと機能が正常に動作していることを確認します。各テストは、他のものから、可能な限り独立していなければなりません。
| 一方で、機能テストはアプリケーションの結果が、全体として正しく動作することを確認します。
| 明日は機能テストに専念し、本日はユニットテストを説明します。
| Symfony2 は独立したライブラリであり、豊富なテスト·フレームワークを提供する PHPUnit と統合されています。
| テストを実行するには、 PHPUnit の 3.5.11 以降をインストールする必要があります。
There are two different kinds of automated tests in Symfony: unit tests and functional tests.
Unit tests verify that each method and function is working properly.Each test must be as independent as possible from the others.
On the other hand, functional tests verify that the resulting application behaves correctly as a whole.
Unit tests will be covered in this post, whereas the next post will be dedicated to funcional tests.
Symfony2 integrates with an independent library, the PHPUnit, to give you a rich testing framework.
To run tests, you will have to install PHPUnit 3.5.11 or later.

.. note::

   PHPUnit がインストールされていない場合は、以下を使用します。
   If you don’t have PHPUnit installed, use the following to get it:

   .. code-block:: bash

      $ sudo apt-get install phpunit
      $ sudo pear channel-discover pear.phpunit.de
      $ sudo pear channel-discover pear.symfony-project.com
      $ sudo pear channel-discover components.ez.no
      $ sudo pear channel-discover pear.symfony.com
      $ sudo pear update-channels
      $ sudo pear upgrade-all
      $ sudo pear install pear.symfony.com/Yaml
      $ sudo pear install --alldeps phpunit/PHPUnit
      $ sudo pear install --force --alldeps phpunit/PHPUnit

| 各テスト（単体テストおよび機能テスト） はバンドルのサブディレクトリ Tests/ におかれる PHP のクラスです。
| この設置場所のルールに沿っている場合は、次のコマンドを使用して、すべてのアプリケーションのテストを実行することができます。
Each test – whether it’s a unit test or a functional test – is a PHP class that should live in the Tests/ subdirectory of your bundles.
If you follow this rule, then you can run all of your application’s tests with the following command:

.. code-block:: bash

   $ phpunit -c app/

| -c オプションは、設定ファイルのためのディレクトリ app/ を参照するように PHPUnit に指示します。
| PHPUnit のオプションについて興味があるなら、 app/phpunit.xml.dist ファイルを確認してください。
| ユニットテストは、通常、特定のPHPクラスに対するテストです。 ``Jobeet::slugify()`` メソッドのためのテストを書くことから始めましょう。
| src/Ibw/JobeetBundle/Tests/Utils フォルダに新しいファイル、 JobeetTest.php を作成します。
| 慣例により、 Tests/ のサブディレクトリは、バンドルのディレクトリ構成を複製する必要があります。
| バンドルのディレクトリ Utils/ のクラスをテストするときは、ディレクトリ Tests/Utils/ にテストを置おきます。
The -c option tells PHPUnit to look in the app/ directory for a configuration file. If you’re curious about the PHPUnit options, check out the app/phpunit.xml.dist file.
A unit test is usually a test against a specific PHP class. Let’s start by writing tests for the Jobeet:slugify() method.
Create a new file, JobeetTest.php, in the src/Ibw/JobeetBundle/Tests/Utils folder. By convention, the Tests/ subdirectory should replicate the directory of your bundle. So, when we are testing a class in our bundle’s Utils/ directory, we put the test in the Tests/Utils/ directory:

src/Ibw/JobeetBundle/Tests/Utils/JobeetTest.php

.. code-block:: php

   namespace Ibw\JobeetBundle\Tests\Utils;

   use Ibw\JobeetBundle\Utils\Jobeet;

   class JobeetTest extends \PHPUnit_Framework_TestCase
   {
       public function testSlugify()
       {
           $this->assertEquals('sensio', Jobeet::slugify('Sensio'));
           $this->assertEquals('sensio-labs', Jobeet::slugify('sensio labs'));
           $this->assertEquals('sensio-labs', Jobeet::slugify('sensio labs'));
           $this->assertEquals('paris-france', Jobeet::slugify('paris,france'));
           $this->assertEquals('sensio', Jobeet::slugify(' sensio'));
           $this->assertEquals('sensio', Jobeet::slugify('sensio '));
       }
   }

このテストのみを実行するには、次のコマンドで行います。
To run only this test, you can use the following command:

.. code-block:: bash

   $ phpunit -c app/ src/Ibw/JobeetBundle/Tests/Utils/JobeetTest

すべてが正常に動作する必要があり、あなたは次のような結果を得るでしょう::
As everything should work fine, you should get the following result::

   PHPUnit 3.7.22 by Sebastian Bergmann.

   Configuration read from /var/www/jobeet/app/phpunit.xml.dist

   .
   Time: 0 seconds, Memory: 8.00Mb

   OK (1 test, 6 assertions)

アサーションの完全なリストは、 PHPUnit のドキュメントで確認してください。
For a full list of assertions, you can check the PHPUnit documentation.

新機能のテストを追加
----------------

| 空の文字列のためのスラグは空の文字列です。それをテストすることはでき、動作もします。しかし、 URL に空の文字列は、よい考えではありません。
| 空の文字列の場合は「n-a」の文字列を返すように slugify() メソッドを変更してみましょう。
| 先にテストを書いてからメソッドの更新、または、その他の修正することができます。
| それは本当に好みの問題ですが、先にテストを書くことは、コードが計画したものを実際に実装しているという自信を与えてくれます。
The slug for an empty string is an empty string. You can test it, it will work.
But an empty string in a URL is not that a great idea.
Let’s change the slugify() method so that it returns the “n-a” string in case of an empty string.
You can write the test first, then update the method, or the other way around.
It is really a matter of taste, but writing the test first gives you the confidence that your code actually implements what you planned:

src/Ibw/JobeetBundle/Tests/Utils/JobeetTest.php

.. code-block:: php

   // ...

   $this->assertEquals('n-a', Jobeet::slugify(''));

   // ...

現状では、テストを再実行すると、failure が発生します。::
Now, if we run the test again, we will have a failure::

   PHPUnit 3.7.22 by Sebastian Bergmann.

   Configuration read from /var/www/jobeet/app/phpunit.xml.dist

   F

   Time: 0 seconds, Memory: 8.25Mb

   There was 1 failure:

   1) Ibw\JobeetBundle\Tests\Utils\JobeetTest::testSlugify
   Failed asserting that two strings are equal.
   --- Expected
   +++ Actual
   @@ @@
   -'n-a'
   +''

   /var/www/jobeet/src/Ibw/JobeetBundle/Tests/Utils/JobeetTest.php:13

   FAILURES!
   Tests: 1, Assertions: 5, Failures: 1.


ここで、Jobeet::slugify メソッドを編集し、先頭に次の条件を追加します。
Now, edit the Jobeet::slugify method and add the following condition at the beginning:

src/Ibw/JobeetBundle/Utils/Jobeet.php

.. code-block:: php

   // ...

       static public function slugify($text)
       {
           if (empty($text)) {
               return 'n-a';
           }

           // ...
       }

このテストは期待どおり通過しなくてはいけません。緑のバーを楽しめるでしょう。
The test must now pass as expected, and you can enjoy the green bar.

バグによるテスト追加
--------------------

| その後、時間が経過し、ユーザーの1人から、「いくつかのジョブのリンク先が404エラーページになる」という奇妙なバグの報告を受けるとしましょう。​​
| いくつかの調査の後、あなたはいくつかの理由で、これらのジョブが空の会社名、役職、住所 のスラグを持っていることがわかります。
| それはどのような場合に起こりえますか？
| あなたは、データベース内のレコードに目を通すとカラムは間違いなく空ではありません。あなたはしばらくの間それについて考え、ビンゴ、原因を見つけます。
| 文字列が非ASCII文字のみで構成されている場合、slugify() メソッドは空の文字列に変換します。
| 原因を発見したのに満足して、Jobeetのクラスを編集してすぐに問題を解決しようとするのは、よい考えではありません。
| 最初に、テストを追加してみましょう：
Let’s say that time has passed and one of your users reports a weird bug: some job links point to a 404 error page.
After some investigation, you find that for some reason, these jobs have an empty company, position, or location slug.
How is it possible?
You look through the records in the database and the columns are definitely not empty.
You think about it for a while, and bingo, you find the cause.
When a string only contains non-ASCII characters, the slugify() method converts it to an empty string.
So happy to have found the cause, you open the Jobeet class and fix the problem right away.
That’s a bad idea. First, let’s add a test:

src/Ibw/JobeetBundle/Tests/Utils/JobeetTest.php

.. code-block:: php

   $this->assertEquals('n-a', Jobeet::slugify(' - '));

テストに合格しないことを確認した後、Jobeetのクラスを編集して、空の文字列チェックをメソッドの最後に移動します。
After checking that the test does not pass, edit the Jobeet class and move the empty string check to the end of the method:

src/Ibw/JobeetBundle/Utils/Jobeet.php

.. code-block:: php

   static public function slugify($text)
   {
       // ...

       if (empty($text))
       {
           return 'n-a';
       }

       return $text;
   }

| 他のテスト同様に、新しいテストも通りました。 slugify() は、100％ のコード網羅率にもかかわらず、バグがありました。
| テストを書くときには、すべてのエッジケースを考えることはできません。それは大丈夫です。
| しかし、あなたが1つを発見したときに、コードを修正する前にテストを書く必要があります。
| また、あなたのコードは時間かけて良くなることを意味します。それは常に良いことです。
The new test now passes, as do all the other ones. The slugify() had a bug despite our 100% coverage.
You cannot think about all edge cases when writing tests, and that’s fine.
But when you discover one, you need to write a test for it before fixing your code.
It also means that your code will get better over time, which is always a good thing.

よりよい slugify メソッドに向けて
---------------------------------

| おそらく、あなたは Symfony がフランス人によって作られていることを知っているでしょう。
| そこで、「アクセント」が含まれているフランス語の単語でテストを追加してみましょう：
You probably know that symfony has been created by French people,
so let’s add a test with a French word that contains an “accent”:

src/Ibw/JobeetBundle/Tests/Utils/JobeetTest.php

.. code-block:: php

   $this->assertEquals('developpeur-web', Jobeet::slugify('Développeur Web'));

| テストは失敗しなければいけません。é を e で置き換える代わりに、slugify（） メソッドは、ダッシュ（ - ） で置き換えました。
| 音訳と呼ばれる厳しい問題です。
| iconv ライブラリがインストールされている場合は、うまくいけば、私たちのために仕事を行います。
| 以下で slugify メソッドのコードを置き換えます。
The test must fail. Instead of replacing é by e, the slugify() method has replaced it by a dash (-).
That’s a tough problem, called transliteration. Hopefully, if you have iconv Library installed, it will do the job for us.
Replace the code of the slugify method with the following:

src/Ibw/JobeetBundle/Utils/Jobeet.php

.. code-block:: php

   static public function slugify($text)
   {
       // replace non letter or digits by -
       $text = preg_replace('#[^\\pL\d]+#u', '-', $text);

       // trim
       $text = trim($text, '-');

       // transliterate
       if (function_exists('iconv'))
       {
           $text = iconv('utf-8', 'us-ascii//TRANSLIT', $text);
       }

       // lowercase
       $text = strtolower($text);

       // remove unwanted characters
       $text = preg_replace('#[^-\w]+#', '', $text);

       if (empty($text))
       {
           return 'n-a';
       }

       return $text;
   }

Symfony のデフォルトのエンコーディングである UTF-8 エンコーディングですべての PHP ファイルを保存することを忘れないでください。
また、 UTF-8 は iconv によって翻訳に使用されるます。
また、 iconv が利用可能である場合にのみ、テストファイルを変更します。
Remember to save all your PHP files with the UTF-8 encoding, as this is the default Symfony encoding, and the one used by iconv to do the transliteration.
Also change the test file to run the test only if iconv is available:

src/Ibw/JobeetBundle/Tests/Utils/JobeetTest.php

.. code-block:: php

   if (function_exists('iconv')) {
       $this->assertEquals('developpeur-web', Jobeet::slugify('Développeur Web'));
   }

コード網羅率
------------

| テストを書くときに、コードの一部を忘れがちです。
| 新しい機能を追加したり、コード網羅率の統計を確認したい場合に必要があるのは --coverage-HTML オプションを使用してコード網羅率をチェックすることです。
When you write tests, it is easy to forget a portion of the code.
If you add a new feature or you just want to verify your code coverage statistics, all you need to do is to check the code coverage by using the --coverage-html option:

.. code-block:: bash

   $ phpunit --coverage-html=web/cov/ -c app/

ブラウザで生成されたページ（ http://jobeet.local/cov/index.html ）を開いて確認してください。
Check the code coverage by opening the generated http://jobeet.local/cov/index.html page in a browser.

.. note::

   コードカバレッジは XDebug が有効になっており、依存するすべてのものがインストールされている場合にのみ動きます。
   The code coverage only works if you have XDebug enabled and all dependencies installed.

   .. code-block:: bash

      $ sudo apt-get install php5-xdebug

cov/index.html のは次のようになります。
Your cov/index.html should look like this:

.. image:: /images/day-8-code-coverage1.jpg

インジケーターのテストユニットがすべて実施されているということは、すべての行が実施されたという意味で、それだけで、すべてのエッジケースがテストされているわけではないことに注意してください。
Keep in mind that when this indicates that your code is fully unit tested, it just means that each line has been executed, not that all the edge cases have been tested.

Doctrine の単体テスト
---------------------

| データベース接続を必要とする Doctrine モデルクラスのユニットテストは、もう少し複雑です。
| あなたはすでに、あなたの開発のために使用するものを持っているが、それはテスト専用のデータベースを作成するには良い習慣です。
| このチュートリアルの初めに、私たちはアプリケーションの設定を変更する方法として環境を導入しました。
| デフォルトでは、すべての Symfony のテストは test 環境で実行されるので、 test 環境用に異なるデータベースを設定しましょう。
| app/config ディレクトリに移動し、 parameters.yml ファイルをコピーして parameters_test.yml を作成します。
| parameters_test.yml を編集し、 jobeet_test ためにデータベースの名前を変更します。
| これをインポートするために、 config_test.yml ファイルに追加する必要があります。
Unit testing a Doctrine model class is a bit more complex as it requires a database connection.
You already have the one you use for your development, but it is a good habit to create a dedicated database for tests.
At the beginning of this tutorial, we introduced the environments as a way to vary an application’s settings. By default, all symfony tests are run in the test environment, so let’s configure a different database for the test environment:
Go to your app/config directory and create a copy of parameters.yml file, called parameters_test.yml.
Open parameters_test.yml and change the name of your database to jobeet_test.
For this to be imported, we have to add it in the config_test.yml file :

app/config/config_test.yml

.. code-block:: yaml

   imports:
       - { resource: config_dev.yml }
       - { resource: parameters_test.yml }
   // ...

Job エンティティのテスト
------------------------

| まず、 Tests/Entity 内に JobTest.php ファイルを作成する必要があります。
| ``setUp`` 関数は、テストを実行するたびにデータベースを操作します。
| 最初に、現在のデータベースをドロップし、再作成し、フィクスチャーからデータをロードします。
| test 環境用に作成したデータベースに、テストを実行する際に、同じ初期データを持っていることは役に立つでしょう。
First, we need to create the JobTest.php file in the Tests/Entity folder.
The setUp function will manipulate your database each time you will run the test.
At first, it will drop your current database,
then it will re-create it and load data from fixtures in it.
This will help you have the same initial data in the database you created for the test environment before running the tests.

src/Ibw/JobeetBundle/Tests/Entity/JobTest.php

.. code-block:: php

   namespace Ibw\JobeetBundle\Entity;

   use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;
   use Ibw\JobeetBundle\Utils\Jobeet as Jobeet;
   use Symfony\Bundle\FrameworkBundle\Console\Application;
   use Symfony\Component\Console\Output\NullOutput;
   use Symfony\Component\Console\Input\ArrayInput;
   use Doctrine\Bundle\DoctrineBundle\Command\DropDatabaseDoctrineCommand;
   use Doctrine\Bundle\DoctrineBundle\Command\CreateDatabaseDoctrineCommand;
   use Doctrine\Bundle\DoctrineBundle\Command\Proxy\CreateSchemaDoctrineCommand;

   class JobTest extends WebTestCase
   {
       private $em;
       private $application;

       public function setUp()
       {
           static::$kernel = static::createKernel();
           static::$kernel->boot();

           $this->application = new Application(static::$kernel);

           // drop the database
           $command = new DropDatabaseDoctrineCommand();
           $this->application->add($command);
           $input = new ArrayInput(array(
               'command' => 'doctrine:database:drop',
               '--force' => true
           ));
           $command->run($input, new NullOutput());

           // we have to close the connection after dropping the database so we don't get "No database selected" error
           $connection = $this->application->getKernel()->getContainer()->get('doctrine')->getConnection();
           if ($connection->isConnected()) {
               $connection->close();
           }

           // create the database
           $command = new CreateDatabaseDoctrineCommand();
           $this->application->add($command);
           $input = new ArrayInput(array(
               'command' => 'doctrine:database:create',
           ));
           $command->run($input, new NullOutput());

           // create schema
           $command = new CreateSchemaDoctrineCommand();
           $this->application->add($command);
           $input = new ArrayInput(array(
               'command' => 'doctrine:schema:create',
           ));
           $command->run($input, new NullOutput());

           // get the Entity Manager
           $this->em = static::$kernel->getContainer()
               ->get('doctrine')
               ->getManager();

           // load fixtures
           $client = static::createClient();
           $loader = new \Symfony\Bridge\Doctrine\DataFixtures\ContainerAwareLoader($client->getContainer());
           $loader->loadFromDirectory(static::$kernel->locateResource('@IbwJobeetBundle/DataFixtures/ORM'));
           $purger = new \Doctrine\Common\DataFixtures\Purger\ORMPurger($this->em);
           $executor = new \Doctrine\Common\DataFixtures\Executor\ORMExecutor($this->em, $purger);
           $executor->execute($loader->getFixtures());
       }

       public function testGetCompanySlug()
       {
           $job = $this->em->createQuery('SELECT j FROM IbwJobeetBundle:Job j ')
               ->setMaxResults(1)
               ->getSingleResult();

           $this->assertEquals($job->getCompanySlug(), Jobeet::slugify($job->getCompany()));
       }

       public function testGetPositionSlug()
       {
           $job = $this->em->createQuery('SELECT j FROM IbwJobeetBundle:Job j ')
               ->setMaxResults(1)
               ->getSingleResult();

           $this->assertEquals($job->getPositionSlug(), Jobeet::slugify($job->getPosition()));
       }

       public function testGetLocationSlug()
       {
           $job = $this->em->createQuery('SELECT j FROM IbwJobeetBundle:Job j ')
               ->setMaxResults(1)
               ->getSingleResult();

           $this->assertEquals($job->getLocationSlug(), Jobeet::slugify($job->getLocation()));
       }

       public function testSetExpiresAtValue()
       {
           $job = new Job();
           $job->setExpiresAtValue();

           $this->assertEquals(time() + 86400 * 30, $job->getExpiresAt()->format('U'));
       }

       protected function tearDown()
       {
           parent::tearDown();
           $this->em->close();
       }
   }

レポジトリクラスのテスト
------------------------

さて、前の日に作成した関数が正しい値を返すかどうかを確認するために、 JobRepository クラスのいくつかのテストを書いてみましょう。
Now, let’s write some tests for the JobRepository class, to see if the functions we created in the previous days are returning the right values:

src/Ibw/JobeetBundle/Tests/Repository/JobRepositoryTest.php

.. code-block:: php

   namespace Ibw\JobeetBundle\Tests\Repository;

   use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;
   use Symfony\Bundle\FrameworkBundle\Console\Application;
   use Symfony\Component\Console\Output\NullOutput;
   use Symfony\Component\Console\Input\ArrayInput;
   use Doctrine\Bundle\DoctrineBundle\Command\DropDatabaseDoctrineCommand;
   use Doctrine\Bundle\DoctrineBundle\Command\CreateDatabaseDoctrineCommand;
   use Doctrine\Bundle\DoctrineBundle\Command\Proxy\CreateSchemaDoctrineCommand;

   class JobRepositoryTest extends WebTestCase
   {
       private $em;
       private $application;

       public function setUp()
       {
           static::$kernel = static::createKernel();
           static::$kernel->boot();

           $this->application = new Application(static::$kernel);

           // drop the database
           $command = new DropDatabaseDoctrineCommand();
           $this->application->add($command);
           $input = new ArrayInput(array(
               'command' => 'doctrine:database:drop',
               '--force' => true
           ));
           $command->run($input, new NullOutput());

           // we have to close the connection after dropping the database so we don't get "No database selected" error
           $connection = $this->application->getKernel()->getContainer()->get('doctrine')->getConnection();
           if ($connection->isConnected()) {
               $connection->close();
           }

           // create the database
           $command = new CreateDatabaseDoctrineCommand();
           $this->application->add($command);
           $input = new ArrayInput(array(
               'command' => 'doctrine:database:create',
           ));
           $command->run($input, new NullOutput());

           // create schema
           $command = new CreateSchemaDoctrineCommand();
           $this->application->add($command);
           $input = new ArrayInput(array(
               'command' => 'doctrine:schema:create',
           ));
           $command->run($input, new NullOutput());

           // get the Entity Manager
           $this->em = static::$kernel->getContainer()
               ->get('doctrine')
               ->getManager();

           // load fixtures
           $client = static::createClient();
           $loader = new \Symfony\Bridge\Doctrine\DataFixtures\ContainerAwareLoader($client->getContainer());
           $loader->loadFromDirectory(static::$kernel->locateResource('@IbwJobeetBundle/DataFixtures/ORM'));
           $purger = new \Doctrine\Common\DataFixtures\Purger\ORMPurger($this->em);
           $executor = new \Doctrine\Common\DataFixtures\Executor\ORMExecutor($this->em, $purger);
           $executor->execute($loader->getFixtures());
       }

       public function testCountActiveJobs()
       {
           $query = $this->em->createQuery('SELECT c FROM IbwJobeetBundle:Category c');
           $categories = $query->getResult();

           foreach($categories as $category) {
               $query = $this->em->createQuery('SELECT COUNT(j.id) FROM IbwJobeetBundle:Job j WHERE j.category = :category AND j.expires_at > :date');
               $query->setParameter('category', $category->getId());
               $query->setParameter('date', date('Y-m-d H:i:s', time()));
               $jobs_db = $query->getSingleScalarResult();

               $jobs_rep = $this->em->getRepository('IbwJobeetBundle:Job')->countActiveJobs($category->getId());
               // This test will verify if the value returned by the countActiveJobs() function
               // coincides with the number of active jobs for a given category from the database
               $this->assertEquals($jobs_rep, $jobs_db);
           }
       }

       public function testGetActiveJobs()
       {
           $query = $this->em->createQuery('SELECT c from IbwJobeetBundle:Category c');
           $categories = $query->getResult();

           foreach ($categories as $category) {
               $query = $this->em->createQuery('SELECT COUNT(j.id) from IbwJobeetBundle:Job j WHERE j.expires_at > :date AND j.category = :category');
               $query->setParameter('date', date('Y-m-d H:i:s', time()));
               $query->setParameter('category', $category->getId());
               $jobs_db = $query->getSingleScalarResult();

               $jobs_rep = $this->em->getRepository('IbwJobeetBundle:Job')->getActiveJobs($category->getId(), null, null);
               // This test tells if the number of active jobs for a given category from
               // the database is the same as the value returned by the function
               $this->assertEquals($jobs_db, count($jobs_rep));
           }
       }

       public function testGetActiveJob()
       {
           $query = $this->em->createQuery('SELECT j FROM IbwJobeetBundle:Job j WHERE j.expires_at > :date');
           $query->setParameter('date', date('Y-m-d H:i:s', time()));
           $query->setMaxResults(1);
           $job_db = $query->getSingleResult();

           $job_rep = $this->em->getRepository('IbwJobeetBundle:Job')->getActiveJob($job_db->getId());
           // If the job is active, the getActiveJob() method should return a non-null value
           $this->assertNotNull($job_rep);

           $query = $this->em->createQuery('SELECT j FROM IbwJobeetBundle:Job j WHERE j.expires_at < :date');         $query->setParameter('date', date('Y-m-d H:i:s', time()));
           $query->setMaxResults(1);
           $job_expired = $query->getSingleResult();

           $job_rep = $this->em->getRepository('IbwJobeetBundle:Job')->getActiveJob($job_expired->getId());
           // If the job is expired, the getActiveJob() method should return a null value
           $this->assertNull($job_rep);
       }

       protected function tearDown()
       {
           parent::tearDown();
           $this->em->close();
       }
   }

CategoryRepository クラスに同じことを行います。
We will do the same thing for CategoryRepository class:

src/Ibw/JobeetBundle/Tests/Repository/CategoryRepositoryTest.php

.. code-block:: php

   namespace Ibw\JobeetBundle\Tests\Repository;

   use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;
   use Symfony\Bundle\FrameworkBundle\Console\Application;
   use Symfony\Component\Console\Output\NullOutput;
   use Symfony\Component\Console\Input\ArrayInput;
   use Doctrine\Bundle\DoctrineBundle\Command\DropDatabaseDoctrineCommand;
   use Doctrine\Bundle\DoctrineBundle\Command\CreateDatabaseDoctrineCommand;
   use Doctrine\Bundle\DoctrineBundle\Command\Proxy\CreateSchemaDoctrineCommand;

   class CategoryRepositoryTest extends WebTestCase
   {
       private $em;
       private $application;

       public function setUp()
       {
           static::$kernel = static::createKernel();
           static::$kernel->boot();

           $this->application = new Application(static::$kernel);

           // drop the database
           $command = new DropDatabaseDoctrineCommand();
           $this->application->add($command);
           $input = new ArrayInput(array(
               'command' => 'doctrine:database:drop',
               '--force' => true
           ));
           $command->run($input, new NullOutput());

           // we have to close the connection after dropping the database so we don't get "No database selected" error
           $connection = $this->application->getKernel()->getContainer()->get('doctrine')->getConnection();
           if ($connection->isConnected()) {
               $connection->close();
           }

           // create the database
           $command = new CreateDatabaseDoctrineCommand();
           $this->application->add($command);
           $input = new ArrayInput(array(
               'command' => 'doctrine:database:create',
           ));
           $command->run($input, new NullOutput());

           // create schema
           $command = new CreateSchemaDoctrineCommand();
           $this->application->add($command);
           $input = new ArrayInput(array(
               'command' => 'doctrine:schema:create',
           ));
           $command->run($input, new NullOutput());

           // get the Entity Manager
           $this->em = static::$kernel->getContainer()
               ->get('doctrine')
               ->getManager();

           // load fixtures
           $client = static::createClient();
           $loader = new \Symfony\Bridge\Doctrine\DataFixtures\ContainerAwareLoader($client->getContainer());
           $loader->loadFromDirectory(static::$kernel->locateResource('@IbwJobeetBundle/DataFixtures/ORM'));
           $purger = new \Doctrine\Common\DataFixtures\Purger\ORMPurger($this->em);
           $executor = new \Doctrine\Common\DataFixtures\Executor\ORMExecutor($this->em, $purger);
           $executor->execute($loader->getFixtures());
       }

       public function testGetWithJobs()
       {
           $query = $this->em->createQuery('SELECT c FROM IbwJobeetBundle:Category c LEFT JOIN c.jobs j WHERE j.expires_at > :date');
           $query->setParameter('date', date('Y-m-d H:i:s', time()));
           $categories_db = $query->getResult();

           $categories_rep = $this->em->getRepository('IbwJobeetBundle:Category')->getWithJobs();
           // This test verifies if the number of categories having active jobs, returned
           // by the getWithJobs() function equals the number of categories having active jobs from database
           $this->assertEquals(count($categories_rep), count($categories_db));
       }

       protected function tearDown()
       {
           parent::tearDown();
           $this->em->close();
       }
   }

テストを書き終えた後、全体の関数のコード網羅率を生成するため、次のコマンドを実行します。
After you finish writing the tests, run them with the following command, in order to generate the code coverage percent for the whole functions :

.. code-block:: bash

   $ phpunit --coverage-html=web/cov/ -c app src/Ibw/JobeetBundle/Tests/Repository/

ブラウザでページ( http://jobeet.local/cov/Repository.html )を開くと、リポジトリのテストのコードカバレッジが100% 完了になっていないことがわかります。
Now, if you go to http://jobeet.local/cov/Repository.html you will see that the code coverage for Repository Tests is not 100% complete.

.. image:: /images/Day-8-coverage-not-complete.jpg

| それでは、100% のコード網羅率を達成するために JobRepository ためのいくつかのテストを追加してみましょう。
| 今のところ、データベースにはアクティブなジョブを持たない二つのカテゴリと、一つだけアクティブなジョブを持つ一つのカテゴリを持ちます。
| $max と $offset パラメーターをテストする際に、以下のテストでカテゴリに少なくとも3つのアクティブなジョブをテストするのはそのためです。
| そのために、 testGetActiveJobs() にて、 foreach 文の内部にこれを追加します。
Let’s add some tests for the JobRepository to achieve 100% code coverage.
At the moment, in our database, we have two job categories having 0 active jobs and one job category having just one active job.
That why, when we will test the $max and $offset parameters, we will run the following tests just on the categories with at least 3 active jobs.
In order to do that, add this inside your foreach statement, from your testGetActiveJobs() function:

src/Ibw/JobeetBundle/Tests/Repository/JobRepositoryTest.php

.. code-block:: php

   // ...
   foreach ($categories as $category) {
       // ...

       // 少なくとも3つのアクティブなジョブが選択したカテゴリにはあり、
       // getActiveJobs() メソッドを limit と offset パラメーターを利用してテストし、
       // 100% のコード網羅率にします。
       // If there are at least 3 active jobs in the selected category, we will
       // test the getActiveJobs() method using the limit and offset parameters too
       // to get 100% code coverage
       if($jobs_db > 2 ) {
           $jobs_rep = $this->em->getRepository('IbwJobeetBundle:Job')->getActiveJobs($category->getId(), 2);
           // This test tells if the number of returned active jobs is the one $max parameter requires
           $this->assertEquals(2, count($jobs_rep));

           $jobs_rep = $this->em->getRepository('IbwJobeetBundle:Job')->getActiveJobs($category->getId(), 2, 1);
           // We set the limit to 2 results, starting from the second job and test if the result is as expected
           $this->assertEquals(2, count($jobs_rep));
       }
   }
   // ...

再びコードカバレッジコマンドを実行します。
Run the code coverage command again :

.. code-block:: bash

   $ phpunit --coverage-html=web/cov/ -c app src/Ibw/JobeetBundle/Tests/Repository/

ここで、コード網羅率を確認するすると、 100% が表示されます。
This time, if you check your code coverage, you will see that it 100% complete.

.. image:: /images/Day-8-coverage-complete.jpg

これで今日はすべてです！明日は、機能テストについてお話します。
That’s all for today! See you tomorrow, when we will talk about functional tests.

.. include:: common/license.rst.inc
