.. _xpack_rbac:

X-Pack plugin and RBAC
========================

Since X-Pack provides us the capabilities of role base access control we can use it to manage Kibana on the right way. 
The default X-Pack configuration uses the **elastic** user to make all. This is not the right way on a production environment.

.. warning:: The **elastic** user is a superuser so it's better to create users with the right roles.

Kibana system user
------------------

We need to create a user to be used by Kibana to connect to Elasticsearch. It also will start the whole plugins installed along X-Pack plugin. Finally it will fetch data related to Wazuh from Elasticsearch and it will write data to Elasticsearch as well.

.. note:: This user will use two roles: **wazuh-admin** and the pre-built role named **kibana_system**. The **wazuh-admin** role will be used to handle data related to Wazuh and the **kibana_system** role will be used by Kibana itself.

1. Defining the wazuh-admin role

At cluster level, it will need the following privileges:

+------------------------------------------------------------------------+-------------------------------------------------------------+
|Cluster privileges                                                      | Check                                                       |
+========================================================================+=============================================================+
|manage                                                                  | **Yes**                                                     |
+------------------------------------------------------------------------+-------------------------------------------------------------+
|manage_index_templates                                                  | **Yes**                                                     |
+------------------------------------------------------------------------+-------------------------------------------------------------+


At index level, it will need the following privileges:

+------------------------------------------------------------------------+-------------------------------------------------------------+
|Indices                                                                 | Privileges                                                  |
+========================================================================+=============================================================+
|.old-wazuh                                                              | **all**                                                     |
+------------------------------------------------------------------------+-------------------------------------------------------------+
|.wazuh                                                                  | **all**                                                     |
+------------------------------------------------------------------------+-------------------------------------------------------------+
|.wazuh-version                                                          | **all**                                                     |
+------------------------------------------------------------------------+-------------------------------------------------------------+
|wazuh-*                                                                 | **all**                                                     |
+------------------------------------------------------------------------+-------------------------------------------------------------+

Wazuh API manager user
------------------

We need a new user who will be able to login through the Kibana UI and add/delete Wazuh API entries too. 

.. note:: This user will use two roles: **wazuh-basic** and **wazuh-api-admin**. The **wazuh-admin** role will be used to handle data related to Wazuh and the **wazuh-api-admin** role will be used to add/delete Wazuh API entries.

1. Defining the wazuh-basic role:

At cluster level, it won't need any privileges. At index level, it will need the following privileges:

+------------------------------------------------------------------------+-------------------------------------------------------------+
|Indices                                                                 | Privileges                                                  |
+========================================================================+=============================================================+
|.kibana                                                                 | **read**                                                    |
+------------------------------------------------------------------------+-------------------------------------------------------------+
|.wazuh                                                                  | **read**                                                    |
+------------------------------------------------------------------------+-------------------------------------------------------------+
|.wazuh-version                                                          | **read**                                                    |
+------------------------------------------------------------------------+-------------------------------------------------------------+
|wazuh-alerts-3.x-*                                                      | **read**                                                    |
+------------------------------------------------------------------------+-------------------------------------------------------------+
|wazuh-monitoring-3.x-*                                                  | **read**                                                    |
+------------------------------------------------------------------------+-------------------------------------------------------------+

2. Defining the wazuh-api-admin role:

At cluster level, it won't need any privileges. At index level, it will need the following privileges:

+------------------------------------------------------------------------+-------------------------------------------------------------+
|Indices                                                                 | Privileges                                                  |
+========================================================================+=============================================================+
|.wazuh                                                                  | **all**                                                     |
+------------------------------------------------------------------------+-------------------------------------------------------------+

Wazuh standard user
------------------

Finally we need one or more users who will be able to login through the Kibana UI with read privileges only. This user only needs
to use the wazuh-basic role. 

How your environment should looks like?
------------------

Take a look at the following table, it should looks like your environment:

+------------------------------------------------------------------------+-------------------------------------------------------------+
|User                                                                    | Roles                                                       |
+========================================================================+=============================================================+
|Kibana system user                                                      | **wazuh-admin**, **kibana_system**                          |
+------------------------------------------------------------------------+-------------------------------------------------------------+
|Wazuh API manager user                                                  | **wazuh-basic**, **wazuh-api-admin**                        |
+------------------------------------------------------------------------+-------------------------------------------------------------+
|Wazuh standard user #1, Wazuh standard user #2...                       | **wazuh-basic**                                             |
+------------------------------------------------------------------------+-------------------------------------------------------------+

How to configure through Kibana with the `elastic` user
------------------

How to configure without Kibana with the `elastic` user
------------------