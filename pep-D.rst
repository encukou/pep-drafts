PEP: 9999
Title: Running extension modules using -m switch
Version: $Revision$
Last-Modified: $Date$
Author: Marcel Plch <gmarcel.plch@gmail.com>,
        Petr Viktorin <encukou@gmail.com>
Status: Draft
Type: Standards Track
Content-Type: x-rst
Created: 25-May-2017
Python-Version: 3.7
Post-History: 


Abstract
========




Specification
=============

This PEP proposes implementation that allows built-in and extension
modules to be executed in the ``__main__`` namespace using
the `PEP 489`_ multi-phase initialization.

With this, a multi-phase initialization enabled module can be run
using following syntax::

    $ python3 -m _testmultiphase
    This is a test module named __main__.


Python-level changes
--------------------

There are going to be few changes at the Python level.
Affected files are going to be Lib/runpy.py and
Lib/importlib/_bootstrap_external.py.

A new optional method for loaders will be added. This method is
called ``exec_in_module`` and will take two positional arguments:
module spec and already initialized module. It will then passe these
arguments to new C function inside the _imp module called the same name.

That means runpy will look for this new attribute and if it is present,
it will execute it and return results.


C-level changes
---------------

A new function will have been added.

This function will be called ``exec_in_module`` and will take two arguments:
a spec, and a module in which's namespace should be the code
executed. This function will find a module def according to spec.

If def belongs to a single-phase initialization module,
an import exception will be raised since this is not feasible
for single-phase initialization.

If def belongs to multi-phase initialization module, is,
new function ``PyModule_ExecInModule`` (described below) will be
called and passed the namespace module and def.

A function called ``PyModule_ExecInModule``, this function will receive
module and def as arguments. Then it will check if the module really
is a module. If not, system exception is raised. Import exception
will be raised even if the module has already been initialized,
so modules will have to make sure that the module has no state assigned.
After all of this, the function checks if there is any ``Py_mod_create``
slot present, if there is, import exception is raised.

Finally, if everything went alright, the function will assign methods,
doc string and will call ``PyModule_ExecDef`` using the passed-in arguments.


Motivation
==========




Rationale
=========

This approach may prove useful with Cython compiled modules.
Author of such module could just compile it and leave it as it
already is without making any changes to make it work.


Backwards Compatibility
=======================

This PEP should cause no problems with the old code, since it
doesn't change the way API is being used and usage of any
additional functions is purely optional.


Reference Implementation
========================

The reference implementation of this PEP has been made available
at GitHub_, so it's quite easy to look at those changes for revision.


References
==========

.. _PEP 489: https://www.python.org/dev/peps/pep-0489/
.. _GitHub: https://github.com/python/cpython/pull/1761


Copyright
=========

This document has been placed in the public domain.



..
   Local Variables:
   mode: indented-text
   indent-tabs-mode: nil
   sentence-end-double-space: t
   fill-column: 70
   coding: utf-8
   End:
