---
layout: post

title: "Facet dependency"

tags: [JavaScript Search Framework, Facets]

author:
  name: Gregory Laporte
  bio: Product Analyst, Coveo for Salesforce
  image: glaporte.jpg
---

Since Coveo offers a unified result list, people tend to want an *All Content* tab with a set of results coming from different repositories. This is pretty simple to setup, but compared to the facets per tab setup, having too much facets targeted on specific metadatas on the main tab can be rather confusing for an end user. What if you could hide and show facets depending on the state of another? That's what we are going to look at.

<!-- more -->

## Building the search page

The first step is to build the page with the standard markup. Since we don't want any tabs, we can remove the `coveo-tab-section` section.
 

Then we should find the trigger facets and the dependant facets. Let's say that `objecttype` is the trigger facet and we want to show case related facets when a user selects `Case` in the type facet.

We start with the following markup. If we reload the page, we will see 3 facets.

{% highlight html %}

<div class="coveo-facet-column">
  <div class="CoveoFacet" data-title="Type" data-field="@objecttype"></div>
  <div class="CoveoFacet" data-title="Case Status" data-field="@sfcasestatus"></div>
  <div class="CoveoFacet" data-title="Case Origin" data-field="@sfcaseorigin"></div>
</div>

{% endhighlight %}

In order to achieve our goal, the trigger facet doesn't need additional information, but the dependant facet needs those two:
- The trigger facet id (defaults to field name if not provided)
- The trigger facet value

In order to remain consistent with the framework, I decided to add those 2 options as data attributes on the dependant facets. I choosed to call those attributes `data-depends-on-id` and `data-depends-on-value`.


Let's add this to our markup.

{% highlight html %}

<div class="coveo-facet-column">
  <div class="CoveoFacet" data-title="Type" data-field="@objecttype"></div>

  <div class="CoveoFacet" data-title="Case Status" data-field="@sfcasestatus" 
  data-depends-on-id="@objecttype"
  data-depends-on-value="Case"></div>

  <div class="CoveoFacet" data-title="Case Origin" data-field="@sfcaseorigin"
  data-depends-on-id="@objecttype"
  data-depends-on-value="Case"></div>
</div>

{% endhighlight %}

Now if we reload the page, we still see 3 facets. Let's do some javascript now.

## Writing the JavaScript code

Now that we have all the information needed, we can start writing the javascript code. The first thing we need to do is to disable the dependant facets, so we don't see them when we load the page. This can be done with this simple code snippet.

{% highlight javascript %}
$(function() {
  //Select all the facets with a dependency attribute
  $('.CoveoFacet[data-depends-on-id]').each(function(index, facet) {
    $(facet).coveo().disable(); //Disable them
  });
});
{% endhighlight %}

Now that we disabled them, let's add some code to re-enable them depending on state.
{% highlight javascript %}
  $('.CoveoFacet[data-depends-on-id]').each(function(index, facet) {
    //First, get the data attributes value.
    var id = $(facet).data('dependsOnId');
    var value = $(facet).data('dependsOnValue');
    $(facet).coveo('disable');
    //Add a state change event to detect the selection of the trigger facet
    $('#search').on('state:change:f:' + id, function(e, args) {
      $(facet).addClass('coveo-empty');
      //Check if the value is in the list of selected values
      if (_.contains(args.value, value)) {
        //If the value we depend on is selected, enable the facet
        $(facet).coveo('enable').coveo('reset');
        console.log('Enabling facet ' + $(facet).attr('data-field'));
      } else {
        //If the value we depend on is not selected, disable the facet
        $(facet).coveo('disable');
        console.log('Disabling facet ' + $(facet).attr('data-field'));
      }
    });
  });
{% endhighlight %}

The first thing that we had to do was to add a state change event listener. The idea here is to be as precise as possible to avoid some useless checks. Now we know that our callback will be invoked when the state of our trigger facet changes.

Once we know that the state of our trigger facet changed, we need to check if the value we depends on is selected. Depending on that, we can easily enable of disable the facet using it's public methods. 

 
You can learn more about the [state events](https://developers.coveo.com/display/public/JsSearch/State) and the [facet's public methods](https://developers.coveo.com/display/public/JsSearch/Facet+Component#FacetComponent-Methods) in our documentation.
