---
layout: post

title: "Building the Elastic Search Demo, Part 2"

tags: [Index Extensions, Query Extensions, JS UI custom components, Push API]

author:
  name: Wim Nijmeijer
  bio: Technical Evangelist
  image: 20180501/wim.jpg
---

_This is the second blog post of a new series entitled “Build it with Coveo”. The series will present innovative use cases for the Coveo Platform, always including full code samples._

## Use case: 
* Build a demo on top of [Coveo for Elasticsearch](https://elastic.coveodemo.com/demo) to show what you can do with the platform. 
* Use public content that everybody is familiar with. 
* Build it in 2-4 weeks time.
This post is divided into two parts [Part 1]((/2018/05/01/building-the-elastic-demo), [Part 2]((/2018/05/01/building-the-elastic-demo-part2).
![RL1]({{ site.baseurl }}/images/20180501/intro.png)

<!-- more -->

## Building the UI
Coveo offers out of the box a [Javascript Framework](https://docs.coveo.com/en/375) for building the UI. It offers a ton of components which you can simply drag and drop using an [Interface Editor](http://www.coveo.com/go?dest=cloudhelp&lcid=9&context=230). Using the editor you can quickly design the basic layout of your UI, create search interfaces, add facets and you are ready to go for a basic search experience. Since we wanted to have very specific result templates, completely tailored to the search audience, we needed some additional configuration directly into the HTML/JS files.

### Result templates
For almost all the content types we have indexed we wanted to have some more dedicated [result templates](https://docs.coveo.com/en/413), than offered out of the box. The Movie result template is probably the most complicated one. It uses a couple of custom controls (later on more on them), and some custom formatting.
Coveo offers out of the box different [result layouts](https://docs.coveo.com/en/360), list, card and table. For each a custom result template can be created.
What the Javascript framework will do is, pick the first template where the condition is met. If none of the conditions is met and there is NO default template, then nothing is shown.

The Movie database result template looks like:
``` html
<script id="Movie" class="result-template" type="text/html" data-layout="list" data-field-mytype="Movie">
	<div class="coveo-result-frame movie">
	<span class="CoveoMyBackground" data-show-poster="true"></span>
	<div class="result-float-right role-based-text-color">
		<span class="CoveoFieldValue" data-field="@date" data-helper="date"></span>
		<span class="CoveoMySentiment"></span>
		<div class="CoveoQuickview"></div>
		<div class="CoveoFieldValue" data-field="@myvoteaverage" data-text-caption="★" style="display:block"></div>
		<div class="CoveoFieldValue" data-field="@mysentimentvalue"></div>
		<div class="CoveoFieldValue" data-field="@myprofitvalue" data-text-caption="Profit (x100M):" style="display:block"></div>
		<div class="CoveoFieldValue" data-field="@mypopularity" data-text-caption="Popularity:" data-helper="currency" data-helper-options-decimals="0" data-helper-options-symbol=" " style="display:block"></div>
		<div class="CoveoMyFeatured"></div>
		<div class="CoveoMyART"></div>
	</div>
	<a class="CoveoResultLink"></a>

	<div class="CoveoExcerpt"></div>

	<span class="CoveoResultFolding"
		data-result-template-id="Review"
		data-normal-caption="Reviews"
		data-more-caption="Show more reviews"
		data-expanded-caption="Reviews"
		data-less-caption="Show less reviews"
		data-one-result-caption="Only one review"></span>

	<div class="CoveoDetailsAndRelated">
		<table class="CoveoFieldTable coveo-details related-content Details query-done" data-allow-minimization="false">
			<tr data-field="@source" data-caption="Source"></tr>
			<tr data-field="@mystatus" data-caption="Status"></tr>
		<tr data-field="@mycountries" data--caption="Production Countries"></tr>
		<tr data-field="@mygenre" data-caption="Genres"></tr>
		<tr data-field="@myrelatedartist" data-caption="Soundtrack (artists)" data-split-values="true"></tr>
		<tr data-field="@myrelatedsongs" data-caption="Soundtrack (songs)"></tr>
		</table>
	</div>

	<div class="CoveoResultsRelated youtube"
	data-result-template-id="YouTubeVideoList"
	data-name="Videos"
	data-normal-caption="Videos"
	data-title-caption="Related videos (based upon title)"
	data-expanded-caption="Hide Related Youtube Videos"
	data-no-results-caption="No related videos found"
	data-query='@title="[FIELD1]" @filetype=YoutubeVideo'
	data-key-check='@filetype;YoutubeVideo'
	data-expanded-comment="Show YouTube videos which title contains '[FIELD1]'"
	data-extra-boost=false
	data-fields="title"
	data-partial-match=false
	data-number-Of-Results=5
	data-help='#ResultHelpYoutube'>
	</div>

	<div class="CoveoResultsRelated music"
	data-result-template-id="Music"
	data-name="Music"
	data-normal-caption="Music"
	data-title-caption="Related music (based upon related songs and related artists)"
	data-expanded-caption="Hide Related Music"
	data-no-results-caption="No related music found"
	data-query='@myrelatedsongs=("[FIELD1]") @myrelatedartist=("[FIELD2]") @source=Music $qre(expression: &#39;soundtrack&#39;,modifier:&#39;300&#39;, isConstant:&#39;true&#39;)'
	data-key-check='@source;Music'
	data-expanded-comment="Show Music with Related Songs/Artists of: [FIELD1] and [FIELD2]"
	data-extra-boost=false
	data-fields="myrelatedsongs;myrelatedartist"
	data-filter-field="@myalbum"
	data-partial-match=false
	data-number-Of-Results=5
	data-help='#ResultHelpMusic'>
	</div>

	<div class="CoveoResultsRelated book"
	data-result-template-id="DefaultBook"
	data-name="Scripts"
	data-normal-caption="Scripts"
	data-title-caption="Related scripts (based upon title)"
	data-expanded-caption="Hide Related Scripts"
	data-no-results-caption="No related Scripts found"
	data-query='@title="[FIELD1]" @source=Books'
	data-key-check='@source;Books'
	data-expanded-comment="Show Scripts with matching title: [FIELD1]"
	data-extra-boost=false
	data-fields="title"
	data-partial-match=false
	data-number-Of-Results=5
	data-help='#ResultHelpScript'>
	</div>

	<div class="CoveoResultsRelated Wiki"
	data-result-template-id="DefaultWiki"
	data-name="Wikipedia"
	data-normal-caption="Wikipedia"
	data-title-caption="Related wikipedia (based upon movie id)"
	data-expanded-caption="Hide Related Wikipedia"
	data-no-results-caption="No related Wikipedia found"
	data-query="@myimdb=[FIELD1] @mytype=Wikipedia"
	data-key-check='@mytype;Wikipedia'
	data-expanded-comment="Show Wikipedia with movie id: [FIELD1]"
	data-extra-boost=false
	data-fields="myimdb"
	data-partial-match=false
	data-number-Of-Results=5
	data-help='#ResultHelpWiki'>
	</div>
	<div class="CoveoResultsRelated Play"
	data-result-template-id="DefaultWiki"
	data-name="Playing"
	data-normal-caption="Playing"
	data-title-caption="Today playing:"
	data-expanded-caption="Hide Playing nearby"
	data-no-results-caption="Nothing found playing today nearby {LOC} (retrieved from Google)"
	data-query='https://www.google.com/search?q=[FIELD1]&amp;oq=[FIELD1]&amp;near={LOC}'
	data-key-check=''
	data-div-to-get='div.tb_c;.lr_c_fcb;a.vk_bk.lr-s-din;div.lr_c_fce'
	data-expanded-comment=""
	data-extra-boost=false
	data-fields="title"
	data-partial-match=false
	data-number-Of-Results=5
	data-help='#ResultHelpWiki'>
	</div>
</div>

</script>
```

And will show in the interface like:
![RL1]({{ site.baseurl }}/images/20180501/RL1.png)

Lots of the components inside the resulttemplate are out of the box (CoveoFieldValue, CoveoExcerpt, CoveoResultLink), others (CoveoMyBackground, CoveoResultsRelated) are custom made (see next section).

## Custom controls
You can use [Typescript](https://docs.coveo.com/en/361) to create custom controls, or you can simply embed them within your HTML/[Javascript](https://docs.coveo.com/en/305) files, and that is what we have used for the demo. A few of the components will be discussed in detail. You can always view them [here](https://elastic.coveodemo.com/demo/js/pages.js).

### CoveoMyBackground
The first custom control is CoveoMyBackground. It offers a custom background based upon a reference to the images we gathered during the indexing process. It also uses the color gradient (from the Indexing Pipeline Extension) to create a nice gradient color, based upon the image of the movie/album.
``` javascript
//***************************
//MyBackground
//  Adds a background image and sets the background gradient of the result
//***************************
let MyBackground = function (element, options, bindings, result) {
    __extends(MyBackground, Coveo.Component);
    Coveo.ComponentOptions.initComponentOptions(element, MyBackground, options);

    if (options.showPoster) {
        let url = result.raw.myimage;
        if (result.raw.mytype === 'Movie') {
            url = url ? '//image.tmdb.org/t/p/w185_and_h278_bestv2' + url : '//elastic.coveodemo.com/icons/noimage-nofill.png';
        }
        else if (result.raw.mytype === 'Music') {
            url = url ? '//lastfm-img2.akamaized.net/i/u/174s/' + url : '//elastic.coveodemo.com/icons/noimage-nofill.png';
        }
        else if (result.raw.mytype === 'Concert') {
            url = url ? `//images.sk-static.com/images/media/profile_images/artists/${url}/large_avatar` : '//elastic.coveodemo.com/icons/noimage-nofill.png';
        }

        $(element).closest('.coveo-result-frame').css('background-image', `url(${url})`).on('click', (e) => {
            $(e.target).find('.CoveoQuickview').coveo('open');
        });

    }

    // calculate gradient
    // the mycolors is added with an IPE during indexing
    let colors = result.raw.mycolors;
    if (colors) {
        colors = colors.split(' ');
        if (colors.length) {
            let i = 0;
            // make sure we have at least 5 colors, by adding previous colors when array is short.
            while (colors.length < 5) {
                colors.push(colors[i++]);
            }
            colors = colors.map((c, idx) => {
                let trans=0.05;
                if (userIdToAdd.sel=="Anonymous" && options.showPoster)
                { 
                    trans=0.95;
                }                
                let t = ((5 - idx) * trans).toFixed(2);
                return c.replace('(', 'rgba(').replace(')', `,${t})`); // add transparency
            });
            
            if (userIdToAdd.sel=="Anonymous" && options.showPoster)
            {
                let gradient = `linear-gradient(to right, ${colors[0]}, ${colors[1]}, ${colors[2]}, ${colors[3]}, ${colors[4]})`;
                $(element).closest('.CoveoResult').prepend("<div class='TopBorder'></div>").css('background', gradient);
            }
            else
            {
                let gradient = `linear-gradient(to bottom right, ${colors[0]}, ${colors[1]}), radial-gradient(circle, ${colors[2]}, ${colors[3]}, ${colors[4]})`;
                $(element).closest('.CoveoResult').prepend("<div class='TopBorder'></div>").css('background', gradient);
            }
        }
    }
    else {
        if (userIdToAdd.sel=="Anonymous" && options.showPoster)
        {
            $(element).closest('.CoveoResult').prepend("<div class='TopBorder'></div>").css('background', 'linear-gradient(to right, rgba(193,193,193,0.35) 0%,rgba(0,0,0,0) 100%)');
        }
        else
        {
            $(element).closest('.CoveoResult').prepend("<div class='TopBorder'></div>").css('background', 'linear-gradient(45deg, rgba(193,193,193,0.35) 0%,rgba(0,0,0,0) 100%)');
        }
    }
};
MyBackground.ID = 'MyBackground';
MyBackground.options = {
    showPoster: Coveo.ComponentOptions.buildBooleanOption({ defaultValue: false })
};
Coveo.CoveoJQuery.registerAutoCreateComponent(MyBackground);
```

### CoveoMyART
Our next custom control is CoveoMyArt, it shows a 'Featured' label on the result when the result was promoted by our Machine Learning [Art](http://www.coveo.com/go?dest=cloudhelp&lcid=9&context=183).
``` javascript
//***************************
//ART result
//  Is triggered when ART (Machine Learning) is kicking in
//***************************
let MyART = function (element, options, bindings, result) {
    __extends(MyART, Coveo.Component);
    this.element = element;
    this.options = Coveo.ComponentOptions.initComponentOptions(element, MyART, options);
    this.bindings = bindings;
    this.result = result;
    if (this.result.isRecommendation) {
        Coveo.$('<div style="position:relative"><div class="coveo-result-highlight coveo-custom-ART" ><div class="mytextART">ML Boost</div></div></div>').addClass('custom-coveo-badge').prependTo($(element).closest('.coveo-result-frame'));
    }
};
MyART.ID = 'MyART';
Coveo.CoveoJQuery.registerAutoCreateComponent(MyART);
```

### CoveoResultsRelated
One of my favorites: CoveoResultsRelated, it shows (based upon a user action) an additional result list. The current result is used as the context. For example we want to show a tab with related youtube video's on the current result (based upon the title of the movie). We can configure the Custom control by using the properties:
``` html
<div class="CoveoResultsRelated youtube"
            data-result-template-id="YouTubeVideoList"
            data-name="Videos"
            data-normal-caption="Videos"
            data-title-caption="Related videos (based upon title)"
            data-expanded-caption="Hide Related Youtube Videos"
            data-no-results-caption="No related videos found"
            data-query='@title="[FIELD1]" @filetype=YoutubeVideo'
            data-key-check='@filetype;YoutubeVideo'
            data-expanded-comment="Show YouTube videos which title contains '[FIELD1]'"
            data-extra-boost=false
            data-fields="title"
            data-partial-match=false
            data-number-Of-Results=5
            data-help='#ResultHelpYoutube'>
          </div>
```
In the above example we specify the data-query to execute as '@title="[FIELD1]" @filetype=YoutubeVideo'. The data-fields (in this case title) will be used to fill up the [FIELD1] in the query. So only when the end user clicks on a button, the query will be executed... at least that was the first behavior of the component. 

Because we changed the layout to use tabs on a result, it does not look very professional if people see 4 different tabs and when they click on them 'No results' are shown. Bad user-experience! 
So we added an additional check inside the CoveoDetailsAndRelated component, which is executing a query with the data-key-check field. If the key with the value is not present in the data, we can hide the tab. A much better user-experience. BUT BE AWARE, if 10 results are displayed this means 10 additional queries!!!
``` javascript
//***************************
//composeQuery
//  Creates a new query for the checkQuery function
//***************************
function composeQuery(result, dataset){
    let newquery = dataset.query;
    let a = dataset.fields.split(";");
    let i = 0;
    let allfieldsmissing = true;
    for (i = 0; i < a.length; i++) {
        let fieldcontent = result.raw[a[i]];
        if (fieldcontent) {
            allfieldsmissing = false;
            fieldcontent = '' + fieldcontent;//make sure it's a string
            //Seems that if you format the field in the UI, it gets the formatted value
            fieldcontent = fieldcontent.replace(/[;()]/g, ' ');
            //Comma's must be replaced by ","
            fieldcontent = fieldcontent.replace(/,/g, '","');
            //In Elastic the : means a field query
            newquery = newquery.replace("[FIELD" + (i + 1) + "]", fieldcontent.replace(':', ' '));
        }
        else
        {
            newquery = newquery.replace("[FIELD" + (i + 1) + "]", "");
        }
    }
    return newquery;
}

//***************************
//checkQuery
//  Checks the query for all the relatedTabs. Based upon the groupby fields the tab will be made visible or not
//***************************
function checkQuery(fields, query, partial, dataset){
    let queryBuilder = new Coveo.QueryBuilder();
    queryBuilder.locale = GetBindings().queryController.lastQueryBuilder.locale;
    queryBuilder.pipeline = 'default';
    queryBuilder.searchHub = 'CheckTabs';
    queryBuilder.enableDebug = false;
    queryBuilder.enableQuerySyntax = true;
    queryBuilder.enableDuplicateFiltering = false;
    queryBuilder.excerptLength = 0;

    queryBuilder.timezone = GetBindings().queryController.lastQueryBuilder.timezone;
    queryBuilder.numberOfResults = 0;
    queryBuilder.groupByRequests = fields;

    queryBuilder.expression.add(query);
    if (partial=="true") {
        queryBuilder.enablePartialMatch = true;
        queryBuilder.partialMatchKeywords = 4;
        queryBuilder.partialMatchThreshold = "50%";
    }
    GetBindings().queryController.getEndpoint().search(queryBuilder.build()).done(function (data) {
        let showit= data.results.length !== 0;
        dataset.forEach(f => {
            if (f.dataset.partialMatch==partial && f.dataset.keyCheck!="")
            {
                //Check if we have the key in the result, if so enable the tab else disable it
                let field=f.dataset.keyCheck.split(';')[0];
                let value=f.dataset.keyCheck.split(';')[1];
                let found=false;
                data.groupByResults.forEach(res => {
                    res.values.forEach(groupval => {
                        if (groupval.value.toLowerCase()==value.toLowerCase() )
                        {
                            //We have it
                            found=true;
                        }
                    });
                });
                let $newTab = $('.RelatedTab[data-name="'+f.tab+'"]', f.parent);
                if (found){
                    $newTab.show();
                }
                else
                {
                    $newTab.hide();
                }
            }
        });
    });
}

//***************************
//checkQueries
//  builds up the total query to execute for the relatedTabs
//***************************
function checkQueries(queries){
    //Partial match queries
    let fields=[];
    let query=[];
    queries.forEach(f => {
        if (f.dataset.partialMatch=="true" && f.dataset.keyCheck!="")
        {
            //Build a single query with an OR on all queries
            //The key will be used to check if we have results
            //Add keyfield to fields to retrieve
            fields.push({ field: f.dataset.keyCheck.split(';')[0], sortCriteria:'nosort'});
            query.push('('+composeQuery(f.result, f.dataset)+')');
        }
    });
    if (query.length!=0) {
        checkQuery( fields, query.join(' OR '), "true", queries );
    }
    //Non partial match queries
    query=[];
    fields=[];
    queries.forEach(f => {
        if (f.dataset.partialMatch=="false" && f.dataset.keyCheck!="")
        {
            //Build a single query with an OR on all queries
            //The key will be used to check if we have results
            //Add keyfield to fields to retrieve
            fields.push({ field: f.dataset.keyCheck.split(';')[0], sortCriteria:'nosort'});
            query.push('('+composeQuery(f.result, f.dataset)+')');
        }
    });
    if (query.length!=0) {
        checkQuery( fields, query.join(' OR '), "false", queries );
    }
}
```

For the full code of the component: [Here](https://elastic.coveodemo.com/demo/js/page.js).

## Styling the UI with custom CSS
By default the CSS of the Coveo interface looks nice, but in most cases you want to change them to your own branding. Everything can be [overruled](https://docs.coveo.com/en/423). Always create your own custom CSS files, and never overwrite the ones provided by Coveo out of the box.

Our resulttemplate is formatted like:
``` css
.coveo-results-column {
    .coveo-list-layout .coveo-result-frame {
        padding: 10px;
        background: #FFFFFF;
        border-radius: 2px;
        background-repeat: no-repeat;
        
        &.movie {
            padding-left: 200px;
            min-height: 278px;
            background-size: 185px auto !important;
        }
        &.people {
            padding-left: 200px;
            min-height: 278px;
            background-size: 138px auto  !important;
            background-position: 20px 20px !important;
        }
        &.default {
            padding-left: 1px;
            min-height: 278px;
        }
        &.music {
            padding-left: 200px;
            min-height: 190px;
            background-size: 174px auto  !important;
        }
        &.concert {
            padding-left: 200px;
            min-height: 160px;
            background-size: 160px auto  !important;
        }
        &.no-image {
            padding-left: 10px;
        }
    }
}
```

## Personalization
More and more we see requests coming in around personalization, one of the reasons why we added it to our demo. In our case we have two profiles: Movie Fan and a Movie Producer. Each has its own color schema and its own relevancy rules. Using the profiles we can quickly change the UI styling and also add specific relevancy rules into the mix. In a normal scenario this could come from all kinds of systems: for example department/country from Active Directory or information from a userprofile in Salesforce or Sharepoint.

With Coveo you can use the [QRE](http://www.coveo.com/go?dest=cloudhelp&lcid=9&context=357) syntax to start boosting results with your preferences to a higher position.
``` javascript

//Create a usermap with help and booster information.
const usersMap = {
    Producer: {
        sel: "Producer",
        activeTab: "Movies",
        help: "They like Documentaries, not Rock music. Movies are boosted with profit and popularity.<BR>Results are influenced with:<br><font style='font-size:small'><i>Using: $qre(expression: \'@mygenre=Documentaries @mytype=Movie\',modifier:\'400\', isConstant:\'true\')  $qre(expression: \'@mygenre=Rock @mytype=Music\',modifier:\'-420\', isConstant:\'true\') $qre(expression: \'@myprofitvalue>100\',modifier:\'200\', isConstant:\'true\') $qre(expression: \'@myprofitvalue>200\',modifier:\'200\', isConstant:\'true\') $qre(expression: \'@myprofitvalue>300\',modifier:\'200\', isConstant:\'true\') $qre(expression: \'@mypopularity<15\', modifier:\'-400\', isConstant:\'true\') $qre(expression: \'@mypopularity>15\', modifier:\'200\', isConstant:\'true\') $qre(expression: \'@mypopularity>250\', modifier:\'200\', isConstant:\'true\') $qre(expression: \'@mypopularity>500\', modifier:\'200\', isConstant:\'true\')</i></font><br>Only results for user producer@coveo.com and their memberships are shown.<br>They can NOT see reviews.",
        booster: "$qre(expression: \'@mytype=Movie\',modifier:\'1600\', isConstant:\'true\') $qre(expression: \'@mygenre=Documentaries @mytype=Movie\',modifier:\'400\', isConstant:\'true\')  $qre(expression: \'@mygenre=Rock @mytype=Music\',modifier:\'-420\', isConstant:\'true\') $qre(expression: \'@myprofitvalue<=5\',modifier:\'-800\', isConstant:\'true\') $qre(expression: \'@myprofitvalue>100\',modifier:\'200\', isConstant:\'true\') $qre(expression: \'@myprofitvalue>200\',modifier:\'200\', isConstant:\'true\') $qre(expression: \'@myprofitvalue>300\',modifier:\'200\', isConstant:\'true\') $qre(expression: \'@mypopularity<15\', modifier:\'-800\', isConstant:\'true\') $qre(expression: \'@mypopularity>15\', modifier:\'200\', isConstant:\'true\') $qre(expression: \'@mypopularity>250\', modifier:\'200\', isConstant:\'true\') $qre(expression: \'@mypopularity>500\', modifier:\'200\', isConstant:\'true\')",
        musicbooster: " $qre(expression: \'@mygenre=Rock @mytype=Music\',modifier:\'-500\', isConstant:\'true\')",
    },
    Anonymous: {
        sel: "Anonymous",
        activeTab: "Movies",
        help: "They like Movies. Movies are boosted with popularity.<BR>Results are influenced with:<br><font style='font-size:small'><i>Using: $qre(expression: \'@mytype=Movie\',modifier:\'1600\', isConstant:\'true\') $qre(expression: \'@mytype=Review\',modifier:\'-1200\', isConstant:\'true\') $qre(expression: \'@mypopularity<15\', modifier:\'-400\', isConstant:\'true\') $qre(expression: \'@mypopularity>15\', modifier:\'200\', isConstant:\'true\') $qre(expression: \'@mypopularity>250\', modifier:\'200\', isConstant:\'true\') $qre(expression: \'@mypopularity>500\', modifier:\'200\', isConstant:\'true\')</i></font><br>They CANNOT see reviews/customer support cases.<br>Only results for Anonymous and their memberships are shown. <br>",
        booster: "$qre(expression: \'@mytype=Movie\',modifier:\'1600\', isConstant:\'true\') $qre(expression: \'@mytype=Review\',modifier:\'-1200\', isConstant:\'true\') $qre(expression: \'@mypopularity<15\', modifier:\'-800\', isConstant:\'true\') $qre(expression: \'@mypopularity>15\', modifier:\'200\', isConstant:\'true\') $qre(expression: \'@mypopularity>250\', modifier:\'200\', isConstant:\'true\') $qre(expression: \'@mypopularity>500\', modifier:\'200\', isConstant:\'true\')",
        musicbooster: "",
    }
};
```

The above will change in the future when we will support rankingfunctions, then the whole set of @mypopularity will be replaced by a single decay function.

## UI, tips and tricks
Some special tips and tricks.

### Adding context
In order to use context like a userRole we added [context](https://docs.coveo.com/en/399) to our Queries using the following code:
``` javascript
$('#search').on("buildingQuery", function (e, args) {
	//Add the context
	//A custom context is added so we can react on it in Query Pipelines and Analytics
	args.queryBuilder.addContext({
			"userRole": userIdToAdd.sel
	});
});
```
All [events](https://docs.coveo.com/en/417) in Coveo can be adjusted to fit to your needs.

### Adding a Relevancy boost when a user profile is selected
When a user profile is selected we want to set the [hiddenQuery](https://docs.coveo.com/en/3) so it will change the relevancy based upon what was set for the current profile. In this case: hd means the description provided to the user (in the breadcrumb section), hq is the actual hidden query to execute.
``` javascript
$('#search').on("buildingQuery", function (e, args) {
	//Set the proper pipeline
	args.queryBuilder.pipeline = (MLactive ? 'ML' : 'default');

	if (userBoost) {
		Coveo.$('.CoveoSearchInterface').coveo('state', 'hd', userHelp.split('<BR>')[0]);
		Coveo.$('.CoveoSearchInterface').coveo('state', 'hq', userBoost);
		userBoost = "";
	}
});
```

### Getting a proper access token from a Lambda function
As mentioned before you should not use public API keys inside your code. So we created a Lambda function for the retrieval of our Access Token.
The Lambda function:
``` javascript
'use strict';
const https = require('https');

const TOKENS = {
  'platformdemoelastic3zlf3f2p': '-IMPERSONATE-TOKEN-'
};

const ROLES = {
  Director: 'director@coveo.com',
  Producer: 'producer@coveo.com',
  Rock: 'rock@coveo.com'
};


let getUsers = (role) => {
  let provider = 'Email Security Provider';
  let users = [{
    name: '*@*',
    provider,
    type: 'Group'
  }];

  let name = ROLES[role];
  if (name) {
    users.push({
      name,
      provider,
      type: 'User',
    });
  }
  return users;
};


// Return an object {token, userIds?} if input is valid, otherwise returns null.
let validateInput = (event) => {
  try {
    let body = JSON.parse(event.body),
      token = TOKENS[body.org],
      userIds = getUsers(body.role);

    if (token && userIds) {
      return { token, userIds };
    }
  }
  catch (e) {
    console.log('Invalid input: ', e);
  }
  return null;
};


let requestToken = (inputParams, callback) => {
  // the post options
  let options = {
    host: 'platform.cloud.coveo.com',
    path: '/rest/search/token',
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': 'Bearer ' + inputParams.token,
    }
  };

  let req = https.request(options, (res) => {
    if (res.statusCode !== 200) {
      callback(res);
      return;
    }
    let body = '';
    res.on('data', chunk => {body += chunk;});
    res.on('end', () => {
      let response = {
        statusCode: 200,
        headers: {
          'Access-Control-Allow-Headers': 'Content-Type,X-Amz-Date,Authorization,X-Api-Key,X-Amz-Security-Token',
          'Access-Control-Allow-Methods': 'POST,OPTIONS',
          'Access-Control-Allow-Origin': '*',
        },
        body,
        isBase64Encoded: false
      };

      // success
      callback(null, response);
    });
  });
  req.on('error', (e) => {
    console.error('HTTP error: ' + e.message);
    callback({
      statusCode: 500,
      headers: {},
      body: '{"message":"server error"}',
      isBase64Encoded: false
    });
  });

  let usersPayload = {
    userIds: inputParams.userIds
  };
  req.write(JSON.stringify(usersPayload));
  req.end();

};

exports.handler = (event, context, callback) => {
  let inputParams = validateInput(event);
  if (inputParams) {
    requestToken(inputParams, callback);
  }
  else {
    callback(null, {
      statusCode: 400,
      headers: {},
      body: '{"message": "Invalid request body"}',
      isBase64Encoded: false
    });
  }

};

```

And in our JS we fetch the token:
``` javascript
//On Load...
	//We set default values
    Coveo.Analytics.options.endpoint.defaultValue = 'https://usageanalytics.coveo.com';
    Coveo.Analytics.options.searchHub.defaultValue = 'Movie';
    Coveo.Analytics.options.organization.defaultValue = ORG_ID;
    Coveo.Analytics.options.pipeline = 'ML';

    //Retrieve the access token
    let xhr = new XMLHttpRequest();
    xhr.open('POST', 'https://5qgxnbkr4f.execute-api.us-east-1.amazonaws.com/prod');
    xhr.setRequestHeader('Content-Type', 'application/json');
    xhr.onload = ()=>{
        let token = JSON.parse(xhr.response).token;
        //Set it for both the interface as the analytics token
        Coveo.SearchEndpoint.endpoints['default'].options.accessToken = token;
        Coveo.Analytics.options.token.defaultValue = token;
		//Init our Components
        Coveo.init(document.querySelector('#search'));
		//And the recommendation component
        Coveo.initRecommendation(document.querySelector('#MyRecommendationsMusic'));
    };
    xhr.send(JSON.stringify({
        org: ORG_ID,
        role: userIdToAdd.sel
    }));
```

### Reacting on 'popular queries', manually execute a query
We added (for demo purposes) a couple of popular queries to the UI. This could execute a query directly, but when you execute a new query by code we also need to add the proper analytics events! We used the following code:
``` javascript
//Change the query based upon the popular ones. This will also add an analytics event
function changeQuery(query) {
    Coveo.$('.CoveoSearchInterface').coveo('state', 'q', query );
    //We are not executing analytics if we call executeQuery directly
    var customEventCause = {name: 'searchboxSubmit', type:'search box'};
    $('#search').coveo('logSearchEvent', customEventCause, {});
    
    $('#search').coveo('executeQuery');
}
```

### Add additional (missing or cleaning) metadata fields to our retrieved results
Sometimes you want to enrich the result set with your own result fields (on rendering time). In our case a default field called 'collection' was missing in our data. So we add it to our results. Coveo has an event for that:
``` javascript
$('#search').on("preprocessResults", (e, args) => {
	//We want to add an additional field (if it is missing) to the results
	((args && args.results && args.results.results) || []).forEach(r=>{
		r.raw.collection = (r.raw.collection || 'default');
	});
});
```

### Setting the culture
Providing a UI in several languages is very user friendly. We did it in our [Partner demo](https://elastic.coveodemo.com/demo-partners). First provide the translation of your custom strings:
``` javascript
//Setting the translation for the custom strings, which would not be in the out-of-the-box translation set
String.toLocaleString({
    "es-es": {
        "Movies, Scripts and Music": "Películas, guiones y música",       // Defines a new string.
        "All Content": "Todo el contenido",
        "Type": "Tipo",
        "Genre": "Género",
        "Artist (Soundtrack)": "Artista (Banda sonora)",
    },
    "nl": {
        "Movies, Scripts and Music": "Films, script en Muziek",       // Defines a new string.
        "All Content": "Alles",
        "Type": "Type",
        "Genre": "Genre",
        "Artist (Soundtrack)": "Artiest (soundtrack)",
    },
    "fr": {
        "Movies, Scripts and Music": "Films, Scripts et Musique",       // Defines a new string.
        "All Content": "Tout le contenu",
        "Type": "Type",
        "Genre": "Genre",
        "Artist (Soundtrack)": "Artist (soundtrack)",
    }
});
```
Then set the locale and culture to the selected one:
``` javascript
//Setting the locale based upon es-es, nl, fr, en
Globalize.cultureSelector = id;
String["locale"] = id;
//Reload the window
window.location.reload();
```

### Get query suggestions on our landing page
In our [Partner demo](https://elastic.coveodemo.com/demo-partners) we provide a landing page with popular queries. Based upon the userRole the [querySuggestions](https://developers.coveo.com/x/iQGwAQ) will be retrieved using our api, using the following code:
``` javascript

function getQuerySuggest(profile) {
    var token = Coveo.SearchEndpoint.endpoints['default'].options.accessToken;
    var url = 'https://platform.cloud.coveo.com/rest/search/v2/querySuggest?'
    + 'access_token='
    + token
    + '&language=en'
    +' &pipeline=MLIntranet'
    + '&context={"userrole": "'+profile+'"}'
    + '&searchHub=Intranet';

    var xhr = new XMLHttpRequest();
    xhr.addEventListener('load', function () {
    handleQuerySuggestResponse(JSON.parse(this.responseText));
    });
    xhr.open("GET", url);
    xhr.send();
}

function toTitleCase(str)
{
    return str.replace(/\w\S*/g, function(txt){return txt.charAt(0).toUpperCase() + txt.substr(1).toLowerCase();});
}

function handleQuerySuggestResponse(response) {
    let addLi = html => {
        var query = toTitleCase( html.expression );
        var a= `<a class="CoveoResultLink RecResult" href="" onClick="changeQuery('${query}');return false;">${query}</a>`;
        return `<div class="coveo-list-layout CoveoResult popquery-item" style="height:32px">${a}</div>`;
      };
    var suggestions = [];
    response.completions.forEach(function(query) {
      suggestions.push(query);
    });
    //Set it up in the UI
    let html = suggestions
    .map(addLi);
    $('#PopularQueries').empty().html(html);
}
```

### Setting customData on our Analytics Events
In our Search Analytics dashboard we want to report if a recommended result is being clicked. For that we needed to add some additional [metadata](https://support.coveo.com/s/article/ka132000000LMYcAAO/2096) on our Analytics events.
``` javascript
$('#search').on('changeAnalyticsCustomData', function (e, args) {
	//We want to inject custom Data into each Analytics call
	args.metaObject.user = userIdToAdd.name;
	args.metaObject.role = userIdToAdd.sel;
	if (args.resultData!=undefined)
	{
		//The below values are first added as Dimensions in the Administration Console
		args.metaObject.c_istop = args.resultData.isTopResult;
		args.metaObject.c_isrecommended =  args.resultData.isRecommendation;
	}
});
```

### Sending a custom Analytics event
Whe want to track if end-users are clicking on the 'Expand to' links in the ResultsRelated component. In order to do so, we need to send a custom event to our Analytics API.
``` javascript
ResultsRelated.prototype.expandQuery = function () {
	Coveo.$('.CoveoFacet').coveo('reset');
	Coveo.$('#search').coveo('state', 'hd', this.expandedComment);
	Coveo.$('#search').coveo('state', 'hq', this.expandquery);
	Coveo.$('#search').coveo('state', 'q', '');
	//Seems we are not executing analytics if we call executeQuery directly
	var customEventCause = {name: 'expand '+this.options.name, type:'search box'};
	Coveo.$('#search').coveo('logSearchEvent', customEventCause, {});
	Coveo.$('#search').coveo('executeQuery');
};

```

### Adding an advanced search
In our case we wanted to add an advanced search feature, so end-users could easily change the results to their preferences. First of all we need to add the settings we want to the HTML page:
``` HTML
      <div class="CoveoPreferencesPanel">
        <fieldset class="coveo-form-group">
          <span class="coveo-form-group-label">Personalize results</span>
          <div class="perso-menu-content">
            <div class="perso-menu-item Filter">
              <label>
                <input class="user-personalisation-option" id="b1" name="choice-personalize" type="radio" value="choice-personalize-b1" />
                 Filter Action movies
              </label>
            </div>
            <div class="perso-menu-item Boost">
              <label>
                <input class="user-personalisation-option" id="b2" name="choice-personalize" type="radio" value="choice-personalize-b2" />
                Boost Action and Willis movies
              </label>
            </div>
            <div class="perso-menu-item Boost">
              <label>
                <input class="user-personalisation-option" id="b3" name="choice-personalize" type="radio" value="choice-personalize-b3" />
                Boost Comedy lower
              </label>
            </div>
            <div class="perso-menu-item Boost">
              <label>
                <input class="user-personalisation-option" id="b4" name="choice-personalize" type="radio" value="choice-personalize-b4" />
                Ignore Title for ranking
              </label>
            </div>
            <div class="perso-menu-item Standard">
              <label>
                <input class="user-personalisation-option" id="standard" name="choice-personalize" type="radio" value="choice-personalize-nothing"/>
                Standard search
              </label>
            </div>
          </div>
        </fieldset>

        <div class="CoveoResultsPreferences"></div>
        <div class="CoveoResultsFiltersPreferences"></div>
      </div>
```
Support that with additional code in our JS:
``` javascript
//***************************
//setSearchParams
//  Executed when the 'advancedsearch' is activated
//***************************
let persoFilter = "standard";
function setSearchParams() {
    let detail = '';

    let queries = {
        b1: `@mygenre=Action`,
        b2: `($qre(expression: '@mygenre=Action',modifier:'220')) ($qre(expression: '@mypeople=Willis',modifier:'220', isConstant:'true'))`,
        b3: `($qre(expression: '@mygenre=Comedy',modifier:'-200', isConstant:'true'))`,
        b4: `$weight(name:'Title', value:'0')`,
    };

    let query = queries[persoFilter] || '';
    if (query) {
        let $elem = $('#' + persoFilter).prop('checked', true);
        detail = $elem.closest('label').text();
    }
    else {
        $('#standard').prop("checked", true);
    }
	//Set the hiddenQuery parameters
    $('#search').coveo('state', 'hd', detail);
    $('#search').coveo('state', 'hq', query);
}

//On Load...

    //If the Advanced search is changed we need to trigger the change
    $('.user-personalisation-option').change(function () {
        persoFilter = $(this).attr('id');
        setSearchParams();
		//Set analytics event
        var customEventCause = {name: 'changePreferences', type:'search box'};
        $('#search').coveo('logSearchEvent', customEventCause, {});
        $('#search').coveo('executeQuery');
    });
```

### Sending a PageView event when clicking on a URL
In a normal situation your public website or other system could sent additional PageView events to our Analytics engine. You need that if you want to use our [Recommendations](http://www.coveo.com/go?dest=cloudhelp&lcid=9&context=237) component. The Recommendations component will look at your current journey and will use our Machine Learning to start recommending articles based upon other peoples journey.

In our case we simply want to add such an event whenever end-users are clicking on the preview/and or are following the hyperlink.
``` javascript
$('#search').on('changeAnalyticsCustomData', function (e, args) {
	//Register a pageview when clicking on a result (used for Recommendations)
	// Normally this would be done by the website directly
	if (args.type === 'ClickEvent') {
		let contentType = null;
		if ((/themoviedb/).test(args.metaObject.documentURL)) {
			contentType = "Movie";
		}
		if ((/last\.fm/).test(args.metaObject.documentURL)) {
			contentType = "Music";
		}

		if (contentType) {
			let meta = args.metaObject;
			coveoua("init", Coveo.Analytics.options.token.defaultValue);
			coveoua('send', 'pageview', {
				contentIdKey: meta.contentIDKey,
				contentIdValue: meta.contentIDValue,
				contentType: contentType
			});
		}
	}
});
```

## Pushing Analytics events using the UABOT
This is a demo instance and not used like any other production system. But for the demo we would still would like to show our [Search Analytics Dashboards](http://www.coveo.com/go?dest=cloudhelp&lcid=9&context=238). In order to show them we need data. That is where our [UABOT](https://github.com/coveo/uabot) comes in. It runs all the time and based upon a JSON configuration file it will start pushing Analytics events to our Analytics service. Based upon the Search Analytics events our Machine Learning will also start working by [suggesting query completions](https://onlinehelp.coveo.com/en/cloud/enabling_coveo_machine_learning_query_suggestions_in_a_coveo_js_search_framework_search_box.htm) and [ART](http://www.coveo.com/go?dest=cloudhelp&lcid=9&context=166).

Our UABOT file:
``` json
{
	"searchendpoint": "https://platform.cloud.coveo.com/rest/search/",
	"analyticsendpoint": "https://usageanalytics.coveo.com/rest/v15/analytics/",
	"orgName": "ORGNAME",
	"pipeline": "ML",
	"searchHub": "Movie",
	"defaultOriginLevel1": "Movie",
	"defaultOriginLevel2": "default",
	"allowAnonymousVisits": true,
	"anonymousThreshold": 0.3,
	"emailSuffixes" : ["@coveo.com"],
	"timeBetweenVisits": 20,
	"timeBetweenActions": 3,
	"randomGoodQueries": [
		"red willis", "captain america", "jungle", "dinosaur",
		"spaceship", "hunger", "smith", "mortal", "jumanji", "frozen",
		"history museum", "toy story", "disney", "fiction", "sience", "weird", "taken",
		"mortal", "king", "aliens", "Indiana Jones", "Jack Ryan",
		"Inglourious Basterds", "Eye of the Tiger", "Ghostbusters", "Bruce Wayne",
		"Marvel", "transformers", "avengers", "rock", "adventure", "comedy", "smile", "knight", "last knight",
		"I Don't Want to Miss a Thing"
	],
	"randomBadQueries": ["woopi godberg", "Inglarious Basserds",
		"Ghostbutter", "Gostbuster", "star treq", "Spuck",
		"jumangi", "frosen", "dsiney", "dysney", "therock", "tranformer"
	],
	"randomCustomData": [
		{ "apiname": "context_userRole", "values": ["Anonymous", "Producer"]},
		{ "apiname": "role", "values": ["Anonymous", "Producer"]},
		{ "apiname": "c_isbot", "values": ["true"]}
	],
	"globalFilter": "NOT @mytype=Wikipedia",
	"scenarios": [{
			"name": "GQ -> C(50%) -> C(15%)",
			"weight": 5,
			"events": [{
					"type": "Search",
					"arguments": {
						"queryText": "",
						"goodQuery": true
					}
				},
				{
					"type": "Click",
					"arguments": {
						"docNo": -1,
						"offset": 0,
						"probability": 0.5
					}
				},
				{
					"type": "Click",
					"arguments": {
						"docNo": -1,
						"offset": 0,
						"probability": 0.25
					}
				}
			]
		}, {
			"name": "BQ -> C(+10,33%)",
			"weight": 2,
			"events": [{
					"type": "Search",
					"arguments": {
						"queryText": "",
						"goodQuery": false
					}
				},
				{
					"type": "Click",
					"arguments": {
						"docNo": -1,
						"offset": 10,
						"probability": 0.10
					}
				}
			]
		},
		{
			"name": "(G) red willis",
			"weight": 3,
			"events": [{
				"type": "SearchAndClick",
				"arguments": {
					"queryText": "red willis",
					"matchField": "title",
					"matchValue": "RED",
					"probability": 0.95
				}
			}
		]
	},
	{
		"name": "(G) red2 willis",
		"weight": 1,
		"events": [{
				"type": "SearchAndClick",
				"arguments": {
					"queryText": "red willis",
					"docClickTitle": "RED 2",
					"probability": 0.50
				}
			}]
		}, {
			"name": "QC(Eye of the Tiger, 95%)",
			"weight": 1,
			"events": [{
				"type": "SearchAndClick",
				"arguments": {
					"queryText": "Eye of the Tiger",
					"docClickTitle": "Eye of the Tiger by Survivor",
					"probability": 0.95
				}
			}]
		}, {
			"name": "QC(The Rock, 95%)",
			"weight": 1,
			"events": [{
					"type": "Search",
					"arguments": {
						"queryText": "The Rock",
						"goodQuery": true
					}
				},
				{
					"type": "FacetChange",
					"arguments": {
						"facetTitle": "Supporting people",
						"facetValue": "Dwayne Johnson",
						"facetField": "@mypeople"
					}
				}
			]
		}, {
			"name": "QC(Eye of the Tiger -> Rocky IV, 85%)",
			"weight": 1,
			"events": [{
				"type": "Search",
				"arguments": {
					"queryText": "Eye of the Tiger",
					"goodQuery": true
				}
			}, {
				"type": "SearchAndClick",
				"arguments": {
					"queryText": "Eye of the Tiger Rocky",
					"docClickTitle": "Rocky IV",
					"probability": 0.85
				}
			}]
		}, {
			"name": "QC(Night at the Museum, 95%)",
			"weight": 3,
			"events": [{
				"type": "SearchAndClick",
				"arguments": {
					"queryText": "history museum",
					"matchField": "title",
					"matchValue": "Night at the Museum",
					"probability": 0.95
				}
			}]
		},
		{
			"name": "QC(Night at the Museum, 95%)",
			"weight": 3,
			"events": [{
				"type": "SearchAndClick",
				"arguments": {
					"queryText": "history museum",
					"docClickTitle": "Night at the Museum: Secret of the Tomb",
					"offset":10,
					"probability": 0.95
				}
			}]
		}, {
			"name": "GQ(fast racing) -> QC(Fast Five, 95%)",
			"weight": 4,
			"events": [{
					"type": "Search",
					"arguments": {
						"queryText": "fast racing",
						"goodQuery": true
					}
				},
				{
					"type": "SearchAndClick",
					"arguments": {
						"queryText": "fast racing",
						"matchField": "title",
						"matchValue": "Fast & Furious",
						"offset": 10,
						"probability": 0.85
					}
				},
				{
					"type": "SearchAndClick",
					"arguments": {
						"queryText": "fast racing",
						"docClickTitle": "Fast & Furious 6",
						"offset": 20,
						"probability": 0.95
					}
				},
				{
					"type": "SearchAndClick",
					"arguments": {
						"queryText": "fast racing",
						"docClickTitle": "The Fast and the Furious: Tokyo Drift",
						"offset": 30,
						"probability": 0.95
					}
				}
			]
		}
	]

}
```

And that is it! I hope you enjoyed it.

Special thanks to my colleague Jérôme Devost who helped me to build this!