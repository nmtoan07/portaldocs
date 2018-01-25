
## Master details browse 

In this discussion, `<dir>` is the `SamplesExtension\Extension\` directory and  `<dirParent>`  is the `SamplesExtension\` directory. Links to the Dogfood environment are working copies of the samples that were made available with the SDK.

 In this example, there is a parent blade that shows a list of resources and a child blade that shows details about an individual resource. This example specifies how to share data across the parent blade and the child blades.

The code for this example comes from the 'master detail browse' sample. The code is located at:
`<dir>\Client\V1\MasterDetail\MasterDetailArea.ts`
`<dir>\Client\V1\MasterDetail\MasterDetailBrowse\MasterDetailBrowse.pdl`
`<dir>\Client\V1\Data\MasterDetailBrowse\MasterDetailBrowseData.ts`
`<dir>\Client\V1\asterDetail\MasterDetailBrowse\ViewModels\DetailViewModels.ts`
`<dir>\Client\V1\MasterDetail\MasterDetailBrowse\ViewModels\MasterViewModels.ts`

The `QueryCache` and the `EntityCache` are the two types of caches that are used in this scenario. In this scenario, the extension retrieves information from the server and displays this data across multiple blades. The information is a list of websites. The server data is cached using one of the previously-mentioned caches, and then uses that cache to display the websites across two blades. The first blade displays the list of websites in a grid. When the user activates one of those websites, a second blade is opened, and displays more details about the activated website. The data for both blades is located in the cache, therefore the server is not queried again when it is time to open the second blade. When the data in the cache is updated, that update is displayed across all blades at the same time. Consequently, the Portal always presents a consistent view of the data.

### The MasterDetail Area and DataContext

The portal uses an `Area` to hold the cache and other data objects that are shared across multiple blades. The code for the area is located in its own folder, as specified in [portalfx-data-overview.md#areas](portalfx-data-overview.md#areas). In this example the area folder is named `MasterDetail` and it is located in the `Client` folder of the extension. 

 Inside the folder, there is a typescript file whose name is a combination of the name of the area and the word 'Area'. For this example, the file is named  `MasterDetailArea.ts` and is located at `<dir>Client/V1/MasterDetail/MasterDetailArea.ts`.  This code is also included in the following example.

<!-- 
gitdown": "include-section", "file":"../Samples/SamplesExtension/Extension/Client/V1/MasterDetail/MasterDetailArea.ts", "section": "data#websitesQueryCache"}
-->

This file contains the `DataContext` class, which is the class that will be sent to all the `ViewModels` associated with the area.  The `DataContext` also contains an `EditScopeCache` which is used in the master detail edit scenario that is located at . This code is also included in the following example.

 <!--TODO:  Locate the gitHub copies of the samples. -->

<!-- determine whether an "as specified in portalfx-forms-editscope*" is relevant here.  -->

If this is a new area for the extension, edit the  `Program.ts` file to create the `DataContext` when the extension is loaded. The SDK edition of the `Program.ts` file is located at `<dir>\Client\Program.ts`. For the extension that is being built, find the `initializeDataContexts` method and then use the `setDataContextFactory` method to set the `DataContext`.

There are two data caches that are used in the master/detail browse scenario. The `QueryCache` can be used to cache a list of items as specified in [#querycache](#querycache). The `EntityCache` can be used to cache a single item, as specified in [#entitycache](#entitycache).

### QueryCache 

When the `QueryCache` that contains the websites is created, two elements are specified.

1. The name of the entityType for a website.

    The QueryCache needs to know the shape of the data contained in it, in order to handle the data appropriately. That information is specified with the entity type.

1. A function that returns the URI to populate the cache.
        
    This function is sent a set of parameters that are sent to a `fetch` call. In this case, `runningStatus` is the only parameter. Based on the presence of the `runningStatus` parameter, the URI is modified to query for the correct data.

<!-- TODO:  determine whether "presence" can be changed to "value". Did they mean true if present and false if absent, with false as the default value?  This sentence needs more information. -->

When all of these items are complete, the `QueryCache` is  configured. It will be populated while Views are being created over the cache, and the `fetch()` method is called. 

### EntityCache

The other cache used in this sample is the EntityCache. The SDK edition of the  file is located at `<dir>Client/V1/MasterDetail/MasterDetailArea.ts`. This code is also included in the following example.

<!-- {"gitdown": "include-section", "file":"../Samples/SamplesExtension/Extension/Client/V1/MasterDetail/MasterDetailArea.ts", "section": "data#websitesEntityCache"} -->

When the EntityCache that contains the websites is created, three elements are specified.

1. The name of the entityType 

   The cache requires this so that it can reason over the data.

1. The `sourceUri` property. 

    This function that, given the parameters from a `fetch()` call, will return the URI that the cache uses to get the data. In this instance, the  `MsPortalFx.Data.uriFormatter()` helper method is used. It fills one or more parameters into the URI provided to it. In this instance there is only  one parameter, the `id` parameter, which is filled into the part of the URI containing `{id}`.

1. The `findCachedEntity` property. 

<!-- TODO:  Determine whether this is a method instead of a property. -->

   This optional property allows the lookup of an entity from the `QueryCache`, instead of retrieving the data a second time from the server, which would create a second copy of the website data on the client. 
    
   The two properties of the `findCachedEntity` property are the `QueryCache` to use and a function that, given a item from the QueryCache, will specify whether this is the object that was requested by the parameters to the `fetch()` call.

### Implementing the master view

The master view is used to display the data in the caches. The advantage of using views is that they allow the `QueryCache` to handle the caching, refreshing, and evicting of data.

1. Make sure that the PDL that defines the blades specifies the  `Area` so the `ViewModels` receive the `DataContext`. In the `<Definition>` tag at the top of the PDL file, include an `Area` attribute whose value corresponds to the name of the area that is being built, as in the example located at `<dir>Client/V1/MasterDetail/MasterDetailBrowse/MasterDetailBrowse.pdl`. This code is also included in the following example.

    <!--{"gitdown": "include-section", "file":"../Samples/SamplesExtension/Extension/Client/V1/MasterDetail/MasterDetailBrowse/MasterDetailBrowse.pdl", "section": "data#pdlArea"} -->

    The `ViewModel` for the list of websites is located in `<dir>\Client\V1\MasterDetail\MasterDetailBrowse\ViewModels\MasterViewModels.ts`. 

1. Create a view on the QueryCache, as in the following example.

    <!-- {"gitdown": "include-section", "file":"../Samples/SamplesExtension/Extension/Client/V1/MasterDetail/MasterDetailBrowse/ViewModels/MasterViewModels.ts", "section": "data#createView"} -->

   The view is the `fetch()` method that is called to populate the `QueryCache`, and allows the  items that are returned by the fetch call to be viewed. 

   **NOTE**:  There may be multiple views over the same `QueryCache`. This happens when there are multiple blades on the screen at the same time, all of which are displaying data from the same cache. 

   There are two controls on this blade, both of which use the view that was just created: a grid and the `OptionGroup` control.  

   * The grid displays the data in the `QueryCache`, as specified in []().

   * The `OptionGroup` control allows the user to select whether to display websites that are in a running state, websites in a stopped state or display both types of sites. The  display created by the `OptionGroup` control is specified in []().

#### QueryCache 

The observable `items` array of the view is sent to the grid constructor as the `items` parameter, as in the following example.

<!--
{"gitdown": "include-section", "file":"../Samples/SamplesExtension/Extension/Client/V1/MasterDetail/MasterDetailBrowse/ViewModels/MasterViewModels.ts", "section": "data#gridConstructor"}
-->

The `fetch()` command has not yet been issued on the QueryCache, because it is asynchronous. When it is issued, the view's `items` array will be observably updated, which populates the grid with the results. This occurs by calling the  `fetch()` method on the blade's `onInputsSet()`, which returns the promise shown in the following example.

<!--
{"gitdown": "include-section", "file":"../Samples/SamplesExtension/Extension/Client/V1/MasterDetail/MasterDetailBrowse/ViewModels/MasterViewModels.ts", "section": "data#onInputsSet"}
-->

This will populate the `QueryCache` with items from the server and display them in the grid.

#### The OptionGroup control 

The  control is initialized, and the extension then subscribes to its value property, as in the following example.

<!-- {"gitdown": "include-section", "file":"../Samples/SamplesExtension/Extension/Client/V1/MasterDetail/MasterDetailBrowse/ViewModels/MasterViewModels.ts", "section": "data#optionGroupValueSubscription"} -->

In the subscription, the extension performs the following actions.

1. Put the grid in a loading mode. It will remain in this mode until all of the data is retrieved.
1. Request the new data by calling the `fetch()` method on the data view with new parameters.
1. When the `fetch()` method completes, take the grid out of loading mode.

There's no need to try to get the results of the fetch and replace the items in the grid because we've pointed the grid's items array to the items array of the view. The view will update its items array as soon as the fetch is complete.

If you look through the rest of the code you'll see we've configured the grid to activate any of the websites when they're clicked on. We'll pass the 'id' of the website that is activated to the details blade as an input.

### Implementing the detail view

The detail view will use the EntityCache (which we hooked up to our QueryCache) from the DataContext to display the details of a website. Once you understand what's going on in the master blade you should have a pretty good handle of what's going on here. The blade starts by creating an view on the EntityCache:

{"gitdown": "include-section", "file":"../Samples/SamplesExtension/Extension/Client/V1/MasterDetail/MasterDetailBrowse/ViewModels/DetailViewModels.ts", "section": "data#entityCacheView"}

Then in the `onInputsSet` we call `fetch` passing the ID of the website we want the data for:

{"gitdown": "include-section", "file":"../Samples/SamplesExtension/Extension/Client/V1/MasterDetail/MasterDetailBrowse/ViewModels/DetailViewModels.ts", "section": "data#onInputsSet"}

When the fetch is completed the data will be available in the view's `item` property. This blade uses the `text` data-binding in it's HTML template to show the name, id and running status of the website but obviously you could do whatever you want with the item.