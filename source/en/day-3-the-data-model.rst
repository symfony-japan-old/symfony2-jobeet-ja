Day 3: The Data Model
=====================

.. include:: common/original.rst.inc

If you’re itching to open your text editor and lay down some PHP, you will be happy to know that today will get us into some development. We will define the Jobeet data model, use an ORM to interact with the database and build the first module of the application. But as Symfony does a lot of work for us, we will have a fully functional web module without writing too much PHP code.

The Relational Model
---------------

The user stories from the previous day describe the main objects of our project: jobs, affiliates, and categories. Here is the corresponding entity relationship diagram:

.. image:: /images/Day3-entity_diagram.png

In addition to the columns described in the stories, we have also added created_at and updated_at columns. We will configure Symfony to set their value automatically when an object is saved or updated.

The Database
------------

To store the jobs, affiliates and categories in the database, Symfony 2.3.2 uses Doctrine ORM. To define the database connection parameters, you have to edit the app/config/parameters.yml file (for this tutorial we will use MySQL):

app/config/parameters.yml

.. code-block:: yaml

   parameters:
       database_driver: pdo_mysql
       database_host: localhost
       database_port: null
       database_name: jobeet
       database_user: root
       database_password: password
       # ...

Now that Doctrine knows about your database, you can have it create the database for you by typing the following command in your terminal:

.. code-block:: bash

    $ php app/console doctrine:database:create

The Schema
----------

To tell Doctrine about our objects, we will create “metadata” files that will describe how our objects will be stored in the database. Now go to your code editor and create a directory named doctrine, inside src/Ibw/JobeetBundle/Resources/config directory. Doctrine will contain three files: Category.orm.yml, Job.orm.yml and Affiliate.orm.yml.

src/Ibw/JobeetBundle/Resources/config/doctrine/Category.orm.yml

.. code-block:: yaml

   Ibw\JobeetBundle\Entity\Category:
       type: entity
       table: category
       id:
           id:
               type: integer
               generator: { strategy: AUTO }
       fields:
           name:
               type: string
               length: 255
               unique: true
       oneToMany:
           jobs:
               targetEntity: Job
               mappedBy: category
       manyToMany:
           affiliates:
               targetEntity: Affiliate
               mappedBy: categories

src/Ibw/JobeetBundle/Resources/config/doctrine/Job.orm.yml

.. code-block:: yaml

   Ibw\JobeetBundle\Entity\Job:
       type: entity
       table: job
       id:
           id:
               type: integer
               generator: { strategy: AUTO }
       fields:
           type:
               type: string
               length: 255
               nullable: true
           company:
               type: string
               length: 255
           logo:
               type: string
               length: 255
               nullable: true
           url:
               type: string
               length: 255
               nullable: true
           position:
               type: string
               length: 255
           location:
               type: string
               length: 255
           description:
               type: text
           how_to_apply:
               type: text
           token:
               type: string
               length: 255
               unique: true
           is_public:
               type: boolean
               nullable: true
           is_activated:
               type: boolean
               nullable: true
           email:
               type: string
               length: 255
           expires_at:
               type: datetime
           created_at:
               type: datetime
           updated_at:
               type: datetime
               nullable: true
       manyToOne:
           category:
               targetEntity: Category
               inversedBy: jobs
               joinColumn:
                   name: category_id
                   referencedColumnName: id
       lifecycleCallbacks:
           prePersist: [ setCreatedAtValue ]
           preUpdate: [ setUpdatedAtValue ]

src/Ibw/JobeetBundle/Resources/config/doctrine/Affiliate.orm.yml

.. code-block:: yaml

   Ibw\JobeetBundle\Entity\Affiliate:
       type: entity
       table: affiliate
       id:
           id:
               type: integer
               generator: { strategy: AUTO }
       fields:
           url:
               type: string
               length: 255
           email:
               type: string
               length: 255
               unique: true
           token:
               type: string
               length: 255
           is_active:
               type: boolean
               nullable: true
           created_at:
               type: datetime
       manyToMany:
           categories:
               targetEntity: Category
               inversedBy: affiliates
               joinTable:
                   name: category_affiliate
                   joinColumns:
                       affiliate_id:
                           referencedColumnName: id
                   inverseJoinColumns:
                       category_id:
                           referencedColumnName: id
       lifecycleCallbacks:
           prePersist: [ setCreatedAtValue ]

The ORM
-------

Now Doctrine can generate the classes that define our objects for us with the command:

