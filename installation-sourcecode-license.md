# Installation
You must follow the [installation process of DSpace 7](https://wiki.lyrasis.org/display/DSDOC7x/Installing+DSpace) starting from the appropriate [source code](#source-code), i.e. the DSpace-CRIS fork or the patched branches.

Two additional cores need to be created in solr
* nbevent: where the data related to the [Data Correction Service](data-correction.md) will be loaded
* suggestion: where the retrieved publications from the OpenAIRE Graph will be stored to be used in the [Publication Claim Service](publication-claim.md)

To add these cores to your solr simply stop the service, copy the cores configuration and restart the service. Assuming that solr is installed in its default location

```
sudo systemctl stop solr
cp -R /dspace-install/solr/nbevent /opt/solr/data/
cp -R /dspace-install/solr/suggestion /opt/solr/data/
sudo systemctl start solr
```

The [Data Correction Service](data-correction.md) requires a json file produced for your repository by the OpenAIRE Team periodically upon request, please [get in touch with them](https://www.openaire.eu/support/helpdesk) to enable the service for your repository.

Out-of-box both services have configuration that work at the best for most Institutions but you can consult the specific sections to learn more about the configuration options for the [Publication Claim Service](publication-claim.md) and the [Data Correction Service](data-correction.md) that allow you for instance to change the metadata mapping, the scorer algorithm used to evaluate suggestion, which topics should be filtered from the Notification Broker and how the different correction should be applied to your local data

## Source Code
The source code related to the project have been merged in the official DSpace-CRIS 7 branch and it will be part of the official release 
* REST Contract: https://github.com/4Science/Rest7Contract/tree/dspace-cris-7
* DSpace REST Backend: https://github.com/4Science/DSpace/tree/dspace-cris-7
* DSpace Angular Frontend: https://github.com/4Science/dspace-angular/tree/dspace-cris-7

DSpace users that want to adopt the solution without endorse the full DSpace-CRIS extension should look to the following branches
* REST Contract: https://github.com/4Science/Rest7Contract/tree/dspace-oaire-eld
* DSpace REST Backend: https://github.com/4Science/DSpace/tree/dspace-oaire-eld
* DSpace Angular Frontend: https://github.com/4Science/dspace-angular/tree/dspace-oaire-eld

from where we have created the corresponding PRs to propose the inclusion of these features in the official DSpace codebase. Your feedback and testing on such PRs is highly appreciated and would enhance the chance to have them accepted.

## License
The source code is released under the same license used by the [DSpace](https://duraspace.org/dspace/) and [DSpace-CRIS](https://wiki.lyrasis.org/display/DSPACECRIS/) community projects: the [BSD 3-Clause license](https://github.com/DSpace/DSpace/blob/main/LICENSE).
The ownership has assigned to the DSpace project owner, LYRASIS, from the start, retaining only the intellectual attribution to simplify the process of acceptance in the mainstream project. 