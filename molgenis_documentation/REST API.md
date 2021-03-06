# REST API

MOLGENIS has a REST API that allows you to access the data and metadata inside MOLGENIS from the outside through HTTP requests. The REST API allows you to login using your molgenis user account, retrieve data and metadata, and query, aggregate, insert or update the data.

These HTTP requests are platform independent, so you should be able to create them quite easily on most client platforms.
For a couple common platforms we provide you with a higher level client-side MOLGENIS API that should make things even easier.


## R

The R API is a script that is hosted on MOLGENIS on the URL `http[s]://<your.molgenis.url>/molgenis.R`.

### [Example: Plotting Allele-Specific Expression data from MOLGENIS in R](id:example-rest-R)
As an example, let's create a plot for publicly available ASE data available on https://molgenis56.target.rug.nl/. For a description of the data, take a look at [http://molgenis.org/ase](http://molgenis.org/ase).

Start up the R environment.

In the shell type:

```R
library('RCurl')
eval(expr = parse(text = getURL("https://molgenis56.target.rug.nl/molgenis.R")))
```

This loads the R API from the molgenis56 server. If you take a look in your workspace by typing

```R
ls()
```
you should see that a couple functions have been added for you to use:

```
 [1] "molgenis.add"                  "molgenis.addAll"               "molgenis.addList"              "molgenis.delete"               "molgenis.env"                 
 [6] "molgenis.get"                  "molgenis.getAttributeMetaData" "molgenis.getEntityMetaData"    "molgenis.login"                "molgenis.logout"              
[11] "molgenis.update"     
```

Let's load some data from the server using `molgenis.get`:

```R
molgenis.get("ASE")
```

This retrieves the top 1000 rows from the ASE entity.

```
P_Value Samples      SNP_ID Chr       Pos                           Genes
1    0.000000000000000020650473933963698652198164782417962682333833636491755847419682368126814253628253936767578125000000000000000000000000000000000000000000000000000000000000000000000     145   rs9901673  17   7484101 ENSG00000129226,ENSG00000264772
2    0.000000000000000008781097353981130661746700850633192724259771276345502150073585312384238932281732559204101562500000000000000000000000000000000000000000000000000000000000000000000     359   rs2597775   4  17503382                 ENSG00000151552
3    0.000000000000000001491745894983400057481059632909089858257546335023040629669255352496293198782950639724731445312500000000000000000000000000000000000000000000000000000000000000000     301      rs3216  11    214421                 ENSG00000177963
[...]
1000 0.000132500824069775005771554265976419628714211285114288330078125000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000      47   rs1056019  12  41337435                 ENSG00000018236
```

Let's retrieve a specific SNP from the ASE entity:

```R
molgenis.get("ASE", q="SNP_ID==rs12460890")
```

```
  Fraction_alternative_allele Likelihood_ratio_test_D Alternative_allele Reference_allele                P_Value Samples     SNP_ID Chr    Pos           Genes
1                       0.527                56.02079               TRUE                C 0.00000000000007170854      21 rs12460890  19 829568 ENSG00000172232
```

This SNP has a mild but significant allele-specific expression, based on expression counts in 21 samples.

Let's retrieve the samples for this SNP:

```R
samples <- molgenis.get("SampleAse", q="SNP_ID==rs12460890")
print(samples)
```

```
       SNP_ID SampleIds Ref_Counts Alt_Counts Chromosome Position      ID
1  rs12460890 ERS194242        130        121         19   829568 1418785
2  rs12460890 ERS326942       4142       4791         19   829568 1418786
3  rs12460890 ERS327006         19         28         19   829568 1418787
4  rs12460890 SRS353551         19         23         19   829568 1418788
5  rs12460890 SRS271084         32         11         19   829568 1418789
6  rs12460890 SRS375020        639        572         19   829568 1418790
7  rs12460890 SRS375024        202        309         19   829568 1418791
8  rs12460890 SRS375022        423        401         19   829568 1418792
9  rs12460890 SRS375030        271        234         19   829568 1418793
10 rs12460890 SRS375026        806       1081         19   829568 1418794
11 rs12460890 SRS375027        213        201         19   829568 1418795
12 rs12460890 SRS376459         74         96         19   829568 1418796
13 rs12460890 SRS375032        730        655         19   829568 1418797
14 rs12460890 SRS376461        584        699         19   829568 1418798
15 rs12460890 SRS376464        331        391         19   829568 1418799
16 rs12460890 SRS376469         13         14         19   829568 1418800
17 rs12460890 SRS376467         70        101         19   829568 1418801
18 rs12460890 SRS376468         47         35         19   829568 1418802
19 rs12460890 SRS418748         19         28         19   829568 1418803
20 rs12460890 SRS418754         44         47         19   829568 1418804
21 rs12460890 SRS418755         60         55         19   829568 1418805
```

