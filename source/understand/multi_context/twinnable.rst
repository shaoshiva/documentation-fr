``Contextable`` / ``Twinnable``
###############################

Novius OS est nativement multi-contextes. Mais chaque application décide de la façon dont elle utilise les contextes.
Trois cas de figures.

Application n'utilisant pas les contextes
*****************************************

Le cas le plus simple. Une application n'implémente pas la notion de contexte. C'est le cas par défaut, elle n'a rien à faire.

Son contenu sera alors le même quelque soit le contexte et elle pourra être utilisée (via ses :term:`enhancers`) dans n'importe quel contexte.

.. index:: Contextable

Application ``Contextable``
***************************

L'application utilise les contextes. Chaque item est associé à un contexte et ne peut être utilisé que dans son contexte.

Techniquement les tables de l'application auront un champ ``context`` de type ``varchar(25)`` contenant le code du contexte
et les :ref:`Models <api:php/models/model>` de l'application implémenteront le behaviour :ref:`api:php/behaviours/contextable`.


.. index:: Twinnable

Application ``Twinnable``
*************************

L'application utilise les contextes et crée des ponts entre-eux.
Chaque item est associé à un contexte et peut-être lié à d'autres items de contextes différents.

Techniquement les tables de l'application auront 3 champs :

:context: 			 de type ``varchar(25)`` et contenant le code du context de l'item.
:common_id_property: de type ``int`` et contenant un ID commun aux items liés entre-eux.
:is_main_property:   de type ``boolean``, chaque groupe d'items liés entre-eux a un seul item principal.

Les :ref:`Models <api:php/models/model>` de l'application implémenteront le behaviour :ref:`api:php/behaviours/twinnable`.

Exemple
=======

Struture de la table d'exemple :

* item_id (clé primaire)
* item_context
* item_common_id_property
* item_is_main_property
* item_title

Créons un premier item :

=======	============ ======================= ===================== ======================
item_id	item_context item_common_id_property item_is_main_property item_title
=======	============ ======================= ===================== ======================
1       main::fr_FR  1                       1                     Premier item
=======	============ ======================= ===================== ======================

| La colonne ``item_common_id_property`` prend le même ID que la clé primaire de l'item.
| L'item est déclaré item principal, donc la colonne ``item_is_main_property`` prend la valeur ``1``.


Ajoutons un autre item, dans un autre contexte, et lié au premier item :

=======	============ ======================= ===================== ======================
item_id	item_context item_common_id_property item_is_main_property item_title
=======	============ ======================= ===================== ======================
1       main::fr_FR  1                       1                     Premier item
2       main::en_GB  1                       0                     First item
=======	============ ======================= ===================== ======================

| La colonne ``item_common_id_property`` prend le ``item_common_id_property`` de l'item auxquel il est lié.
| La colonne ``item_is_main_property`` prend la valeur ``0``, ce n'est pas l'item principal.

Observons la table après plusieurs ajouts :

=======	============ ======================= ===================== ======================
item_id	item_context item_common_id_property item_is_main_property item_title
=======	============ ======================= ===================== ======================
1       main::fr_FR  1                       1                     Premier item
2       main::en_GB  1                       0                     First item
3       main::en_GB	 3                       1                     Second item
4       main::fr_FR	 3                       0                     Second item (fr)
5       event::fr_FR 5                       1                     Item du site event
6       main::es_ES	 1                       0                     First item (es)
=======	============ ======================= ===================== ======================

| Les items ``1``, ``2`` et ``6`` sont liés entre-eux et l'item principal est ``1`` / ``main::fr_FR``.
| Les items ``3`` et ``4`` sont liés entre-eux et l'item principal est ``3`` / ``main::en_GB``.

Supprimons l'item ``1`` :

=======	============ ======================= ===================== ======================
item_id	item_context item_common_id_property item_is_main_property item_title
=======	============ ======================= ===================== ======================
2       main::en_GB  *1*                     **1**                 First item
3       main::en_GB	 3                       1                     Second item
4       main::fr_FR	 3                       0                     Second item
5       event::fr_FR 5                       1                     Item du site vent
6       main::es_ES	 *1*                     0                     First item
=======	============ ======================= ===================== ======================

L'item ``2`` a récupéré le rôle principal, mais l'``item_common_id_property`` du ``2`` et du ``6`` n'a **pas** changé.

