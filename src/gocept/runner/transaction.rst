Multiple commit helper
======================


Sometimes you have several actions and want to commit in between each step.
``gocept.runner`` provides a decorator ``transaction_per_item`` that simplifies
writing this down, for example::

    @gocept.runner.transaction_per_item
    def multiple_steps():
        for item in source:
            yield lambda: process(item)

This will call each ``process(item)`` in turn and commit after each. If an
exception is raised by a step, the transaction is aborted and the loop
continues with the next step. ZODB ConflictErrors are ignored likewise.