.. code-block:: bash

    $ php app/console doctrine:generate:entities IbwJobeetBundle

If you take a look into Entity directory from IbwJobeetBundle, you will find the newly generated classes in there: Category.php, Job.php and Affiliate.php. Open Job.php and set the created_at and updated_at values as below:

src/Ibw/JobeetBundle/Entity/Job.php

.. code-block:: php

   // ...

       /**
        * @ORM\PrePersist
        */
       public function setCreatedAtValue()
       {
           if(!$this->getCreatedAt()) {
               $this->created_at = new \DateTime();
           }
       }

       /**
        * @ORM\PreUpdate
        */
       public function setUpdatedAtValue()
       {
           $this->updated_at = new \DateTime();
       }

You will do the same for created_at value of the Affiliate class:

src/Ibw/JobeetBundle/Entity/Affiliate.php

.. code-block:: php

   // ...

       /**
        * @ORM\PrePersist
        */
       public function setCreatedAtValue()
       {
           $this->created_at = new \DateTime();
       }

   // ...

This will make Doctrine to set the created_at and updated_at values when saving or updating objects. This behaviour was defined in the Affiliate.orm.yml and Job.orm.yml files listed above.
We will also ask Doctrine to create our database tables with the command below:

.. code-block:: bash

    $ php app/console doctrine:schema:update --force

.. note::

   This task should only be used during the development. For a more robust method of systematically updating your production database, read about Doctrine migrations.

The tables have been created in the database but there is no data in them. For any web application, there are three types of data: initial data (this is needed for the application to work, in our case we will have some initial categories and an admin user), test data (needed for the application to be tested) and user data (created by users during the normal life of the application).
To populate the database with some initial data, we will use `DoctrineFixturesBundle`_. To setup this bundle, we have to follow the next steps:

1. Add the following to your composer.json file, in the require section:

.. code-block:: json

   // ...
       "require": {
           // ...
           "doctrine/doctrine-fixtures-bundle": "dev-master",
           "doctrine/data-fixtures": "dev-master"
       },

   // ...

2. Update the vendor libraries:

.. code-block:: bash

    $ php composer.phar update

3. Register the bundle DoctrineFixturesBundle in app/AppKernel.php:

app/AppKernel.php

.. code-block:: php

   // ...

   public function registerBundles()
   {
       $bundles = array(
           // ...
           new Doctrine\Bundle\FixturesBundle\DoctrineFixturesBundle()
       );

       // ...
   }

Now that everything is set up, we will create some new classes to load data in a new folder, named src/Ibw/JobeetBundle/DataFixtures/ORM, in our bundle:

src/Ibw/JobeetBundle/DataFixtures/ORM/LoadCategoryData.php

.. code-block:: php

   <?php
   namespace Ibw\JobeetBundle\DataFixtures\ORM;

   use Doctrine\Common\Persistence\ObjectManager;
   use Doctrine\Common\DataFixtures\AbstractFixture;
   use Doctrine\Common\DataFixtures\OrderedFixtureInterface;
   use Ibw\JobeetBundle\Entity\Category;

   class LoadCategoryData extends AbstractFixture implements OrderedFixtureInterface
   {
       public function load(ObjectManager $em)
       {
           $design = new Category();
           $design->setName('Design');

           $programming = new Category();
           $programming->setName('Programming');

           $manager = new Category();
           $manager->setName('Manager');

           $administrator = new Category();
           $administrator->setName('Administrator');

           $em->persist($design);
           $em->persist($programming);
           $em->persist($manager);
           $em->persist($administrator);
           $em->flush();

           $this->addReference('category-design', $design);
           $this->addReference('category-programming', $programming);
           $this->addReference('category-manager', $manager);
           $this->addReference('category-administrator', $administrator);
       }

       public function getOrder()
       {
           return 1; // the order in which fixtures will be loaded
       }
   }

src/Ibw/JobeetBundle/DataFixtures/ORM/LoadJobData.php

