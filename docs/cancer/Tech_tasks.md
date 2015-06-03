##Code4Health Cancer MDT Output Ehrscape API Technical Guide

This document describes the series of [Ehrscape API](https://dev.ehrscape.com/api-explorer.html) calls required for the Ripple Cancer MDT Output Report app and assumes a base level of understanding of that API and the use of openEHR - further details can be found at [Overview of openEHR and Ehrscape](/docs/training/openehr_intro.md).

Ehrscape is an API wrapper for an implementation of the [openEHR](openehr.org) specification and an outline of the general structure of an openEHR system, to which all openEhr systems comply, will be helpful

###Outline of openEHR system structure

**CDR - Care Data Repository**
All of the records within a single namespaced patient data repository. In the Ehrscape environment, a single physical repository is represented as a number of independent virtual ‘domains’ each of which can be regarded as a separate CDR.

**EHR - Electronic Health Record**
The top-level container of all clinical records for a single patient

**FOLDER (optional)**
A high-level grouping of Compositions. Not implemented in Ehrscape, and not in common use generally

**COMPOSITION**
A document-level container for all clinical records. This equates to an Encounter, a Lab Report, a Discharge summary, a set of Nursing Observations. All openEHR data is stored within the context of a composition and always includes clinical/medico-legal context such as patient, clinical author details, encounter times etc.

The Composition is the container for structured/ coded data expressed as ENTRIES, within which the leaf data is contained in ELEMENTS.

Compositions are versioned and fully-audit trailed, so that previous versions can always be retrieved.
openEHR systems handle versioning automatically. Any time a new version of an existing document is committed to the system, its unique compositionId identifier ‘version suffix’ is updated e.g

798e27b1-f2e8-48c3-8ced-42d4d27d1db3::c4h.hopd.com::1”
 ->
798e27b1-f2e8-48c3-8ced-42d4d27d1db3::c4h.hopd.com::2”
  
In normal operation only the most recent version of the composition is returned by querying or composition retrieval.

The online of the overall structure is …
````
Physical CDR
 	Virtual CDR (Ehrscape Domain)
		EHR
     (FOLDER)
       COMPOSITION	
					(SECTION)
 						ENTRY
							(CLUSTER)
                 ELEMENT	
										name
										value
````
###Technical challenge steps

* Open an Ehrscape Session

* Retrieve an existing Patient’s ehrId from Ehrscape, based on their subjectId e.g. NHSNumber.

or if no EHR exists for that patient …

Create a new Patient EHR and retrieve their ehrId, based on an external subjectId identifier e.g NHS Number.

* Retrieve the compositionID and dates of Patient's previous Cancer MDT Output Report compositions

* Persist a new Cancer MDT Output Report Composition as a POST i.e create a completely new instance of the composition.

* Run an AQL query which returns a 'flat list' of key datapoints from the Cancer MDT Output Report for read-only display.

* Close the Ehrscape Session

* Other API services
 * Access the Indizen SNOMED CT Terminology browser service  

The baseURL for all Ehrscape API calls is https://rest.ehrscape.com

###Key API parameters for the Ehrscape domain

You should have received a login and password for your Ehrscape domain.

These are only used once, to open an Ehrscape session.

Your domain should also have been provisioned with a single patient record (EHR)

”subjectId": “7430555” (Steve Walford)  
"subjectNamespace": "uk.nhs.nhsnumber"  

You can check if this has been set correctly by running Step B below.

You may have other local parties setup.

###A. Open the Ehrscape session

The first step in working with Ehrscape is to open a Session and retrieve the ``sessionId`` token. This allows subsequent API calls to be made without needing to login on each occasion.  
The session should be formally closed when you are finished.

The baseURL for all Ehrscape calls is https://rest.ehrscape.com

i.e. the call below should be to https://rest.ehrscape.com/rest/v1/session?username=answer&password=answer99q

#####Call: Create a new openEHR session:
 ````
 POST /rest/v1/session?username=answer&password=answer99q
 ````
#####Returns:
````json
{
"sessionId": "fc234d24-7b59-49f5-a724-8d37072e832b"
}

````

###B. Retrieve Patient's ehrId from Ehrscape based on their subjectId

We now need to retrieve the patient's internal `ehrID` linked their subjectId to their external subjectId. The ehrID is a unique string which, for security reasons, cannot be associated with the patient, if for instance their openEHR records were leaked.

#####Call: Returns the EHR for the specified subject ID and namespace.
````
GET /rest/v1/ehr/?subjectId=67022&subjectNamespace=uk.nhs.nhsnumber
Headers:
 Ehr-Session: {{sessionId}} //The value of the sessionId
````
#####Return:
````json
{
  "ehrId": “852d7d4e-ac6b-4bbf-90f0-d568748e02bf”
}
````

###D. Retrieve the compositionId and dates of Patient's most recent Cancer MDT Output Report compositions

Now that we have the patient's ehrId we can use it to locate their existing records.
We use an Archetype Query Language (AQL) call to retrieve a list of the identifiers and dates of existing Cancer MDT Output Report ``composition`` records. Compositions are document-level records which act as the container for all openEHR patient data.

The `name/value` of the Composition is the root name of the composition archetype `Cancer MDT Output Report` (case-sensitive). In a real-world example we would query on other factors to ensure we had the 'correct' list.

#####AQL statement

````
select
    a/uid/value as uid_value,
    a/context/start_time/value as context_start_time
from EHR e[ehr_id/value= ‘852d7d4e-ac6b-4bbf-90f0-d568748e02bf’]
contains COMPOSITION a[openEHR-EHR-COMPOSITION.report.v1]
where a/name/value=‘Cancer MDT Output Report’
order by a/context/start_time/value desc
offset 0 limit 20
````
The query API call returns a `resultset` which is a nested set of name/value pairs whose format is determined by the AQL query.

The `uid_value` element in the response is the unique identifier for the composition and `context_start_time` is the time that the document was authored.

#####Call: Run AQL query and return a Resultset
````
GET https://rest.ehrscape.com/rest/v1/query?aql=select     a/uid/value as uid_value,     a/context/start_time/value as context_start_time from EHR e[ehr_id/value= ‘852d7d4e-ac6b-4bbf-90f0-d568748e02bf’] contains COMPOSITION a[openEHR-EHR-COMPOSITION.report.v1] where a/name/value=‘Cancer MDT Output Report’ order by a/context/start_time/value desc offset 0 limit 20
Headers:
 Ehr-Session: {{sessionId}} //The value of the sessionId
````

#####Return:
````json
"resultSet": [
{
  “meta”: {
    “href”: “https://rest.ehrscape.com/rest/v1/query/?aql=select%20%20%20%20%20a/uid/value%20as%20uid_value,%20%20%20%20%20a/context/start_time/value%20as%20context_start_time%20from%20EHR%20e%5Behr_id/value%3D%20'852d7d4e-ac6b-4bbf-90f0-d568748e02bf’%5D%20contains%20COMPOSITION%20a%5BopenEHR-EHR-COMPOSITION.report.v1%5D%20where%20a/name/value%3D’Cancer%20MDT%20Output%20Report’%20order%20by%20a/context/start_time/value%20desc%20offset%200%20limit%2020”
  },
  “aql”: “select     a/uid/value as uid_value,     a/context/start_time/value as context_start_time from EHR e[ehr_id/value= ‘852d7d4e-ac6b-4bbf-90f0-d568748e02bf’] contains COMPOSITION a[openEHR-EHR-COMPOSITION.report.v1] where a/name/value=‘Cancer MDT Output Report’ order by a/context/start_time/value desc offset 0 limit 20”,
  “executedAql”: “select     a/uid/value as uid_value,     a/context/start_time/value as context_start_time from EHR e[ehr_id/value= ‘852d7d4e-ac6b-4bbf-90f0-d568748e02bf’] contains COMPOSITION a[openEHR-EHR-COMPOSITION.report.v1] where a/name/value=‘Cancer MDT Output Report’ order by a/context/start_time/value desc offset 0 limit 20”,
  “resultSet”: [
    {
      “context_start_time”: “2015-02-21T23:11:02.518+01:00”,
      “uid_value”: “839928d7-1607-4622-a885-24ae2c051e4e::answer.hopd.com::1”
    }
]
````



###D. Persist a new instance of the Cancer MDT Output Report  Composition as a POST

All openEHR data is persisted as a COMPOSITION (document) class. openEHR data can be highly structured and potentially complex. To simplify the challenge of persisting openEHR data, examples of  'target composition' data instances have been provided in the Ehrscape ``FLAT JSON`` format.

Once the data is assembled in the correct format, the actual service call is very simple requiring only the setting of simple parameters and headers.

[Example Ehrscape Flat JSON Composition](/technical/instances/Cancer MDT/IDCR - Cancer MDT Output report FLAT.json)  

#####Call: Updates a new openEHR composition and returns the new CompositionId
````
POST rest.ehrscape.com/rest/v1/composition?ehrId=852d7d4e-ac6b-4bbf-90f0-d568748e02bf&templateId=IDCR - Cancer MDT Output Report.v0&committerName=handi&format=FLAT

Headers:
 Ehr-Session: {{sessionId}} //The value of the sessionId

 {
  "ctx/composer_name": "Dr Joyce Smith",
  "ctx/health_care_facility|id": "999999-345",
  "ctx/health_care_facility|name": "Northumbria Community NHS",
  "ctx/id_namespace": "NHS-UK",
  "ctx/id_scheme": "2.16.840.1.113883.2.1.4.3",
  "ctx/language": "en",
  "ctx/territory": "GB",
	  "ctx/time": "2015-02-24T00:11:02.518+02:00",
    …..
}
````
#####Return:
````json
{
 "href": "https://rest.ehrscape.com/rest/v1/composition/798e27b1-f2e8-48c3-8ced-42d4d27d1db3::answer.hopd.com::3"
}
````


###F. Close the Ehrscape session

The last step in working with Ehrscape is to close the session.

#####Call: Create a new openEHR session:
 ````
 DELETE /rest/v1/session?sessionId={{sessionId}}
 ````
#####Returns:
````json
{
  "sessionId": "2dcd6528-0471-4950-82fa-a018272f1339"
}
````

##G. Other Services

These are other non-Ehrscape API services.

###1. Access the ALISS 'Local community resources' service

[ALISS](http://www.aliss.org/) (A Local Information System for Scotland) is a search and collaboration tool for Health and Wellbeing resources. Originally developed in Scotland, it is now us used in regions across the UK.

Further search options e.g by locality are avaiable from the [ALISS API documentation site](http://aliss.readthedocs.org/en/latest/search_api/).

#####Call: Search the ALLIS database for resources related to asthma
````
GET http://www.aliss.org/api/v2/search/?q=asthma

Headers:
 None
````
#####Return:
````json
{
    "count": 91,
    "next": "http://www.aliss.org/api/v2/search/?page=2&q=asthma",
    "previous": null,
    "results": [
        {
   "id": 351,
   "title": "Asthma - a booklist to help you manage the condition - from Renfrewshire Libraries",
   "description": "Renfrewshire Libraries have developed a facility for creating booklists online via a simple search function. One can find out whether Renfrewshire Libraries has a particular title, whether it is available and from which library, etc.\r\n\r\nThis one is about Asthma.",
   "uri": "https://libcat.renfrewshire.gov.uk/vs/List.csp?SearchT1=asthma&Index1=Keywords&Database=1&PublicationType=NoPreference&Location=NoPreference&SearchMethod=Find_1&SearchTerm1=asthma&OpacLanguage=eng&Profile=Default&EncodedRequest=*D5*2C2*D8*EE*C4*D4*5B*B2*9C*00*26*D8*5Cb*8F&EncodedQuery=*D5*2C2*D8*EE*C4*D4*5B*B2*9C*00*26*D8*5Cb*8F&Source=SysQR&PageType=Start&PreviousList=RecordListFind&WebPageNr=1&NumberToRetrieve=20&WebAction=NewSearch&StartValue=0&RowRepeat=0&ExtraInfo=SearchFromList&SortIndex=Author&SortDirection=1",
   "locations": [
       {
           "lat": 55.8561,
           "lon": -4.4057,
           "formatted_address": "Renfrew South & Gallowhill ward, Renfrewshire, PA3 4SF"
       }
   ],
   "tags": [
       "books",
       "library",
       "Asthma",
       "asthma"
   ],
   "owner": "Living Well at the Library",
   "event_start": null,
   "event_end": null,
   "created_on": "2011-07-13T12:46:44.732000+00:00",
   "modified_on": "2014-04-22T07:15:37.249920+00:00"
},

````

###2. Access the NHS Choices Patient advice service - HTML

This API call retreives NHS Choices patient information page for Asthma in HTML format.

#####Call: Search the NHS Choices database for resources related to asthma, returning an HTML page
````
GET http://v1.syndication.nhschoices.nhs.uk/conditions/articles/asthma/introduction?apikey=GFENGBJA

Headers:
 None
````
#####Return:
````html
<html xmlns="http://www.w3.org/1999/xhtml" >
<head>
    <title>NHS Choices Syndication</title>
    <meta http-equiv="Content-Style-Type" content="text/css" />
.....
</head>
````

###3. Access the NHS Choices Patient advice service - XML

This API call retreives NHS Choices patient information for Asthma in XML format.

#####Call: Search the NHS Choices database for resources related to asthma, returning an XML document.
````
GET http://v1.syndication.nhschoices.nhs.uk/conditions/articles/asthma/introduction.xml?apikey=GFENGBJA

Headers:
 None
````
#####Return:
````xml
<?xml version="1.0" encoding="utf-8"?>
     <feed xmlns="http://www.w3.org/2005/Atom"><title type="text">NHS Choices - Introduction</title><id>uuid:6e093010-1013-46b4-b231-42795514008d;id=1950</id><rights type="text">© Crown Copyright 2009</rights><updated>2015-02-11T15:54:40Z</updated><category term="asthma" /><logo>http://www.nhs.uk/nhscwebservices/documents/logo1.jpg</logo>
	...
````

###4. Access the Indizen SNOMED CT Terminology browser service

This API call to the [Indizen](www.indizen.com/index.php/en/) Terminology serviceretreives SNOMED CT terms matching ``asthma`` in XML format.

The baseURL for Indizen ITSNode is http://www.itserver.es:9080/

#####Call: Search the Indizen terminology service database for terms matching asthma
````
GET /ITSNode/rest/snomed/descriptions?matchvalue=asthma&referencelanguage=en&fuzzy=false&numberOfElements=110&filtercomponent=all&spellingCorrection=true

Headers:
 Authorization: Basic aGFuZGlob3BkOmRmZzU4cGE=
````
#####Return:
````xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<sctDescriptionss>
	<description>
		<conceptid>195967001</conceptid>
		<descriptionid>301485011</descriptionid>
		<descriptionstatus>0</descriptionstatus>
		<descriptiontype>1</descriptiontype>
		<inititalcapitalstatus>0</inititalcapitalstatus>
		<languagecode>en</languagecode>
		<similarity>100.0</similarity>
		<sourceName/>
		<term>Asthma</term>
	</description>
	<description>
		<conceptid>161527007</conceptid>
		<descriptionid>251716013</descriptionid>
		<descriptionstatus>0</descriptionstatus>
		<descriptiontype>1</descriptiontype>
		<inititalcapitalstatus>1</inititalcapitalstatus>
		<languagecode>en</languagecode>
		<similarity>98.0</similarity>
		<sourceName/>
		<term>H/O: asthma</term>
	</description>
	<description>
		<conceptid>160377001</conceptid>
		<descriptionid>249995012</descriptionid>
		<descriptionstatus>0</descriptionstatus>
		<descriptiontype>1</descriptiontype>
		<inititalcapitalstatus>1</inititalcapitalstatus>
		<languagecode>en</languagecode>
		<similarity>98.0</similarity>
		<sourceName/>
		<term>FH: Asthma</term>
	</description>
   ...
</sctDescriptionss>
````
