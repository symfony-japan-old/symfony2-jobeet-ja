Day 14: Feeds
=============

.. include:: common/original.rst.inc

If you are looking for a job, you will probably want to be informed as soon as a new job is posted. Because it is not very convenient to check the website every other hour, we will add several job feeds here to keep our Jobeet users up-to-date.

Template Formats
----------------

Templates are a generic way to render content in any format. And while in most cases you’ll use templates to render HTML content, a template can just as easily generate JavaScript, CSS, XML or any other format.
For example, the same “resource” is often rendered in several different formats. To render an article index page in XML, simply include the format in the template name:

* XML template name: AcmeArticleBundle:Article:index.xml.twig
* XML template filename: index.xml.twig

In reality, this is nothing more than a naming convention and the template isn’t actually rendered differently based on its format.
In many cases, you may want to allow a single controller to render multiple different formats based on the “request format”. For that reason, a common pattern is to do the following:

.. code-block:: php

   public function indexAction()
   {
       $format = $this->getRequest()->getRequestFormat();

       return $this->render('AcmeBlogBundle:Blog:index.'.$format.'.twig');
   }

The getRequestFormat on the Request object defaults to html, but can return any other format based on the format requested by the user. The request format is most often managed by the routing, where a route can be configured so that /contact sets the request format to html while /contact.xml sets the format to xml.
To create links that include the format parameter, include a _format key in the parameter hash:

.. code-block:: html+jinja

   <a href="{{ path('article_show', {'id': 123, '_format': 'pdf'}) }}">
       PDF Version
   </a>

Feeds
-----

LATEST JOBS FEED
~~~~~~~~~~~~~~~~

Supporting different formats is as easy as creating different templates. To create an Atom feed for the latest jobs, create an index.atom.twig template:

src/Ibw/JobeetBundle/Resources/views/Job/index.atom.twig

.. code-block:: xml

   <?xml version="1.0" encoding="utf-8"?>
   <feed xmlns="http://www.w3.org/2005/Atom">
       <title>Jobeet</title>
       <subtitle>Latest Jobs</subtitle>
       <link href="" rel="self"/>
       <link href=""/>
       <updated></updated>
       <author><name>Jobeet</name></author>
       <id>Unique Id</id>

       <entry>
           <title>Job title</title>
           <link href="" />
           <id>Unique id</id>
           <updated></updated>
           <summary>Job description</summary>
           <author><name>Company</name></author>
       </entry>
   </feed>

In the Jobeet footer, update the link to the feed:

src/Ibw/JobeetBundle/Resources/views/layout.html.twig

.. code-block:: html+jinja

   <!-- ... -->

   <li class="feed"><a href="{{ path('ibw_job', {'_format': 'atom'}) }}">Full feed</a></li>

   <!-- ... -->

Add a <link> tag in the head section of the layout to allow automatic discover by the browser of our feed:

src/Ibw/JobeetBundle/Resources/views/layout.html.twig

.. code-block:: html+jinja

   <!-- ... -->

   <link rel="alternate" type="application/atom+xml" title="Latest Jobs" href="{{ url('ibw_job', {'_format': 'atom'}) }}" />

   <!-- ... -->

In the JobController change the indexAction to render the template according to the _format:

.. code-block:: php

   src/Ibw/JobeetBundle/Controller/JobController.php

   // ...

   $format = $this->getRequest()->getRequestFormat();

   return $this->render('IbwJobeetBundle:Job:index.'.$format.'.twig', array(
       'categories' => $categories
   ));

   // ...

Replace the Atom template header with the following code:

src/Ibw/JobeetBundle/Resources/views/Job/index.atom.twig

.. code-block:: jinja

   <!-- ... -->

       <title>Jobeet</title>
       <subtitle>Latest Jobs</subtitle>
       <link href="{{ url('ibw_job', {'_format': 'atom'}) }}" rel="self"/>
       <link href="{{ url('ibw_jobeet_homepage') }}"/>
       <updated>{{ lastUpdated }}</updated>
       <author><name>Jobeet</name></author>
       <id>{{ feedId }}</id>

   <!-- ... -->

