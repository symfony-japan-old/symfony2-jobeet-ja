Day 18: AJAX
============

.. include:: common/original.rst.inc

Yesterday, we implemented a very powerful search engine for Jobeet, thanks to the Zend Lucene library. In the following lines, to enhance the responsiveness of the search engine, we will take advantage of AJAX to convert the search engine to a live one.
As the form should work with and without JavaScript enabled, the live search feature will be implemented using unobtrusive JavaScript. Using unobtrusive JavaScript also allows for a better separation of concerns in the client code between HTML, CSS, and the JavaScript behaviors.

Installing jQuery
-----------------

Go to the jQuery website, download the latest version, and put the .js file under src/Ibw/JobeetBundle/Resources/public/js/.
After putting the .js file in the js directory, run the following command to tell Symfony to make it available to the public:

.. code-block:: bash

   $ php app/console assets:install web --symlink

Including jQuery
----------------

As we will need jQuery on all pages, update the layout to include it:

src/Ibw/JobeetBundle/Resources/views/layout.html.twig

.. code-block:: html+jinja

   <!-- ... -->
       {% block javascripts %}
           <script type="text/javascript" src="{{ asset('bundles/ibwjobeet/js/jquery-2.0.3.min.js') }}"></script>
       {% endblock %}
   <!-- ... -->

Adding Behaviours
-----------------

Implementing a live search means that each time the user types a letter in the search box, a call to the server needs to be triggered; the server will then return the needed information to update some regions of the page without refreshing the whole page.
Instead of adding the behavior with an on*() HTML attributes, the main principle behind jQuery is to add behaviors to the DOM after the page is fully loaded. This way, if you disable JavaScript support in your browser, no behavior is registered, and the form still works as before.
The first step is to intercept whenever a user types a key in the search box:
Explaining code before implementing
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: javascript

   $('#search_keywords').keyup(function(key)
   {
       if (this.value.length >= 3 || this.value == '')
       {
           // do something
       }
   });

.. note::

   Don’t add the code for now, as we will modify it heavily. The final JavaScript code will be added to the layout in the next section.

Every time the user types a key, jQuery executes the anonymous function defined in the above code, but only if the user has typed more than 3 characters or if he removed everything from the input tag.
Making an AJAX call to the server is as simple as using the load() method on the DOM element:

Explaining code before implementing
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: javascript

   $('#search_keywords').keyup(function(key)
   {
       if (this.value.length >= 3 || this.value == '')
       {
           $('#jobs').load(
               $(this).parents('form').attr('action'), { query: this.value + '*' }
           );
       }
   });

To manage the AJAX Call, the same action as the “normal” one is called. The needed changes in the action will be done in the next section.
Last but not least, if JavaScript is enabled, we will want to remove the search button:

Explaining code before implementing
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: javascript

   $('.search input[type="submit"]').hide();

User Feedback
-------------

Whenever you make an AJAX call, the page won’t be updated right away. The browser will wait for the server response to come back before updating the page. In the meantime, you need to provide visual feedback to the user to inform him that something is going on.
A convention is to display a loader icon during the AJAX call. Update the layout to add the loader image and hide it by default:

src/Ibw/JobeetBundle/Resources/views/layout.html.twig

.. code-block:: html+jinja

   <!-- ... -->
       <div class="search">
           <h2>Ask for a job</h2>
           <form action="{{ path('ibw_job_search') }}" method="get">
               <input type="text" name="query" value="{{ app.request.get('query') }}" id="search_keywords" />
               <input type="submit" value="search" />
               <img id="loader" src="{{ asset('bundles/ibwjobeet/images/loader.gif') }}" style="vertical-align: middle; display: none" />
               <div class="help">
                   Enter some keywords (city, country, position, ...)
               </div>
           </form>
       </div>
   <!-- ... -->

Now that you have all the pieces needed to make the HTML work, create a search.js file that contains the JavaScript we have explained so far:

src/Ibw/JobeetBundle/Resources/public/js/search.js

.. code-block:: javascript

   $(document).ready(function()
   {
       $('.search input[type="submit"]').hide();

       $('#search_keywords').keyup(function(key)
       {
           if(this.value.length >= 3 || this.value == '') {
               $('#loader').show();
               $('#jobs').load(
                   $(this).parent('form').attr('action'),
                   { query: this.value ? this.value + '*' : this.value },
                   function() {
                       $('#loader').hide();
                   }
               );
           }
       });
   });

