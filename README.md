# GeoFire for iOS — Realtime location queries with Firebase

GeoFire is an open-source library for iOS that allows you to store and query a
set of keys based on their geographic location.

At its heart, GeoFire simply stores locations with string keys. Its main
benefit however, is the possibility of querying keys within a given geographic
area - all in realtime.

GeoFire uses [Firebase](https://www.firebase.com/?utm_source=geofire-objc) for
data storage, allowing query results to be updated in realtime as they change.
GeoFire *selectively loads only the data near certain locations, keeping your
applications light and responsive*, even with extremely large datasets.

A compatible GeoFire client is also available for [Java](https://github.com/firebase/geofire-java)
and [JavaScript](https://github.com/firebase/geofire-js).

### Integrating GeoFire with your data

GeoFire is designed as a lightweight add-on to Firebase. However, to keep things
simple, GeoFire stores data in its own format and its own location within
your Firebase. This allows your existing data format and security rules to
remain unchanged and for you to add GeoFire as an easy solution for geo queries
without modifying your existing data.

### Example Usage
Assume you are building an app to rate bars and you store all information for a
bar, e.g. name, business hours and price range, at `/bars/<bar-id>`. Later, you
want to add the possibility for users to search for bars in their vicinity. This
is where GeoFire comes in. You can store the location for each bar using
GeoFire, using the bar IDs as GeoFire keys. GeoFire then allows you to easily
query which bar IDs (the keys) are nearby. To display any additional information
about the bars, you can load the information for each bar returned by the query
at `/bars/<bar-id>`.


## Upgrading GeoFire

### Upgrading from GeoFire 1.0.x to 1.1.x

With the release of GeoFire for iOS 1.1.0, this library now uses [the new query functionality found in
Firebase 2.0.0](https://www.firebase.com/blog/2014-11-04-firebase-realtime-queries.html). As a
result, you will need to upgrade to Firebase 2.x.x and add a new `.indexOn` rule to your Security
and Firebase Rules to get the best performance. You can view [the updated rules
here](https://github.com/firebase/geofire-js/blob/master/examples/securityRules/rules.json)
and [read our docs for more information about indexing your data](https://www.firebase.com/docs/security/guide/indexing-data.html).


## Downloading GeoFire for iOS

In order to use GeoFire in your project, you need to download the framework and
add it to your project. You also need to [add the Firebase
framework](https://www.firebase.com/docs/ios-quickstart.html?utm_source=geofire-objc)
and the CoreLocation framework to your project.

You can download the latest version of the [GeoFire.framework from the releases
page](https://github.com/firebase/geofire-objc/releases) or include the GeoFire
Xcode project from this repo in your project.

Alternatively, if you're using [CocoaPods](http://cocoapods.org/?q=geofire), add
the following to your `Podfile`:

```
pod 'GeoFire', '>= 1.1'
```

### Using GeoFire with Swift

In order to use GeoFire in a Swift project, you'll also need to setup a bridging
header, in addition to adding the Firebase, GeoFire, and CoreLocation frameworks
to your project. To do that, [follow these instructions](https://www.firebase.com/docs/ios/guide/setup.html#section-swift), and then add the following line to your bridging header:

````objective-c
#import <GeoFire/GeoFire.h>
````


## Getting Started with Firebase

GeoFire requires Firebase in order to store location data. You can [sign up here for a free
account](https://www.firebase.com/signup/?utm_source=geofire-objc).


## GeoFire for iOS Quickstart

This is a quickstart on how to use GeoFire's core features. There is also a
[full API reference available
online](https://geofire-ios.firebaseapp.com/docs/).

### GeoFire

A `GeoFire` object is used to read and write geo location data to your Firebase
and to create queries. To create a new `GeoFire` instance you need to attach it to a Firebase reference:

##### Objective-C
```objective-c
Firebase *geofireRef = [[Firebase alloc] initWithUrl:@"https://<your-firebase>.firebaseio.com/"];
GeoFire *geoFire = [[GeoFire alloc] initWithFirebaseRef:geofireRef];
```

##### Swift
````swift
let geofireRef = Firebase(url: "https://<your-firebase>.firebaseio.com/")
let geoFire = GeoFire(firebaseRef: geofireRef)
````

Note that you can point your reference to anywhere in your Firebase, but don't
forget to [setup security rules for
GeoFire](https://github.com/firebase/geofire-js/blob/master/examples/securityRules/rules.json).

#### Setting location data

In GeoFire you can set and query locations by string keys. To set a location for a key
simply call the `setLocation:forKey` method:

##### Objective-C
```objective-c
[geoFire setLocation:[[CLLocation alloc] initWithLatitude:37.7853889 longitude:-122.4056973]
              forKey:@"firebase-hq"];
```

##### Swift
````swift
geoFire.setLocation(CLLocation(latitude: 37.7853889, longitude: -122.4056973), forKey: "firebase-hq")
````

Alternatively a callback can be passed which is called once the server
successfully saves the location:

##### Objective-C
```objective-c
[geoFire setLocation:[[CLLocation alloc] initWithLatitude:37.7853889 longitude:-122.4056973]
              forKey:@"firebase-hq"
 withCompletionBlock:^(NSError *error) {
     if (error != nil) {
         NSLog(@"An error occurred: %@", error);
     } else {
         NSLog(@"Saved location successfully!");
     }
 }];
```

##### Swift
````swift
geoFire.setLocation(CLLocation(latitude: 37.7853889, longitude: -122.4056973), forKey: "firebase-hq") { (error) in
  if (error != nil) {
    println("An error occured: \(error)")
  } else {
    println("Saved location successfully!")
  }
}
````

To remove a location and delete the location from Firebase simply call:

##### Objective-C
```objective-c
[geoFire removeKey:@"firebase-hq"];
```

##### Swift
````swift
geoFire.removeKey("firebase-hq")
````

#### Retrieving a location

Retrieving locations happens with callbacks. If the key is not present in
GeoFire, the callback will be called with `nil`. If an error occurred, the
callback is passed the error and the location will be `nil`.

##### Objective-C
```objective-c
[geoFire getLocationForKey:@"firebase-hq" withCallback:^(CLLocation *location, NSError *error) {
    if (error != nil) {
        NSLog(@"An error occurred getting the location for \"firebase-hq\": %@", [error localizedDescription]);
    } else if (location != nil) {
        NSLog(@"Location for \"firebase-hq\" is [%f, %f]",
              location.coordinate.latitude,
              location.coordinate.longitude);
    } else {
        NSLog(@"GeoFire does not contain a location for \"firebase-hq\"");
    }
}];
```

##### Swift
````swift
geoFire.getLocationForKey("firebase-hq", withCallback: { (location, error) in
  if (error != nil) {
    println("An error occurred getting the location for \"firebase-hq\": \(error.localizedDescription)")
  } else if (location != nil) {
    println("Location for \"firebase-hq\" is [\(location.coordinate.latitude), \(location.coordinate.longitude)]")
  } else {
    println("GeoFire does not contain a location for \"firebase-hq\"")
  }
})
````

### Geo Queries

GeoFire allows you to query all keys within a geographic area using `GFQuery`
objects. As the locations for keys change, the query is updated in realtime and fires events
letting you know if any relevant keys have moved. `GFQuery` parameters can be updated
later to change the size and center of the queried area.

##### Objective-C
```objective-c
CLLocation *center = [[CLLocation alloc] initWithLatitude:37.7832889 longitude:-122.4056973];
// Query locations at [37.7832889, -122.4056973] with a radius of 600 meters
GFCircleQuery *circleQuery = [geoFire queryAtLocation:center withRadius:0.6];

// Query location by region
MKCoordinateSpan span = MKCoordinateSpanMake(0.001, 0.001);
MKCoordinateRegion region = MKCoordinateRegionMake(center.coordinate, span);
GFRegionQuery *regionQuery = [geoFire queryWithRegion:region];
```

#### Swift
````swift
let center = CLLocation(latitude: 37.7832889, longitude: -122.4056973)
// Query locations at [37.7832889, -122.4056973] with a radius of 600 meters
var circleQuery = geoFire.queryAtLocation(center, withRadius: 0.6)

// Query location by region
let span = MKCoordinateSpanMake(0.001, 0.001)
let region = MKCoordinateRegionMake(center.coordinate, span)
var regionQuery = geoFire.queryWithRegion(region)
````

#### Receiving events for geo queries

There are three kinds of events that can occur with a geo query:

1. **Key Entered**: The location of a key now matches the query criteria.
2. **Key Exited**: The location of a key no longer matches the query criteria.
3. **Key Moved**: The location of a key changed but the location still matches the query criteria.

Key entered events will be fired for all keys initially matching the query as well as any time
afterwards that a key enters the query. Key moved and key exited events are guaranteed to be
preceded by a key entered event.

To observe events for a geo query you can register a callback with `observeEventType:withBlock:`:

##### Objective-C
```objective-c
FirebaseHandle queryHandle = [query observeEventType:GFEventTypeKeyEntered withBlock:^(NSString *key, CLLocation *location) {
    NSLog(@"Key '%@' entered the search area and is at location '%@'", key, location);
}];
```

##### Swift
````swift
var queryHandle = query.observeEventType(GFEventTypeKeyEntered, withBlock: { (key: String!, location: CLLocation!) in
  println("Key '\(key)' entered the search area and is at location '\(location)'")
})
````

To cancel one or all callbacks for a geo query, call
`removeObserverWithFirebaseHandle:` or `removeAllObservers:`, respectively.

#### Waiting for queries to be "ready"

Sometimes you want to know when the data for all the initial keys has been
loaded from the server and the corresponding events for those keys have been
fired. For example, you may want to hide a loading animation after your data has
fully loaded. `GFQuery` offers a method to listen for these ready events:

##### Objective-C
```objective-c
[query observeReadyWithBlock:^{
    NSLog(@"All initial data has been loaded and events have been fired!");
}];
```

##### Swift
````swift
query.observeReadyWithBlock({
  println("All initial data has been loaded and events have been fired!")
})
````

Note that locations might change while initially loading the data and key moved and key
exited events might therefore still occur before the ready event was fired.

When the query criteria is updated, the existing locations are re-queried and the
ready event is fired again once all events for the updated query have been
fired. This includes key exited events for keys that no longer match the query.

#### Updating the query criteria

To update the query criteria you can use the `center` and `radius` properties on
the `GFQuery` object. Key exited and key entered events will be fired for
keys moving in and out of the old and new search area, respectively. No key moved
events will be fired as a result of the query criteria changing; however, key moved
events might occur independently.


## API Reference

[A full API reference is available here](https://geofire-ios.firebaseapp.com/docs/).


## Contributing

If you'd like to contribute to GeoFire for iOS, you'll need to run the
following commands to get your environment set up:

```bash
$ git clone https://github.com/firebase/geofire-objc.git
$ cd geofire-objc
$ ./setup.sh
```
