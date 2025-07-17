.. _aci_dev_guide:

****************************
Developing Cisco ACI modules
****************************
This is a developer guide for contributing modules to the Cisco ACI-Ansible collection. It is for developers who wish to contribute code, fixes, or new modules to the collection. This document will outline various steps and the expected conventions for contributions.

For more information about Cisco ACI, consult the `Cisco ACI user guide <https://www.cisco.com/c/en/us/solutions/collateral/data-center-virtualization/application-centric-infrastructure/solution-overview-c22-741487.html>`_.

What is covered in this section:

.. contents::
  :depth: 4
  :local:

.. _aci_dev_guide_intro:

Introduction
============
The `cisco.aci collection <https://galaxy.ansible.com/cisco/aci>`_ already includes a large number of Cisco ACI modules; however, the ACI object model is extensive, and covering all possible functionality would easily require more than 1,500 individual modules. Therefore, Cisco develops modules requested on a just-in-time basis.

If a specific functionality is required, there are three options:

- Open an issue using https://github.com/CiscoDevNet/ansible-aci/issues/new/choose so that Cisco developers can build, enhance, or fix the modules.
- Learn the ACI object model and utilize the low-level APIC REST API using the `aci_rest <https://docs.ansible.com/ansible/latest/collections/cisco/aci/aci_rest_module.html>`_ module.
- Contribute to Cisco's ansible-aci project by writing dedicated modules, proposing a fix or an enhancement, and becoming part of the Cisco ansible-aci community.

.. _aci_dev_guide_git:

This guide will concentrate on the third option to demonstrate how to build a module, fix an issue, or improve an existing module and contribute it back to the ansible-aci project. The initial step in this process is to retrieve the latest version of the ansible-aci collection code. By retrieving the latest version, one will be able to modify existing code.

