.. _logging-scala:

#################
 Logging (Scala)
#################

.. sidebar:: Contents

   .. contents:: :local:

How to Log
==========

Create a ``LoggingAdapter`` and use the ``error``, ``warning``, ``info``, or ``debug`` methods,
as illustrated in this example:

.. includecode:: code/akka/docs/event/LoggingDocSpec.scala
   :include: my-actor

For convenience you can mixin the ``log`` member into actors, instead of defining it as above.

.. code-block:: scala

  class MyActor extends Actor with akka.actor.ActorLogging {
    ...
  }

The second parameter to the ``Logging`` is the source of this logging channel.
The source object is translated to a String according to the following rules:

  * if it is an Actor or ActorRef, its path is used
  * in case of a String it is used as is
  * in case of a class an approximation of its simpleName
  * and in all other cases a compile error occurs unless and implicit
    :class:`LogSource[T]` is in scope for the type in question.

The log message may contain argument placeholders ``{}``, which will be substituted if the log level
is enabled.

Translating Log Source to String and Class
------------------------------------------

The rules for translating the source object to the source string and class
which are inserted into the :class:`LogEvent` during runtime are implemented
using implicit parameters and thus fully customizable: simply create your own
instance of :class:`LogSource[T]` and have it in scope when creating the
logger.

.. includecode:: code/akka/docs/event/LoggingDocSpec.scala#my-source

This example creates a log source which mimics traditional usage of Java
loggers, which are based upon the originating object’s class name as log
category. The override of :meth:`getClazz` is only included for demonstration
purposes as it contains exactly the default behavior.

.. note::

  You may also create the string representation up front and pass that in as
  the log source, but be aware that then the :class:`Class[_]` which will be
  put in the :class:`LogEvent` is
  :class:`akka.event.DummyClassForStringSources`.

  The SLF4J event listener treats this case specially (using the actual string
  to look up the logger instance to use instead of the class’ name), and you
  might want to do this also in case you implement your own loggin adapter.

Event Handler
=============

Logging is performed asynchronously through an event bus. You can configure
which event handlers that should subscribe to the logging events. That is done
using the ``event-handlers`` element in the :ref:`configuration`.  Here you can
also define the log level.

.. code-block:: ruby

  akka {
    # Event handlers to register at boot time (Logging$DefaultLogger logs to STDOUT)
    event-handlers = ["akka.event.Logging$DefaultLogger"]
    # Options: ERROR, WARNING, INFO, DEBUG
    loglevel = "DEBUG"
  }

The default one logs to STDOUT and is registered by default. It is not intended
to be used for production. There is also an :ref:`slf4j-scala`
event handler available in the 'akka-slf4j' module.

Example of creating a listener:

.. includecode:: code/akka/docs/event/LoggingDocSpec.scala
   :include: my-event-listener

.. _slf4j-scala:

SLF4J
=====

Akka provides an event handler for `SL4FJ <http://www.slf4j.org/>`_. This module is available in the 'akka-slf4j.jar'.
It has one single dependency; the slf4j-api jar. In runtime you also need a SLF4J backend, we recommend `Logback <http://logback.qos.ch/>`_:

  .. code-block:: scala

     lazy val logback = "ch.qos.logback" % "logback-classic" % "1.0.0" % "runtime"


You need to enable the Slf4jEventHandler in the 'event-handlers' element in
the :ref:`configuration`. Here you can also define the log level of the event bus.
More fine grained log levels can be defined in the configuration of the SLF4J backend
(e.g. logback.xml). The String representation of the source object that is used when
creating the ``LoggingAdapter`` correspond to the name of the SL4FJ logger.

.. code-block:: ruby

  akka {
    event-handlers = ["akka.event.slf4j.Slf4jEventHandler"]
    loglevel = "DEBUG"
  }

Logging Thread and Akka Source in MDC
-------------------------------------

Since the logging is done asynchronously the thread in which the logging was performed is captured in
Mapped Diagnostic Context (MDC) with attribute name ``sourceThread``.
With Logback the thread name is available with ``%X{sourceThread}`` specifier within the pattern layout configuration::

  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <layout>
      <pattern>%date{ISO8601} %-5level %logger{36} %X{sourceThread} - %msg%n</pattern>
    </layout>
  </appender>

.. note::
  
  It will probably be a good idea to use the ``sourceThread`` MDC value also in
  non-Akka parts of the application in order to have this property consistently
  available in the logs.

Another helpful facility is that Akka captures the actor’s address when
instantiating a logger within it, meaning that the full instance identification
is available for associating log messages e.g. with members of a router. This
information is available in the MDC with attribute name ``akkaSource``::

  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <layout>
      <pattern>%date{ISO8601} %-5level %logger{36} %X{akkaSource} - %msg%n</pattern>
    </layout>
  </appender>

For more details on what this attribute contains—also for non-actors—please see
`How to Log`_.
