:title: Running a Redis service
:description: Installing and running an redis service
:keywords: docker, example, package installation, networking, redis

.. _running_redis_service:

Redis Service
=============

.. include:: example_header.inc

Very simple, no frills, Redis service attached to a web application using a link.

Create a docker container for Redis
-----------------------------------

Firstly, we create a ``Dockerfile`` for our new Redis image.

.. code-block:: bash

    FROM        ubuntu:12.10
    RUN         apt-get update
    RUN         apt-get -y install redis-server
    EXPOSE      6379
    ENTRYPOINT  ["/usr/bin/redis-server"]

Next we build an image from our ``Dockerfile``. Replace ``<your username>`` 
with your own user name.

.. code-block:: bash

    sudo docker build -t <your username>/redis .

Run the service
---------------

Use the image we've just created and name your container ``redis``.

Running the service with ``-d`` runs the container in detached mode, leaving the
container running in the background.

Importantly, we're not exposing any ports on our container. Instead we're going to 
use a container link to provide access to our Redis database.

.. code-block:: bash

    sudo docker run --name redis -d <your username>/redis

Create your web application container
-------------------------------------

Next we can create a container for our application. We're going to use the ``-link`` 
flag to create a link to the ``redis`` container we've just created with an alias of 
``db``. This will create a secure tunnel to the ``redis`` container and expose the 
Redis instance running inside that container to only this container.

.. code-block:: bash

    sudo docker run --link redis:db -i -t ubuntu:12.10 /bin/bash

Once inside our freshly created container we need to install Redis to get the 
``redis-cli`` binary to test our connection.

.. code-block:: bash

    apt-get update
    apt-get -y install redis-server
    service redis-server stop

As we've used the ``--link redis:db`` option, Docker has created some environment 
variables in our web application container.

.. code-block:: bash

    env | grep DB_
    
    # Should return something similar to this with your values 
    DB_NAME=/violet_wolf/db
    DB_PORT_6379_TCP_PORT=6379
    DB_PORT=tcp://172.17.0.33:6379
    DB_PORT_6379_TCP=tcp://172.17.0.33:6379
    DB_PORT_6379_TCP_ADDR=172.17.0.33
    DB_PORT_6379_TCP_PROTO=tcp

We can see that we've got a small list of environment variables prefixed with ``DB``.
The ``DB`` comes from the link alias specified when we launched the container. Let's use 
the ``DB_PORT_6379_TCP_ADDR`` variable to connect to our Redis container.

.. code-block:: bash

    redis-cli -h $DB_PORT_6379_TCP_ADDR
    redis 172.17.0.33:6379>
    redis 172.17.0.33:6379> set docker awesome
    OK
    redis 172.17.0.33:6379> get docker
    "awesome"
    redis 172.17.0.33:6379> exit

We could easily use this or other environment variables in our web application to make a 
connection to our ``redis`` container.

