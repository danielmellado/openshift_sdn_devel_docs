2. Edit the ``/etc/openshift_devel_docs/openshift_devel_docs.conf`` file and complete the following
   actions:

   * In the ``[database]`` section, configure database access:

     .. code-block:: ini

        [database]
        ...
        connection = mysql+pymysql://openshift_devel_docs:OPENSHIFT_DEVEL_DOCS_DBPASS@controller/openshift_devel_docs