There they are.
Let's plot the expression counts in these samples in a scatter plot.

```R
plot(samples$Ref_Counts, samples$Alt_Counts, xlim = c(0, 5000), ylim = c(0, 5000), xlab='Reference Allele', ylab='Alternative Allele', main = 'Allele-Specific Expression for rs12460890')
```

And add a line for the non-specific expression.

```R
lines(c(0,5000), c(0, 5000))
```
![image](images/rs12460890-r.png)

### Query format
The query must be in [fiql/rsql format](https://github.com/jirutka/rsql-parser).

### Non-anonymous access
The ASE data from the previous example is publicly available.
To access private data, you can log in using

```R
molgenis.login("your username", "your password")
```
This will create a molgenis token on the server and set it in the `molgenis.token` variable in your R workspace.

When you're done, you can log out using

```R
molgenis.logout()
```

### Retrieving more rows
By default, `molgenis.get` will retrieve up to 1000 rows only.

If you need more rows, you can request up to 10000 rows by adding the `num` parameter:

```R
molgenis.get("ASE", num=2000)
```
will retrieve the top 2000 rows from the ASE entity.

### Pagination
You can retrieve the data page-by-page.

```R
molgenis.get("ASE", num=5)
```
will retrieve a first page of 5 rows and

```R
molgenis.get("ASE", num=5, start=5)
```
will retrieve the second page of 5 rows.

### Server
By default, the MOLGENIS R API retrieves its data from the server you retrieved it from, so if you want to retrieve data from a different server, simply source the molgenis.R from that server.

But if you want to combine data from multiple server, you can specify a different REST API URL to use by setting `molgenis.api.url` in the `molgenis.env` environment. For example:

```R
local({
  molgenis.api.url <- "https://molgenis56.target.rug.nl:443/api/v1/"
}, env = molgenis.env)
```
or

```R
local({
  molgenis.api.url <- "http://localhost:8080/api/v1/"
}, env = molgenis.env)
```

### Other methods
You can manage your data using the other `molgenis.*` methods. See [the technical reference documentation](https://github.com/molgenis/molgenis/wiki/R-project-API-v1).

## Python
TODO: Merge to molgenis master, show how to install.

### Example: Plotting Allele-Specific Expression data from MOLGENIS in Python
As an example, let's create a plot for publicly available ASE data available on https://molgenis56.target.rug.nl/. For a description of the data, take a look at [http://molgenis.org/ase](http://molgenis.org/ase).

We'll be creating a scatter plot so if you haven't already, install matplotlib from the commandline:

```python
pip install matplotlib
```

Start an interactive python shell and create a molgenis connection:

```python
import molgenis
c = molgenis.Connection("https://molgenis56.target.rug.nl/api/")
```
This imports the molgenis package and instantiates a new Connection and points it at the molgenis56 server. If you take a look at the connection by typing

```python
dir(c)
```
you should see the methods you can call:

```python
['__doc__', '__init__', '__module__', '_getTokenHeader', 'add', 'addAll', 'delete', 'get', 'getAttributeMetaData', 'getEntityMetaData', 'login', 'logout', 'url']     
```

Let's load some data from the server using `c.get`:

```python
c.get("ASE")
```
This retrieves the top 1000 rows from the ASE entity.

```python
[{u'Alternative_allele': u'A', u'P_Value': 2.06504739339637e-17, u'Genes': {u'href': u'/api/v1/ASE/rs9901673/Genes'}, u'Fraction_alternative_allele': 0.479, u'Pos': 7484101, u'Reference_allele': u'C', u'Chr': u'17', u'href': u'/api/v1/ASE/rs9901673', u'Samples': u'145', u'Likelihood_ratio_test_D': 72.0813644150712, u'SNP_ID': u'rs9901673'}, {u'Alternative_allele': u'T', u'P_Value': 8.78109735398113e-18, u'Genes': {u'href': u'/api/v1/ASE/rs2597775/Genes'}, u'Fraction_alternative_allele': 0.479, u'Pos': 17503382, u'Reference_allele': u'C', u'Chr': u'4', u'href': u'/api/v1/ASE/rs2597775', u'Samples': u'359', u'Likelihood_ratio_test_D': 73.769089117417, u'SNP_ID': u'rs2597775'}, {u'Alternative_allele': u'C', u'P_Value': 1.4917458949834e-18, u'Genes': {u'href': u'/api/v1/ASE/rs3216/Genes'}, u'Fraction_alternative_allele': 0.479, u'Pos': 214421, u'Reference_allele': u'G', u'Chr': u'11', u'href': u'/api/v1/ASE/rs3216', u'Samples': u'301', u'Likelihood_ratio_test_D': 77.2691957930797, u'SNP_ID': u'rs3216'}, [...],{u'Alternative_allele': u'T', u'P_Value': 0.000132500824069775, u'Genes': {u'href': u'/api/v1/ASE/rs1056019/Genes'}, u'Fraction_alternative_allele': 0.482, u'Pos': 41337435, u'Reference_allele': u'C', u'Chr': u'12', u'href': u'/api/v1/ASE/rs1056019', u'Samples': u'47', u'Likelihood_ratio_test_D': 14.605874945467, u'SNP_ID': u'rs1056019'}
```
Let's retrieve a specific SNP from the ASE entity:

```python
print c.get("ASE", q=[{"field":"SNP_ID", "operator":"EQUALS", "value":"rs12460890"}])
```
```python
[{u'Alternative_allele': u'T', u'P_Value': 7.1708540619282e-14, u'Genes': {u'href': u'/api/v1/ASE/rs12460890/Genes'}, u'Fraction_alternative_allele': 0.527, u'Pos': 829568, u'Reference_allele': u'C', u'Chr': u'19', u'href': u'/api/v1/ASE/rs12460890', u'Samples': u'21', u'Likelihood_ratio_test_D': 56.0207947348388, u'SNP_ID': u'rs12460890'}]
```
This SNP has a mild but significant allele-specific expression, based on expression counts in 21 samples.

Let's retrieve the samples for this SNP:

```python
samples = c.get("SampleAse", q=[{"field":"SNP_ID", "operator":"EQUALS", "value":"rs12460890"}])
print samples
```

```python
[{u'Ref_Counts': u'130', u'href': u'/api/v1/SampleAse/1418785', u'SampleIds': {u'href': u'/api/v1/SampleAse/1418785/SampleIds'}, u'Position': 829568, u'SNP_ID': {u'href': u'/api/v1/SampleAse/1418785/SNP_ID'}, u'Alt_Counts': u'121', u'ID': u'1418785', u'Chromosome': u'19'}, {u'Ref_Counts': u'4142', u'href': u'/api/v1/SampleAse/1418786', u'SampleIds': {u'href': u'/api/v1/SampleAse/1418786/SampleIds'}, u'Position': 829568, u'SNP_ID': {u'href': u'/api/v1/SampleAse/1418786/SNP_ID'}, u'Alt_Counts': u'4791', u'ID': u'1418786', u'Chromosome': u'19'}, {u'Ref_Counts': u'19', u'href': u'/api/v1/SampleAse/1418787', u'SampleIds': {u'href': u'/api/v1/SampleAse/1418787/SampleIds'}, u'Position': 829568, u'SNP_ID': {u'href': u'/api/v1/SampleAse/1418787/SNP_ID'}, u'Alt_Counts': u'28', u'ID': u'1418787', u'Chromosome': u'19'}, {u'Ref_Counts': u'19', u'href': u'/api/v1/SampleAse/1418788', u'SampleIds': {u'href': u'/api/v1/SampleAse/1418788/SampleIds'}, u'Position': 829568, u'SNP_ID': {u'href': u'/api/v1/SampleAse/1418788/SNP_ID'}, u'Alt_Counts': u'23', u'ID': u'1418788', u'Chromosome': u'19'}, {u'Ref_Counts': u'32', u'href': u'/api/v1/SampleAse/1418789', u'SampleIds': {u'href': u'/api/v1/SampleAse/1418789/SampleIds'}, u'Position': 829568, u'SNP_ID': {u'href': u'/api/v1/SampleAse/1418789/SNP_ID'}, u'Alt_Counts': u'11', u'ID': u'1418789', u'Chromosome': u'19'}, {u'Ref_Counts': u'639', u'href': u'/api/v1/SampleAse/1418790', u'SampleIds': {u'href': u'/api/v1/SampleAse/1418790/SampleIds'}, u'Position': 829568, u'SNP_ID': {u'href': u'/api/v1/SampleAse/1418790/SNP_ID'}, u'Alt_Counts': u'572', u'ID': u'1418790', u'Chromosome': u'19'}, {u'Ref_Counts': u'202', u'href': u'/api/v1/SampleAse/1418791', u'SampleIds': {u'href': u'/api/v1/SampleAse/1418791/SampleIds'}, u'Position': 829568, u'SNP_ID': {u'href': u'/api/v1/SampleAse/1418791/SNP_ID'}, u'Alt_Counts': u'309', u'ID': u'1418791', u'Chromosome': u'19'}, {u'Ref_Counts': u'423', u'href': u'/api/v1/SampleAse/1418792', u'SampleIds': {u'href': u'/api/v1/SampleAse/1418792/SampleIds'}, u'Position': 829568, u'SNP_ID': {u'href': u'/api/v1/SampleAse/1418792/SNP_ID'}, u'Alt_Counts': u'401', u'ID': u'1418792', u'Chromosome': u'19'}, {u'Ref_Counts': u'271', u'href': u'/api/v1/SampleAse/1418793', u'SampleIds': {u'href': u'/api/v1/SampleAse/1418793/SampleIds'}, u'Position': 829568, u'SNP_ID': {u'href': u'/api/v1/SampleAse/1418793/SNP_ID'}, u'Alt_Counts': u'234', u'ID': u'1418793', u'Chromosome': u'19'}, {u'Ref_Counts': u'806', u'href': u'/api/v1/SampleAse/1418794', u'SampleIds': {u'href': u'/api/v1/SampleAse/1418794/SampleIds'}, u'Position': 829568, u'SNP_ID': {u'href': u'/api/v1/SampleAse/1418794/SNP_ID'}, u'Alt_Counts': u'1081', u'ID': u'1418794', u'Chromosome': u'19'}, {u'Ref_Counts': u'213', u'href': u'/api/v1/SampleAse/1418795', u'SampleIds': {u'href': u'/api/v1/SampleAse/1418795/SampleIds'}, u'Position': 829568, u'SNP_ID': {u'href': u'/api/v1/SampleAse/1418795/SNP_ID'}, u'Alt_Counts': u'201', u'ID': u'1418795', u'Chromosome': u'19'}, {u'Ref_Counts': u'74', u'href': u'/api/v1/SampleAse/1418796', u'SampleIds': {u'href': u'/api/v1/SampleAse/1418796/SampleIds'}, u'Position': 829568, u'SNP_ID': {u'href': u'/api/v1/SampleAse/1418796/SNP_ID'}, u'Alt_Counts': u'96', u'ID': u'1418796', u'Chromosome': u'19'}, {u'Ref_Counts': u'730', u'href': u'/api/v1/SampleAse/1418797', u'SampleIds': {u'href': u'/api/v1/SampleAse/1418797/SampleIds'}, u'Position': 829568, u'SNP_ID': {u'href': u'/api/v1/SampleAse/1418797/SNP_ID'}, u'Alt_Counts': u'655', u'ID': u'1418797', u'Chromosome': u'19'}, {u'Ref_Counts': u'584', u'href': u'/api/v1/SampleAse/1418798', u'SampleIds': {u'href': u'/api/v1/SampleAse/1418798/SampleIds'}, u'Position': 829568, u'SNP_ID': {u'href': u'/api/v1/SampleAse/1418798/SNP_ID'}, u'Alt_Counts': u'699', u'ID': u'1418798', u'Chromosome': u'19'}, {u'Ref_Counts': u'331', u'href': u'/api/v1/SampleAse/1418799', u'SampleIds': {u'href': u'/api/v1/SampleAse/1418799/SampleIds'}, u'Position': 829568, u'SNP_ID': {u'href': u'/api/v1/SampleAse/1418799/SNP_ID'}, u'Alt_Counts': u'391', u'ID': u'1418799', u'Chromosome': u'19'}, {u'Ref_Counts': u'13', u'href': u'/api/v1/SampleAse/1418800', u'SampleIds': {u'href': u'/api/v1/SampleAse/1418800/SampleIds'}, u'Position': 829568, u'SNP_ID': {u'href': u'/api/v1/SampleAse/1418800/SNP_ID'}, u'Alt_Counts': u'14', u'ID': u'1418800', u'Chromosome': u'19'}, {u'Ref_Counts': u'70', u'href': u'/api/v1/SampleAse/1418801', u'SampleIds': {u'href': u'/api/v1/SampleAse/1418801/SampleIds'}, u'Position': 829568, u'SNP_ID': {u'href': u'/api/v1/SampleAse/1418801/SNP_ID'}, u'Alt_Counts': u'101', u'ID': u'1418801', u'Chromosome': u'19'}, {u'Ref_Counts': u'47', u'href': u'/api/v1/SampleAse/1418802', u'SampleIds': {u'href': u'/api/v1/SampleAse/1418802/SampleIds'}, u'Position': 829568, u'SNP_ID': {u'href': u'/api/v1/SampleAse/1418802/SNP_ID'}, u'Alt_Counts': u'35', u'ID': u'1418802', u'Chromosome': u'19'}, {u'Ref_Counts': u'19', u'href': u'/api/v1/SampleAse/1418803', u'SampleIds': {u'href': u'/api/v1/SampleAse/1418803/SampleIds'}, u'Position': 829568, u'SNP_ID': {u'href': u'/api/v1/SampleAse/1418803/SNP_ID'}, u'Alt_Counts': u'28', u'ID': u'1418803', u'Chromosome': u'19'}, {u'Ref_Counts': u'44', u'href': u'/api/v1/SampleAse/1418804', u'SampleIds': {u'href': u'/api/v1/SampleAse/1418804/SampleIds'}, u'Position': 829568, u'SNP_ID': {u'href': u'/api/v1/SampleAse/1418804/SNP_ID'}, u'Alt_Counts': u'47', u'ID': u'1418804', u'Chromosome': u'19'}, {u'Ref_Counts': u'60', u'href': u'/api/v1/SampleAse/1418805', u'SampleIds': {u'href': u'/api/v1/SampleAse/1418805/SampleIds'}, u'Position': 829568, u'SNP_ID': {u'href': u'/api/v1/SampleAse/1418805/SNP_ID'}, u'Alt_Counts': u'55', u'ID': u'1418805', u'Chromosome': u'19'}]
```

There they are.

Let's format the expression counts

```python
for sample in samples:
	print "{Ref_Counts:5}\t{Alt_Counts:5}".format(**sample)
```

```python
130  	121
4142 	4791
19   	28
19   	23
32   	11
639  	572
202  	309
423  	401
271  	234
806  	1081
213  	201
74   	96
730  	655
584  	699
331  	391
13   	14
70   	101
47   	35
19   	28
44   	47
60   	55
```

Let's plot the expression counts in these samples in a scatter plot.

```python
import matplotlib.pyplot as plt
plt.scatter([sample["Ref_Counts"] for sample in samples], [sample["Alt_Counts"] for sample in samples])
plt.xlim([0, 5000])
plt.ylim([0, 5000])
plt.xlabel("Reference Allele")
plt.ylabel("Alternative Allele")
plt.title("Allele-Specific Expression for rs12460890")
```


And add a line for the non-specific expression.

```python
plt.plot([0, 5000], [0, 5000])
plt.show()
```
![image](images/rs12460890-py.png)

## Java
The `molgenis-data-rest-client` module contains an `org.molgenis.data.rest.clientMolgenisClient` class that we use to access the REST API for our own integration tests. It's built using the Spring `RestTemplate`.

The method calls are synchronous. Not all REST API methods are supported yet, it's not well documented and it has no users that we know of other than ourselves. But if you're interested in accessing a remote MOLGENIS instance from Java, you'll want to take a look there for inspiration.

## JavaScript

Internally, MOLGENIS uses `molgenis.RestClient` and `molgenis.RestClientV2` defined in `molgenis.js`. Their methods are implemented using `jquery.ajax`. This interface is not yet supported externally, but if you're planning to access MOLGENIS from JavaScript, you'll want to take a look at that for inspiration.

## HTTP

This is the raw HTTP REST API that the other APIs are built upon.
There are two versions, v1 and v2. v2 should be seen as an extension of v1.

The main differences between v2 and v1 are that v2 retrieves more entity metadata and that it allows you to query using RSQL.
But beware: the v1 csv endpoint already queries using RSQL.

# Scripts

Scripts are defined in the Scripts plugin.
By default you can find it in the menu under Plugins, Scripts.
The Scripts plugin allows you to create, edit and delete the available scripts.

## Creating a script

You open the script editor by clicking the add button ![image](images/new.png).

### Name
The name uniquely identifies the script

### Type
The script type determines how the script is executed.

* R scripts are executed in R
* Python scripts are executed in Python
* Magma JavaScript and JavaScript scripts are executed in a JavaScript context

### Content
The actual script content goes here.

### Result file extension
Allows you to render output in your script. For instance, if you set this to `png`, the script will be provided with a parameter `outputFile`.
You can write your png image to this file. 
When the script is done, the contents of the file will be served as a result of the script invocation request.

###Parameters
Your script may define parameters that it needs in order to run.
**All parameters are of type String**.
Multiple scripts can share the same parameter, here you just specify the parameter names.

#### Passing parameters to R, JavaScript and Python
If your Type is R, JavaScript or Python, the Script Content is interpreted as a [Freemarker](http://freemarker.apache.org) template.
The parameters are provided as Freemarker variables.
You can render them in the script by typing `${parameter}`, but the full Freemarker syntax is at your disposal, so for instance `<#if><#else><#/if>` constructs are also available.

This means that if you want to initialize an R or Python string variable with the value of a parameter string, you'll need to explicitly encapsulate it with quotes.

So in R:

```FreeMarker
# Assign string parameter
name <- "${name}"
# Assign numeric parameter
amount <- ${amount}
```

And in Python:

```FreeMarker
# Assign string parameter
name = "${name}"
# Assign numeric parameter
amount = ${amount}
```

And in JavaScript:

```FreeMarker
// Assign string parameter
var name = "${name}"
// Assign numeric parameter
var amount = ${amount}
```

The script template will be rendered to a file in the File Store and this file will be interpreted using the R or Python interpreter.

#### Passing parameters to Magma JavaScript
If your Type is [Magma JavaScript](http://wiki.obiba.org/display/OPALDOC/Magma+Javascript+API), the parameters are available through Magma's [$](http://wiki.obiba.org/display/OPALDOC/value+selector+method) selector method.
JavaScript will have to do the type conversions for you. Or if you want to be sure, you should cast the values explicitly

```javascript
// Assign string parameter
var name = $("name")
// Assign numeric parameter
var amount = int($("amount"))
```

### Generate molgenis security token
Your script can access data inside a MOLGENIS repository through the REST API.
If the data is private data, you can set Generate Security Token to Yes.
Then a molgenis security token will be generated for the user running the script and added as a parameter named `molgenisToken`.
You can pass the token on to the REST API.

#### Use security token in R script
Pass the token as a parameter when you request the molgenis.R api script:

```Freemarker
library('RCurl')
eval(expr = parse(text = getURL("https://molgenis01.target.rug.nl/molgenis.R?molgenis-token=${molgenisToken}")))
molgenis.get("MolgenisUser")
```

## Running a script
You can run your script by pressing the ![image](images/execute.png) execute button.
If the script has parameters, you'll be presented with a popup form to specify them.

You can also run the script through an HTTP request.
For instance, to run the `bmi` script with parameters `weight=50` and `height=1.60` on server `http://molgenis09.target.rug.nl` you can surf to [https://molgenis09.target.rug.nl/scripts/bmi/run?weight=50&height=1.60](https://molgenis09.target.rug.nl/scripts/bmi/run?weight=50&height=1.60)

>Beware that you need to URL-encode parameter values if they contain special characters

### Permissions
Note that in order to execute scripts, users need

* to be authenticated (i.e. anonymous users cannot execute your scripts)
* View permissions on the Scripts entity
* View permissions on the ScriptParameter entity
* View permissions on the ScriptType entity

## Putting it all together
Let's take the R script from [the previous example](#example-rest-R) and add it to the Script plugin. The script will fetch the public ASE data from [https://molgenis56.target.rug.nl/](https://molgenis56.target.rug.nl/)

You'll need a running instance of MOLGENIS.
So either run this example locally on your own MOLGENIS instance or Sign Up on our demo server [https://molgenis09.target.rug.nl/](https://molgenis09.target.rug.nl/)

### Create the script

Go to the Script plugin and create a new script.

#### Name
Any name will do, as long as it's unique.
Since the result will be a plot of Allele-Specific Expression for a SNP, we suggest the name `plot-ase`.

#### Type
We're creating an R script, so pick `R`.

#### Generate security token
No, since the data is publicly available and lives on a different server anyways.

#### Result file extension
R can plot postscript, pdf, png and jpeg. 
Let's pick `png`.

#### Parameters
In the example we plotted one specific ASE, let's make the snp ID a parameter.
Select `snp_id`.
If it's not yet available you can add it by pushing the **+** button to the right of the select box.

#### Content

```
library('RCurl')
eval(expr = parse(text = getURL("https://molgenis56.target.rug.nl/molgenis.R")))

samples <- molgenis.get("SampleAse", q="SNP_ID==${snp_id}")
jpeg('${outputFile}')

max <- max(samples$Ref_Counts, samples$Alt_Counts)

plot(samples$Ref_Counts, samples$Alt_Counts, xlim = c(0, max), ylim = c(0, max), xlab='Reference Allele', ylab='Alternative Allele', main = 'Allele-Specific Expression for ${snp_id}')
lines(c(0,max), c(0, max))
```

Let's take a closer look at what happens here.

First we fetch and source the MOLGENIS R API from molgenis56.
This means it'll be set up to retrieve its data from molgenis56.

Then we retrieve the samples using `molgenis.get`.

The `snp_id` parameter gets filled in into the rsql query q.
So if for instance `snp_id` equals `rs2287988`, Freemarker will fill the parameter in where it says `${snp_id}` so the query becomes `q="SNP_ID==rs2287988"`.

The outputFile parameter gets filled in into `jpeg('${outputFile}')` so that R plots to the correct output file.

In [the previous example](#example-rest-R) we manually set the axes to `c(0, 5000)`, but here the amount of reads depends on the chosen SNP. So we compute the maximum amount of reads in both the `Ref_Counts` and `Alt_Counts` attributes of the `samples` dataframe.

Next we create the scatter plot and the reference line, like we did before.
Note how we use the `snp_id` parameter a second time, to render the plot title.

Save the script.

#### Call the script
Push the ![image](images/execute.png) execute button.

In the popup, specify the value for `snp_id`, for example `rs2287988`.

Push Run.

![image](images/rs2287988.png)

#### Entity Report
If you are plotting your own data, a nice trick is to define a FreemarkerTemplate with an Entity Report for the entity you're plotting.

For instance, if you have an entity with a SNP_ID attribute, it's as easy as adding `<img src="https://molgenis09.target.rug.nl/scripts/plot-ase/run?snp_id=${entity.getString("SNP_ID")}"/>` into the FreemarkerTemplate for that entity.

This will allow you to generate one or more plots for entities you select in the Data Explorer. See the documentation for Entity Report.