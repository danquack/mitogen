
.. _changelog:

Release Notes
=============


.. raw:: html

    <style>
        div#release-notes h2 {
            border-bottom: 1px dotted #c0c0c0;
            margin-top: 40px;
        }
    </style>


v0.2.2 (2018-07-??)
-------------------

Mitogen for Ansible
~~~~~~~~~~~~~~~~~~~

* `#291 <https://github.com/dw/mitogen/issues/291>`_: compatibility:
  ``ansible_*_interpreter`` variables are parsed using UNIX hashbang syntax,
  i.e. with support for a single space-separated argument. This supports a
  common idiom where ``ansible_python_interpreter`` is set to ``/usr/bin/env
  python``.

* `#299 <https://github.com/dw/mitogen/issues/299>`_: fix the ``network_cli``
  connection type when the Mitogen strategy is active.

* `#303 <https://github.com/dw/mitogen/pull/303>`_: the ``doas`` become method
  is now supported. Contributed by Mike Walker.

Core Library
~~~~~~~~~~~~

* `#291 <https://github.com/dw/mitogen/issues/291>`_: the ``python_path``
  paramater may specify an argument vector prefix rather than a single string
  program path.

* `#303 <https://github.com/dw/mitogen/pull/303>`_: the ``doas`` become method
  is now supported. Contributed by Mike Walker.

* `#307 <https://github.com/dw/mitogen/issues/307>`_: SSH login banner output
  containing the word 'password' is no longer confused for a password prompt.

* Debug logs containing command lines are printed with the minimal quoting and
  escaping required.


v0.2.1 (2018-07-10)
-------------------

Mitogen for Ansible
~~~~~~~~~~~~~~~~~~~

* `#297 <https://github.com/dw/mitogen/issues/297>`_: compatibility: local
  actions set their working directory to that of their defining playbook, and
  inherit a process environment as if they were executed as a subprocess of the
  forked task worker.


v0.2.0 (2018-07-09)
-------------------

Mitogen 0.2.x is the inaugural feature-frozen branch eligible for fixes only,
except for problem areas listed as in-scope below. While stable from a
development perspective, it should still be considered "beta" at least for the
initial releases.

**In Scope**

* Python 3.x performance improvements
* Subprocess reaping improvements
* Major documentation improvements
* PyPI/packaging improvements
* Test suite improvements
* Replacement CI system to handle every supported OS
* Minor deviations from vanilla Ansible behaviour
* Ansible ``raw`` action support

The goal is a *tick/tock* model where even-numbered series are a maturation of
the previous unstable series, and unstable series are released on PyPI with
``--pre`` enabled. The API and user visible behaviour should remain unchanged
within a stable series.


Mitogen for Ansible
~~~~~~~~~~~~~~~~~~~

* Support for Ansible 2.3 - 2.5.x and any mixture of Python 2.6, 2.7 or 3.6 on
  controller and target nodes.

* Drop-in support for many Ansible connection types.

* Preview of Connection Delegation feature.

* Built-in file transfer compatible with connection delegation.


**Known Issues**

* The ``raw`` action executes as a regular Mitogen connection, which requires
  Python on the target, precluding its use for installing Python. This will be
  addressed in a future 0.2 release. For now, simply mix Mitogen and vanilla
  Ansible strategies in your playbook:

  .. code-block:: yaml

    - hosts: web-servers
      strategy: linear
      tasks:
      - name: Install Python if necessary.
        raw: test -e /usr/bin/python || apt install -y python-minimal

    - hosts: web-servers
      strategy: mitogen_linear
      roles:
      - nginx
      - initech_app
      - y2k_fix

* When running with ``-vvv``, log messages such as *mitogen: Router(Broker(0x7f5a48921590)): no route
  for Message(..., 102, ...), my ID is ...* may be visible. These are due to a
  minor race while initializing logging and can be ignored.

* Performance does not scale linearly with target count. This requires
  significant additional work, as major bottlenecks exist in the surrounding
  Ansible code. Performance-related bug reports for any scenario remain
  welcome with open arms.

* Performance on Python 3 is significantly worse than on Python 2. While this
  has not yet been investigated, at least some of the regression appears to be
  part of the core library, and should therefore be straightforward to fix as
  part of 0.2.x.

* *Module Replacer* style Ansible modules are not supported.

* Actions are single-threaded for each `(host, user account)` combination,
  including actions that execute on the local machine. Playbooks may experience
  slowdown compared to vanilla Ansible if they employ long-running
  ``local_action`` or ``delegate_to`` tasks delegating many target hosts to a
  single machine and user account.

* Connection Delegation remains in preview and has bugs around how it infers
  connections. Connection establishment will remain single-threaded for the 0.2
  series, however connection inference bugs will be addressed in a future 0.2
  release.

* Connection Delegation does not support automatic tunnelling of SSH-dependent
  actions, such as the ``synchronize`` module. This will be addressed in the
  0.3 series.

* Configurations will break that rely on the `hashbang argument splitting
  behaviour <https://github.com/ansible/ansible/issues/15635>`_ of the
  ``ansible_python_interpreter`` setting, contrary to the Ansible
  documentation. This will be addressed in a future 0.2 release.


Core Library
~~~~~~~~~~~~

* Synchronous connection establishment via OpenSSH, sudo, su, Docker, LXC and
  FreeBSD Jails, local subprocesses and :func:`os.fork`. Parallel connection
  setup is possible using multiple threads. Connections may be used from one or
  many threads after establishment.

* UNIX masters and children, with Linux, MacOS, FreeBSD, NetBSD, OpenBSD and
  Windows Subsystem for Linux explicitly supported.

* Automatic tests covering Python 2.6, 2.7 and 3.6 on Linux only.


**Known Issues**

* Serialization is still based on :mod:`pickle`. While there is high confidence
  remote code execution is impossible in Mitogen's configuration, an untrusted
  context may at least trigger disproportionately high memory usage injecting
  small messages (*"billion laughs attack"*). Replacement is an important
  future priority, but not critical for an initial release.

* Child processes are not reliably reaped, leading to a pileup of zombie
  processes when a program makes many short-lived connections in a single
  invocation. This does not impact Mitogen for Ansible, however it limits the
  usefulness of the core library. A future 0.2 release will address it.

* Some races remain around :class:`mitogen.core.Broker <Broker>` destruction,
  disconnection and corresponding file descriptor closure. These are only
  problematic in situations where child process reaping is also problematic.

* The `fakessh` component does not shut down correctly and requires flow
  control added to the design. While minimal fixes are possible, due to the
  absence of flow control the original design is functionally incomplete.

* The multi-threaded :ref:`service` remains in a state of design flux and
  should be considered obsolete, despite heavy use in Mitogen for Ansible. A
  future replacement may be integrated more tightly with, or entirely replace
  the RPC dispatcher on the main thread.

* Documentation is in a state of disrepair. This will be improved over the 0.2
  series.
