.. role:: xml(code)
	:language: xml

.. _Sec:input:model:

ModelV4.xml
===========

Fluid mechanics models used in the computation are specified in the *modelv4.xml* file. It is **mandatory** located in the folder of the current case. A typical form of this file is:

.. code-block:: xml

	<?xml version = "1.0" encoding = "UTF-8" standalone = "yes"?>
	<model>
	  <flowModel name="Kapila" numberPhases="2" alphaNull="false"/>
	  <EOS name="IG_air.xml"/>
	  <EOS name="SG_water.xml"/>
	</model>

.. _Sec:input:FlowModel:

Flow model
----------

.. code-block:: xml

	<flowModel name="Kapila" numberPhases="2" numberTransports="1" alphaNull="false"/>

The :xml:`<flowModel>` markup is **mandatory** to specify the mathematical model to solve during the computation. This markup may contain the following attributes:

- :xml:`name`: Name of the mathematical flow model. This attribute can take the values: *Euler* :cite:`euler1757principes`, *Kapila* :cite:`relaxjcp`:cite:`kapila2001`, *multip*, *ThermalEq* :cite:`martelotboiling` or *EulerHomogeneous*.
- :xml:`numberPhases`: Integer number corresponding to the number of phases present in the simulation. The total amount of equations is related to this number. This attribute is not necessary when the values of :xml:`name` are *Euler* or *EulerHomogeneous*.
- :xml:`numberTransports`: This attribute is optionnal and is set to 0 as default. It can be used to add specific variables advected in the flow (color functions).
- :xml:`alphaNull`: For *Kapila*'s model, the volume fraction can either be null (*true*) or not (*false*) and this choice is determined by this parameter. Default value is *false*.

**Remark:**

If *EulerHomogeneous* model is chosen, two additional attributes may be used: :xml:`liquid` and :xml:`vapor` to specify the number corresponding to the liquid phase and the vapor phase. They are phase 0 for the first and 1 for the second.

.. code-block:: xml

	<flowModel name="EulerHomogeneous" liquid="0" vapor="1"/>
	<EOS name="SG_waterLiq.xml"/>
	<EOS name="IG_waterVap.xml"/>

Equations of state (EOS)
------------------------

.. code-block:: xml

	<EOS name="IG_air.xml"/>

The *modelV4.xml* input file **must contain** as many :xml:`<EOS>` markups as number of phases specified in the :ref:`Sec:input:FlowModel` markup. Each phase is described thanks to relations and parameters. The values of these parameters are specified in a separate file: The attribute name contains the name of this file that must be placed in the folder **ECOGEN/libEOS/**. Some fluid files are already present in the ECOGEN package.

.. _Sec:input:Transport:

Advected additional variables
-----------------------------

.. code-block:: xml

	<transport name="color"/>

The *modelV4.xml* input fle **must contain** as many :xml:`<transport>` markups as number of transports specified in the :ref:`Sec:input:FlowModel` markup. Each transported variable is described by its name. The default number of advected variable is 0.

Relaxation procedures
---------------------

.. code-block:: xml

	<relaxation type="PT"/>

An additional markup :xml:`<relaxation>` may be used to impose some specific equilibrium between the phases depending on the flow model used. The attribute :xml:`type` specifies the type of equilibrium:

- *P*: A pressure equilibrium is imposed at every location of the flow. It does not require additional attributes.
- *PT*: Both pressure and thermal equilibrium are imposed at every location of the flow. It does not require additional attributes.
- *PTMu*: A thermodynamical equilibrium is imposed at every location of the flow. It must be associated with the node :xml:`<dataPTMu>` whose attributes are :xml:`liquid` and :xml:`vapor` to specify the name of the EOS of the liquid and the vapor phase. Hereafter the complete node when *PTMu* is used:

.. code-block:: xml

	<relaxation type="PTMu">
	  <dataPTMu liquid="SG_waterLiq.xml" vapor="IG_waterVap.xml"/>
	</relaxation>
 
Source terms
------------

The additional :xml:`<sourceTerms>` markup can be used to numerically integrate some source terms in the equations. The attribute :xml:`type` selects the source term:

- *heating*: Related to a thermal energy heating/cooling. This attribute requires the :xml:`<dataHeating>` node with the attribute :xml:`volumeHeatPower`; a real number corresponding to the power by volume unit added to the flow (unit: W/m3 (SI)).

.. code-block:: xml

	<sourceTerms type="heating">
	  <dataHeating volumeHeatPower="1.e6"/>
	</sourceTerms>

- *gravity*: If the gravity is considered. The node :xml:`<dataGravity>` must be present with the attributes :xml:`x`, :xml:`y` and :xml:`z` giving the coordinates for the gravity acceleration vector in real numbers (unit: m/s2 (SI)).

.. code-block:: xml

	<sourceTerms type="gravity">
	  <gravity x="0." y="-9.81" z="0."/>
	</sourceTerms>

- *MRF*: For a simulation in a moving reference frame. Allow to compute solution in a rotating frame. The node :xml:`<omega>` requires the attributes :xml:`x`, :xml:`y` and :xml:`z` giving the coordinates for the rotating vector in real numbers (unit: rad/s (SI)). The node :xml:`<timeToOmega>` is optional and allow to specify a progressing acceleration (linear) to the final rotating velocity (requires the attribute :xml:`tf` for acceleration time).

.. code-block:: xml

	<sourceTerms type="MRF">
	  <omega x="0." y="0." z="1."/>
	  <timeToOmega tf="1.e-3"/>  <!-- Optional: If activated, the angular velocity increase linearly to omega in during tf -->
	</sourceTerms>

Symmetry terms
--------------

Both cylindrical (2D) and spherical (1D) symmetries are implemented. The additional :xml:`<symmetryTerm>` markup can be used. It requires the attribute :xml:`type` that can take the value *cylindrical* or *spherical*. It also requires an additional node to specify the radial axis:

- Cylindrical:

.. code-block:: xml

	<symmetryTerm type="cylindrical">
	  <dataSymCyl radialAxis="X"/>
	</symmetryTerm>

- Spherical:

.. code-block:: xml

	<symmetryTerm type="spherical">
	  <dataSymSpher radialAxis="X"/>
	</symmetryTerm>

Additional physics (dev)
------------------------

Depending on the model chosen in section :ref:`Sec:input:FlowModel`, additional physical effects can be added. This is the case for surface tension, viscosity and conductive heat transfers. These additional physical effects are obtained thanks to the additional markup :xml:`additionalPhysics` with the attribute :xml:`type` that can take different value according to the chosen effect.

Surface tension
~~~~~~~~~~~~~~~
This physical effect is obtained by using the type *surface tension*. Then it requires the node :xml:`dataSurfaceTension` with following attributes:

- :xml:`transport`: This is the name of advected variable used as color function for surface-tension terms. This advected variable has been precised in the section :ref:`Sec:input:Transport`. The name should be the same.
- :xml:`sigma`: A real number for the surface-tension coefficient (unit: N/m (SI)).

.. code-block:: xml

	<additionalPhysic type="surfaceTension" >
	  <dataSurfaceTension transport="color" sigma="72.e-3"/>
	</additionalPhysic>

Viscosity
~~~~~~~~~
This physical effect is obtained by using the type *viscosity* and it needs the attribute :xml:`mu` of the :xml:`<physicalParameters>` to be set in the EOS files (:ref:`Sec:IO:materials`).

.. code-block:: xml

	<additionalPhysic type="viscosity"/>

Others
~~~~~~

In dev...