<h2>How to work around using ArcGIS Online and Portal with the ArcGIS Web App Builder </h2>

*Still a work in progress, I'm just trying to get the basic instructions in place for now.

This is a method that I presented at the ESRI 2015 DevSummit as a lightning talk presentation as a way to easily get around the requirement of using ArcGIS Online and/or Portal with the Web App Builder. I'll skip the initial set up of the Web App Builder in terms of adding in widgets and layout of the web map and move forward from the point that you download the application to work with the code (However, I will try to add this later on if desired).

So you've downloaded your WAB based web map and would like to use your ArcGIS Server services without having to work through the additional layer of ArcGIS Online. Doing this is actually rather simple, you just need to edit two files:
  -config.json (the main config file in the main directory of the WAB download)
  -MapManager.js (found inside the "jimu.js" folder)
  
Step 1: Edit the config.json file
In your config.json file, you will want to look for the two instances where it asks for the two instances where it asks for portalUrl and make sure that it is referencing your organization URL (if you don't have one you can also use a free Developer url as well (you can sign up here https://developers.arcgis.com/en/sign-up/). Though you may not be using any services hosted on the URL, the Web App Builder still needs something to take in in order to get through it's set up so make sure a valid url is filled out for those two instances of "portalUrl" (ex. "portalUrl": "http://maruttangt.maps.arcgis.com" (example of using a free ESRI Developer account url)).

Step 2: Edit the MapManager.js file
Found inside the "jimu.js" folder, the MapManager.js file contains functions where the WAB attempts to load up your ArcGIS Online Web Map. 

The first thing that you will do is find the initial instance of function "_show2DWebMap", which you should find at line 79. Comment out the call to this function and put in a call to a new function there "_show2DLayersMap" (ex: this._show2DLayersMap(appConfig);)

```
 if (appConfig.map.itemId) {
            //this._show2DWebMap(appConfig);
			this._show2DLayersMap(appConfig);
          } else {
			this._show2DLayersMap(appConfig);
            console.log('No webmap found. Please set map.itemId in config.json.');
          }
  ```

For now, just place this new function above where the _show2DWebMap function is (where ever else is fine) then the function you make should look like this:

```

 _show2DLayersMap: function(appConfig) {
		//You can load required modules here, just like with the ArcGIS JavaScript API you can use both new and old coding methods
        require(['esri/map', "esri/layers/FeatureLayer", "esri/tasks/query"], lang.hitch(this, function(Map, FeatureLayer, Query) {
          var map = new Map(this.mapDivId, 
        {//map options
			extent: new Extent(-11508936.84, 4292809.73, -10408243.64, 5026605.20, new SpatialReference(102100)),
			zoom:8
		});
		  //You can define an ArcGIS Server REST service for the basemap
		  var basemap = new esri.layers.ArcGISTiledMapServiceLayer(
		  "http://services.arcgisonline.com/arcgis/rest/services/Canvas/World_Light_Gray_Base/MapServer",{
		  id: "basemap"
		  });
		  map.addLayer(basemap);
			var content = "<b>Status</b>: ${STATUS}" +
						  "<br><b>Cumulative Gas</b>: ${CUMM_GAS} MCF" +
						  "<br><b>Total Acres</b>: ${APPROXACRE}" +
						  "<br><b>Avg. Field Depth</b>: ${AVG_DEPTH} meters";
			var infoTemplate = new InfoTemplate("${FIELD_NAME}", content);
			//The below is a layer I grabbed from an ArcGIS JavaScript API code sample
			//You can change this to whatever REST feature service you would 
			//like to load from your ArcGIS Server, you can use the layer list widget to zoom to it
			featureLayer = new FeatureLayer("http://sampleserver3.arcgisonline.com/ArcGIS/rest/services/Petroleum/KSPetro/MapServer/1",
			  {
				mode: FeatureLayer.MODE_ONDEMAND,
				infoTemplate: infoTemplate,
				outFields: ["*"]
			  });
			map.addLayer(featureLayer);
			//This generates the map
			this._publishMapEvent(map);
        }));
      },
      
```
