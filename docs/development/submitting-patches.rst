Submitting Patches
==================

* Always make a new branch for your work.
* Patches should be small to facilitate easier review. `Studies have shown`_
  that review quality falls off as patch size grows. Sometimes this will result
  in many small PRs to land a single large feature.
* Larger changes should be discussed on `our mailing list`_ before submission.
* New features and significant bug fixes should be documented in the
  :doc:`/changelog`.

If you believe you've identified a security issue in ``cryptography``, please
follow the directions on the :doc:`security page </security>`.

Code
----

When in doubt, refer to :pep:`8` for Python code.

`Write comments as complete sentences.`_

Every code file must start with the boilerplate notice of the Apache License.
Additionally, every Python code file must contain

.. code-block:: python

    from __future__ import absolute_import, division, print_function

API Considerations
~~~~~~~~~~~~~~~~~~

Most projects' APIs are designed with a philosophy of "make easy things easy,
and make hard things possible". One of the perils of writing cryptographic code
is that secure code looks just like insecure code, and its results are almost
always indistinguishable. As a result ``cryptography`` has, as a design
philosophy: "make it hard to do insecure things". Here are a few strategies for
API design that should be both followed, and should inspire other API choices:

If it is necessary to compare a user provided value with a computed value (for
example, verifying a signature), there should be an API provided that performs
the verification in a secure way (for example, using a constant time
comparison), rather than requiring the user to perform the comparison
themselves.

If it is incorrect to ignore the result of a method, it should raise an
exception, and not return a boolean ``True``/``False`` flag. For example, a
method to verify a signature should raise ``InvalidSignature``, and not return
whether the signature was valid.

.. code-block:: python

    # This is bad.
    def verify(sig):
        # ...
        return is_valid

    # Good!
    def verify(sig):
        # ...
        if not is_valid:
            raise InvalidSignature

Every recipe should include a version or algorithmic marker of some sort in its
output in order to allow transparent upgrading of the algorithms in use, as
the algorithms or parameters needed to achieve a given security margin evolve.

APIs at the :doc:`/hazmat/primitives/index` layer should always take an
explicit backend, APIs at the recipes layer should automatically use the
:func:`~cryptography.hazmat.backends.default_backend`, but optionally allow
specifying a different backend.

C bindings
~~~~~~~~~~

When binding C code with ``cffi`` we have our own style guide, it's pretty
simple.

Don't name parameters:

.. code-block:: c

    // Good
    long f(long);
    // Bad
    long f(long x);

...unless they're inside a struct:

.. code-block:: c

    struct my_struct {
        char *name;
        int number;
        ...;
    };

Include ``void`` if the function takes no arguments:

.. code-block:: c

    // Good
    long f(void);
    // Bad
    long f();

Wrap lines at 80 characters like so:

.. code-block:: c

    // Pretend this went to 80 characters
    long f(long, long,
           int *)

Include a space after commas between parameters:

.. code-block:: c

    // Good
    long f(int, char *)
    // Bad
    long f(int,char *)

Values set by ``#define`` should be assigned the appropriate type. If you see
this:

.. code-block:: c

    #define SOME_INTEGER_LITERAL 0x0;
    #define SOME_UNSIGNED_INTEGER_LITERAL 0x0001U;
    #define SOME_STRING_LITERAL "hello";

...it should be added to the bindings like so:

.. code-block:: c

    static const int SOME_INTEGER_LITERAL;
    static const unsigned int SOME_UNSIGNED_INTEGER_LITERAL;
    static const char *const SOME_STRING_LITERAL;

Tests
-----

All code changes must be accompanied by unit tests with 100% code coverage (as
measured by the combined metrics across our build matrix).

When implementing a new primitive or recipe ``cryptography`` requires that you
provide a set of test vectors. See :doc:`/development/test-vectors` for more
details.

Documentation
-------------

All features should be documented with prose in the ``docs`` section.

Because of the inherent challenges in implementing correct cryptographic
systems, we want to make our documentation point people in the right directions
as much as possible. To that end:

* When documenting a generic interface, use a strong algorithm in examples.
  (e.g. when showing a hashing example, don't use
  :class:`~cryptography.hazmat.primitives.hashes.MD5`)
* When giving prescriptive advice, always provide references and supporting
  material.
* When there is real disagreement between cryptographic experts, represent both
  sides of the argument and describe the trade-offs clearly.

When documenting a new module in the ``hazmat`` package, its documentation
should begin with the "Hazardous Materials" warning:

.. code-block:: rest

    .. hazmat::

When referring to a hypothetical individual (such as "a person receiving an
encrypted message") use gender neutral pronouns (they/them/their).

Docstrings are typically only used when writing abstract classes, but should
be written like this if required:

.. code-block:: python

    def some_function(some_arg):
        """
        Does some things.

        :param some_arg: Some argument.
        """

So, specifically:

* Always use three double quotes.
* Put the three double quotes on their own line.
* No blank line at the end.
* Use Sphinx parameter/attribute documentation `syntax`_.


.. _`Write comments as complete sentences.`: http://nedbatchelder.com/blog/201401/comments_should_be_sentences.html
.. _`syntax`: http://sphinx-doc.org/domains.html#info-field-lists
.. _`Studies have shown`: http://www.ibm.com/developerworks/rational/library/11-proven-practices-for-peer-review/
.. _`our mailing list`: https://mail.python.org/mailman/listinfo/cryptography-dev
