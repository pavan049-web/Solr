1. Requirement :
Search suggestions (also called key word suggestions) should always offer at least one positive result

Example "green m" keyword brings up "green medium," as a search suggestion which leads to "zero successful search results". Out of the first 5 search suggestions returned by the application, display only the ones with positive results and suppress the suggestions with zero results

2. changes made in solrconfig.xml: 
To achieve best suggestion for given term, Need to use new dictionary in solr and comment existing dictionary.  Following changes are required in solrconfig.xml.

// to be modified
<searchComponent name="suggest" class="solr.SuggestComponent">
      
        <lst name="suggester">
            <str name="name">wfstl_en</str>
            <str name="lookupImpl">org.apache.solr.spelling.suggest.fst.WFSTLookupFactory</str>          
            <str name="suggestAnalyzerFieldType">text</str>
            <str name="field">name_autocomplete</str>
            <str name="buildOnStartup">false</str>            
            <str name="buildOnCommit">false</str>
            <str name="buildOnOptimize">false</str>
            <str name="accuracy">0.35</str>
        </lst>
//dictionary will used for french content
 		<lst name="suggester">
            <str name="name">wfstl_fr</str>
            <str name="lookupImpl">org.apache.solr.spelling.suggest.fst.WFSTLookupFactory</str>          
            <str name="suggestAnalyzerFieldType">text</str>
            <str name="field">name_autocomplete_fr</str>
            <str name="buildOnStartup">false</str>            
            <str name="buildOnCommit">false</str>
            <str name="buildOnOptimize">false</str>
            <str name="accuracy">0.35</str>
        </lst>
    </searchComponent>
    
    <requestHandler name="/suggest" class="solr.SearchHandler" startup="lazy" >
        <lst name="defaults">
            <str name="suggest">true</str>            
            <str name="suggest.dictionary">wfstl_en</str>
            <str name="suggest.count">10</str>
            <str name="suggest.build">true</str>            
        </lst>
        <arr name="components">
            <str>suggest</str>
        </arr>
    </requestHandler>
	
3. changes made in schema.xml: 
Need to define new text field type in schema.xml with WhitespaceTokenizerFactory analyzer and ShingleFilterFactory filter factory.

Need to define new field "name_autocomplete" for en and  "name_autocomplete_fr" for fr language of type "text_auto" to fetch suggestions for given term. Data will be copied from autosuggest_en and autosuggest field to  "name_autocomplete" field and autosuggest_fr  to  "name_autocomplete_fr" .

The fields autosuggest_en,  autosuggest_fr and autosuggest contains data of all those hybris product/category attribute that are marked as auto complete in solr configuration.

<fieldType class="solr.TextField" name="text_auto" positionIncrementGap="100">
        <analyzer>
            <tokenizer class="solr.WhitespaceTokenizerFactory"/>
            <filter class="solr.LowerCaseFilterFactory"/>
            <filter class="solr.ShingleFilterFactory" minShingleSize="3" maxShingleSize="20" outputUnigrams="true"/>
        </analyzer>      
 </fieldType>

	<field name="name_autocomplete" type="text_auto" indexed="true" stored="true" multiValued="true" />
	<field name="name_autocomplete_fr" type="text_auto" indexed="true" stored="true" multiValued="true" />
     <copyField source="autosuggest_en" dest="name_autocomplete" />
     <copyField source="autosuggest" dest="name_autocomplete" />
	<copyField source="autosuggest_fr" dest="name_autocomplete_fr" />
	
4. based on above configuration, Following prooperties are used

request :  /suggest
handler: searchhandler
suggest: true
suggest.dictionary : wfstl (weighted automaton representation)
suggest.build: true (build the suggestion index)
suggest.count: 10 (number of suggestions for request to return)
suggester component: SuggestComponent
LookupImpl: WFSTLookupFactory
Suggest field: name_autocomplete
Field type: TextField
Suggest Query format:  {q=green+m&qt=/suggest&suggest.q=green+m&wt=javabin&version=2}

Full request: http://localhost:8983/solr/master_usb2c_Product/suggest?wt=json&indent=true&{q=green+m&qt=/suggest&suggest.q=green+m&wt=javabin&version=2}	

5. Suggestion Response: 

{
  "responseHeader":{
    "status":0,
    "QTime":55},
  "command":"build",
  "suggest":{"fstl":{
      "green m":{
        "numFound":7,
        "suggestions":[{
            "term":"green mountain coffee®",
            "weight":190,
            "payload":""},
          {
            "term":"green mountain naturals®",
            "weight":6,
            "payload":""},
          {
            "term":"green mountain colombian",
            "weight":2,
            "payload":""},
          {
            "term":"green mountain colombian decaf",
            "weight":2,
            "payload":""},
          {
            "term":"green mountain coffee",
            "weight":1,
            "payload":""},
          {
            "term":"green mountain coffee artisan",
            "weight":1,
            "payload":""},
          {
            "term":"green mountain coffee artisan mug",
            "weight":1,
            "payload":""}]}}}}
	