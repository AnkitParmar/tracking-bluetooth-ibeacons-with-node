
UNSTABLE - WORK IN PROGRESS

Tracking mobile iBeacons and BlueTooth Low Energy engagements with Node.js
--###WhatBluetooth Low Energy(BLE) ‘beacons’ have come into the world around us.  iBeacon is Apple’s branded convention and integration of Bluetooth LE into 3 core ‘location’ features: broadcast, monitoring and ranging.iBeacon is primarily a location service similar to GPS location revolution that has made mobile devices so valuable. Unlike GPS which gives gives devices 'vicinity' awareness (limited to outdoor scenarios that have line of sight to GPS satellites), BLE iBeacon’s gives devices 'proximity' awareness.###Your phone is your signatureThe branding and assimilation of BLE iBeacons into Apples iOS7 platform is causing a proliferation of Bluetooth ‘beaconing’ technology. Large implementations of the technology such as MLB's installation into ball parks , and PayPal's Beacon payment system are making facilities and locations smart. Startups like [Estimote](http://), [Brick Trends](http://), [Pebble](http://) are making it easier for developers and businesses to integrate the technology into their customers’ daily lives and their mobile app’s.  The technology is poised to forever change kiosks, retail end-caps, payment terminals and the facilities around us. Historically these engagement touch points were blind, dumb and mute to your mobile device. Effectively the urinal in the men's bathroom is smarter than your average retail kiosk; at least the urinal knew to flush when you stepped away. iBeacons will make facilities, displays and the devices that power them aware of the mobile device ( and user ) in front of them, and more importantly give insight into the previously dark world of “in store” behavior and engagement analytics.###Why - AwarenessWhen you open am iBeacon enabled mobile app your Blue Tooth ‘signature' your relative location can be tracked inside the walls of a facility. In a similar way to how a web cookie allows retailers to track your behavior through a web site, you can track ‘engagements’ between a ‘sniffer’ device and the mobile app as proxy to the human holding it.###How – PledgeIt’s very easy to track a Bluetooth device using Node.js on your local machine and track the engagement in a MongoDB for later analysis and reporting.  We can do this using Node.js, LoopBack, and Bleacon in 3 commands and as little as 6 lines of code.###How – DemoYou can build the demo on your own and see it run on a mac book pro (requires Mavericks) with Node.js by simply cloning the repo ‘git clone https://github.com/mschmulen/tracking-bluetooth-ibeacons-with-node’ and running it with ‘slc run app.js’or build it from scratch by following the instructions below:Commands:
```npm install -g strong-clislc lb project ibeacon-node-collectorcd ibeacon-node-collectornpm install --save bleaconslc lb model ibeaconslc lb model engagement```

