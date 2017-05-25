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

This implementation allows built-in and extension modules to be
executed in the __main__ namespace using the `PEP 489`_ multi-phase
initialization.

With this, a multi-phase initialization enabled module can be run
using following syntax::

    $ python3 -m _testmultiphase
    This is a test module named __main__.
    $ ...

This approach may prove useful with Cython compiled modules.
Author of such module could just compile it and leave it as it
already is without making any changes to make it work.


Python-level changes
--------------------

There are few minor changes at the Python level. Affected files are
Lib/runpy.py and Lib/importlib/_bootstrap_external.py.


Lib/runpy.py
''''''''''''

Here, two functions were modified.

_get_module_details has been split into two functions. Now it returns
only module name and spec.

_run_module_as_main can therefore inspect the loader and
if it has new optional method (described below) called
exec_in_module, it executes it and returns globals, otherwise
_get_code is called to obtain script code and runpy behaves the
same way it used to.


Lib/importlib/_bootstrap_external.py
''''''''''''''''''''''''''''''''''''

A new optional method for loaders has been added. This method is
called exec_in_module and takes two positional arguments:
module spec and already initialized module. It then passes these
arguments to new C function inside the _imp module called same way.



C-level changes
---------------

On C level, there have been some additions in form of new functions
and again, one split.


Python/importdl.c
'''''''''''''''''

_PyImport_LoadDynamicModuleWithSpec has been split into two functions,
one has the old name and always returns a module. The second,
PyImport_LoadDynamicModuleDef, is called by the first one and
returns what the PyInit* function returns: either a fully initialized
module (for single-phase init) or PyModuleDef (for multi-phase init).
_PyImport_LoadDynamicModuleWithSpec then typechecks the result of
PyImport_LoadDynamicModuleDef and either initializes a new
module (def), or just passes the result upwards (module).


Python/import.c
'''''''''''''''

A new function has been added.

This function is called exec_in_module and takes two arguments:
a spec, and a module in which's namespace should be the code
executed. This function finds a module def using newly created
function "PyImport_LoadDynamicModuleDef" passing it the given spec.

If a module object is returned, an exception is raised
since this is not feasible for single-phased initialization.

If a def is returned, PyModule_ExecInModule (described below) is
called and passed the namespace module and spec.


Objects/moduleobject.c
''''''''''''''''''''''

There is also just one change in this file.
A function called PyModule_ExecInModule, this function receives
module and def as arguments. Then it checks if module is really
a module. If not, exception is raised. Exception is raised even
if the module has already been initialized, so **make sure** the
module does **not** have any **module state** assigned yet.
After all of this, the function checks if there is any Py_mod_create
slot present, if there is, exception is raised.

Finally, if everything went alright, the function assigns methods and
doc string and calls PyModule_ExecDef using the passed-in arguments.


Header Files
''''''''''''

Header files have been modified so they satisfy the need of using
newly implemented functions.


Motivation
==========




Rationale
=========




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
.. _GitHub: https://github.com/Traceur759/cpython/tree/main_c_modules_namespace


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
