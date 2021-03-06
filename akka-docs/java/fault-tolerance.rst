.. _fault-tolerance-java:

Fault Handling Strategies (Java)
=================================

.. sidebar:: Contents

   .. contents:: :local:

As explained in :ref:`actor-systems` each actor is the supervisor of its
children, and as such each actor is given a fault handling strategy when it is
created. This strategy cannot be changed afterwards as it is an integral part
of the actor system’s structure.

Creating a Fault Handling Strategy
----------------------------------

For the sake of demonstration let us consider the following strategy:

.. includecode:: code/akka/docs/actor/FaultHandlingTestBase.java
   :include: strategy

I have chosen a few well-known exception types in order to demonstrate the
application of the fault handling actions described in :ref:`supervision`.
First off, it is a one-for-one strategy, meaning that each child is treated
separately (an all-for-one strategy works very similarly, the only difference
is that any decision is applied to all children of the supervisor, not only the
failing one). There are limits set on the restart frequency, namely maximum 10
restarts per minute; each of these settings defaults to could be left out, which means
that the respective limit does not apply, leaving the possibility to specify an
absolute upper limit on the restarts or to make the restarts work infinitely.

Practical Application
---------------------

The following section shows the effects of the different actions in practice,
wherefor a test setup is needed. First off, we need a suitable supervisor:

.. includecode:: code/akka/docs/actor/FaultHandlingTestBase.java
   :include: supervisor

This supervisor will be used to create a child, with which we can experiment:

.. includecode:: code/akka/docs/actor/FaultHandlingTestBase.java
   :include: child

The test is easier by using the utilities described in :ref:`akka-testkit`,
where ``TestProbe`` provides an actor ref useful for receiving and inspecting replies.

.. includecode:: code/akka/docs/actor/FaultHandlingTestBase.java
   :include: testkit

Using the strategy shown above let us create actors:

.. includecode:: code/akka/docs/actor/FaultHandlingTestBase.java
   :include: create

The first test shall demonstrate the ``Resume`` action, so we try it out by
setting some non-initial state in the actor and have it fail:

.. includecode:: code/akka/docs/actor/FaultHandlingTestBase.java
   :include: resume

As you can see the value 42 survives the fault handling action. Now, if we
change the failure to a more serious ``NullPointerException``, that will no
longer be the case:

.. includecode:: code/akka/docs/actor/FaultHandlingTestBase.java
   :include: restart

And finally in case of the fatal ``IllegalArgumentException`` the child will be
terminated by the supervisor:

.. includecode:: code/akka/docs/actor/FaultHandlingTestBase.java
   :include: stop

Up to now the supervisor was completely unaffected by the child’s failure,
because the actions set did handle it. In case of an ``Exception``, this is not
true anymore and the supervisor escalates the failure.

.. includecode:: code/akka/docs/actor/FaultHandlingTestBase.java
   :include: escalate-kill

The supervisor itself is supervised by the top-level actor provided by the
:class:`ActorSystem`, which has the default policy to restart in case of all
``Exception`` cases (with the notable exceptions of
``ActorInitializationException`` and ``ActorKilledException``). Since the
default action in case of a restart is to kill all children, we expected our poor
child not to survive this failure.

In case this is not desired (which depends on the use case), we need to use a
different supervisor which overrides this behavior.

.. includecode:: code/akka/docs/actor/FaultHandlingTestBase.java
   :include: supervisor2

With this parent, the child survives the escalated restart, as demonstrated in
the last test:

.. includecode:: code/akka/docs/actor/FaultHandlingTestBase.java
   :include: escalate-restart

