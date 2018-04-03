.. _xpack_rbac:

X-Pack plugin and RBAC
========================

Since X-Pack provides us the capabilities of RBAC (role base access control) we can use it to manage Kibana on the right way. 
The default X-Pack configuration uses the **elastic** user to make all. This is not the right way on a production environment.

.. warning:: The **elastic** user is a superuser so it's better to create users with the right roles.

Preparation
-----------------

.. note:: Follow the official Elastic guide https://www.elastic.co/downloads/x-pack for a more in deep explanation.

On a default installation you could follow the next steps to prepare the environment:

1. Install X-Pack plugin for Elasticsearch:

    .. code-block:: console

        # /usr/share/elasticsearch/bin/elasticsearch-plugin install x-pack

2. Restart Elasticsearch:

    .. code-block:: console

        # systemctl start elasticsearch

3. Generate the credentials and note down them:

    .. code-block:: console

        # /usr/share/elasticsearch/bin/x-pack/setup-passwords auto

4. Install X-Pack plugin for Kibana:

    .. code-block:: console

        # /usr/share/kibana/bin/kibana-plugin install x-pack

5. Set temporary the `elastic` user for Kibana, edit /etc/kibana/kibana.yml as follow:

    .. code-block:: console

        elasticsearch.username: "elastic"
        elasticsearch.password: "elastic_password_from_step3"

6. Restart Kibana

    .. code-block:: console

        # systemctl restart kibana

7. Login the Kibana UI using the `elastic` user too.

Kibana system user
------------------

We need to create a user to be used by Kibana to connect to Elasticsearch. It also will start the whole plugins installed along X-Pack plugin. Finally it will fetch data related to Wazuh from Elasticsearch and it will write data to Elasticsearch as well.

.. note:: This user will use two roles: **wazuh-admin** and the pre-built role named **kibana_system**. The **wazuh-admin** role will be used to handle data related to Wazuh and the **kibana_system** role will be used by Kibana itself.

1. Defining the wazuh-admin role

    a) At cluster level, it will need the following privileges:

    +------------------------------------------------------------------------+-------------------------------------------------------------+
    |Cluster privileges                                                      | Check                                                       |
    +========================================================================+=============================================================+
    |manage                                                                  | **Yes**                                                     |
    +------------------------------------------------------------------------+-------------------------------------------------------------+
    |manage_index_templates                                                  | **Yes**                                                     |
    +------------------------------------------------------------------------+-------------------------------------------------------------+


    b) At index level, it will need the following privileges:

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

    a) At cluster level, it won't need any privileges. At index level, it will need the following privileges:

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

    a) At cluster level, it won't need any privileges. At index level, it will need the following privileges:

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

1. Login on Kibana using the `elastic` user:

    .. thumbnail:: ../../images/x-pack/xpack1.png
        :title: Data flow
        :align: center
        :width: 40%

2. Go to Management > Security > Roles:

    .. thumbnail:: ../../images/x-pack/xpack2.png
        :title: Data flow
        :align: center
        :width: 100%

3. Creating the **wazuh-admin** role:

    .. thumbnail:: ../../images/x-pack/xpack3.png
        :title: Data flow
        :align: center
        :width: 100%

4. Creating the **wazuh-basic** role:

    .. thumbnail:: ../../images/x-pack/xpack4.png
        :title: Data flow
        :align: center
        :width: 100%

5. Creating the **wazuh-api-admin** role:

    .. thumbnail:: ../../images/x-pack/xpack5.png
        :title: Data flow
        :align: center
        :width: 100%

6. Go to Management > Security > Users:

    .. thumbnail:: ../../images/x-pack/xpack6.png
        :title: Data flow
        :align: center
        :width: 100%

7. Creating the Wazuh API manager user:

    .. thumbnail:: ../../images/x-pack/xpack7.png
        :title: Data flow
        :align: center
        :width: 100%

8. Creating a standard user:

    .. note:: This user is not able to add/remove/edit a Wazuh API, use the Wazuh API manager user instead (step 7).

    .. thumbnail:: ../../images/x-pack/xpack8.png
        :title: Data flow
        :align: center
        :width: 100%