From the JobController (index action) we have to send the lastUpdated and feedId to the template:

src/Ibw/JobeetBundle/Controller/JobController.php

.. code-block:: php

   // ...

   $latestJob = $em->getRepository('IbwJobeetBundle:Job')->getLatestPost();

   if($latestJob) {
      $lastUpdated = $latestJob->getCreatedAt()->format(DATE_ATOM);
   } else {
      $lastUpdated = new \DateTime();
      $lastUpdated = $lastUpdated->format(DATE_ATOM);
   }

   $format = $this->getRequest()->getRequestFormat();
   return $this->render('IbwJobeetBundle:Job:index.'.$format.'.twig', array(
         'categories' => $categories,
         'lastUpdated' => $lastUpdated,
         'feedId' => sha1($this->get('router')->generate('ibw_job', array('_format'=> 'atom'), true)),
   ));
   // ...

To get the date of the latest post, we have to create the getLatestPost() method in the JobRepository:

src/Ibw/JobeetBundle/Repository/JobRepository.php

.. code-block:: php

   // ...

   public function getLatestPost($category_id = null)
   {
     $query = $this->createQueryBuilder('j')
         ->where('j.expires_at > :date')
         ->setParameter('date', date('Y-m-d H:i:s', time()))
         ->andWhere('j.is_activated = :activated')
         ->setParameter('activated', 1)
         ->orderBy('j.expires_at', 'DESC')
         ->setMaxResults(1);

     if($category_id) {
         $query->andWhere('j.category = :category_id')
             ->setParameter('category_id', $category_id);
     }

     try{
         $job = $query->getQuery()->getSingleResult();
     } catch(\Doctrine\Orm\NoResultException $e){
         $job = null;
     }

     return $job;
   }
   // ...

The feed entries can be generated with the following code:

src/Ibw/JobeetBundle/Resources/views/Job/index.atom.twig

.. code-block:: html+jinja

   {% for category in categories %}
       {% for entity in category.activejobs %}
           <entry>
               <title>{{ entity.position }} ({{ entity.location }})</title>
               <link href="{{ url('ibw_job_show', { 'id': entity.id, 'company': entity.companyslug, 'location': entity.locationslug, 'position': entity.positionslug }) }}" />
               <id>{{ entity.id }}</id>
               <updated>{{ entity.createdAt.format(constant('DATE_ATOM')) }}</updated>
               <summary type="xhtml">
                   <div xmlns="http://www.w3.org/1999/xhtml">
                       {% if entity.logo %}
                           <div>
                               <a href="{{ entity.url }}">
                                   <img src="http://{{ app.request.host }}/uploads/jobs/{{ entity.logo }}" alt="{{ entity.company }} logo" />
                               </a>
                           </div>
                       {% endif %}
                       <div>
                           {{ entity.description|nl2br }}
                       </div>
                       <h4>How to apply?</h4>
                       <p>{{ entity.howtoapply }}</p>
                   </div>
               </summary>
               <author><name>{{ entity.company }}</name></author>
           </entry>
       {% endfor %}
   {% endfor %}

LATEST JOBS IN A CATEGORY FEED
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

One of the goals of Jobeet is to help people find more targeted jobs. So, we need to provide a feed for each category.
First, let’s update the links to category feeds in the templates:

src/Ibw/JobeetBundle/Resources/views/Job/index.html.twig

.. code-block:: html+jinja

   <div class="feed">
       <a href="{{ path('IbwJobeetBundle_category', { 'slug': category.slug, '_format': 'atom' }) }}">Feed</a>
   </div>

src/Ibw/JobeetBundle/Resources/views/Category/show.html.twig

