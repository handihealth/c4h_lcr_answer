# RippleOS openEHR modelling Implementation Notes

## Reference model (RM) Attribute mappings
Some GUI fields map to attributes in the openEHR Reference Model, rather than to archetype elements. This will generally be documented in the notes below

http://www.openehr.org/releases/trunk/UML/#Package__BOUML_0x1f602_22

points to an online UML resource documenting the Reference Model.

## ACTION archetypes and the RM state_machine

ACTION archetypes act as a constraint on the ACTION class in the openEHR Reference Model

## Composition Commit Styles

Depending on the clinical requirement, 3 styles of commit strategy are suggested.

**’Event’**

Each time the composition is committed, create a new instance via a POST.

**’Episodic’**

Create a new composition via POST for each Period of Care i.e an admission. If it needs to updated use a PUT to modify i.e Each patient has a single instance per Period of Care.

**’Longitudinal’**

Create a new composition via POST for each patient. If it needs to updated use a PUT to modify i.e. each patient has only a single instance over their lifetime. This will be unusual in a hospital record where there is generally limited ability to curate the patient record in this way.

## Composition context detail ##
The composition context detail cluster is used to carry common information recorded against all (or at least most) of the compositions in the patient record e.g. Identifiers for the Episode/Period of Care (admission) and the Details checked and complete’ flag.

## Capturing pick-list list + ‘Other’

This a very common pattern in health records where the user is presented with a choice of codes from a pick list along with the ability to enter ‘Other’ text.

In openEHR this kind of data is captured with a DV_TEXT/DV_CODED_TEXT multiple constraint, normally with the DV_CODED_TEXT aspect being defined by a local internal coddles, but also with an external terminology.

This means

Either select an item from the internal codelist (DV_CODED_TEXT)

or

add some free text (DV_TEXT)

In the archetype we do not add ‘OTHER’ as one of the internal coded list items, as this is a feature of the UI not of the data constraint required i.e ‘Other’ never appears in the captured data, it is just a UI prompt to tell the user to enter some free text, perhaps triggering the appearance of a simple text entry box.

## Limit to List (Templated valuesets)

In some circumstances, local value sets are added at template level e.g. Breathing Signs in the Nursing Observation template. Here the Limit to List attribute tells whether ‘Other’ text can be added to whether the valueset must be strictly adhered to.

## IDCR - Procedures List.v0


### Composition Commit Style
Episodic

### Composition RM attribute mappings
Ehrscape
Author: ctx/composer_name
Authored Date: ctx/date


## IDCR - End of Life Patient Preferences.v0

**Design Notes**

This template
### Composition Commit Style
Longitudinal

### Composition RM attribute mappings
Ehrscape
Author: ctx/composer_name
Authored Date: ctx/date



**Technical Notes**

## IDCR - Procedures List.v0

This templated composition maintains a single source-of-truth list of significant patient procedures.

### Composition Commit Style
Episodic/ Longitudinal (currently longitudinal)

### Composition (Ehrscape ctx) RM attribute mappings

Author(committer):
 ctx/composer_name
Date (authored):
 ctx/date
Procedure Performed :
 ctx/participation_name
 ctx/participation_function
 ctx/participation_mode

e.g.
```
"ctx/composer_name": "{{Author}}::Dr Joyce Smith",
"ctx/time": "{{Date}}::2015-02-22T00:11:02.518+02:00",  
"ctx/participation_name:0": "{{Procedure Performed By}}::Dr. Marcus Johnson",
"ctx/participation_function:0": "Performer",
"ctx/participation_mode:0": "face-to-face communication"
```
### Archetype RM attribute mappings

Procedure Date and Time:
 procedure/time

`time` is an RM attribute of the ACTION archetype class which records the datetime that the current ACTION/curent_status became true.
See xxx for more detail on the state_machine model used by ACTION archetypes.  

e.g.
```
"procedures_list/procedures:0/procedure:0/time": "{{Date Performed}}::2015-07-21T15:31:33.829Z"
```


## RIPPLE - Minimal Referral.v0

This template captures the original referral and any subsequent updates to the referral process e.g the referral being made, appointment being scheduled and the patent being seen, as a set of optional ACTIONs.

In this scenario, the first POST of the composition will include the Referral Request as an INSTRUCTION, along with an ACTION which sets the current_state to 'openehr::526::planned' and the careflow_step to "local::at0002::Referral planned".

When the appointment is scheduled, the original POST is read back including the original INSTRUCTION and 'Referral Planned' ACTION. A new ACTION is added
### Composition Commit Style
Episodic

### Composition RM attribute mappings

**Ehrscape**
Author: ctx/composer_name
Authored Date: ctx/date
