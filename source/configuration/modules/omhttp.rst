********************************************
omhttp: HTTP REST API Output Module
********************************************

===========================  ===========================================================================
**Module Name:**             **omhttp**
**Author:**                  `Christian Tramnitz <https://www.trasec.de/>`_ <tramnitz@trasec.de>
===========================  ===========================================================================


Purpose
=======

This module provides support for logging to HTTP
REST API endpoint.


Notable Features
================

- :ref:`omhttp-statistic-counter`


Configuration Parameters
========================

.. note::

   Parameter names are case-insensitive.


Action Parameters
-----------------

Server
^^^^^^

.. csv-table::
   :header: "type", "default", "mandatory", "|FmtObsoleteName| directive"
   :widths: auto
   :class: parameter-table

   "array", "localhost", "no", "none"

An array of HTTP REST servers in the specified format. If no scheme
is specified, it will be chosen according to usehttps_. If no port is
specified, serverport_ will be used. Defaults to "localhost".

Requests to multiple servers will be load-balanced between all servers in
round-robin fashion.

.. code-block:: none

  Examples:
       server="localhost:9200"
       server=["api-server-1", "api-server-2"]


.. _serverport:

Serverport
^^^^^^^^^^

.. csv-table::
   :header: "type", "default", "mandatory", "|FmtObsoleteName| directive"
   :widths: auto
   :class: parameter-table

   "integer", "443", "no", "none"

Default HTTPS port to use to connect to HTTP REST API. If none is specified
on a server_. Defaults to 443


.. _healthchecktimeout:

HealthCheckTimeout
^^^^^^^^^^^^^^^^^^

.. csv-table::
   :header: "type", "default", "mandatory", "|FmtObsoleteName| directive"
   :widths: auto
   :class: parameter-table

   "integer", "3500", "no", "none"

Specifies the number of milliseconds to wait for a successful health check
on a server_. Before trying to submit events to the HTTP REST endpoint,
rsyslog will execute an *HTTP HEAD* to the configured check path and expect
an *HTTP OK* within this timeframe. Defaults to 3500.

*Note, the health check is verifying connectivity only, not the state of
the HTTP REST endpoint.*


.. _restPath:

restPath
^^^^^^^^^^^

.. csv-table::
   :header: "type", "default", "mandatory", "|FmtObsoleteName| directive"
   :widths: auto
   :class: parameter-table

   "word", "none", "yes", "none"

HTTP path for your REST API to send your logs to.


.. _dynRestPath:

dynRestPath
^^^^^^^^^^^^^^

.. csv-table::
   :header: "type", "default", "mandatory", "|FmtObsoleteName| directive"
   :widths: auto
   :class: parameter-table

   "binary", "off", "no", "none"

Whether the string provided for restPath_ should be taken as a
`rsyslog template <http://www.rsyslog.com/doc/rsyslog_conf_templates.html>`_.
Defaults to "off", which means the index name will be taken
literally. Otherwise, it will look for a template with that name, and
the resulting string will be the index name. For example, let's
assume you define a template named "date-days" containing
"%timereported:1:10:date-rfc3339%". Then, with dynRestPath="on",
if you say restPath="date-days", each log will be sent to an URL
named after the first 10 characters of the timestamp, like
"2013-03-22".

.. _usehttps:

usehttps
^^^^^^^^

.. csv-table::
   :header: "type", "default", "mandatory", "|FmtObsoleteName| directive"
   :widths: auto
   :class: parameter-table

   "binary", "on", "no", "none"

Default scheme to use when sending events to HTTP REST endpoint if none
is specified on a  server_. The issuing CA of the server must be trusted by
the system.

.. _template:

template
^^^^^^^^

.. csv-table::
   :header: "type", "default", "mandatory", "|FmtObsoleteName| directive"
   :widths: auto
   :class: parameter-table

   "word", "see below", "no", "none"

This is the JSON document that will be send to the HTTP REST endpoint.
For most endpoints, the resulting string needs to be a valid JSON.
Defaults to:

.. code-block:: none

    $template JSONDefault, "{\"message\":\"%msg:::json%\",\"fromhost\":\"%HOSTNAME:::json%\",\"facility\":\"%syslogfacility-text%\",\"priority\":\"%syslogpriority-text%\",\"timereported\":\"%timereported:::date-rfc3339%\",\"timegenerated\":\"%timegenerated:::date-rfc3339%\"}"

Which will produce this sort of documents (pretty-printed here for
readability):

