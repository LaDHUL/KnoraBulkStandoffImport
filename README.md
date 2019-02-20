# How to create a standoff mapping and use bulk import

Using Lumieres Lausanne ontology as example to manage xml encoding:

```html
<span class="sic" data-corr="voici l'adaptation" title="voici l'adaptation"
  >Adaptation éditoriale</span
>
```

Knora docs:

- https://docs.knora.org/paradox/03-apis/api-v1/xml-to-standoff-mapping.html#creating-a-custom-mapping
- https://docs.knora.org/paradox/03-apis/api-v2/xml-to-standoff-mapping.html

## Create standoff mapping

1.  Preapre ontology for standoff tags into file `lumieres-lausanne-standoff.ttl`:

```turtle
@prefix owl: <http://www.w3.org/2002/07/owl#> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix xml: <http://www.w3.org/XML/1998/namespace> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix knora-base: <http://www.knora.org/ontology/knora-base#> .
@prefix salsah-gui: <http://www.knora.org/ontology/salsah-gui#> .
@prefix lumieres-lausanne: <http://www.knora.org/ontology/0113/lumieres-lausanne#> .
@base <http://www.knora.org/ontology/0113/lumieres-lausanne> .

##########################################################
# <span class="" data-corr="" title=""></span
##########################################################
lumieres-lausanne:StandoffEditionTag rdf:type owl:Class ;
    rdfs:subClassOf knora-base:StandoffTag,
                   [
                      rdf:type owl:Restriction ;
                      owl:onProperty lumieres-lausanne:standoffEditionTagHasFix ;
                      owl:cardinality "1"^^xsd:nonNegativeInteger
                   ] ,
                   [
                      rdf:type owl:Restriction ;
                      owl:onProperty lumieres-lausanne:standoffEditionTagHasTitle ;
                      owl:cardinality "1"^^xsd:nonNegativeInteger
                   ] ;
    rdfs:label "Represents an edition fix in a TextValue"@en.


lumieres-lausanne:standoffEditionTagHasFix rdf:type owl:DatatypeProperty ;
                  knora-base:subjectClassConstraint lumieres-lausanne:StandoffEditionTag ;
                  knora-base:objectDatatypeConstraint xsd:string .

lumieres-lausanne:standoffEditionTagHasTitle rdf:type owl:DatatypeProperty ;
                  knora-base:subjectClassConstraint lumieres-lausanne:StandoffEditionTag ;
                  knora-base:objectDatatypeConstraint xsd:string .

```

2. Copy default Knora HTML standoff mapping (`webapi/_test_data/test_route/texts/mappingForStandardHTML.xml`) into file `mappingForLL.xml`

(copy schema for validation: `webapi/src/main/resources/mappingXMLToStandoff.xsd`)

3. Add at the end of this file the mapping for `<span>` with its attributes:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<mapping ns:noNamespaceSchemaLocation="mappingXMLToStandoff.xsd"
  xmlns:ns="http://www.w3.org/2001/XMLSchema-instance">

[mappingForStandardHTML.xml mapping elements]

<mappingElement>
        <tag>
            <name>span</name>
            <class>sic</class>
            <namespace>noNamespace</namespace>
            <separatesWords>true</separatesWords>
        </tag>
        <standoffClass>
            <classIri>http://www.knora.org/ontology/0113/lumieres-lausanne#StandoffEditionTag</classIri>
            <attributes>
                <attribute>
                    <attributeName>data-corr</attributeName>
                    <namespace>noNamespace</namespace>
                    <propertyIri>http://www.knora.org/ontology/0113/lumieres-lausanne#standoffEditionTagHasFix</propertyIri>
                </attribute>
                <attribute>
                    <attributeName>title</attributeName>
                    <namespace>noNamespace</namespace>
                    <propertyIri>http://www.knora.org/ontology/0113/lumieres-lausanne#standoffEditionTagHasTitle</propertyIri>
                </attribute>
            </attributes>
        </standoffClass>
</mappingElement>

</mapping>
```

4.  Create description file of this mapping `mappingForLL.json`:

```json
{
  "knora-api:mappingHasName": "LLStandoffMapping",
  "knora-api:attachedToProject": {
    "@id": "http://rdfh.ch/projects/0113"
  },
  "rdfs:label": "LLStandoffMapping",
  "@context": {
    "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
    "knora-api": "http://api.knora.org/ontology/knora-api/v2#"
  }
}
```

## Add standoff mapping to Knora

Using an existing Knora server having the LL ontology installed:

1.  Load `lumieres-lausanne-standoff.ttl` in graphdb (into `http://www.knora.org/ontology/0113/lumieres-lausanne`)

2.  Load the mapping:

```shell
$ curl -u admin_test@unil.ch:test -X POST -F json=@mappingForLL.json -F xml=@mappingForLL.xml  http://localhost:3333/v2/mapping
```