Development Environment
======================
The collection code is located in a git repository (https://github.com/CiscoDevNet/ansible-aci). 

Fork, Clone and Branch
-----------------------
One can directly clone this repository to retrieve the latest version of the code, but to later contribute code back to the project, it will be necessary to create a fork to enable a proper pull request.

It is recommended to fork the repository first, then clone the forked copy to the local machine and create a branch to push the changes while keeping the master branch clean. For further information, refer to the `Fork, Clone and Branch <fork_clone_branch_collection>`_ section.

Tools required for installation and build
------------------------------------------------

Refer the documentation in the `README.md <https://github.com/CiscoDevNet/ansible-aci?tab=readme-ov-file#ansible-aci>`_ file of the collection repository.

.. _aci_dev_guide_module_structure:

ACI module structure
====================

    **Note**: Copy the module outline from aci_module_name.py in the docs/docsite/rst/sample_module directory and use it as a template for a new module. This will ensure that the module follows the same structure and conventions as the existing modules in the collection.
    If the python file is copied, ensure to rename the file_name to match the name of the module (aci_module_name.py => aci_l3out_static_routes.py).

The structure of the cisco.aci collection contains an ansible-aci repository which consists of directories and files. The repository in detail is explained in the `cisco_aci_collection_structure <cisco_aci_collection_structure>`_ section.

Building a module
------------------
Having explained and reviewed the components of the ACI module structure, this guide will now proceed to build a module. The following section demonstrates a basic and practical approach to building a module with the help of an existing module. This approach simplifies the creation of a new module without requiring everything to be written from scratch.

The purpose of this section is to illustrate how to build a module based on an existing module. This is achieved by using the template provided in aci_module_name.py in the docs/docsite/rst/sample_module. A module similar to the one intended for creation should be selected to reuse the required sections. For this, one can either take the parent object and append the attributes required for the module.

1. In the modules directory located in the plugins directory of the collection, create a new file in .py format or use the template provided in aci_module_name.py in the docs/docsite/rst/sample_module directory. To create a name for the new module, always use the format aci_<name_of_module>.

    The first 15 lines of the new module should include the same import statements and metadata as any other module. The only change here is in the copyright section.

**Note**: For reference, select any module from the collection that is at the same level as the new module or the parent module.

2. Change the copyright section by replacing <year> by current year, <name> by author's name and <author_github_handle> by author's github handle:

.. code-block:: python

  #!/usr/bin/python
  # -*- coding: utf-8 -*-

  # Copyright: (c) <year>, <Name> (<author_github_handle>)
  # GNU General Public License v3.0+ (see LICENSE or https://www.gnu.org/licenses/gpl-3.0.txt)

  from __future__ import absolute_import, division, print_function
  __metaclass__ = type

  ANSIBLE_METADATA = {
      'metadata_version': '1.1',
      'status': ['preview'],
      'supported_by': 'community'
  }

Documentation Section
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

3. In the documentation section, begin by changing the name of the module, its short description, and the description of the functions being performed on the object. The description of the module must be followed by the options, which is a list of attributes. Each attribute should include the name, description, data type, aliases (if applicable), choices (if applicable), and default (if applicable) of all the parameters that will be consumed by the object.
 * The options section includes all the parameters that will be defined in the argument_spec, such as the object_id, configurable properties of the object, parent object_id, state, etc., and these need to be documented in the same file as the module in the DOCUMENTATION section.
    + Description must be clear and concise, providing enough detail for users to understand the purpose and usage of the object.
    + Description must include specific details about the object, such as its purpose, how it is used, and any important considerations.
    + For example,
        + The APIC defaults to C(default_value) when unset during creation. Explains that when an object value is not explicitly provided in a task, the APIC automatically assigns a default value to that object.
        + The object_prop1 must be a valid choice from the list: [choice1, choice2, choice3]. This explains that the values for object_prop1 must be one of the specified choices.
        + The object_prop1 must be in the range 1 to 100. The default value is 50.
        + The object_prop3 is only applicable when using 'object_prop2' is set to <specific_value>.
        + default: <xyz> , the default values should not be provided for configuration arguments, unless API adds a default_value to the payload when creating the object. Default values could cause unintended changes to the object.
        + required: true; should be used only for parameters that are mandatory in all the states (present,query,absent) of the module. This ensures that users must provide a value for these parameters when using the module.

    **Note**: If a parameter is required in some states but not in others, then it should **not** be marked as required: true. Instead, it should be added in the argument_spec with the appropriate required_if conditions.

 * The options section must be followed by the extends_documentation_fragment section, which is used to include the common reusable documentation fragments for all ACI modules.
    + **plugins/doc_fragments** directory of the collection contain the common documentation fragments; these are mentioned in the extends_documentation_fragment section.
    + This includes the cisco.aci.aci fragment, which contains the common parameters used in all ACI modules.
        + cisco.aci.annotation is added to the extends_documentation_fragment section if the module supports the annotation parameter.
        + cisco.aci.owner is added to the extends_documentation_fragment section if the module supports the owner parameter.

The format of documentation is shown below:

.. code-block:: yaml

  DOCUMENTATION = r"""
  ---
  module: aci_<name_of_module>
  short_description: Short description of the module being created (config:<name_of_class>).
  description:
  - Functionality one.
  - Functionality two.
  options:
    object_id:
      description:
      - Description of the object.
      type: Data type of object eg. 'str'
      aliases: [ Alternate name of the object ]
    object_prop1:
      description:
      - Description of property one.
      type: Property's data type eg. 'int'
      choices: [ choice one, choice two ]
    object_prop2:
      description:
      - Description of property two.
      - This attribute is only configurable in ACI versions 6.0(2h) and above.
      type: Property's data type eg. 'bool'
    object_prop3:
      description:
      - Description of property three.
      - The APIC defaults to C(default_value) when unset during creation.
      - The object_prop3 is only applicable when using 'object_prop2' is set to <specific_value>.
      - The object_prop3 must be in the range 1 to 100. The default value is 50.
      type: Property's data type eg. 'str'
      required: true
    state:
      description:
      - Use C(present) or C(absent) for adding or removing.
      - Use C(query) for listing an object or multiple objects.
      type: str
      choices: [ absent, present, query ]
      default: present
  extends_documentation_fragment:
  - cisco.aci.aci

4. The options are followed by notes, which usually contain any dependencies of the module being created with the parent modules that exist in the collection. A "see also" section is also included, which provides a link to the class being used in the module, followed by the author's name and GitHub ID as shown below.

.. code-block:: yaml

      notes:
      - The C(root_object), C(parent_object), C(object_prop), used must exist before using this module in your playbook.
        The M(cisco.aci.aci_root_object_module) and M(cisco.aci.parent_object_module) modules can be used for this.
      seealso:
      - module: cisco.aci.aci_root_object_module
      - module: cisco.aci.aci_parent_object_module
      - name: APIC Management Information Model reference
        description: More information about the internal APIC class B(config:<name_of_class>).
        link: https://developer.cisco.com/docs/apic-mim-ref/
      author:
      - <author's name> (<author's github id>)
      """

Examples Section
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

5. The examples section of the copied module should be modified by adding the necessary parameters to all the examples. Please note that removing and querying an object will only contain the object name and no object parameters. "Query All" will not have any parameters other than the one that are set to required, ensuring that all the objects of the class being worked upon are returned.

  - The examples section must consist of Ansible tasks which can be used as a reference to build playbooks.
  - The example section must include CRUD operations that can be performed using the module. It should include examples for adding, updating, querying, and removing an object. Each example should include the required parameters and the expected state of the object.
The format of this section is shown below:

.. code-block:: yaml

  EXAMPLES = r"""
  - name: Add a new object
    cisco.aci.aci_<name_of_module>:
      host: apic
      username: admin
      password: SomeSecretePassword
      object_id: id
      object_prop1: prop1
      object_prop2: prop2
      state: present
    delegate_to: localhost

  - name: Query an object
    cisco.aci.aci_<name_of_module>:
      host: apic
      username: admin
      password: SomeSecretePassword
      object_id: id
      state: query
    delegate_to: localhost

  - name: Query all objects
    cisco.aci.aci_<name_of_module>:
      host: apic
      username: admin
      password: SomeSecretePassword
      state: query
    delegate_to: localhost

  - name: Remove an object
    cisco.aci.aci_<name_of_module>:
      host: apic
      username: admin
      password: SomeSecretePassword
      object_id: id
      state: absent
    delegate_to: localhost
  """

.. note:: Ensure to test the examples since users generally copy and paste examples to use the module.

Return Section
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The RETURN section is used in every module and has the same content, so copy and paste it from any module and do not modify it

.. code-block:: python

  RETURN = r"""
current:
  ...
"""

Importing objects from Python libraries
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

7. The following import section is generally left untouched, but if a shared method is added in the library, it might need to be imported here.

The following imports are standard across ACI modules:

.. code-block:: python

    from ansible.module_utils.basic import AnsibleModule
    from ansible.module_utils.aci.plugins.module_utils.aci import ACIModule, aci_argument_spec


**ansible.module_utils.aci** is used to import the superclass ACIModule and the aci_argument_spec definition from the library aci.py in the module_utils directory mentioned earlier. ACIModule is imported because it has basic functions to make API requests and other capabilities that allow modules to manipulate objects. The aci.py library also contains a generic argument definition called **aci_argument_spec**. It is used by all the modules and allows them to accept shared parameters such as username and password.
  - **aci_annotation_spec** and **aci_owner_spec** are also imported for modules supporting annotation and owner parameters, respectively. Add this only if used by the module.

Similarly, the AnsibleModule is imported, which contains common code for quickly building an Ansible module in Python.

To understand more about the AnsibleModule, refer to the `Ansible documentation <https://docs.ansible.com/ansible/latest/dev_guide/developing_program_flow_modules.html#ansiblemodule>`_.

.. code-block:: python

  # Importing constants for ACI modules when needed.
  # This import is used to access predefined constants and mappings for ACI objects.
  from ansible_collections.cisco.aci.plugins.module_utils.constants import *

- the '*' can be replaced with the specific constants needed, such as:
  from ansible_collections.cisco.aci.plugins.module_utils.constants import FILTER_PORT_MAPPING, IPV4_REGEX

The imported constants from plugins/module_utils/constants.py file define the collection of fixed values and mapping dictionaries used to standardize and normalize for ACI-specific parameters.

Defining the argument_spec variable
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
8. In the main function, the argument_spec variable defines all the arguments necessary for this module and is based on aci_argument_spec. All arguments defined previously in the documentation section are added to this variable.

The **argument_spec** variable is based on **aci_argument_spec** and allows a module to accept additional parameters from the user specific to the module.

To understand what argument_spec is and how it is used, refer to the `Ansible documentation <https://docs.ansible.com/ansible/latest/dev_guide/developing_program_flow_modules.html#argument-spec>`_.

* the object_id (usually the name)
* the configurable properties of the object
* the object_id of each parent up to the root (usually the name)
* The child classes that have a 1-to-1 relationship with the main object do not need their own dedicated module and can be incorporated into the parent module. If the relationship is 1-to-many/many-to-many, this child class will require a dedicated module. In some corner cases, deviations from this pattern might occur.
* the state
  + ``state: absent`` to ensure the object does not exist
  + ``state: present`` to ensure the object and configurations exist; this is also the default
  + ``state: query`` to retrieve information about a specific object or all objects of the class

.. code-block:: python

    def main():
        argument_spec = aci_argument_spec()
        argument_spec.update(
            object_id=dict(type='str', aliases=['name']),
            object_prop1=dict(type='str'),
            object_prop2=dict(type='str', choices=['choice1', 'choice2', 'choice3']),
            object_prop3=dict(type='int'),
            parent_id=dict(type='str'),
            child_object_id=dict(type='str'),
            child_object_prop=dict(type='str'),
            state=dict(type='str', default='present', choices=['absent', 'present', 'query']),
        )

**Note**: It is recommended not to provide default values for configuration arguments. Default values could cause unintended changes to the object.

Using the AnsibleModule object
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The following section creates an instance of AnsibleModule and then adds to the constructor a series of properties such as the argument_spec. The module should support check-mode, which validates the working of a module without making any changes to the ACI object. The first attribute passed to the constructor is ``argument_spec``; the second argument is ``supports_check_mode``. It is highly recommended that every module support check mode in this collection. The last element is required_if, which is used to specify conditional required attributes, and since these modules support querying the APIC for all objects of the module's class, the object/parent IDs should only be required if ``state: absent`` or ``state: present``.

.. code-block:: python

    module = AnsibleModule(
        argument_spec=argument_spec,
        supports_check_mode=True,
        required_if=[
            ['state', 'absent', ['object_id', 'parent_id']],
            ['state', 'present', ['object_id', 'parent_id']],
        ],
    )

9. The required_if variable has the following arguments. These arguments are not set for all states because "Query All" does not require them. However, users are still required to provide these arguments when creating or deleting something. This is why they are included in required_if, which specifies which attributes are required when state is present or absent. If any of the attributes in required_if are missing in the task that adds or deletes the object in the playbook, Ansible will immediately warn the user that the attributes are missing.

Mapping variable definition
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
10. The above instantiation (required for all modules) is followed by code that is used to get attributes from the playbook that correspond to all the properties of objects defined in the main() function above. This is also where validations and string concatenations are performed.

Once the AnsibleModule object has been instantiated as module, the necessary parameter values should be extracted from the ``module.params`` dictionary and all additional data should be validated. Usually, the only parameters that need to be extracted are those related to the ACI object configuration and its child configuration. If integer objects require validation, then the validation should be performed here.

.. code-block:: python

    object_id = module.params.get('object_id')
    object_prop1 = module.params.get('object_prop1')
    object_prop2 = module.params.get('object_prop2')
    object_prop3 = module.params.get('object_prop3')
    if object_prop3 is not None and object_prop3 not in range(x, y):
        module.fail_json(msg='Valid object_prop3 values are between x and (y-1)')
    child_object_id = module.params.get('child_object_id')
    child_object_prop = module.params.get('child_object_prop')
    state = module.params.get("state")

**Note**:
  * Sometimes the APIC will require special characters ([, ], and -) or will use object metadata in the name ("vlanns" for VLAN pools); the module should handle adding special characters or joining multiple parameters in order to keep expected inputs simple.
  * Most type conversions, checks and validations that are done at this level are minimal and are usually done to ensure the the correct formatted data is passed further down the code.

Using the ACIModule object
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The ACIModule class handles most of the logic for the ACI modules. The ACIModule extends the functionality of the AnsibleModule object, so the module instance must be passed into the class instantiation.

.. code-block:: python

    aci = ACIModule(module)

The ACIModule has 7 main methods that are used by most modules in the collection:

* construct_url
* get_existing
* payload
* get_diff
* post_config
* delete_config
* exit_json

The first 2 methods are used regardless of what value is passed to the ``state`` parameter.

Constructing URLs
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
11. The following section constructs a filter to target a set of entries that match certain criteria at the level of the target DN and in the subtree below it. The construct_url function below is used to build the appropriate DN by using the tenant as the root class and other subsequent subclasses up to object of the module.

The ``construct_url()`` method is used to dynamically build the REST API URL and query parameters to retrieve or configure ACI objects at various levels of the object hierarchy, supporting flexible depth and child class filtering for APIC requests.

* When the ``state`` is not ``query``, the URL consists of the base URL (to access the APIC) combined with the distinguished name of the object (to access the object). The filter string limits the returned data to configuration information only.
* When ``state`` is ``query``, the URL and filter string used depend on which parameters are passed to the object. This method handles the complexity so that it is easier to add new modules and ensures that all modules are consistent in the type of data returned.
  * In query specific object, the URL is constructed to target a specific object within the module's class using its distinguished name. The filter string is typically not applied, allowing retrieval of the full object data. This approach simplifies module development by handling the URL construction dynamically and ensures consistent data retrieval for individual objects.
  * In query all objects, the URL is built to query all objects of the specified class. If a target filter is provided, it is applied as a query parameter to restrict the returned data to matching objects. This method manages the complexity of querying collections, making it easier to add new modules and maintain uniformity in the data returned across modules.
* `https://www.cisco.com/c/en/us/td/docs/dcn/aci/apic/all/apic-rest-api-configuration-guide/cisco-apic-rest-api-configuration-guide-42x-and-later/m_using_the_rest_api.html`_ provides more information about the APIC REST API and how to construct URLs.

    **Note**: The design goal is to take all ID parameters that have values and return the most specific data possible. If no ID parameters are supplied to the task, then all objects of the class will be returned. If the task does consist of ID parameters, then the data for the specific object is returned. If a partial set of ID parameters is passed, then the module will use the IDs that are passed to build the URL and filter strings appropriately.

The ``construct_url()`` method takes 2 required arguments and 7 optional arguments; the first 6 optional arguments are subclasses of the root class, and the last argument is a list of child classes. The method builds the URL and filter string based on the provided arguments, allowing for flexible querying of ACI objects.

    The required arguments of the method ``construct_url()`` are:
        * **self** - passed automatically with the class instance
        * **root_class** - A dictionary consisting of ``aci_class``, ``aci_rn``, ``target_filter``, and ``module_object`` keys

        + **aci_class**: The name of the class used by the APIC.

        + **aci_rn**: The relative name of the object.

        + **target_filter**: A dictionary with key-value pairs that make up the query string for selecting a subset of entries.

        + **module_object**: The particular object for this class.

            Some modules, like ``aci_tenant``, are the root class and so would not need to pass any additional arguments to the method.

    The optional arguments of the method ``construct_url()`` are:

        * subclass_1 - A dictionary consisting of ``aci_class``, ``aci_rn``, ``target_filter``, and ``module_object`` keys

        * subclass_2 - A dictionary consisting of ``aci_class``, ``aci_rn``, ``target_filter``, and ``module_object`` keys

        * subclass_3 - A dictionary consisting of ``aci_class``, ``aci_rn``, ``target_filter``, and ``module_object`` keys

        * subclass_4 - A dictionary consisting of ``aci_class``, ``aci_rn``, ``target_filter``, and ``module_object`` keys

        * subclass_5 - A dictionary consisting of ``aci_class``, ``aci_rn``, ``target_filter``, and ``module_object`` keys

        * subclass_6 - A dictionary consisting of ``aci_class``, ``aci_rn``, ``target_filter``, and ``module_object`` keys

        * child_classes - The list of APIC names for the child classes supported by the modules.
            + This is a list, even if it contains only one item.
            + These are the child class object names used by the APIC.
            + These are used to limit the returned child_classes when possible.

**Note**:
    * The ``aci_rn`` is the relative name of the object, which is one section of the distinguished name (DN) that uniquely identifies the object in the ACI fabric. It should not contain the entire DN, as the method will automatically construct the full DN using the provided RNs of all arguments.
    * RN is one section of DN, with the ID of the specific argument. Do not put the entire DN in the **aci_rn** of each argument. The method automatically constructs the DN using the RN of all the arguments above.
    * Refer to the modules aci_l3out_static_routes_nexthop for creation of object (ip:NexthopP) and aci_l3out_hsrp_secondary_vip for creation of object (hsrp:SecVip) for insights on how to use the ``construct_url()`` method.

Example:

.. code-block:: python

  # If "dn" = "uni/tn-ansible_tenant/out-ansible_l3out/lnodep-ansible_node_profile/", then the construct_url() will be constructed as follows:

  aci.construct_url(
      root_class=dict(
          aci_class='fvTenant',
          aci_rn='tn-{0}'.format(tenant),
          module_object=tenant,
          target_filter={'name': tenant}
      ),
      subclass_1=dict(
          aci_class='l3extOut',
          aci_rn='out-{0}'.format(l3out),
          module_object=l3out,
          target_filter={'name': l3out}
      ),
      subclass_2=dict(
          aci_class='l3extLNodeP',
          aci_rn='lnodep-{0}'.format(node_profile),
          module_object=node_profile,
          target_filter={'name': node_profile}
      )target_filter={'name': nexthop}
      )
  )

