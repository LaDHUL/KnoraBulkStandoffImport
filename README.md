# How to create a standoff mapping and use bulk import

(Using Lumieres Lausanne ontology as example)

## Create standoff mapping

1.  Copy default Knora HTML standoff mapping (`mappingForStandardHTML.xml`) into file `mappingForLL.xml`

2.  Add in this file the mapping for `<span>` with its attributes:

```xml
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
```

3.  Create description file of this mapping `mappingForLL.json`:

```json
{
  "project_id": "http://rdfh.ch/projects/0113",
  "label": "LL Standoff mapping",
  "mappingName": "LLStandoffMapping"
}
```

4.  Create a turtle file (`lumieres-lausanne-standoff.ttl`) for our standoff classes:

```
lumieres-lausanne:standoffEditionTagHasFix rdf:type owl:DatatypeProperty ;
                  knora-base:subjectClassConstraint lumieres-lausanne:StandoffEditionTag ;
                  knora-base:objectDatatypeConstraint xsd:string .

lumieres-lausanne:standoffEditionTagHasTitle rdf:type owl:DatatypeProperty ;
                  knora-base:subjectClassConstraint lumieres-lausanne:StandoffEditionTag ;
                  knora-base:objectDatatypeConstraint xsd:string .

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
```

## Add standoff mapping to Knora

Using an existing Knora server having the LL ontology installed:

1.  Load `lumieres-lausanne-standoff.ttl` in graphdb (into `http://www.knora.org/ontology/0113/lumieres-lausanne`)

2.  Load the mapping:

```shell
$ curl -u lumieres@unil.ch:test -X POST -F json=@mappingForLL.json -F xml=@mappingForLL.xml  http://localhost:3333/v1/mapping

{"mappingIri":"http://rdfh.ch/projects/0113/mappings/LLStandoffMapping","status":0}
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

    <p0113-lumieres-lausanne:Person id="person_import">
        <knoraXmlImport:label>person_import</knoraXmlImport:label>
        <p0113-lumieres-lausanne:hasBiography knoraType="richtext_value" mapping_id="http://rdfh.ch/projects/0113/mappings/LLStandoffMapping">
            <text xmlns="">
                <p>
                    <span class="sic" data-corr="voici l'adaptation" title="voici l'adaptation">Adaptation éditoriale</span>
                </p>
            </text>
        </p0113-lumieres-lausanne:hasBiography>
        <p0113-lumieres-lausanne:hasName knoraType="richtext_value">hasName</p0113-lumieres-lausanne:hasName>
        <p0113-lumieres-lausanne:hasOldId knoraType="richtext_value">hasOldId</p0113-lumieres-lausanne:hasOldId>
        <p0113-lumieres-lausanne:mayHaveBiography knoraType="boolean_value">true</p0113-lumieres-lausanne:mayHaveBiography>
    </p0113-lumieres-lausanne:Person>

</knoraXmlImport:resources>
```

## Launch bulk import

```shell
$ curl -u lumieres@unil.ch:test -X POST -d @data.xml http://localhost:3333/v1/resources/xmlimport/http%3A%2F%2Frdfh.ch%2Fprojects%2F0113

{"createdResources":[{"clientResourceID":"person_import","resourceIri":"http://rdfh.ch/0113/oLtoCaxDTNOCdNUEeM_qNA","label":"person_import"}],"status":0}
```

## Hack Salsah 1 to render correctly the new standoff mapping

By default, rich text property are edited in Salsah with CKEditor configured to manage the default Knora HTML standoff mapping.

1.  Add **Source** button to CKEditor toolbar by patching the file `salsah1/src/public/js/jquery.htmleditor.js:194`:

```javascript
        { name: 'tools', items: [ 'Maximize', 'Source' ] }
```

2.  Allow CKEditor to accept `<span>` tag and its attributes by patching the file `salsah1/src/public/js/jquery.htmleditor.js:145`:

```javascript
var filter =
  ' p em strong strike u sub sup hr h1 h2 h3 h4 h5 h6 pre table tbody tr td ol ul li cite blockquote code; a[!href](salsah-link); span(sic)[data-corr,title] ';
```

3.  **FIXME:** on save, Salsah/CKEditor should write text values with the mapping_id of the value and not with the default provided by Knora (probably hardcoded)