9. Creating the Kibana system user:

    .. note:: Ensure the password is enough strong, it will be the superuser for your environment.

    .. thumbnail:: ../../images/x-pack/xpack9.png
        :title: Data flow
        :align: center
        :width: 100%

10. Set the right user on `kibana.yml` file:

    .. code-block:: console

        # vi /etc/kibana/kibana.yml

        elasticsearch.username: "wazuhsystem"
        elasticsearch.password: "wazuhsystem"

11. Restart Kibana:

    .. code-block:: console

        # systemctl restart kibana


How to configure out of Kibana with the `elastic` user
------------------

.. note:: Before configure the roles and users you must to install X-Pack.

1. Creating the **wazuh-admin** role:

    .. code-block:: console
    
        # curl -XPOST "http://localhost:9200/_xpack/security/role/wazuh-admin" -H 'Content-Type: application/json' -d'
        {
        "cluster": [ "manage", "manage_index_templates" ],
        "indices": [
            {
            "names": [ ".old-wazuh", ".wazuh", ".wazuh-version", "wazuh-*" ],
            "privileges": ["all"]
            }
        ]
        }' -u elastic:elastic_password

        {"role":{"created":true}}

2. Creating the **wazuh-basic** role:

    .. code-block:: console

        # curl -XPOST "http://localhost:9200/_xpack/security/role/wazuh-basic" -H 'Content-Type: application/json' -d'
        {
        "cluster": [],
        "indices": [
            {
            "names": [ ".kibana", ".wazuh", ".wazuh-version", "wazuh-alerts-3.x-*", "wazuh-monitoring-3.x-*" ],
            "privileges": ["read"]
            }
        ]
        }' -u elastic:elastic_password

        {"role":{"created":true}}

3. Creating the **wazuh-api-admin** role:

    .. code-block:: console

        # curl -XPOST "http://localhost:9200/_xpack/security/role/wazuh-api-admin" -H 'Content-Type: application/json' -d'
        {
        "cluster": [],
        "indices": [
            {
            "names": [ ".wazuh" ],
            "privileges": ["all"]
            }
        ]
        }' -u elastic:elastic_password

        {"role":{"created":true}}

4. Creating the Kibana system user:

    .. note:: Ensure the password is enough strong, it will be the superuser for your environment.

    .. code-block:: console

        # curl -XPOST "http://localhost:9200/_xpack/security/user/wazuhsystem" -H 'Content-Type: application/json' -d'
        {
            "password": "wazuhsystem",
            "roles":["wazuh-admin","kibana_system"],
            "full_name":"Wazuh System",
            "email":"wazuhsystem@wazuh.com"                           
        }' -u elastic:elastic_password

        {"user":{"created":true}}

5. Creating the Wazuh API manager user:

    .. code-block:: console

        # curl -XPOST "http://localhost:9200/_xpack/security/user/jack" -H 'Content-Type: application/json' -d'
        {
            "password": "jackjack",
            "roles":["wazuh-basic","wazuh-api-admin"],
            "full_name":"Jack",
            "email":"jack@wazuh.com"                           
        }' -u elastic:elastic_password

        {"user":{"created":true}}

6. Creating a standard user:

    .. note:: This user is not able to add/remove/edit a Wazuh API, use the Wazuh API manager user instead (step 5).

    .. code-block:: console

        # curl -XPOST "http://localhost:9200/_xpack/security/user/john" -H 'Content-Type: application/json' -d'
        {
            "password": "johnjohn",
            "roles":["wazuh-basic"],
            "full_name":"John",
            "email":"john@wazuh.com"                           
        }' -u elastic:elastic_password

        {"user":{"created":true}}

7. Set the right user on `kibana.yml` file:

    .. code-block:: console

        # vi /etc/kibana/kibana.yml

        elasticsearch.username: "wazuhsystem"
        elasticsearch.password: "wazuhsystem"

8. Restart Kibana:

    .. code-block:: console

        # systemctl restart kibana