**Note**: Any requirements/changes for values of arguments (object,object_prop1, etc.) such as conversion to boolean, letter case, or formatting/validating the inputs must be done before the ``construct_url()`` method is called. This is because the method will use the values as they are passed in the task, and it will not perform any additional validation or conversion.

Getting the existing configuration
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
12. aci.get_existing() should remain as is. It is used to get the existing configuration of the object.

Once the URL and filter string have been built, the module is ready to retrieve the existing configuration for the object:

* ``state: present`` retrieves the configuration to use as a comparison against what was entered in the task. All values that are different from the existing values will be updated.
* ``state: absent`` uses the existing configuration to see if the item exists and needs to be deleted.
* ``state: query`` uses this to perform the query for the task and report back the existing data.

.. code-block:: python

    aci.get_existing()

When state is present
^^^^^^^^^^^^^^^^^^^^^
When ``state: present``, the module needs to perform a diff against the existing configuration and the task entries. If any value needs to be updated, the module will make a POST request with only the items that need to be updated. In other words, the payload is built with the expected configuration and this is compared with the existing configuration that was retrieved. If a change is needed, then the changed configuration will be pushed to APIC. Some modules have children that are in a 1-to-1 relationship with another object; for these cases, the module can be used to manage the child objects.

Building the ACI payload
""""""""""""""""""""""""
The ``aci.payload()`` method is used to build a dictionary of the proposed object configuration. All parameters that were not provided a value in the task will be removed from the dictionary (both for the object and its children). Any parameter that does have a value will be converted to a string and added to the final dictionary object that will be used for comparison against the existing configuration.