Run the command for telling Symfony to make it available to the public:

.. code-block:: bash

   $ php app/console assets:install web --symlink

You also need to update the layout to include this new file:

src/Ibw/JobeetBundle/Resources/views/layout.html.twig

.. code-block:: html+jinja

   <!-- ... -->
       {% block javascripts %}
           <script type="text/javascript" src="{{ asset('bundles/ibwjobeet/js/jquery-2.0.3.min.js') }}"></script>
           <script type="text/javascript" src="{{ asset('bundles/ibwjobeet/js/search.js') }}"></script>
       {% endblock %}
   <!-- ... -->

AJAX in an Action

If JavaScript is enabled, jQuery will intercept all keys typed in the search box, and will call the search action. If not, the same search action is also called when the user submits the form by pressing the enter key.
So, the search action now needs to determine if the call is made via AJAX or not. Whenever a request is made with an AJAX call, the isXmlHttpRequest() method of the request object returns true.

src/Ibw/JobeetBundle/Controller/JobController.php

.. code-block:: php

   use Symfony\Component\HttpFoundation\Response;

   class JobController extends Controller
   {
       // ...

       public function searchAction(Request $request)
       {
           $em = $this->getDoctrine()->getManager();
           $query = $this->getRequest()->get('query');

           if(!$query) {
               if(!$request->isXmlHttpRequest()) {
                   return $this->redirect($this->generateUrl('ibw_job'));
               } else {
                   return new Response('No results.');
               }
           }

           $jobs = $em->getRepository('IbwJobeetBundle:Job')->getForLuceneQuery($query);

           if($request->isXmlHttpRequest()) {

               return $this->render('IbwJobeetBundle:Job:list.html.twig', array('jobs' => $jobs));
           }

           return $this->render('IbwJobeetBundle:Job:search.html.twig', array('jobs' => $jobs));
       }
   }

If the search returns no result, we need to display a message instead of a blank page. We will return just a simple string:

src/Ibw/JobeetBundle/Controller/JobController.php

.. code-block:: php

   public function searchAction(Request $request)
   {
     $em = $this->getDoctrine()->getManager();
     $query = $this->getRequest()->get('query');

     if(!$query) {
         if(!$request->isXmlHttpRequest()) {
             return $this->redirect($this->generateUrl('ibw_job'));
         } else {
             return new Response('No results.');
         }
     }

     $jobs = $em->getRepository('IbwJobeetBundle:Job')->getForLuceneQuery($query);

     if($request->isXmlHttpRequest()) {
         if('*' == $query || !$jobs || $query == '') {
             return new Response('No results.');
         }

         return $this->render('IbwJobeetBundle:Job:list.html.twig', array('jobs' => $jobs));
     }

     return $this->render('IbwJobeetBundle:Job:search.html.twig', array('jobs' => $jobs));
   }

Testing AJAX
------------

As the symfony browser cannot simulate JavaScript, you need to help it when testing AJAX calls. It mainly means that you need to manually add the header that jQuery and all other major JavaScript libraries send with the request:

src/Ibw/JobeetBundle/Tests/Controller/JobControllerTest.php

.. code-block:: php

   class JobControllerTest extends WebTestCase
   {
       // ...

       public function testSearch()
       {
           $client = static::createClient();

           $crawler = $client->request('GET', '/job/search');
           $this->assertEquals('Ibw\JobeetBundle\Controller\JobController::searchAction', $client->getRequest()->attributes->get('_controller'));

           $crawler = $client->request('GET', '/job/search?query=sens*', array(), array(), array(
               'X-Requested-With' => 'XMLHttpRequest',
           ));
           $this->assertTrue($crawler->filter('tr')->count()== 2);
       }
   }

In day 17, we used the Zend Lucene library to implement the search engine. Today, we used jQuery to make it more responsive. The symfony framework provides all the fundamental tools to build MVC applications with ease, and also plays well with other components. As always, try to use the best tool for the job.
Tomorrow, we will explain how to internationalize the Jobeet website.

.. include:: common/license.rst.inc