.. code-block:: php
   <?php
   namespace Ibw\JobeetBundle\DataFixtures\ORM;

   use Doctrine\Common\Persistence\ObjectManager;
   use Doctrine\Common\DataFixtures\AbstractFixture;
   use Doctrine\Common\DataFixtures\OrderedFixtureInterface;
   use Ibw\JobeetBundle\Entity\Job;

   class LoadJobData extends AbstractFixture implements OrderedFixtureInterface
   {
       public function load(ObjectManager $em)
       {
            $job_sensio_labs = new Job();
            $job_sensio_labs->setCategory($em->merge($this->getReference('category-programming')));
            $job_sensio_labs->setType('full-time');
            $job_sensio_labs->setCompany('Sensio Labs');
            $job_sensio_labs->setLogo('sensio-labs.gif');
            $job_sensio_labs->setUrl('http://www.sensiolabs.com/');
            $job_sensio_labs->setPosition('Web Developer');
            $job_sensio_labs->setLocation('Paris, France');
            $job_sensio_labs->setDescription('You\'ve already developed websites with symfony and you want to work with Open-Source technologies. You have a minimum of 3 years experience in web development with PHP or Java and you wish to participate to development of Web 2.0 sites using the best frameworks available.');
            $job_sensio_labs->setHowToApply('Send your resume to fabien.potencier [at] sensio.com');
            $job_sensio_labs->setIsPublic(true);
            $job_sensio_labs->setIsActivated(true);
            $job_sensio_labs->setToken('job_sensio_labs');
            $job_sensio_labs->setEmail('job@example.com');
            $job_sensio_labs->setExpiresAt(new \DateTime('+30 days'));
            $job_extreme_sensio = new Job();
            $job_extreme_sensio->setCategory($em->merge($this->getReference('category-design')));
            $job_extreme_sensio->setType('part-time');
            $job_extreme_sensio->setCompany('Extreme Sensio');
            $job_extreme_sensio->setLogo('extreme-sensio.gif');
            $job_extreme_sensio->setUrl('http://www.extreme-sensio.com/');
            $job_extreme_sensio->setPosition('Web Designer');
            $job_extreme_sensio->setLocation('Paris, France');
            $job_extreme_sensio->setDescription('Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in.');
            $job_extreme_sensio->setHowToApply('Send your resume to fabien.potencier [at] sensio.com');
            $job_extreme_sensio->setIsPublic(true);
            $job_extreme_sensio->setIsActivated(true);
            $job_extreme_sensio->setToken('job_extreme_sensio');
            $job_extreme_sensio->setEmail('job@example.com');
            $job_extreme_sensio->setExpiresAt(new \DateTime('+30 days'));

            $em->persist($job_sensio_labs);
            $em->persist($job_extreme_sensio);
            $em->flush();
       }

       public function getOrder()
       {
           return 2; // the order in which fixtures will be loaded
       }
   }

Once your fixtures have been written, you can load them via the command line by using thedoctrine:fixtures:load command:

.. code-block:: bash

    $ php app/console doctrine:fixtures:load

Now, if you check your database, you should see the data loaded into tables.

See it in the browser
---------------------

If you run the command below, it will create a new controller src/Ibw/JobeetBundle/Controllers/JobController.php with actions for listing, creating, editing and deleting jobs (and their corresponding templates, form and routes):

.. code-block:: bash

    $ php app/console doctrine:generate:crud --entity=IbwJobeetBundle:Job --route-prefix=ibw_job --with-write --format=yml

After running this command, you will need to do some configurations the prompter requires you to. So just select the default answers for them.
To view this in the browser, we must import the new routes that were created in src/Ibw/JobeetBundle/Resources/config/routing/job.yml into our bundle main routing file:

src/Ibw/JobeetBundle/Resources/config/routing.yml

.. code-block:: yaml

   IbwJobeetBundle_job:
           resource: "@IbwJobeetBundle/Resources/config/routing/job.yml"
           prefix:   /job
   # ...

We will also need to add a _toString() method to our Category class to be used by the category drop down from the edit job form:

src/Ibw/JobeetBundle/Entity/Category.php

.. code-block:: php

   // ...

   public function __toString()
   {
       return $this->getName() ? $this->getName() : "";
   }

   // ...

Clear the cache:

.. code-block:: bash

    $ php app/console cache:clear --env=dev
    $ php app/console cache:clear --env=prod

You can now test the job controller in a browser: http://jobeet.local/job/ or, in development environment, http://jobeet.local/app_dev.php/job/ .

.. image:: /images/Day-3-index_page.png

You can now create and edit jobs. Try to leave a required field blank, or try to enter invalid data. That’s right, Symfony has created basic validation rules by introspecting the database schema.
That’s all. Today, we have barely written PHP code but we have a working web module for the job model, ready to be tweaked and customized. Tomorrow, we will get familiar with the controller and the view. See you next time!

.. include:: common/license.rst.inc

.. _`DoctrineFixturesBundle`: http://symfony.com/doc/current/bundles/DoctrineFixturesBundle/index.html