Values of parameters that are set to "None" are removed. If there is a previous configuration for a non-default value, then the parameter will not be modified if it is not reset. For example, if the description is set to something and then the module is run again with no description, it will not change it to the default, but by setting it to None, it will remove the description from the payload.

If parameters of the payload have been added in a recent version, it is recommended to add the new parameters to the payload when the parameter is assigned a value. This is done to maintain backward compatibility.

The ``aci.payload()`` method takes 2 required arguments and one optional argument, depending on whether the module manages child objects.

* ``aci_class`` is the APIC name for the object's class.
* ``class_config`` is the set of attributes of the aci class objects to be used as the payload for the POST request

  + The keys should match the names used by the APIC.
  + The formatted values should be the values retrieved from ``module.params`` and modified if necessary to comply with the object model.

* ``child_configs`` is optional and is a list of child config dictionaries.

  + The child configs include the full child object dictionary, not just the attributes configuration portion.
  + The configuration portion is built the same way as the parent object.

* ``annotation`` is an optional string that can be used to add additional information to the object.
  + If annotation is a supported attribute for a module it will be populated in the payload of that respective module.

**Note**: When the class configuration or child configuration depends on specific parameters, it is recommended to create these configurations beforehand. This approach ensures that the payload passed to the aci.payload() function is accurately constructed based on the parameter values, allowing for precise and flexible management of both class and child objects before the final payload is built and applied.