The commands above installs the [strong-cli](http://https://npmjs.org/package/strong-cli)making giving us access to the ```slc lb``` commands to quickly scaffold an API tier and add our ```beacon``` and ```engagement``` data models. Installs the [bleacon](https://npmjs.org/package/bleacon) npm package and dependencies.

Code:

Add the 10 lines of code below to your app.'s file after line 121 in app.js file, right after ``` app.get('/', loopback.status()); ```  to [upsert](http://strongloop.com) a beacon engagement and beacon signature into the default in memory data store.
```
var ibeacon = app.models.ibeacon;
var Bleacon = require('bleacon');
var hist = [];

var ibeaconCache = new Array();

Bleacon.on('discover', function(bleacon) {
	
	guid = bleacon.uuid + bleacon.major + bleacon.minor;
	
  ibeacon.upsert({ guid:guid,uuid:bleacon.uuid, major:bleacon.major, minor:bleacon.minor},
		function( error, _ibeacon) 
		{
			if ( error ) { console.log( "error on upsert " ); }
			else {
				var eng = app.models.engagement;
				eng.create({ pBeaconID: _ibeacon.guid, timestamp: Date.now(), rssi: bleacon.rssi, proximity: bleacon.proximity });
			} //end else success
	}); //end upsert
	
  hist.unshift(bleacon.rssi);
  if (hist.length > 50) hist.splice(50, 1);
  var avg = hist.reduce(function(total, current) { return total + current; }) / hist.length;
  var xs = new Array(Math.floor(-avg) + 1).join('x');
	//console.log( "discovered " + bleacon.uuid + " " + bleacon.major + ":" + bleacon.minor +" rssi:" + bleacon.rssi );
  console.log(xs);
	
});//end on discover

Bleacon.startScanning();```
Test the implementation by installing an app that uses iBeacons from the from the app store. The [Estimote Virtual Beacon on the App Store](https://itunes.apple.com/us/app/estimote-virtual-beacon/id686915066?mt=8) is what I'm using.

Start the Estimote App in 'Beacons' mode.

<img src="screenshots/estimote-ibeacon.png" alt="tab 1" height="209" width="120">
and then simply start the node application on your host machine with 
```slc run app.js```

In addition to entering the engagement in the in memory data store the code above will also show the BLE [rssi](http://) to the console log as your Node application receives Bleacon discover events.
<img src="screenshots/console-run.png" alt="tab 1" width="420">###Resolutions – Turn You can install this on a Raspberry-Pi (just follow these instructions) to create a simple “sniffer” device that when plugged into the wall will track all the Bluetooth engagements that occur in your home, office or store.
If you want to persist your analytics data you can simply change the LoopBack model binding in ```datasources.json``` to point to a MongoDB instance, you can find more information on binding your Node.js API tier LoopBack supported data stores [here](http://strongloop.com).
It's also interesting to turn on StrongOps and watch the memory and CPU profile of your machine as your machine 'sniffer' captures.
Lets look at some of the beacon signatures and engagements that we have captured using the LoopBack Explorer feature.[http://localhost:3000/explorer/](http://localhost:3000/explorer/)<img src="screenshots/loopback-explorer.png" alt="tab 1" width="420">###Resolutions – Prestige
Show the dashboard with search and filter.

Show all beacons in the system:
[http://localhost:3000/api/engagements](http://localhost:3000/api/engagements)

<img src="screenshots/api-ibeacons.png" alt="tab 1" height="209" width="120">

Filter engagements based on a GUID:
[http://localhost:3000/api/engagements?filter[where][cBeaconGUID]=87209302c7f24d56b1d114eadd0ce41f15](http://localhost:3000/api/engagements?filter[where][cBeaconGUID]=87209302c7f24d56b1d114eadd0ce41f15)

Filter engagements based on GUID and time:
[http://localhost:3000/api/engagements?filter[where][cBeaconGUID]=87209302c7f24d56b1d114eadd0ce41f15&filter[where][pBeaconGUID]=87209302c7f24d56b1d114eadd0ce41f16](http://localhost:3000/api/engagements?filter[where][cBeaconGUID]=87209302c7f24d56b1d114eadd0ce41f15&filter[where][pBeaconGUID]=87209302c7f24d56b1d114eadd0ce41f16)

If you want to build your own iOS app you can start with this sample:

Filter engagements based on 2 GUID interactions:
[http://localhost:3000/api/engagements?filter[where][cBeaconGUID]=87209302c7f24d56b1d114eadd0ce41f15&filter[where][pBeaconGUID]=87209302c7f24d56b1d114eadd0ce41f16](http://localhost:3000/api/engagements?filter[where][cBeaconGUID]=87209302c7f24d56b1d114eadd0ce41f15&filter[where][pBeaconGUID]=87209302c7f24d56b1d114eadd0ce41f16)
###Resolutions – MonsterMatch the user to the anon signature. And then get the behavior of the user. If you want to go the extra mile, simply use the built in LoopBack ‘user model object by adding an index reference to the ‘beacon model’. Do this and you convert all the anonymous engagement tracking analytics that you are recording to known user behavior.  This is conversion from an anonymous unknown user BLE iBeacon signature to known user behavior is the reason that retailers want you to open their customer loyalty app in the store and login.  Binding your user profiler to the device and allowing them to track your in store loitering and behavior ( and possibly target engagements at you ) in the same way they track your shopping behaviors online. Whats NextYak yack.


