#NHS Code4Health:  Answer/LCR application Demo

##A. The Clinical Handover Summary Scenario

This scenario represents the key clinical information typically recorded and updated as part of a clinical handover, including current problems, medications, allergies and relevant contacts.

In the first part of this scenarion, we will simply explore how to read and update a set of 'curated lists' e.g Allergies List, Problems List.

Later we will explore how to contruct a nightly Handover REport by querying into these documents and saving the queried data along with other new dat pertinent to the Handover process.

###What has been pre-prepared

A C4H/HANDI-HOPD domain login:`answer` password:`answerq9`with access to the Marand EhrExplorer view of the [Answer domain](https://dev.ehrscape.com/explorer). This will allow templates to be viewed and AQL Queries to be constructed.

A set of openEHR archetypes and templates
LCR Allergies List, LCR Medications List, LCR Problems List, LCR Relevent Contacts List, LCR Handover Summary. Thes have been uplaoded to theAnswer domain alonfg with some dummy data, with the [instance examples](/technical/instance/leeds) being available.

[Answer/LCR Handover Summary Ehrscape Technical tasks](/docs/leeds/Leeds_tech_tasks.md)

[Overview of openEHR and Ehrscape](/docs/training/openehr_intro.md)
