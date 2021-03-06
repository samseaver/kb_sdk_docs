Adding UI Elements
===========================

Control of narrative interaction is accomplished in files in the
``ui/narrative/methods/<MyMethod>`` directory.

Configure the input interface
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The input options are specified in the "parameters" section of the
spec.json file. In the following example, the user will supply two
required parameters: an input name and an output name. If you specify
'valid\_ws\_types', the user will be presented with a searchable
dropdown of objects that match the specified types. If
"is\_output\_name" is set to true, the user will be warned if a name
will overwrite an existing object or if the name contains invalid
characters, and a widget displaying that object will appear beneath the
app cell.

.. code:: json

        "parameters": [ 
            {
                "id": "read_library_name",
                "optional": false,
                "advanced": false,
                "allow_multiple": false,
                "default_values": [ "" ],
                "field_type": "text",
                "text_options": {
                    "valid_ws_types": ["KBaseAssembly.PairedEndLibrary","KBaseFile.PairedEndLibrary"]
                }
            },
            {
                "id" : "output_contigset_name",
                "optional" : false,
                "advanced" : false,
                "allow_multiple" : false,
                "default_values" : [ "MegaHit.contigs" ],
                "field_type" : "text",
                "text_options" : {
                    "valid_ws_types" : [ "KBaseGenomes.ContigSet" ],
                    "is_output_name":true
                }
            }
        ],

Another common input type is a dropdown, which is demonstrated below.
For each option, the "value" is what will be passed to the app while the
"ui\_name" is what the user will see. In this example, the parameter is
hidden by default because "advanced" is true (meaning the parameter will
be hidden in the "Advanced options" section of the input widget).

.. code:: json

    {
        "id": "prune",
        "optional": false,
        "advanced": true,
        "allow_multiple": false,
        "default_values": [ "biochemistry" ],
        "field_type": "dropdown",
        "dropdown_options": {
            "options": [
                {
                    "value": "biochemistry",
                    "display": "Known biochemistry",
                    "id": "biochemistry",
                    "ui_name": "Known biochemistry"
                },
                {
                    "value": "model",
                    "display": "Input model",
                    "id": "model",
                    "ui_name": "Input model"
                },
                {
                    "value": "none",
                    "display": "Do not prune",
                    "id": "none",
                    "ui_name": "Do not prune"
                }
            ]
        }
    },


There are many additional interface options available. One of the best
ways to discover them is to explore `this
gallery <https://narrative.kbase.us/narrative/ws.23109.obj.1>`__ which
contains a variety of KBase apps along with the spec file that generated
the user interface.

Configure passing variables from Narrative Input to SDK method
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In the 'behavior' section of the spec.json, the output of the user
interface is mapped to input to your function. If you have maintained a
consistent naming though these mappings can be pretty pro forma.
However, transformations can also be applied to input from the user
interface.

In the example below, the user interface accepts the names of assemblies
from the user and transforms them into object references before passing
them on to the method. (This prevents a race condition from occurring if
multiple apps are writing to the same object name or if the object is
renamed.) In the output mapping, the output (a single object in this
example) is unpacked into target properties. These output properties are
used to visualize the result of the app (thus the need to return
information about the report object).

::

    "behavior" : {
        "service-mapping": {
                "url": "",
                "name": "kb_quast",
                "method": "run_QUAST_app",
                "input_mapping": [
                    {
                        "narrative_system_variable": "workspace",
                        "target_property": "workspace_name"
                    },
                    {
                        "input_parameter": "assemblies",
                        "target_type_transform": "list<ref>",
                        "target_property": "assemblies"
                    }
                ],
                "output_mapping": [
                    {
                        "service_method_output_path": [0,"report_name"],
                        "target_property": "report_name"
                    },
                    {
                        "service_method_output_path": [0,"report_ref"],
                        "target_property": "report_ref"
                    },
                    {
                        "constant_value": "5",
                        "target_property": "report_window_line_height"
                    },
                    {
                        "service_method_output_path": [0],
                        "target_property": "QUAST_result"
                    },
                    {
                        "input_parameter": "assemblies",
                        "target_property": "input_assemblies"
                    },
                    {
                        "narrative_system_variable": "workspace",
                        "target_property": "workspace_name"
                    }
                ]
            }
        }

In the above example the Narrative take an object looking like this from
the App UI:

.. code:: json

    {
      "assemblies": [
        "AssemblyA",
        "AssemblyB"
      ]
    }

and passes an object looking like this to the implementation function:

.. code:: json

    {
      "assemblies": [
        "765/1/1",
        "765/2/1"
      ],
      "Workspace_name": "<username>:narrative_<long_number>"
    }

Similarly, the Narrative accepts an output object like this:

.. code:: json

    [
      {
        "report_name": "QUAST_Report_<uuid>",
        "report_ref": "765/3/1"
      }
    ]

And presents an object like this one to the report visualization:

.. code:: json

    {
        "report_name": "QUAST_Report_<uuid>",
        "report_ref": "765/3/1",
        "report_window_line_height": 5,
        "QUAST_result": {
            "report_name": "QUAST_Report_<uuid>",
            "report_ref": "765/3/1"
        },
        "assemblies": [
        "AssemblyA",
        "AssemblyB"
        ],
        "Workspace_name": "<username>:narrative_<long_number>"   
    }

Naming fields in the input widget cell
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The ``display.yaml`` file primarily contains text to describe the app (shown in the narrative and in the app catalog). Minimally this file should define: 

* A module name 
* A module tooltip 
* A ui-name for each parameter 
* A short hint for each parameter

Details on Narrative UI specification
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Further details on specification of Narrative app interfaces are
available
`here <../references/UI_spec.html>`__
