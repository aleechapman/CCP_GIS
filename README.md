<!DOCTYPE html>
<html>
  <head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
    <!--The viewport meta tag is used to improve the presentation and behavior of the samples 
      on iOS devices-->
    <meta name="viewport" content="initial-scale=1, maximum-scale=1,user-scalable=no">
    <title>Feature Layer - display results as an InfoWindow onHover</title>

    <link rel="stylesheet" href="http://js.arcgis.com/3.10/js/dojo/dijit/themes/tundra/tundra.css">
    <link rel="stylesheet" href="http://js.arcgis.com/3.10/js/esri/css/esri.css">
    <style>
      html, body, #mapDiv {
        padding:0;
        margin:0;
        height:100%;
      }
		#leftDiv {
				width:280px;
				height:100%;		
				float:left;
		}
		#mapDiv {
			margin-left: 280px;
				height:100%;		        
		}

      #info {
        background: #00BBBB;
        box-shadow: 0 0 5px #001111;
        right: 2em;
        padding: 0.5em;
        position: absolute;
        top: 1em;
        z-index: 40;
      }

	  .gaugeContainer {
		text-align:center;
	  }

	  #resultDiv {
		height:200px;
	  }

    </style>

    <script src="http://js.arcgis.com/3.10/"></script>
    <script>
      var map, dialog;
      require([
        "esri/map", "esri/layers/FeatureLayer", "esri/dijit/Legend",
		"esri/dijit/Gauge",
        "esri/symbols/SimpleLineSymbol", 
        "esri/renderers/SimpleRenderer", "esri/graphic", "esri/lang",
		"esri/tasks/query", "dojo/dom", "dojo/on",
        "esri/Color", "dojo/number", "dojo/dom-style", 
        "dijit/TooltipDialog", "dijit/popup", "dojo/domReady!"
      ], function(
        Map, FeatureLayer, Legend,
		Gauge,
        SimpleLineSymbol,
        SimpleRenderer, Graphic, esriLang,
		Query, dom, on,
	   	  Color, number, domStyle, 
		  TooltipDialog, dijitPopup
      ) {
        map = new Map("mapDiv", {
          basemap: "streets",
          center: [-82, 300],
          zoom: 6,
          slider: false
        });
        
        var operationalLayer = new FeatureLayer("https://services2.arcgis.com/No7KRrFgpO516cMP/arcgis/rest/services/Historical_Earthquakes_and_Hurricanes/FeatureServer/1", {
          mode: FeatureLayer.MODE_ONDEMAND,
          outFields: ["Name", "wmo_wind", "wmo_pres", "H_DATE"]
        });
		
		  operationalLayer.minScale = 10000000;
       
	    map.on("layers-add-result", function (evt) {
	       var legend = new Legend({
                map: map,
                layerInfos: [{layer:operationalLayer,title:'Hurricanes'}]
            }, "legendDiv");
			legend.startup();	
		   
		   var gaugeParams = {
				"caption": "Wind Speed",
				"color": "#CC00CC",
					"dataField": "wmo_wind", // attribute used for the gauge value
					"numberFormat": {"places": 1},
				"dataLabelField": "Name",
				"layer": operationalLayer,
				"maxDataValue": 210, // gauge max value 
			}
			var gauge = new esri.dijit.Gauge(gaugeParams, "gaugeDiv");
			gauge.startup();
			
		});

	    map.addLayers([operationalLayer]);

        map.infoWindow.resize(245,125);
        
        dialog = new TooltipDialog({
          id: "tooltipDialog",
          style: "position: absolute; width: 250px; font: normal normal normal 10pt Helvetica;z-index:100"
        });
        dialog.startup();
        
        var highlightSymbol = new SimpleLineSymbol(SimpleLineSymbol.STYLE_SOLID,
                new Color([255,0,0]), 3);


        //close the dialog when the mouse leaves the highlight graphic
        map.on("load", function(){
          map.graphics.enableMouseEvents();
          map.graphics.on("mouse-out", closeDialog);
          
        });
                
        //listen for when the onMouseOver event fires on the countiesGraphicsLayer
        //when fired, create a new graphic with the geometry from the event.graphic and add it to the maps graphics layer
		operationalLayer.on("mouse-over", function(evt){
          var t = "<b>${Name}</b><hr><b> Wind Speed: </b>${wmo_wind}<br>"
			+ "<b> Pressure: </b>${wmo_pres}<br>"
            + "<b> Date: </b>${H_DATE: DateFormat}";

  
          var content = esriLang.substitute(evt.graphic.attributes,t);
          var highlightGraphic = new Graphic(evt.graphic.geometry,highlightSymbol);
          map.graphics.add(highlightGraphic);
          
          dialog.setContent(content);

          domStyle.set(dialog.domNode, "opacity", 0.85);
          dijitPopup.open({
            popup: dialog, 
            x: evt.pageX,
            y: evt.pageY
          });
        });
    
        function closeDialog() {
          map.graphics.clear();
          dijitPopup.close(dialog);
        }

		  operationalLayer.setSelectionSymbol(highlightSymbol);
		  on(dom.byId("execute"), "click", execute);

		  function execute() {
			  var query = new Query();
			  query.where="wmo_wind >" + dom.byId("inputValue").value;		  
			  operationalLayer.selectFeatures(query, 
							 operationalLayer.SELECTION_NEW, 
							  processResult);		
		  }
		   function processResult(features) {		
			  dom.byId("resultDiv").innerHTML = "Daily hurricane paths found: " + features.length;
		  }

      });
    </script>
  </head>
  <body class="tundra">
	<div id="leftDiv">
		<div id="inputDiv">
			Select hurricane paths with winds faster than:
			<input type="text" id="inputValue" value="100"">
			<input id="execute" type="button" value="Go">
		</div>
		<div id="resultDiv">			
		</div>	
		<div id="legendDiv">
		</div>
		<div id="gaugeDiv">
		</div>
	</div>
    <div id="mapDiv">
      <div id="info">
        Hover over a hurricane to get more information.
      </div>
    </div>
  </body>
</html>

