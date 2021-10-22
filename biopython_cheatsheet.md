---
title: "Bioyython cheatsheet"
subtitle: "Summary from the Udemy course on Biopython"
author: Sébastien Wieckowski
date: \today
output:
    pdf_document:
        latex_engine: xelatex
        toc: true
        pandoc_args: ["--variable=geometry:margin=1.4cm",
                      '--variable=mainfont="Arial"',
                      "--variable=fontsize=10pt",
                      "--toc-depth=4",
                      ]
---

## Advanced search at NCBI

```python
from Bio import Entrez

Entrez.email = 'sbwiecko@free.fr' # always tell Entrez who you are
```

### Obtaining information about the Entrez databases with `einfo()`

See also [https://biopython.org/DIST/docs/tutorial/Tutorial.html#sec145](https://biopython.org/DIST/docs/tutorial/Tutorial.html#sec145).

```python
record = Entrez.read(Entrez.einfo())
print(record['DbList'])
```

See more details in [A General Introduction to the E-utilities](https://www.ncbi.nlm.nih.gov/books/NBK25497/).

| Entrez Database   | UID common name    | E-utility Database Name |
| ----------------- | ------------------ | ----------------------- |
| BioProject        | BioProject ID      | bioproject              |
| BioSample         | BioSample ID       | biosample               |
| Biosystems        | BSID               | biosystems              |
| Books             | Book ID            | books                   |
| Conserved Domains | PSSM-ID            | cdd                     |
| dbGaP             | dbGaP ID           | gap                     |
| dbVar             | dbVar ID           | dbvar                   |
| Epigenomics       | Epigenomics ID     | epigenomics             |
| EST               | GI number          | nucest                  |
| Gene              | Gene ID            | gene                    |
| Genome            | Genome ID          | genome                  |
| GEO Datasets      | GDS ID             | gds                     |
| GEO Profiles      | GEO ID             | geoprofiles             |
| GSS               | GI number          | nucgss                  |
| HomoloGene        | HomoloGene ID      | homologene              |
| MeSH              | MeSH ID            | mesh                    |
| NCBI C++ Toolkit  | Toolkit ID         | toolkit                 |
| NCBI Web Site     | Web Site ID        | ncbisearch              |
| NLM Catalog       | NLM Catalog ID     | nlmcatalog              |
| Nucleotide        | GI number          | nuccore                 |
| OMIA              | OMIA ID            | omia                    |
| PopSet            | PopSet ID          | popset                  |
| Probe             | Probe ID           | probe                   |
| Protein           | GI number          | protein                 |
| Protein Clusters  | Protein Cluster ID | proteinclusters         |
| PubChem BioAssay  | AID                | pcassay                 |
| PubChem Compound  | CID                | pccompound              |
| PubChem Substance | SID                | pcsubstance             |
| PubMed            | PMID               | pubmed                  |
| PubMed Central    | PMCID              | pmc                     |
| SNP               | rs number          | snp                     |
| SRA               | SRA ID             | sra                     |
| Structure         | MMDB-ID            | structure               |
| Taxonomy          | TaxID              | taxonomy                |
| UniGene           | UniGene Cluster ID | unigene                 |
| UniSTS            | STS ID             | unists                  |

### Read information from a particular database

```python
handle = Entrez.einfo(db='sra') # added `db` param

# convert from XML to python datatype
record = Entrez.read(handle)

for key in record['DbInfo'].keys(): # changed the key name
    print(key, ':', record['DbInfo'][key])

# or directly
print(Entrez.read(Entrez.einfo(db='pcassay'))['DbInfo']['Description'])
```

Get all the field names from the PubMed database:

```python
handle = Entrez.einfo(db="pubmed")
record = Entrez.read(handle)

for field in record["DbInfo"]["FieldList"]:
    print(f"{field['Name']}, {field['FullName']}, {field['Description']}")
```

### Obtaining spelling suggestions with `espell()`

Suggests spelling corrections. See also [https://biopython.org/DIST/docs/tutorial/Tutorial.html#sec152](https://biopython.org/DIST/docs/tutorial/Tutorial.html#sec152). The result includes the original query, the complete corrected query, and a breakdown of the terms in the complete corrected query flagged as either "replaced" or "original".

```python
for term in misspelled_terms: # given a list of misspelled terms
    handle = Entrez.espell(
        db='taxonomy', # also works with PMC and certainly other databases
        term=sciName,
    )

    if bool(record['SpelledQuery']): # when the list containing the spelled terms is not empty
        print("Query: ", record['Query'], " - Corrected query: ", record['CorrectedQuery'])
```

### Search NCBI databases

To search any of these databases, we use [esearch](https://biopython.org/DIST/docs/tutorial/Tutorial.html#sec146). In the output, you see PubMed IDs. You can also use ESearch to search GenBank, where each of the IDs is a GenBank identifier (Accession number). You can do things like Jones[AUTH] to search the author field, or Sanger[AFFL] to restrict to authors at the Sanger Centre. This can be very handy - especially if you are not so familiar with a particular database.

```python
term = "(U-937 OR U937) AND interferon"
handle = Entrez.esearch(
    db='sra',
    term=term,
)

record = Entrez.read(handle)

print(record)
```

### Retrieving documents summaries

[esummary](https://biopython.org/DIST/docs/tutorial/Tutorial.html#sec148) retrieves document summaries from a list of primary IDs.

```python
handle = Entrez.esearch(
    db='pubmed',
    term="(bromodomain[TITL] OR BRD4[TITL] OR BET inhibitor[TITL]) AND (review[PTYP] OR review[TITL])",
    retmax=100,
    sort='most+recent',
    mindate=2017,
    maxdate=2021, # daterange; (mindate, maxdate) must be used together
)

record = Entrez.read(handle)

list_pubs = record['IdList']

filename = "./biblio.txt"
if os.path.exists(filename): os.remove(filename)

for id in record['IdList']:
    handle = Entrez.esummary(
        db='pubmed',
        id=id,
    )

    summary = Entrez.read(handle)[0]

    # either we print them
    #print(f"{summary['Id']}\t{summary['Title']}\t{summary['PubDate']}\t{summary['FullJournalName']}")

    # or save them into a txt file
    with open(filename, "a+", encoding="utf-8") as file:
        file.write(f"{summary['Id']}\t{summary['Title']}\t{summary['PubDate']}\t{summary['FullJournalName']}\n")
```

### ## Global search at NCBI

[EGQuery](https://biopython.org/DIST/docs/tutorial/Tutorial.html#sec151) provides counts for a search term in each of the Entrez databases (i.e. a global query). This is particularly useful to find out how many items your search terms would find in each database without actually performing lots of separate searches with ESearch

```python
handle = Entrez.egquery(term="Cancer")
record = Entrez.read(handle)

for row in record["eGQueryResult"]:
    if (db:=row['DbName']) in ['pubmed', 'mesh', 'books', 'nuccore']:
        print(db, ": ", row['Count'])
```

