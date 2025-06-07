---
title: Self-Hosted Repositories
weight: 50
bookToc: true
---

# Self-Hosted Repositories

By default LSUClient fetches packages from the official, public Lenovo Update Catalog at https://download.lenovo.com/catalog.
However, just like System Update, it also supports custom, self-hosted package repositories.

It is currently not possible to **create** or **manage** such a custom package repository with LSUClient.
Self-hosted repositories must either be created with the "Lenovo Update Retriever" program or finagled in
place manually or with your own scripts if you understand the structure (see [repository formats](#repository-formats)).

Maintaining your own package repository may be of interest if you:

- Require offline or network-internal operation
- Require total control over packages distributed
- Want to hack around, manually edit package metadata or create your own packages

{{< hint info >}}
If you want to run LSUClient on machines that do not have direct internet acccess, or you want to speed up package
download times as you deploy the same model of computer (and download the same packages) again and again, setting up
a caching proxy server may be a lower-maintenance alternative to a fully internal package repository.
{{< /hint >}}

To retrieve packages from a custom package repository, use the `Repository` parameter of `Get-LSUpdate` and point it to your repository:

{{< tabs "custom-repo-1" >}}
{{< tab "Web-based repository" >}}
You can serve your custom repository over HTTPS (or just plain HTTP) from a webserver as static files.

The webserver must accept GET and HEAD requests.
POST, PUT or any other methods are not required and can be disabled or blocked.

Example of retrieving packages from an internal webserver:
{{< render-markdown >}}
```powershell
Get-LSUpdate -Repository 'https://pkgs.domain.tld/lenovo-drivers'
```
{{< /render-markdown >}}
{{< /tab >}}
{{< tab "Filesystem-based repository" >}}
You can host your custom repository via a Windows filesystem path, either local or via SMB using a network drive or UNC-path.

The user account running LSUClient must have read-access to the repository. Write or Modify access is not required.

Example of retrieving packages from an internal fileserver:
{{< render-markdown >}}
```powershell
Get-LSUpdate -Repository '\\pkgs.domain.tld\lenovo-drivers$\'
```
{{< /render-markdown >}}
{{< /tab >}}
{{< /tabs >}}

## Repository formats

A package repository is simply a file and directory structure with one of two specific, expected package index
files in the root directory that I call Model-XML and Database-XML.

LSUClient supports fetching packages from both of these types of repositories directly.

### Model-XML 

This is the structure the official, public Lenovo repository uses. There are many XML files directly in the
repository root, one for each computer-model and major-OS-version combination, using the name scheme
`{{ Lenovo Machine Type Model (MTM) Code }}_Win{{ Major Version }}.xml`. These XML files then each contain
a list of links to individual packages relevant for that system.

{{< hint info >}}
I know of no automatic or official method to create a custom repository of this structure. The easiest way would be to download the XML files off of the public repository with a script and editing the "location" of each package to point to an internal or relative path instead of the public downloads.lenovo.com site.
{{< /hint >}}

For example [this](https://download.lenovo.com/catalog/20ls_win10.xml) is the "Model-XML" file
for a Lenovo ThinkPad L480 (Type 20LS) running Windows 10.

### Database-XML

This is the repository structure that Update Retriever creates. It uses a single file named `database.xml` in the repository root that contains a list of all packages in the repository and the information which computer-and-OS combinations each one is for.

{{< hint warning >}}
The `database.xml` package index does not preserve package Category information. This means that all packages sourced from repositories of this type will always have empty Categories.
{{< /hint >}}

{{< hint info >}}
You can also create and manage a repository with Update Retriever and then serve it (its root directory) via HTTP(S).
{{< /hint >}}

## Package finding process

Assuming a ThinkPad A275, model **20KD**001LGE, running Windows 10 as an example,
LSUClients process for discovering packages in a repository is as follows:

{{< mermaid >}}
%%{init:{"theme":"neutral"}}%%
flowchart TD;
    Start([Start]) --> B{"Does 20KD_Win10.xml exist\nin the repository root?"}
    B --> |No| C{"Does database.xml exist\nin the repository root?"}
    C --> |No| F([Error: No packages found])
    C --> |Yes| OK[Read it and find packages]
    B --> |Yes| H
    OK -->G{Are there packages for\nmodel 20KD + Windows 10?}
    G -->|No| F
    G -->|Yes| H[Process packages]
{{< /mermaid >}}

This means that when both a `database.xml` file and model-and-OS-specific XML files are present in a repository,
LSUClient will look for and use the Model-XML file first and then fall back to `database.xml` only when none was found.

