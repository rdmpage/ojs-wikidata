# OJS Wikidata

Adding data on Open Journal Systems to Wikidata. Data source is [Details of publications using software by the Public Knowledge Project](https://doi.org/10.7910/DVN/OCZNVY) from the paper Khanna et al. 2022 [Recalibrating the scope of scholarly publishing: A modest step in a vast decolonization process](https://doi.org/10.1162/qss_a_00228).

## Data

Primary data comes from https://doi.org/10.7910/DVN/OCZNVY.

Other data can be found in the [Open Journal Systems Global](https://github.com/j-a-ball/ojs-global) repository.

### Notebooks

There are some interesting notebooks associated with Khanna et al., e.g. [https://nbviewer.org/github/j-a-ball/ojs-global/blob/main/notebooks/JNB3_Languages.ipynb#Heuristic-approach-to-language-verification:](Heuristic-approach-to-language-verification)

### Errors in data

Some journals have the wrong ISSN leading to the wrong mapping with Wikidata.

Journal | Wrong ISSN
--|--
Revista Rezende | 1687-4110 
SENDERO: Investigación en Ciencias Sociales, Humanas y Estudios Territoriales | 0020-1812

It also seems that many URLs for journals don’t work.

## Wikidata

To bring this data into Wikidata I first imported `beacon.tsv` into a SQLIte database to make searching and editing easier.

We need to map journal ISSNs to Wikidata, and also country codes. Khanna et al. determine country codes based on ISSNs, domain names, and IP geolocation. This is not infallible, and some codes will be incorrect. For instance, the fashionable domain `.io` is unlikely to actually be from the British Indian Ocean Territory, nor is `.tk` likely to be from Tokelau. Some country codes are also not countries but smaller regions (e.g., `HK` for Hong Kong, and `PR` for Puerto Rico).

## Quickstatements

SQL to generate quickstatements

### Add OJS engine, version, and date at which this statement was true

```sql
SELECT rdmp_wikidata, "P408", "Q1710177", "S248", "Q116742355", "P348", """" || version || """", "P585",  "+" || SUBSTRING(lastbeacon, 1, 10) || "T00:00:00Z/11" FROM beacon WHERE rdmp_wikidata IS NOT NULL AND totalrecordcount > 0 AND version <> "";
```

20,683 records

### Publication should be an instance of online publication

```sql
SELECT rdmp_wikidata, "P31", "Q1714118" FROM beacon WHERE rdmp_wikidata IS NOT NULL AND totalrecordcount > 0;
```

### Country

```sql
SELECT rdmp_wikidata, "P495", rdmp_country_wikidata, "S248", "Q116742355" FROM beacon WHERE rdmp_wikidata IS NOT NULL AND totalrecordcount > 0;
```

## Queries

### How many use OJS?

```
SELECT (COUNT(?journal) AS ?c) WHERE
{
  ?journal wdt:P408 wd:Q1710177 .
}
```

Before adding data this gives 4337 items using OJS

### List type(s) of publications using OJS

```
SELECT ?label (COUNT(?journal) AS ?c) WHERE
{
  ?journal wdt:P31/wdt:P279* wd:Q1002697 .
  ?journal wdt:P31 ?type .
  ?type rdfs:label ?label . 
  FILTER(LANG(?label) = "en") .
  ?journal wdt:P408 wd:Q1710177 .
}
GROUP BY ?label
```

### What countries host OJS?

```
SELECT (COUNT(?country) AS ?c) ?label  WHERE {
  ?journal wdt:P408 wd:Q1710177 .
  ?journal wdt:P495 ?country .
  ?country rdfs:label ?label . 
  FILTER(LANG(?label) = "en") .  
}
GROUP BY ?label
ORDER BY DESC(?c)
```


### What publishing engines do journals use?

```
SELECT ?label (COUNT(DISTINCT(?journal)) AS ?c) WHERE
{
  ?journal wdt:P31/wdt:P279* wd:Q1002697 .
 
  ?journal wdt:P408 ?engine .
  ?engine rdfs:label ?label . 
  FILTER(LANG(?label) = "en") .  
}
GROUP BY ?label
```

## Reading

http://dx.doi.org/10.31274/jlsc.12915

http://hdl.handle.net/1805/22819

https://educopia.org/mapping-scholarly-communications-infrastructure/



