.. _cartridge_getting_started:

--------------------------------------------------------------------------------
Getting started with Tarantool Cartridge
--------------------------------------------------------------------------------

Here we'll walk you through developing a simple cluster application.

First, set up the development environment:

#. `Install <https://github.com/tarantool/cartridge-cli#installation>`_
   ``cartridge-cli``, a command-line tool for developing, deploying, and
   managing Tarantool applications.

#. `Install <https://git-scm.com/book/en/v2/Getting-Started-Installing-Git>`_
   ``git``, a version control system.

#. `Install <https://www.npmjs.com/get-npm>`_
   ``npm``, a package manager for ``node.js``.

#. `Install <https://linuxize.com/post/how-to-unzip-files-in-linux/>`_
   the ``unzip`` utility.

Now create an application named ``myapp``. Say:

.. code-block:: console

   cartridge create --name myapp
   cd ./myapp

This will create a Cartridge application in the ``./myapp`` directory with
a handful of template
`files and directories <https://www.tarantool.io/en/doc/latest/book/cartridge/cartridge_cli/#creating-an-application-from-template>`_
inside.

Make a dry run now:

.. code-block:: console

   cartridge build
   cartridge start

This will build the application locally, start 5 instances of Tarantool and run
our application as it is, with no business logic yet.

Why 5 instances? Check out the ``instances.yml`` file in your application
directory. By default, it defines configuration for 5 Tarantool instances:
one router, two storage masters, and two storage replicas.
That's exactly what we need for this exercise.

Now it's time to add some business logic to our application. This will be an
evergreen "Hello, world!" -- just to keep things simple.

Duplicate the ``app/roles/custom.lua`` file and rename the copy to
``hello_world.lua``:

.. code-block:: console

   cp app/roles/custom.lua app/roles/hello_world.lua

This will be our custom role.
Here we'll need to do the following:

* Add our logic to the ``init()`` function:

  .. code-block:: lua
     :emphasize-lines: 3-4

     local function init(opts)

         local log = require('log')
         log.info('Hello, world!')

         return true
     end

  This will write ``Hello, world!`` to the console on cluster start.
  No rocket science.

* Add our role to the "return" section:

  .. code-block:: lua
     :emphasize-lines: 2

     return {
         role_name = 'app.roles.hello_world',
         init = init,
         stop = stop,
         validate_config = validate_config,
         apply_config = apply_config
     }

Why ``role_name = 'app.roles.hello_world'``? The role name should match the path
from the application root (``./myapp``) to the custom role file
(``app/roles/hello_world.lua``). If  we say ``role_name = 'hello_world'``
in "return", the cluster won't find the role.

There's one more thing to do before we can run the application: we need to
add our role to the list of cluster roles in the ``init.lua`` file.

.. code-block:: lua
   :emphasize-lines: 6

    local ok, err = cartridge.cfg({
        workdir = 'tmp/db',
        roles = {
            'cartridge.roles.vshard-storage',
            'cartridge.roles.vshard-router',
            'app.roles.hello_world'
        },
        cluster_cookie = 'lenkis_app_1-cluster-cookie',
    })

Now the cluster will be aware of our role.

Fine! Let's re-build the application and start the cluster now:

.. code-block:: console

   cartridge build
   cartridge start

In the console output, we'll see a line like this:

.. code-block:: console

   router | 2020-04-10 14:23:30.675 [8209] main/108/lua I> Hello, world!

Yeah, that's our business logic speaking. The application is ready to deploy!

Now open the cluster management web interface at
``http://localhost:8081`` (this is the default HTTP port of the router
instance specified in ``init.lua``) and follow
:ref:`this guide <cartridge-deployment-steps>` to set up the cluster and try out
some cool cluster management features.

To stop the cluster after you're done with experiments, press ``Ctrl + C``.

As a final touch, let's pack our application into a distributable, e.g. into an
RPM package:

   .. code-block::

      cartridge pack rpm

You can also
`pack <https://www.tarantool.io/en/doc/2.2/book/cartridge/cartridge_cli/#packing-an-application>`_.
it as a DEB package, a TGZ archive, or a Docker image.