.. code-block:: html+jinja

   <div class="feed">
       <a href="{{ path('IbwJobeetBundle_category', { 'slug': category.slug, '_format': 'atom' }) }}">Feed</a>
   </div>

Update the CategoryController showAction to render the corresponding template:

src/Ibw/JobeetBundle/Controller/CategoryController.php

.. code-block:: php

   // ...
   public function showAction($slug, $page)
   {
     $em = $this->getDoctrine()->getManager();

     $category = $em->getRepository('IbwJobeetBundle:Category')->findOneBySlug($slug);

     if (!$category) {
         throw $this->createNotFoundException('Unable to find Category entity.');
     }

     $latestJob = $em->getRepository('IbwJobeetBundle:Job')->getLatestPost($category->getId());

     if($latestJob) {
         $lastUpdated = $latestJob->getCreatedAt()->format(DATE_ATOM);
     } else {
         $lastUpdated = new \DateTime();
         $lastUpdated = $lastUpdated->format(DATE_ATOM);
     }

     $total_jobs = $em->getRepository('IbwJobeetBundle:Job')->countActiveJobs($category->getId());
     $jobs_per_page = $this->container->getParameter('max_jobs_on_category');
     $last_page = ceil($total_jobs / $jobs_per_page);
     $previous_page = $page > 1 ? $page - 1 : 1;
     $next_page = $page < $last_page ? $page + 1 : $last_page;
     $category->setActiveJobs($em->getRepository('IbwJobeetBundle:Job')->getActiveJobs($category->getId(), $jobs_per_page, ($page - 1) * $jobs_per_page));

     $format = $this->getRequest()->getRequestFormat();

     return $this->render('IbwJobeetBundle:Category:show.' . $format . '.twig', array(
         'category' => $category,
         'last_page' => $last_page,
         'previous_page' => $previous_page,
         'current_page' => $page,
         'next_page' => $next_page,
         'total_jobs' => $total_jobs,
         'feedId' => sha1($this->get('router')->generate('IbwJobeetBundle_category', array('slug' => $category->getSlug(), 'format' => 'atom'), true)),
         'lastUpdated' => $lastUpdated
     ));
   }

Eventually, create the show.atom.twig template:

src/Ibw/JobeetBundle/Resources/views/Category/show.atom.twig

.. code-block:: html+jinja

   <?xml version="1.0" encoding="utf-8"?>
   <feed xmlns="http://www.w3.org/2005/Atom">
       <title>Jobeet ({{ category.name }})</title>
       <subtitle>Latest Jobs</subtitle>
       <link href="{{ url('IbwJobeetBundle_category', { 'slug': category.slug, '_format': 'atom' }) }}" rel="self" />
       <updated>{{ lastUpdated }}</updated>
       <author><name>Jobeet</name></author>
       <id>{{ feedId }}</id>

       {% for entity in category.activejobs %}
           <entry>
               <title>{{ entity.position }} ({{ entity.location }})</title>
               <link href="{{ url('ibw_job_show', { 'id': entity.id, 'company': entity.companyslug, 'location': entity.locationslug, 'position': entity.positionslug }) }}" />
               <id>{{ entity.id }}</id>
               <updated>{{ entity.createdAt.format(constant('DATE_ATOM')) }}</updated>
               <summary type="xhtml">
                   <div xmlns="http://www.w3.org/1999/xhtml">
                       {% if entity.logo %}
                           <div>
                               <a href="{{ entity.url }}">
                                   <img src="http://{{ app.request.host }}/uploads/jobs/{{ entity.logo }}" alt="{{ entity.company }} logo" />
                               </a>
                           </div>
                       {% endif %}
                       <div>
                           {{ entity.description|nl2br }}
                       </div>
                       <h4>How to apply?</h4>
                       <p>{{ entity.howtoapply }}</p>
                   </div>
               </summary>
               <author><name>{{ entity.company }}</name></author>
           </entry>
       {% endfor %}
   </feed>

.. include:: common/license.rst.inc