Performing the request
""""""""""""""""""""""
13. When state is present, a payload needs to be constructed which will be posted to APIC. Payload takes class_config and child_config. The class_config has the main attributes. If new attributes are added in new versions of APIC, that attribute will be added to class_config only if it is assigned a value.

Note - aci_rn must not contain the DN of the individual class. It is construct_url()'s task to build the entire DN leading to the target object using the series of RNs in the root class and the subsequent subclasses.

14. ``get_diff()`` method takes one required argument, ``aci_class``, which is the APIC name for the class of the object being configured. The get_diff() method compares the existing configuration with the proposed configuration and returns a dictionary of the differences. Replace ``<object APIC class>`` with the appropriate APIC class name for the object being configured.
        + The ``get_diff()`` method is used to perform the diff and takes only one required argument, ``aci_class``. In other words, it is used to make a comparison between the ACI payload and the existing configuration, and only create what's actually needed between the two.
        + ``required_properties`` parameter in the ``get_diff()``function is used to ensure that certain essential properties are always included in the configuration difference output, even if they are not part of the changed attributes. When there is a difference between the proposed and existing configurations, and if required_properties is provided as a dictionary, its key-value pairs are added to the configuration dictionary before it is finalized. This guarantees that these required properties are present in the resulting configuration update sent to the APIC, supporting consistent and complete configuration management.

    ``post_config()`` method is used to make the POST request to the APIC by taking the result from ``get_diff()``. This method doesn't take any arguments and handles check mode.

Example code
""""""""""""
.. code-block:: text

    if state == 'present':
        aci.payload(
            aci_class='<object APIC class>',
            class_config=dict(
                name=object_id,
                prop1=object_prop1,
                prop2=object_prop2,
                prop3=object_prop3,
            ),
            child_configs=[
                dict(
                    '<child APIC class>'=dict(
                        attributes=dict(
                            child_key=child_object_id,
                            child_prop=child_object_prop
                        ),
                    ),
                ),
            ],
        )

        aci.get_diff(aci_class='<object APIC class>')

        aci.post_config()

15. The end of the module does not change and generally remains as is.

When state is absent
^^^^^^^^^^^^^^^^^^^^
If the task sets the state to absent, then the ``delete_config()`` method is all that is needed. This method does not take any arguments and handles check mode.

.. code-block:: text

        elif state == 'absent':
            aci.delete_config()

Exiting the module
^^^^^^^^^^^^^^^^^^
To have the module exit, call the ACIModule method ``exit_json()``. This method automatically takes care of returning the common return values.

.. code-block:: text

        aci.exit_json()


    if __name__ == "__main__":
        main()

Testing the Module
============

Now that the module is created, it is time to test it. The module can be tested using the Ansible playbook. The playbook (main.yml) is added in the collection under tests/integration/targets/<aci_module_name>/tasks directory. The playbook is used to test the module and ensure that it works as expected.
  + Step 1: Under the tests/integration/targets/ create a folder with the name of the module being created. For example, replace <aci_module_name> with aci_l3out_logical_node.
  + Step 2: Under the <aci_module_name> directory copy paste the aliases file from any other module folder under tests/integration/targets/.
  + Step 3: Under the <aci_module_name> directory create a folder named tasks.
    + Step 4: Under the tasks directory create a file named main.yml. Preferred name for the file is main.yml.
    + In main.yml add tasks to test the module. The preferred order of tasks is:
        * Create, update, query and delete the object.
            * Create tasks include 3 tasks with check_mode, regular_run and idempotency
                * 2 types of create tasks are supported:
                    + Create a new object with all the parameters.
                    + Create a new object with only the required parameters.
            * Update tasks include 3 tasks with check_mode, regular_run and idempotency
            * Query tasks include 2 tasks; one to query a specific object and another to query all objects of the class.
            * Delete tasks include 3 tasks with check_mode, regular_run and idempotency

For complete guidelines on how to write the playbook, refer to `Testing the modules <testing_modules>`_ documentation.

**Note**:

  - A newline should be added at the end of the file to ensure that the file ends with a newline character, which is a good practice in Python coding.
  - Avoid using whitespaces or tabs at the end of lines, as this can lead to syntax errors or unexpected behavior in Python.
  - If aci_module_name.py file under the plugins/modules directory was used to create the new module, then remove all the comments in the file, except the copyright section at the top (first 5 lines) of the file. The comments in the aci_module_name.py file are only for reference and should not be included in the new module.

Checks before making a Pull Request
------------------------------------------------

Before making a pull request, ensure that the following checks are performed:
1. The module is tested using the Ansible playbook in the tests/integration/targets/<aci_module_name>/tasks directory. Use the sanity and black tests to ensure that the module is working as expected.
2. The module has the necessary code coverage.
3. The commit message is clear and concise, following the `Ansible commit message guidelines <https://docs.ansible.com/ansible/latest/dev_guide/developing_modules_general.html#commit-message-guidelines>`_.
    * The commit message should begin with "[<commit_type>] Short description of the changes."
        + <commit_type> can be one of the following: 
            + [minor_changes] for small changes made in the module which doesn't affect the current functionality.
            + [major_changes] for changes made in the module which affects the current working code(breaking changes).
            + [bugfix] for changes made to fix a bug in the module.
            + [docs] for changes made only to the documentation of the module.
            + [tests] for changes made to the tests of the module.
            + [ignore] for commit made after the initial commit which includes fixing sanity or whitespaces or spelling mistakes.

**Note**:

  `ACI Fundamentals: ACI Policy Model <https://www.cisco.com/c/en/us/td/docs/switches/datacenter/aci/apic/sw/1-x/aci-fundamentals/b_ACI-Fundamentals/b_ACI-Fundamentals_chapter_010001.html>`_
      A good introduction to the ACI object model.
  `APIC Management Information Model reference <https://developer.cisco.com/docs/apic-mim-ref/>`_
      Complete reference of the APIC object model.
  `APIC REST API Configuration Guide <https://www.cisco.com/c/en/us/td/docs/switches/datacenter/aci/apic/sw/2-x/rest_cfg/2_1_x/b_Cisco_APIC_REST_API_Configuration_Guide.html>`_
      Detailed guide on how the APIC REST API is designed and used, including many examples.  