.. code-block:: none

    {
        "message": " this is a test message",
        "fromhost": "test-host",
        "facility": "user",
        "priority": "info",
        "timereported": "2013-03-12T18:05:01.344864+02:00",
        "timegenerated": "2013-03-12T18:05:01.344864+02:00"
    }


.. _bulkmode:

bulkmode
^^^^^^^^

.. csv-table::
   :header: "type", "default", "mandatory", "|FmtObsoleteName| directive"
   :widths: auto
   :class: parameter-table

   "binary", "on", "no", "none"

The default "on" setting means multiple logs are shipped in the same request,
delimited by a newline (\n) character. The maximum number of logs sent in a
single bulk request depends on your maxbytes_ and queue settings.
When setting to "off", each log line will be transmitted one by one; each in
its own HTTP request.


.. _maxbytes:

maxbytes
^^^^^^^^

.. csv-table::
   :header: "type", "default", "mandatory", "|FmtObsoleteName| directive"
   :widths: auto
   :class: parameter-table

   "word", "10m", "no", "none"

When shipping logs with bulkmode_ **on**, maxbytes specifies the maximum
size of the request body sent to the HTTP REST endpoint. Logs are batched until
either the buffer reaches maxbytes or the the `dequeue batch
size <http://www.rsyslog.com/doc/node35.html>`_ is reached. In order to
ensure the endpoint does not reject requests due to content length, verify
this value is set accoring to the limits of the endpoint. Defaults to 10m.

.. _uid:

uid
^^^

.. csv-table::
   :header: "type", "default", "mandatory", "|FmtObsoleteName| directive"
   :widths: auto
   :class: parameter-table

   "word", "none", "no", "none"

If you have basic HTTP authentication deployed (eg through the
`elasticsearch-basic
plugin <https://github.com/Asquera/elasticsearch-http-basic>`_), you
can specify your user-name here.


.. _pwd:

pwd
^^^

.. csv-table::
   :header: "type", "default", "mandatory", "|FmtObsoleteName| directive"
   :widths: auto
   :class: parameter-table

   "word", "none", "no", "none"

Password for basic authentication.

.. _httpheaderkey:

httpheaderkey
^^^

.. csv-table::
:header: "type", "default", "mandatory", "|FmtObsoleteName| directive"
:widths: auto
:class: parameter-table

"word", "none", "no", "none"

If you have HTTP header based authentication, you set the key of the header.
This must be used together with httpheadervalue and is especially useful for
API endpoint that use X-header based authentication, i.e. AWS API Gateway.


.. _httpheadervalue:

httpheadervalue
^^^

.. csv-table::
:header: "type", "default", "mandatory", "|FmtObsoleteName| directive"
:widths: auto
:class: parameter-table

"word", "none", "no", "none"

If you have HTTP header based authentication, you set the value of the header.


.. _errorfile:

errorFile
^^^^^^^^^

.. csv-table::
   :header: "type", "default", "mandatory", "|FmtObsoleteName| directive"
   :widths: auto
   :class: parameter-table

   "word", "none", "no", "none"

If specified, records failed in bulk mode are written to this file, including
their error cause. Rsyslog itself does not process the file any more, but the
idea behind that mechanism is that the user can create a script to periodically
inspect the error file and react appropriately. As the complete request is
included, it is possible to simply resubmit messages from that script.

*Please note:* when rsyslog has problems connecting to elasticsearch, a general
error is assumed and the submit is retried. However, if we receive negative
responses during batch processing, we assume an error in the data itself
(like a mandatory field is not filled in, a format error or something along
those lines). Such errors cannot be solved by simpy resubmitting the record.
As such, they are written to the error file so that the user (script) can
examine them and act appropriately. Note that e.g. after search index
reconfiguration (e.g. dropping the mandatory attribute) a resubmit may
be succesful.

.. _omhttp-tls.cacert:

tls.cacert
^^^^^^^^^^

.. csv-table::
   :header: "type", "default", "mandatory", "|FmtObsoleteName| directive"
   :widths: auto
   :class: parameter-table

   "word", "none", "no", "none"

This is the full path and file name of the file containing the CA cert for the
CA that issued the Elasticsearch server cert.  This file is in PEM format.  For
example: `/etc/rsyslog.d/http-ca.crt`

.. _tls.mycert:

tls.mycert
^^^^^^^^^^

.. csv-table::
   :header: "type", "default", "mandatory", "|FmtObsoleteName| directive"
   :widths: auto
   :class: parameter-table

   "word", "none", "no", "none"