```json
{
  "@id": "http://rdfh.ch/projects/0113/mappings/LLStandoffMapping",
  "@type": "http://api.knora.org/ontology/knora-api/v2#XMLToStandoffMapping",
  "http://api.knora.org/ontology/knora-api/v2#attachedToProject": {
    "@id": "http://rdfh.ch/projects/0113"
  },
  "rdfs:label": "LLStandoffMapping",
  "@context": {
    "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
    "rdf": "http://www.w3.org/1999/02/22-rdf-syntax-ns#",
    "owl": "http://www.w3.org/2002/07/owl#",
    "xsd": "http://www.w3.org/2001/XMLSchema#"
  }
}
```

3. Create a `ttl` representation to load at project ontology intialization:

https://github.com/dhlab-basel/Knora/issues/1212

```sparql
PREFIX knora-base: <http://www.knora.org/ontology/knora-base#>

Construct {
    ?mapping ?p ?o .

    ?mapping knora-base:hasMappingElement ?mEle .
    ?mEle ?pp ?oo .
    ?oo ?ppp ?ooo .
}
FROM <http://www.knora.org/data/0113/lumieres-lausanne>
Where {
    BIND(<http://rdfh.ch/projects/0113/mappings/LLStandoffMapping> as ?mapping)
    ?mapping ?p ?o .

    OPTIONAL {
        ?mapping knora-base:hasMappingElement ?mEle .
        ?mEle ?pp ?oo .

        ?oo ?ppp ?ooo .
    }
}
```

## Prepare bulk import

1.  Get ontology schema:

```shell
$ wget http://localhost:3333/v1/resources/xmlimportschemas/http%3A%2F%2Fwww.knora.org%2Fontology%2F0113%2Flumieres-lausanne -O LL-schema.zip
```

2.  Unzip downloaded file:

```shell
$ unzip LL-schema.zip
Archive:  LL-schema.zip
  inflating: p0113-lumieres-lausanne.xsd
  inflating: knoraXmlImport.xsd
```

3.  Create data file (`data.xml`) with rich text data containing `<span>` tag:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<knoraXmlImport:resources xmlns="http://api.knora.org/ontology/0113/lumieres-lausanne/xml-import/v1#"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://api.knora.org/ontology/0113/lumieres-lausanne/xml-import/v1# p0113-lumieres-lausanne.xsd"
    xmlns:p0113-lumieres-lausanne="http://api.knora.org/ontology/0113/lumieres-lausanne/xml-import/v1#"
    xmlns:knoraXmlImport="http://api.knora.org/ontology/knoraXmlImport/v1#">

    <Person id="person_import">
        <knoraXmlImport:label>person_import</knoraXmlImport:label>
        <hasBiography knoraType="richtext_value" mapping_id="http://rdfh.ch/projects/0113/mappings/LLStandoffMapping">
            <text>
                <p>
                    <span class="sic" data-corr="voici l'adaptation" title="voici l'adaptation">Adaptation éditoriale</span>
                </p>
            </text>
        </hasBiography>
        <hasName knoraType="richtext_value">Auteur à intégrer - via LL</hasName>
        <isModern knoraType="boolean_value">true</isModern>
        <mayHaveBiography knoraType="boolean_value">false</mayHaveBiography>
        <noticeIsCreated knoraType="boolean_value">false</noticeIsCreated>
        <noticeIsValid knoraType="boolean_value">false</noticeIsValid>
    </Person>
</knoraXmlImport:resources>
```

## Launch bulk import

```shell
$ curl -u admin_test@unil.ch:test -X POST -d @data.xml http://localhost:3333/v1/resources/xmlimport/http%3A%2F%2Frdfh.ch%2Fprojects%2F0113

{"createdResources":[{"clientResourceID":"person_import","resourceIri":"http://rdfh.ch/0113/oLtoCaxDTNOCdNUEeM_qNA","label":"person_import"}],"status":0}
```

## Hack Salsah 1 to render correctly the new standoff mapping

By default, rich text property are edited in Salsah with CKEditor configured to manage the default Knora HTML standoff mapping.

1.  Add **Source** button to CKEditor toolbar by patching the file [jquery.htmleditor.js:194](https://github.com/dhlab-basel/Knora/blob/51b19ac26f7e30f7e47b6e66ed00206bd2932df0/salsah1/src/public/js/jquery.htmleditor.js#L194):

```javascript
        { name: 'tools', items: [ 'Maximize', 'Source' ] }
```

2.  Allow CKEditor to accept `<span>` tag and its attributes by patching the file [jquery.htmleditor.js:145](https://github.com/dhlab-basel/Knora/blob/51b19ac26f7e30f7e47b6e66ed00206bd2932df0/salsah1/src/public/js/jquery.htmleditor.js#L145):

```javascript
var filter =
  ' p em strong strike u sub sup hr h1 h2 h3 h4 h5 h6 pre table tbody tr td ol ul li cite blockquote code; a[!href](salsah-link); span(sic)[data-corr,title] ';
```

**FIXME:** ([Salsah issue](https://github.com/dhlab-basel/Knora/issues/908)) this configuration should be created on the fly by getting the mapping associated to the value (Knora provides the possibility to have one mapping per value!)

3.  **FIXME:** ([Salsah issue](https://github.com/dhlab-basel/Knora/issues/907)) on save, Salsah/CKEditor should write text values with the mapping_id of the value and not with the default provided by Knora (probably hardcoded)
