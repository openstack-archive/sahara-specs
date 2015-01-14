..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=====================================
Scenario integration tests for Sahara
=====================================

https://blueprints.launchpad.net/sahara/+spec/scenario-integration-tests

For now the Sahara project has not functional integration tests.
We need to create new integration tests that will be more flexible.


Problem description
===================

Current integration tests allow to test only limited number of test scenarios
and cluster configurations that are hardcoded in code of tests.
In many cases we should test various configurations of Sahara clusters but
current integration tests don't have this functionality. Also we have many
copy-pasted code in test files and this code requires large work
for its refactoring.


Proposed change
===============

It is proposed to create new integration tests that will be more flexible.
Test scenarios will be defined in YAML files and it is supposed this approach
will provide more flexibility in testing. The usual scenario will have the
following look:

.. sourcecode:: yaml

    credentials:
        os_username: dev-user
        os_password: swordfish
        os_tenant: devs
        os_auth_url: http://os_host:5000/v2.0
        sahara_url: http://sahara_host:8336/v1.1  # optional

    network:
        type: neutron  # or nova-network
        private_network: private  # for neutron
        auto_assignment_floating_ip: false  # for nova-network
        public_network: public  # or floating_ip_pool for nova-network

    clusters:
        - plugin_name: vanilla
          plugin_version: 2.6.0
          image: some_id
          node_group_templates:  # optional
            - name: master
              node_processes:
                - namenode
                - resourcemanager
              flavor_id: '3'
            - name: worker
              node_processes:
                - datanode
                - nodemanager
              flavor_id: '3'
          cluster_template:  # optional
              name: vanilla
              node_group_templates:
                  master: 1
                  worker: 3
          scenario:  # optional
              - run_jobs
              - scale
              - run_jobs

        - plugin_name: hdp
          plugin_version: 2.0.6
          image: some_id


Minimal scenario will look the following way:

.. sourcecode:: yaml

    clusters:
        - plugin_name: vanilla
          plugin_version: 2.6.0
          image: some_id


Full scenario will look the following way:

.. sourcecode:: yaml

    concurrency: 3

    credentials:
        os_username: dev-user
        os_password: swordfish
        os_tenant: devs
        os_auth_url: http://os_host:5000/v2.0
        sahara_url: http://sahara_host:8336/v1.1  # optional

    network:
        type: neutron  # or nova-network
        private_network: private  # for neutron
        auto_assignment_floating_ip: false  # for nova-network
        public_network: public  # or floating_ip_pool for nova-network

    clusters:
          # required
        - plugin_name: vanilla
          # required
          plugin_version: 2.6.0
          # required (id or name)
          image: some_id
          node_group_templates:  # optional
            - name: master
              node_processes:
                - namenode
                - resourcemanager
              flavor_id: '3'
              description: >-
                  Some description
              volumes_per_node: 2
              volumes_size: 2
              node_configs:
                  HDFS:
                      dfs.datanode.du.reserved: 10
              security_groups: ~
              auto_security_group: true
              availability_zone: nova
              volumes_availability_zone: nova
              volume_type: lvm
            - name: worker
              node_processes:
                - datanode
                - nodemanager
              flavor_id: 3
          cluster_template:  # optional
              name: vanilla
              description: >-
                  Some description
              cluster_configs:
                  HDFS:
                      dfs.replication: 1
              node_group_templates:
                  master: 1
                  worker: 3
              anti_affinity: true
          cluster:
              name: test-cluster
              is_transient: true
              description: >-
                  Cluster description
          scaling:
              - operation: resize
                node_group: worker
                size: 4
              - operation: add
                node_group: worker
                size: 2
          scenario:  # optional
              - run_jobs
              - scale
              - run_jobs
          edp_jobs_flow: example
          retain_resource: true  # optional

    edp_jobs_flow:
        example:
          - type: Pig
            main_lib:
                source: swift
                path: path_to_pig_script.pig
            input_datasource:
                type: swift
                source: etc/edp-examples/edp-pig/top-todoers/data/input
            output_datasource:
                type: hdfs
                destination: /user/hadoop/edp-output
            configs:
                dfs.replication: 1
          - type: Java
            additional_libs:
                - type: database
                  source: |
                      etc/edp-examples/.../hadoop-mapreduce-examples-2.4.1.jar
            configs:
                edp.java.main_class: |
                    org.apache.hadoop.examples.QuasiMonteCarlo
            args:
                - 10
                - 10


After we described test scenario in YAML file we run test as usual.
The python test code will be generated from these YAML files.

We will use the Mako library to generate the python code. The generated code
will look like:

.. sourcecode:: python

    from sahara.tests.scenario import base


    class vanilla2_4_1TestCase(base.BaseTestCase):
        @classmethod
        def setUpClass(cls):
            super(vanilla2_4_1TestCase, cls).setUpClass()
            cls.credentials = {
                'os_username': 'dev-user',
                'os_password': 'swordfish',
                'os_tenant': 'devs',
                'os_auth_url': 'http://172.18.168.5:5000/v2.0',
                'sahara_url': None
            }
            cls.network = {
                'type': 'neutron',
                'public_network': 'net04_ext',
                'auto_assignment_floating_ip': False,
                'private_network': 'dev-network'
            }
            cls.testcase = {
                'image': 'sahara-juno-vanilla-2.4.1-ubuntu-14.04',
                'plugin_name': 'vanilla',
                'retain_resources': False,
                'class_name': 'vanilla2_4_1',
                'edp_jobs_flow': [
                    {
                        'configs': {
                            'dfs.replication': 1
                        },
                        'output_datasource': {
                            'type': 'hdfs',
                            'destination': '/user/hadoop/edp-output'
                        },
                        'input_datasource': {
                            'type': 'swift',
                            'source':
                            'etc/edp-examples/edp-pig/top-todoers/data/input'
                        },
                        'main_lib': {
                            'type': 'swift',
                            'source':
                            'etc/edp-examples/edp-pig/top-todoers/example.pig'
                        },
                        'type': 'Pig'
                    },
                    {
                        'type': 'Java',
                        'args': [10, 10],
                        'additional_libs': [
                            {
                                'type': 'database',
                                'source':
                                'etc/edp-examples/hadoop2/edp-java/'
                                'hadoop-mapreduce-examples-2.4.1.jar'
                            }
                        ],
                        'configs': {
                            'edp.java.main_class':
                            'org.apache.hadoop.examples.QuasiMonteCarlo'
                        }
                    }
                ],
                'scenario': ['run_jobs', 'scale', 'run_jobs'],
                'plugin_version': '2.4.1'
            }

        def test_plugin(self):
            self.create_cluster()
            self.check_run_jobs()
            self.check_scale()
            self.check_run_jobs()


    class hdp2_0_6TestCase(base.BaseTestCase):
        @classmethod
        def setUpClass(cls):
            super(hdp2_0_6TestCase, cls).setUpClass()
            cls.credentials = {
                'os_username': 'dev-user',
                'os_password': 'swordfish',
                'os_tenant': 'devs',
                'os_auth_url': 'http://172.18.168.5:5000/v2.0',
                'sahara_url': None
            }
            cls.network = {
                'type': 'neutron',
                'public_network': 'net04_ext',
                'auto_assignment_floating_ip': False,
                'private_network': 'dev-network'
            }
            cls.testcase = {
                'image': 'f3c4a228-9ba4-41f1-b100-a0587689d4dd',
                'plugin_name': 'hdp',
                'retain_resources': False,
                'class_name': 'hdp2_0_6',
                'edp_jobs_flow': None,
                'scenario': ['run_jobs', 'scale', 'run_jobs'],
                'scaling': [
                    {
                        'operation': 'resize',
                        'size': 5,
                        'node_group': 'worker'
                    }
                ],
                'plugin_version': '2.0.6'
            }

        def test_plugin(self):
            self.create_cluster()
            self.check_run_jobs()
            self.check_scale()
            self.check_run_jobs()


Mako template will look the following way:

.. sourcecode:: mako

    from sahara.tests.scenario import base

    % for testcase in testcases:
        ${make_testcase(testcase)}
    % endfor

    <%def name="make_testcase(testcase)">
    class ${testcase['class_name']}TestCase(base.BaseTestCase):
        @classmethod
        def setUpClass(cls):
            super(${testcase['class_name']}TestCase, cls).setUpClass()
            cls.credentials = ${credentials}
            cls.network = ${network}
            cls.testcase = ${testcase}

        def test_plugin(self):
            self.create_cluster()
        % for check in testcase['scenario']:
            self.check_${check}()
        % endfor
    </%def>


By default concurrency will be equal to number of cpu cores. This value
can be changed in YAML file.

We are going to use new integration tests for CI as soon as they are completely
implemented.


Alternatives
------------

We can use current integration tests for further testing Sahara but they
don't have sufficient coverage of Sahara use cases.


Data model impact
-----------------

None


REST API impact
---------------

None


Other end user impact
---------------------

None


Deployer impact
---------------

None


Developer impact
----------------

Developers will be able much better to test their changes in Sahara.


Sahara-image-elements impact
----------------------------

None


Sahara-dashboard / Horizon impact
---------------------------------

None


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  sreshetniak

Other contributors:
  ylobankov
  slukjanov


Work Items
----------

The work items will be the following:

* Add python code to sahara/tests
* Add examples of test scenarios
* Add documentation for new integration tests


Dependencies
============

Mako
tempest-lib


Testing
=======

None


Documentation Impact
====================

We need to add a note about new tests in Sahara documentation.


References
==========

None