This is the full path and file name of the file containing the client cert for
doing client cert auth against Elasticsearch.  This file is in PEM format.  For
example: `/etc/rsyslog.d/http-client-cert.pem`

.. _tls.myprivkey:

tls.myprivkey
^^^^^^^^^^^^^

.. csv-table::
   :header: "type", "default", "mandatory", "|FmtObsoleteName| directive"
   :widths: auto
   :class: parameter-table

   "word", "none", "no", "none"

This is the full path and file name of the file containing the private key
corresponding to the cert `tls.mycert` used for doing client cert auth against
Elasticsearch.  This file is in PEM format, and must be unencrypted, so take
care to secure it properly.  For example: `/etc/rsyslog.d/http-client-key.pem`

.. _omhttp-retryfailures:

retryfailures
^^^^^^^^^^^^^

.. csv-table::
   :header: "type", "default", "mandatory", "|FmtObsoleteName| directive"
   :widths: auto
   :class: parameter-table

   "binary", "off", "no", "none"

If this parameter is set to `"on"`, then the module will look for an
`"errors":true` in the HTTP response.  If found, each element in the
response will be parsed to look for errors, since a bulk request may have some
records which are successful and some which are failures.  Failed requests will
be converted back into records and resubmitted back to rsyslog for
reprocessing.  Each failed request will be resubmitted with a local variable
called `$.omes`.  This is a hash consisting of the fields from the response.
See below :ref:`omhttp-retry-example` for an example of how retry
processing works.
*NOTE* The retried record will be resubmitted at the "top" of your processing
pipeline.  If your processing pipeline is not idempotent (that is, your
processing pipeline expects "raw" records), then you can specify a ruleset to
redirect retries to.  See :ref:`omhttp-retryruleset` below.

`$.omes` fields:

* status - the HTTP status code - typically an error will have a `4xx` or `5xx`
  code - of particular note is `429` - this means Elasticsearch was unable to
  process this bulk record request due to a temporary condition e.g. the bulk
  index thread pool queue is full, and rsyslog should retry the operation.
* the metadata associated with the request
* error - a hash containing one or more, possibly nested, fields containing
  more detailed information about a failure.  Typically there will be fields
  `$.omes!error!type` (a keyword) and `$.omes!error!reason` (a longer string)
  with more detailed information about the rejection.  NOTE: The format is
  apparently not described in great detail, so code must not make any
  assumption about the availability of `error` or any specific sub-field.

There may be other fields too - the code just copies everything in the
response.

.. _omhttp-retryruleset:

retryruleset
^^^^^^^^^^^^

.. csv-table::
   :header: "type", "default", "mandatory", "|FmtObsoleteName| directive"
   :widths: auto
   :class: parameter-table

   "word", "none", "no", "none"

If `retryfailures` is not `"on"` (:ref:`omhttp-retryfailures`) then
this parameter has no effect.  This parameter specifies the name of a ruleset
to use to route retries.  This is useful if you do not want retried messages to
be processed starting from the top of your processing pipeline, or if you have
multiple outputs but do not want to send retried Elasticsearch failures to all
of your outputs, and you do not want to clutter your processing pipeline with a
lot of conditionals.  See below :ref:`omhttp-retry-example` for an
example of how retry processing works.

.. _omhttp-ratelimit.interval:

ratelimit.interval
^^^^^^^^^^^^^^^^^^

.. csv-table::
   :header: "type", "default", "mandatory", "|FmtObsoleteName| directive"
   :widths: auto
   :class: parameter-table

   "integer", "600", "no", "none"

If `retryfailures` is not `"on"` (:ref:`omhttp-retryfailures`) then
this parameter has no effect.  Specifies the interval in seconds onto which
rate-limiting is to be applied. If more than ratelimit.burst messages are read
during that interval, further messages up to the end of the interval are
discarded. The number of messages discarded is emitted at the end of the
interval (if there were any discards).
Setting this to value zero turns off ratelimiting.

.. _omhttp-ratelimit.burst:

ratelimit.burst
^^^^^^^^^^^^^^^

.. csv-table::
   :header: "type", "default", "mandatory", "|FmtObsoleteName| directive"
   :widths: auto
   :class: parameter-table

   "integer", "20000", "no", "none"

If `retryfailures` is not `"on"` (:ref:`omhttp-retryfailures`) then
this parameter has no effect.  Specifies the maximum number of messages that
can be emitted within the ratelimit.interval interval. For futher information,
see description there.

.. _omhttp-statistic-counter:

Statistic Counter
=================

