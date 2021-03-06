Storage API
===========

This document details the HTTP API used by the :doc:`http`. It is targeted at
developers interested in developing their own backend servers that integrate
with Annotator, or developing tools that integrate with existing instances of
the API.

The storage API attempts to follow the principles of `REST
<http://en.wikipedia.org/wiki/Representational_state_transfer>`__, and uses JSON
as its primary interchange format.

.. contents::

Endpoints
---------

root
~~~~

.. http:get:: /api

   API root. Returns an object containing store metadata, including hypermedia
   links to the rest of the API.

   **Example request**:

   .. sourcecode:: http

      GET /api
      Host: example.com
      Accept: application/json


   **Example response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Access-Control-Allow-Origin: *
      Access-Control-Expose-Headers: Content-Length, Content-Type, Location
      Content-Length: 1419
      Content-Type: application/json

      {
          "message": "Annotator Store API",
          "links": {
              "annotation": {
                  "create": {
                      "desc": "Create a new annotation",
                      "method": "POST",
                      "url": "http://example.com/api/annotations"
                  },
                  "delete": {
                      "desc": "Delete an annotation",
                      "method": "DELETE",
                      "url": "http://example.com/api/annotations/:id"
                  },
                  "read": {
                      "desc": "Get an existing annotation",
                      "method": "GET",
                      "url": "http://example.com/api/annotations/:id"
                  },
                  "update": {
                      "desc": "Update an existing annotation",
                      "method": "PUT",
                      "url": "http://example.com/api/annotations/:id"
                  }
              },
              "search": {
                  "desc": "Basic search API",
                  "method": "GET",
                  "url": "http://example.com/api/search"
              }
          }
      }

   :reqheader Accept: desired response content type
   :resheader Content-Type: response content type
   :statuscode 200: no error


read
~~~~

.. http:get:: /api/annotations/(string:id)

   Retrieve a single annotation.

   **Example request**:

   .. sourcecode:: http

     GET /api/annotations/utalbWjUaZK5ifydnohjmA
     Host: example.com
     Accept: application/json

   **Example response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json; charset=UTF-8

      {
          "created": "2013-08-26T13:31:49.339078+00:00",
          "updated": "2013-08-26T14:09:14.121339+00:00",
          "id": "utalbWjUQZK5ifydnohjmA",
          "uri": "http://example.com/foo",
          "user": "acct:johndoe@example.org",
          ...
      }

   :reqheader Accept: desired response content type
   :resheader Content-Type: response content type
   :statuscode 200: no error
   :statuscode 404: annotation with the specified `id` not found


create
~~~~~~

.. http:post:: /api/annotations

   Create a new annotation.

   **Example request**:

   .. sourcecode:: http

      POST /api/annotations
      Host: example.com
      Accept: application/json
      Content-Type: application/json;charset=UTF-8

      {
          "uri": "http://example.org/",
          "user": "joebloggs",
          "permissions": {
              "read": ["group:__world__"],
              "update": ["joebloggs"],
              "delete": ["joebloggs"],
              "admin": ["joebloggs"],
          },
          "target": [ ... ],
          "text": "This is an annotation I made."
      }

   **Example response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json; charset=UTF-8

      {
          "id": "AUxWM-HasREW1YKAwhil",
          "uri": "http://example.org/",
          "user": "joebloggs",
          ...
      }

   :param id: annotation's unique id
   :reqheader Accept: desired response content type
   :reqheader Content-Type: request body content type
   :resheader Content-Type: response content type
   :>json string id: unique id of new annotation
   :statuscode 200: no error
   :statuscode 400: could not create annotation from your request (bad payload)


update
~~~~~~

.. http:put:: /api/annotations/(string:id)

   Update the annotation with the given `id`. Requires a valid authentication
   token.

   **Example request**:

   .. sourcecode:: http

      PUT /api/annotations/AUxWM-HasREW1YKAwhil
      Host: example.com
      Accept: application/json
      Content-Type: application/json;charset=UTF-8

      {
          "uri": "http://example.org/foo",
      }

   **Example response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json; charset=UTF-8

      {
          "id": "AUxWM-HasREW1YKAwhil",
          "updated": "2015-03-26T13:09:42.646509+00:00"
          "uri": "http://example.org/foo",
          "user": "joebloggs",
          ...
      }

   :param id: annotation's unique id
   :reqheader Accept: desired response content type
   :reqheader Content-Type: request body content type
   :resheader Content-Type: response content type
   :statuscode 200: no error
   :statuscode 400: could not update annotation from your request (bad payload)
   :statuscode 404: annotation with the given `id` was not found


delete
~~~~~~

.. http:delete:: /api/annotations/(string:id)

   Delete the annotation with the given `id`. Requires a valid authentication
   token.

   **Example request**:

   .. sourcecode:: http

      DELETE /api/annotations/AUxWM-HasREW1YKAwhil
      Host: example.com
      Accept: application/json

   **Example response**:

   .. sourcecode:: http

      HTTP/1.1 204 No Content
      Content-Length: 0

   :param id: annotation's unique id
   :reqheader Accept: desired response content type
   :resheader Content-Type: response content type
   :statuscode 200: no error
   :statuscode 404: annotation with the given `id` was not found


search
~~~~~~

.. http:get:: /api/search

   Search the database of annotations. Search for fields using query string
   parameters.

   **Example request**:

   .. sourcecode:: http

      GET /api/search?text=foobar&limit=10
      Host: example.com
      Accept: application/json

   **Example response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Length: 6771
      Content-Type: application/json

      {
          "total": 43127,
          "rows": [
              {
                  "id": "d41d8cd98f00b204e9800998ecf8427e",
                  "text": "Updated annotation text",
                  ...
              },
              ...
          ]
      }

   :query offset: return results starting at `offset`
   :query limit: return only `limit` results
   :reqheader Accept: desired response content type
   :reqheader Content-Type: request body content type
   :resheader Content-Type: response content type
   :>json int total: total number of results across all pages
   :>json array rows: array of matching annotations
   :statuscode 200: no error
   :statuscode 400: could not search the database with your request (invalid query)

Storage Implementations
-----------------------

-  Reference backend, a Python Flask app:
   https://github.com/okfn/annotator-store (in particular, see
   `store.py <https://github.com/okfn/annotator-store/blob/master/annotator/store.py>`__,
   although be aware that this file also deals with authentication and
   authorization, making the code a good deal more complex than would be
   required to implement what is described above).
-  PHP (Silex) and MongoDB-based basic implementation:
   https://github.com/julien-c/annotator-php (in particular, see
   `index.php <https://github.com/julien-c/annotator-php/blob/master/index.php>`__).
-  eXanore an eXist-db library implementing the Annotator Storage API (currently under development)
   https://github.com/bwbohl/eXanore
