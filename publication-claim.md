# Publication Claim

> Scenario 2: A new researcher joins the institution and logins for the first time in the repository. The publication claim services found most of their publications in the OpenAIRE network and prompts for import. The researcher reviews the list, confirms the authorship and imports the publication saving a significant amount of (often publicly payed) time. Moreover, the authorship confirmation will come back later to OpenAIRE offering useful information about the data quality and potential enrichment. The same applies for publications authored by researchers in different institutes, having the data in multiple repositories makes the data more reliable and raises the chance to get more information and content from any of the authors.  

The goal of the Publication Claim service is to support the scenario above.

The service has been designed to be independent from a specific provider or implementation so that it can be easily extended and maintained over time. Moreover, multiple providers can be active at the same time improving the chance to save researchers time.

In the original plan an integration with the [ReCiter open source platform](https://github.com/wcmc-its/ReCiter) was originally planned but over the phase 2 we found that the internal data structure of ReCiter was too tight to the PubMed Article Model to be adapted to work with the data provided by the openAIRE Research Graph within the budget limit and for such reason we switched to a direct integration with the openAIRE Research Graph via the [Publication REST API](http://api.openaire.eu/api.html#pubs). Other good candidates to be integrated via such framework are ORCID or commercial databases via their authors' IDs.

## Data source
The openAIRE Publication REST API are used to retrieve publication that could be authored by researcher at the Institution. The openAIRE Publication REST API are queried using the names known by the repository for its researchers, the retrieve list is later reduced passing identified publications to a pipeline of JAVA classes that can promote or reject his inclusion in the suggestion list. Publications previously discarded by the researcher are automatically filter out avoiding to re-present the same publication again and again.

This pipelines allow a future refinement of the procedure introducing for instance support for researcher preference that could exclude specific sources (pubmed, crossref, datacite, etc.) or keywords/subjects unrelated with his research interests.
Right now a single scorer is in place, to validate the finding against the researcher name as it has been found that searching the openAIRE Publication API for author such as Bollini Susanna would find also publications co-authored by Andrea Bollini and Susanna Mornati.

The pipeline is defined in the `/config/spring/api/oaire-publications.xml` spring configuration file inside the OAIREPublicationLoader bean

```xml
    <bean id="org.dspace.app.suggestion.oaire.OAIREPublicationLoader"
        class="org.dspace.app.suggestion.oaire.OAIREPublicationLoader">
        <property name="names">
            <list>
                <value>dc.title</value>
                <value>crisrp.name</value>
                <value>crisrp.translated</value>
                <value>crisrp.variants</value>
            </list>
        </property>
        <property name="pipeline">
            <list>
                <bean
                    class="org.dspace.app.suggestion.oaire.AuthorNamesScorer">
                    <property name="contributorMetadata">
                        <list>
                            <value>dc.contributor.author</value>
                        </list>
                    </property>
                    <property name="names">
                        <list>
                            <value>dc.title</value>
                            <value>crisrp.name</value>
                            <value>crisrp.translated</value>
                            <value>crisrp.variants</value>
                        </list>
                    </property>
                </bean>
            </list>
        </property>
    </bean>
```

the names attribute defines the metadata to use to build the search query over the openAIRE Research Graph to retrieve the list of publications to evaluate as suggestions. It is responsibility of the scorers defined in the pipeline to compute a score for each retrieved publication and eventually discard the ones that are not good enough.

The dspace script class `org.dspace.app.suggestion.OAIREPublicationLoaderRunnableCli` is used to run the queries and store the identified publication in the dedicated SOLR core **suggestion** for further processing.

The dspace script can be run both from the CLI than from the UI.

To run the loader from the dspace installation bin folder

```
./dspace import-oaire-suggestions [-s uuid-of-single-researcher]
```

without the `s` parameter the script will process all the researcher available in the system.

The script can be also run from the Script UI so that it is also available to repository manager that cannot be access the CLI

![publication claim suggestions retrieval script UI](/_media/suggestion-ui-script.png)

Two external source providers, openAIRE Publications By Title and By Author have been defined according to the standard [DSpace 7 External Sources framework](https://github.com/DSpace/Rest7Contract/blob/main/external-authority-sources.md). It is activated in the `config/spring/api/external-services.xml` as follow

```xml
     <bean id="openaireLiveImportDataProviderByAuthor" class="org.dspace.external.provider.impl.LiveImportDataProvider">
        <property name="metadataSource" ref="openaireImportServiceByAuthor"/>
        <property name="sourceIdentifier" value="openaire"/>
        <property name="recordIdMetadata" value="dc.identifier.other"/>
        <property name="supportedEntityTypes">
            <list>
                <value>Publication</value>
            </list>
        </property>
    </bean>

    <bean id="openaireLiveImportDataProviderByTitle" class="org.dspace.external.provider.impl.LiveImportDataProvider">
        <property name="metadataSource" ref="openaireImportServiceByTitle"/>
        <property name="sourceIdentifier" value="openaireTitle"/>
        <property name="recordIdMetadata" value="dc.identifier.other"/>
        <property name="supportedEntityTypes">
            <list>
                <value>Publication</value>
            </list>
        </property>
    </bean>
```

with the importer services defined via the Live Import Framework in `/dspace-api/src/main/resources/spring/spring-dspace-addon-import-services.xml` as follow

```
    <bean id="openaireImportServiceByAuthor"
          class="org.dspace.importer.external.openaire.service.OpenAireImportMetadataSourceServiceImpl" scope="singleton">
        <property name="metadataFieldMapping" ref="openaireMetadataFieldMapping"/>
        <property name="queryParam" value="author"/>
    </bean>
    <bean id="openaireImportServiceByTitle"
          class="org.dspace.importer.external.openaire.service.OpenAireImportMetadataSourceServiceImpl" scope="singleton">
        <property name="metadataFieldMapping" ref="openaireMetadataFieldMapping"/>
        <property name="queryParam" value="title"/>
    </bean>
    <bean id="openaireMetadataFieldMapping"
          class="org.dspace.importer.external.openaire.service.metadatamapping.OpenAireFieldMapping">
    </bean>
```

The mapping between the openAIRE Publications metadata and the dspace metadata is provided in the `config/spring/api/openaire-integration.xml` using the usual xpath approach of the DSpace Live Import Framework.

> Having used the Live Import Framework internally to the loader to perform the query has had the side benefit to make available the publication data of the openAIRE Research Graph also to the direct import functionality of DSpace, so that the researcher can now query the openAIRE graph and import publication on demand.

The SOLR **suggestion** core has the following structure

```
<fields>
    <field name="source" type="string" indexed="true" stored="true" omitNorms="true" />
    <field name="suggestion_fullid" type="string" indexed="true" stored="true" omitNorms="true" />
    <field name="suggestion_id" type="string" indexed="true" stored="true" omitNorms="true" />
    <field name="target_id" type="string" indexed="true" stored="true" omitNorms="true" />
    <field name="title" type="string" indexed="true" stored="true" omitNorms="true" />
    <field name="date" type="string" indexed="true" stored="true" omitNorms="true" />
    <field name="display" type="string" indexed="true" stored="true" omitNorms="true" />
    <field name="contributors" type="string" indexed="true" stored="true" omitNorms="true" multiValued="true" />
    <field name="abstract" type="string" indexed="true" stored="true" omitNorms="true" />
    <field name="category" type="string" indexed="true" stored="true" omitNorms="true" multiValued="true"/>
    <field name="external-uri" type="string" indexed="true" stored="true" omitNorms="true" />
    <field name="processed" type="boolean" indexed="true" stored="true" omitNorms="true" />
    <field name="score" type="double" indexed="true" stored="true" omitNorms="true" />
    <field name="evidences" type="string" indexed="false" stored="true" omitNorms="true" />
</fields>    
<uniqueKey>suggestion_id</uniqueKey>
```

the `source` field would allow the reuse of such structure by other sources than openAIRE. 
 
Three endpoints have been designed to expose the result of the processing to the DSpace UI and so to the Repository Managers and single researchers:

* `/api/integration/suggestionsources` to provide access to summary information about the available suggestion from each source (openaire, orcid, etc.)
* `/api/integration/suggestiontargets` to provide access to summary information about the available suggestions for a specific researcher
* `/api/integration/suggestions` to provide access to the detailed suggestions so that they can be reviewed and managed by the repository manager or the researcher to whom they related

The detailed REST contract for such endpoints are available on the [4Science Rest7Contract repository](https://github.com/4Science/Rest7Contract/tree/oaire-eld) and embedded at the bottom of the page for easy reference.

## Repository Manager UI
The resulting UI is accessible for the Repository Manager from the administrative menu. As entry point for the features a “Notifications” menu entry has been added to the DSpace administrative menu, from where the repository manager will be able to manage the suggestions got from the different sources.

![menu](/_media/admin-menu.jpg)

A list of local profiles with candidate publications will be shown so that the repository manager can review them directly or support the researcher:
![source-dashboard](/_media/source-dashboard.jpg)

For each candidate the available suggestions are shown, sorted by the evaluated total score (summing up all the processed evidences ). The suggested authorship of each article can be confirmed importing the data locally, or rejected. This operation can be performed individually but also simultaneously for all the selected suggestions, speeding up the process. The decision can also be guided by inspecting the matching evidences which are displayed for each suggestion by clicking on 'See evidence'

![source-suggesions](/_media/source-suggestions.png)

The suggestions list can be sorted by total score descending or ascending (highlighting the weakest candidates).

![source-suggesions](/_media/suggestions-sorting.png)

## Researcher UI
The single researcher is also allowed to directly review his suggestions. Upon login he is informed about the availability of suggestions from one or more providers and can proceed to review the suggestions list in the same way than the Repository Manager

![source-suggesions](/_media/suggestion-login.jpg)

## Processing the decisions
The backend is responsible to process the repository manager or researcher decisions taken over the received suggestions. 
The publication to be imported are processed according to the Import from External Sources normal data flow of DSpace 7. Upon import the suggestion document is removed from the SOLR core, in case of rejection the document is updated flagging it as *rejected* so that it will be not longer proposed to the user.

## Rest Contract
Three endpoints have been designed to interact with the publication claim service

* `/api/integration/suggestionsources` to provide access to summary information about the available suggestion from each source (openaire, orcid, etc.)
* `/api/integration/suggestiontargets` to provide access to summary information about the available suggestions for a specific researcher
* `/api/integration/suggestions` to provide access to the detailed suggestions so that they can be reviewed and managed by the repository manager or the researcher to whom they related

### /api/integration/suggestionsources
[suggestionsources endpoint](https://raw.githubusercontent.com/4Science/Rest7Contract/oaire-eld/suggestionsources.md ':include')

### /api/integration/suggestiontargets
[suggestiontargets endpoint](https://raw.githubusercontent.com/4Science/Rest7Contract/oaire-eld/suggestiontargets.md ':include')
 
### /api/integration/suggestions
[suggestions endpoint](https://raw.githubusercontent.com/4Science/Rest7Contract/oaire-eld/suggestions.md ':include')