This plugin maintains global :doc:`statistics <../rsyslog_statistic_counter>`,
which accumulate all action instances. The statistic is named "omhttp".
Parameters are:

-  **submitted** - number of messages submitted for processing (with both
   success and error result)

-  **fail.httprequests** - the number of times a http request failed. Note
   that a single http request may be used to submit multiple messages, so this
   number may be (much) lower than fail.http.

-  **fail.http** - number of message failures due to connection like-problems
   (things like remote server down, broken link etc)

-  **fail.es** - number of failures due to elasticsearch error reply; Note that
   this counter does NOT count the number of failed messages but the number of
   times a failure occured (a potentially much smaller number). Counting messages
   would be quite performance-intense and is thus not done.

The following counters are available when `retryfailures="on"` is used:

-  **response.success** - number of records successfully sent in bulk index
   requests - counts the number of successful responses

-  **response.bad** - number of times omhttp received a response in a
   bulk index response that was unrecognized or unable to be parsed.  This may
   indicate that omhttp is attempting to communicate with a version of
   Elasticsearch that is incompatible, or is otherwise sending back data in the
   response that cannot be handled

-  **response.duplicate** - number of records in the bulk index request that
   were duplicates of already existing records - this will only be reported if
   using `writeoperation="create"` and `bulkid` to assign each record a unique
   ID

-  **response.badargument** - number of times omhttp received a
   response that had a status indicating omhttp sent bad data to
   Elasticsearch.  For example, status `400` and an error message indicating
   omhttp attempted to store a non-numeric string value in a numeric
   field.

-  **response.bulkrejection** - number of times omhttp received a
   response that had a status indicating Elasticsearch was unable to process
   the record at this time - status `429`.  The record can be retried.

-  **response.other** - number of times omhttp received a
   response not recognized as one of the above responses, typically some other
   `4xx` or `5xx` http status.

**The fail.httprequests and fail.http counters reflect only failures that
omhttp detected.** Once it detects problems, it (usually, depends on
circumstances) tell the rsyslog core that it wants to be suspended until the
situation clears (this is a requirement for rsyslog output modules). Once it is
suspended, it does NOT receive any further messages. Depending on the user
configuration, messages will be lost during this period. Those lost messages will
NOT be counted by impstats (as it does not see them).

Note that some previous (pre 7.4.5) versions of this plugin had different counters.
These were experimental and confusing. The only ones really used were "submits",
which were the number of successfully processed messages and "connfail" which were
equivalent to "failed.http".

How Retries Are Handled
=======================

When using `retryfailures="on"` (:ref:`omhttp-retryfailures`), the
original `Message` object (that is, the original `smsg_t *msg` object) **is not
available**.  This means none of the metadata associated with that object, such
as various timestamps, hosts/ip addresses, etc. are not available for the retry
operation.  The only thing available is the original JSON string sent in the
original request, and whatever data is returned in the error response, which
will contain the response metadata and will be made available in the `$.omes`
fields.  For the message to retry, the code will take the original JSON string
and parse it back into an internal `Message` object.  This means you **may need
to use a different template** to output messages for your retry ruleset.  For
example, if you used the following template to format the message for the initial submission:

.. code-block:: none

    template(name="http_output_template"
             type="list"
             option.json="on") {
               constant(value="{")
                 constant(value="\"timestamp\":\"")      property(name="timereported" dateFormat="rfc3339")
                 constant(value="\",\"message\":\"")     property(name="msg")
                 constant(value="\",\"host\":\"")        property(name="hostname")
                 constant(value="\",\"severity\":\"")    property(name="syslogseverity-text")
                 constant(value="\",\"facility\":\"")    property(name="syslogfacility-text")
                 constant(value="\",\"syslogtag\":\"")   property(name="syslogtag")
               constant(value="\"}")
             }

You would have to use a different template for the retry, since none of the
`timereported`, `msg`, etc. fields will have the same values for the retry as
for the initial try.

Examples
========

Example 1
---------

The following sample does the following:

-  loads the omhttp module
-  outputs all logs to an HTTP REST endpoint at the given path

.. code-block:: none

    module(load="omhttp")
    *.*     action(type="omhttp" server="api-gateway-1" restpath="api/logs")

Example 1
---------

The following sample does the following:

-  loads the omhttp module
-  outputs all logs to an HTTP REST endpoint at the given path
-  authenticates using header based authentication

.. code-block:: none

    module(load="omhttp")
    *.*     action(type="omhttp" server="api-gateway-1" restpath="api/logs"
               httpheaderkey="x-api-key" httpheadervalue="mySecretapiKEY")
