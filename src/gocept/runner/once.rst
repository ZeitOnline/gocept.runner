Run once commands
=================

It is often needed to run a command once (or via cron) with the full component
architecture. Usually ``zopectl run`` is used for this.

>>> import logging
>>> import gocept.runner
>>> import zope.component.hooks
>>> @gocept.runner.once()
... def worker():
...     log = logging.getLogger('test')
...     log.info("Working.")
...     site = zope.component.hooks.getSite()
...     if hasattr(site, 'store'):
...         log.info("Having attribute store.")
...     site.store = True


Define a Zope environment:

>>> import os.path
>>> import tempfile
>>> zodb_path = tempfile.mkdtemp()
>>> site_zcml = os.path.join(
...     os.path.dirname(__file__), 'ftesting.zcml')
>>> fd, zope_conf = tempfile.mkstemp()
>>> zope_conf_file = os.fdopen(fd, 'w')
>>> _ = zope_conf_file.write('''\
... site-definition %s
... <zodb>
...   <filestorage>
...     path %s/Data.fs
...   </filestorage>
... </zodb>
... <product-config test>
...     foo bar
... </product-config>
... <eventlog>
...   <logfile>
...     formatter zope.exceptions.log.Formatter
...     path STDOUT
...   </logfile>
... </eventlog>
... ''' % (site_zcml, zodb_path))
>>> zope_conf_file.close()


So call the worker for the first time. It will be terminated after one call:

>>> worker(None, zope_conf)
------
... INFO test Working.


Calling it a second time indicates that a property was changed in the first
run:

>>> worker(None, zope_conf)
------
... INFO test Working.
------
... INFO test Having attribute store.
...
