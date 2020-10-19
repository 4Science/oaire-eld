# Publication Claim

> Scenario 2: A new researcher joins the institution and logins for the first time in the repository. The publication claim services found most of their publications in the OpenAIRE network and prompts for import. The researcher reviews the list, confirms the authorship and imports the publication saving a significant amount of (often publicly payed) time. Moreover, the authorship confirmation will come back later to OpenAIRE offering useful information about the data quality and potential enrichment. The same applies for publications authored by researchers in different institutes, having the data in multiple repositories makes the data more reliable and raises the chance to get more information and content from any of the authors.  

The goal of the Publication Claim service is to support the scenario above.

The service has been designed to be independent from a specific provider or implementation so that it can be easily extended and maintained over time. Moreover, multiple providers can be active at the same time improving the chance to save researchers time.

In the original plan an integration with the [ReCiter open source platform](https://github.com/wcmc-its/ReCiter) was originally planned but over the phase 2 we found that the internal data structure of ReCiter was too tight to the PubMed Article Model to be adapted to work with the data provided by the openAIRE Research Graph within the budget limit and for such reason we switched to a direct integration with the openAIRE Research Graph via the [Publication REST API](http://api.openaire.eu/api.html#pubs). Other good candidates to be integrated via such framework are ORCID or commercial databases via their authors' IDs.

Three endpoints have been designed to expose the result of the processing to the DSpace UI and so to the Repository Managers and single researchers:

* `/api/integration/suggestionsources` to provide access to summary information about the available suggestion from each source (openaire, orcid, etc.)
* `/api/integration/suggestiontargets` to provide access to summary information about the available suggestions for a specific researcher
* `/api/integration/suggestions` to provide access to the detailed suggestions so that they can be reviewed and managed by the repository manager or the researcher to whom they related

The detailed REST contract for such endpoints are available on the [4Science Rest7Contract repository](https://github.com/4Science/Rest7Contract/tree/oaire-eld) and embedded at the bottom of the page for easy reference.

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