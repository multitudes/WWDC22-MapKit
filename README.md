# WWDC22-MapKit
Notes from the WWDC22 talk 

---
contributors: multitudes
---


Originally launched with U.S. support, the all-new map and Look Around coverage has been expanding since and now includes Canada, many European countries, Japan, and more.

![maps][maps1]

[maps1]: /images/maps1.jpg

Last year, Apple Maps took things to the next level by introducing 3D City Experiences, featuring turn lanes, crosswalks, bike lanes, and beautifully handcrafted 3D landmarks like the Ferry Building.

![Ferry Building][ferryBuilding]

[ferryBuilding]: ../../../images/notes/wwdc22/10035/ferryBuilding.jpg

The additional detail of the map allows you to provide context and precision that was never before possible, and the addition of 3D terrain elevation provides a level of realism like no other map! 


![London St Paul][london]

[london]: ../../../images/notes/wwdc22/10035/london.jpg

![Golden Gate][goldenGate]

[goldenGate]: ../../../images/notes/wwdc22/10035/goldenGate.jpg


New MapKit features. 
- The Map Configuration API. 
- Improvements to the overlay APIs 
- New blend modes support 
- brand-new Selectable Map Features API. 
- Integrating the immersive Look Around experience 

# Map Configuration API. 

Adopting the all-new map in your iOS, macOS, or tvOS application couldn't be easier. Simply recompile your app with the new SDK, and it will be automatically opted in to the all-new Apple map, including the 3D City Experience, where available. For many applications, a simple recompile is all that's needed.


Of course, there may be situations where you need more control over the presentation of the map. In iOS 15, the way you configure the map is through various properties on MKMapView. In iOS 16, however, we're soft deprecating those properties. 


![Deprecated API][DeprecatedAPI]

[DeprecatedAPI]: ../../../images/notes/wwdc22/10035/DeprecatedAPI.jpg

and we're introducing our new Map Configuration API as a replacement.

```swift
// Configuring your map view
class MKMapView {
    ...
    var preferredConfiguration: MKMapConfiguration
}
```
MKMapConfiguration is the central class of the new Map Configuration API. MKMapConfiguration is an abstract base class with three concrete subclasses.

```swift
class MKMapConfiguration {
}

class MKImageryMapConfiguration : MKMapConfiguration {
}

class MKHybridMapConfiguration : MKMapConfiguration {
}

class MKStandardMapConfiguration : MKMapConfiguration {
}
```

The imagery map configuration is used to present satellite-style imagery. The hybrid map configuration is used to present an imagery-based map with added map features such as road labels and points of interest.
The standard map configuration is used to present a fully graphics-based map.  


![Maps Imagery][mapsImagery]

[mapsImagery]: ../../../images/notes/wwdc22/10035/mapsImagery.jpg


These three map configurations may sound familiar to you, as they're similar to our existing map types.

The base map configuration class supports an `elevationStyle` property, which can be either flat or realistic. A flat elevation style means that the ground appears flat. Roads, including bridges and overpasses, also appear flat. 

Flat is the default elevation style. A realistic elevation style means that the ground terrain reproduces the real-world elevation such as hills and mountains. Roads are depicted with realistic elevation details. 

```swift
class MKMapConfiguration {
    var elevationStyle: ElevationStyle
}

extension MKMapConfiguration {
    enum ElevationStyle {
        case flat 
        case realistic
    }
}
```

Now let's take a closer look at the map configuration subclasses. The imagery map configuration only shows satellite imagery with no additional map features, so it doesn't have any other properties.

```swift
class MKImageryMapConfiguration : MKMapConfiguration {
    
}
```

The hybrid map configuration has additional properties to control filtering of point of interest categories and whether to show traffic or not.

```swift
class MKHybridMapConfiguration : MKMapConfiguration {
    var point0fInterestFilter: MKPointOfInterestFilter?
    var showsTraffic: Bool
}
```

The standard map configuration supports an `emphasisStyle` property which can be either default or muted.

class MKStandardMapConfiguration : MKMapConfiguration {
    var emphasisStyle: EmphasisStyle
}

As the name implies, this is the default emphasis style unless otherwise stated. The muted emphasis style softens the contrasts of the map details, allowing you to bring more attention to additional graphical information you might want to show on top. 

![Standard Map Config][standardMapConfig]

[standardMapConfig]: ../../../images/notes/wwdc22/10035/standardMapConfig.jpg

The standard map configuration also has additional properties to control filtering of point of interest categories and whether to show traffic or not. 

```swift
class MKStandardMapConfiguration : MKMapConfiguration {
    var emphasisStyle: EmphasisStyle 
    var pointOfInterestFilter: MKPoint0fInterestFilter?
    var showsTraffic: Bool
}
```

## Map type mapping

To recap what we just discussed, here's a table that shows the correspondence between the new map configuration classes and the `MKMapType` property.

![Map type mapping][mapTypeMapping]

[mapTypeMapping]: ../../../images/notes/wwdc22/10035/mapTypeMapping.jpg

The all-new map with the 3D City Experience requires compatible hardware. On iOS, the new map support requires A12-based iPhones and iPads or later. On macOS, the new map support requires any M1-based computer or later.

![Device Support][deviceSupport]

[deviceSupport]: ../../../images/notes/wwdc22/10035/deviceSupport.jpg

In areas where the 3D City Experience is not available, the map will automatically fall back to present the all-new map with a flat elevation. On all other devices, the all-new map will be presented with a flat elevation.

On M1 Macs, Xcode allows you to simulate both experiences simply by changing the OS version. We encourage you to try out both, to ensure that your app looks great on all devices! The 3D City Experience is available in many metropolitan areas around the world:

## 3D City Experience
- San Francisco Bay Area
- Los Angeles
- New York
- London
- Washington, DC
- San Diego 
- Philadelphia 
- Toronto 
- Montreal 
- Vancouver 

We're continuously adding new cities to this list, so I encourage you to check out the 3D City Experience section on the feature availability website linked in the session notes. 

# Overlays

MapKit has supported overlays with several styling options for years. In iOS 16, we are improving our existing APIs to allow your overlays to seamlessly integrate with the map. Let's start with a quick recap of overlay levels.

Overlays can be rendered at two different levels: above roads and above labels. You can specify the rendering level at insertion time using one of MapKit's many overlay insertion functions.

```swift
// Overlay levels
class MKMapView {
    func addOverlay(_: MKOverlay, level: MKOverlayLevel)
    // ...
}

public enum MKOverlayLevel {
    case aboveLabels 
    case aboveRoads
}
```

Above labels renders your overlay above everything, including labels. Since labels provide important context information, we encourage you to only use above labels in those rare cases where you don't want your data to interact with the map at all. If what you're trying to achieve is for your content to stand out against the map, you might want to consider using the muted map emphasis or the blend modes, which we'll talk about later.

Above roads means the overlay will be shown on top of the terrain, including roads, land cover, or bodies of water. It will, however, be shown below labels and, to some degree, trees and buildings. See the screenshot on the right below. Above roads will be the new default mode in iOS 16. 

![Above Levels][aboveLevels]

[aboveLevels]: ../../../images/notes/wwdc22/10035/aboveLevels.jpg

# Transparent buildings
There is a new feature we're introducing in iOS 16 called transparent buildings.  
Regardless of whether your overlay level is above roads or above labels, your overlay will always be rendered on top of buildings when viewed top-down with no pitch.  

![Above Levels][aboveLevels2]

[aboveLevels2]: ../../../images/notes/wwdc22/10035/aboveLevels2.jpg

However, we've made some improvements to the experience when using above roads in combination with a pitched map. Ground objects such as trees and buildings are now automatically rendered with transparency when appearing above overlays, so as not to fully obscure them. The alpha value varies with the map's pitch angle. If we revert to showing the map top-down with a 0º pitch angle, colliding ground objects effectively disappear from view, leaving your overlays fully visible.

Transparent buildings also work for semi-transparent overlays. The alpha value of the overlay will be added to combine with the alpha value of the transparent buildings. 

![Above Levels][aboveLevels3]

[aboveLevels3]: ../../../images/notes/wwdc22/10035/aboveLevels3.jpg

There's one more change we're making to overlays. When adding an overlay to a map with realistic terrain, 

![Above Levels][aboveLevels4]

[aboveLevels4]: ../../../images/notes/wwdc22/10035/aboveLevels4.jpg

MapKit will automatically transition the map to a flat representation. 

![Above Levels][aboveLevels5]

[aboveLevels5]: ../../../images/notes/wwdc22/10035/aboveLevels5.jpg

The map will automatically go back to realistic when you remove the last overlay. One notable exception to this rule are overlays sourced through MapKit's directions API. Those overlays automatically follow the terrain. 

![Above Levels][aboveLevels6]

[aboveLevels6]: ../../../images/notes/wwdc22/10035/aboveLevels6.jpg

## Adding Polygon Overlays
 
I will be using our sample app, which helps users rent scooters to tour San Francisco.

Our app offers a number of features, as shown by the rows in this table view. "Operating Area" allows the user to see where they can take our scooters. "Ride" takes the user on a tour across the Golden Gate Bridge. "Explore" gives the user an interactive map of downtown San Francisco, which they can use to explore attractions near the waterfront. "Highlights" offers a closer look at must-see places. We will implement or upgrade these features throughout this session. 

I'm going to use the Operating Area feature to demonstrate how easy it is to adopt the all-new map, and then I'll show you some of the improvements we have made to overlays.

Here we are! We get a map view with rich details, including these beautiful lighting and hill-shading effects on the terrain. As we zoom in, you will see buildings, trees... and landmarks.

![Sample App Levels][sampleApp1]

[sampleApp1]: ../../../images/notes/wwdc22/10035/sampleApp1.jpg

Now let's add a polygon overlay to visualize our operating area.

Here I already have the polygon data ready for the operating area. When the view is loaded, we'll first set the region and the camera boundary. Make sure we look at the correct area. Then, we simply add the overlay. 

```swift
import MapKit

// This class displays elevated route line overlay loaded from MapKit server on a map view.

class PolygonOverlayViewController: UIViewController, MKMapViewDelegate {

    @IBOutlet var mapView: MKMapView!
    
    private let dataSource: OverlayDataSource

    required init? (coder: NSCoder) {
        dataSource = OverlavDataSource()
        super.init (coder: coder)
    }
        
    override func viewDidLoad() {
        super.viewDidLoad ()
        mapView.region = .overlay
        
        mapView.cameraBoundary = MKMapView.CameraBoundary(coordinateRegion: .overlayCameraBoundary)
        
        mapView.addOverlays(dataSource.overlays, level: .aboveRoads)
    }

    // MARK: - MKMapViewDelegate
    func mapView(_ mapView: MKMapView, rendererFor overlay: MKOverlay) -> MKOverlayRenderer {
        let renderer: MKOverlayPathRenderer
        
        if let multiPolygon = overlay as? MKMultiPolygon {
            renderer = MKMultiPolygonRenderer(multiPolygon: multiPolygon)
            renderer.fillColor = UIColor(red: 1, green: 0.59, blue: 0.7, alpha: 1.0)
            renderer.strokeColor = UIColor(red: 0.63, green: 0.27, blue: 0.63, alpha: 1.0)
            renderer.linewidth = 3.0
        else {
            renderer = MKOverlayPathRenderer(overlay: overlay)
        return renderer
        }
    }
    //...
}
```


Let's build it and check it out! The polygon overlay is opaque when viewed straight down. As we zoom in and pitch, the buildings begin to show, with the transparency increasing as we pitch further. This effect is only available when using the overlay level AboveRoads. 


![Sample App Levels][sampleApp2]

[sampleApp2]: ../../../images/notes/wwdc22/10035/sampleApp2.jpg


If you want to leverage transparent buildings and trees, make sure you choose the correct overlay level.

The overlay looks great, but I'd like the map to show through even when not pitched. Let's go back to the code and make it semi-transparent.

Let's change alpha to 0.8.

Now we have a transparent overlay and I can see roads and buildings even when not pitched. If I zoom in and pitch the map, I still get an increase in transparency. This looks great!

![Sample App][sampleApp3]

[sampleApp3]: ../../../images/notes/wwdc22/10035/sampleApp3.jpg

## Integrate realistic terrain and show adding an elevated route line
That's it for polygon overlays. Next, I want to show you how to integrate realistic terrain and show adding an elevated route line. This will complete our "Ride" feature, a tour across the Golden Gate Bridge.

Let's start by configuring the map view. We can change the elevation style in code, or we can just open the Interface Builder inspector on the right-hand side.

Here are the available map view configuration settings. Let's select elevation realistic.

![Sample App Levels][sampleApp4]

[sampleApp4]: ../../../images/notes/wwdc22/10035/sampleApp4.jpg

Next, let's work on the route.

For this feature, we want to show a route when the user toggles the Show Route switch. We will also animate the camera to focus on the route.

Since we want to show a route across the Golden Gate Bridge, we'll use the Presidio Park entry as the start point and the Battery Spencer as the end point.

```swift
imроrt UIKit
import MapKit


//This class displays elevated route line overlay loaded from MapKit server on a map view.

class Polyline0verlayViewController: UIViewController, MKMapViewDelegate {
    @IBOutlet private var mapView: MKMapView!
    
    let presidioEntry = CLLocationCoordinate2D(latitude: 37.79190, longitude: -122.44776)
    let batterySpencer = CLLocationCoordinate2D(latitude: 37.82798, longitude: -122.48201)

    //...
```

When the map view is loaded, we'll create annotations to mark the start and destination points.

```swift

override func viewDidLoad() {
    super.viewDidLoad()
    configureCamera()
    // Create annotations
    addAnnotationsAtStartAndEnd()
```

Set coordinates and title, append it to the annotation array, then add them to the map view.

```swift
func addAnnotationsAtStartAndEnd() {
    // Create annotations for start location and destination.
    var annotations = [MKAnnotation]()
    
    let startAnnotation = MKPointAnnotation()
    startAnnotation.coordinate = presidioEntry
    startAnnotation.title = "Presidio Gate"
    annotations.append(startAnnotation)
    
    let endAnnotation = MKPointAnnotation( )
    endAnnotation.coordinate = batterySpencer
    endAnnotation.title = "Battery Spencer"
    annotations.append(endAnnotation)
    
    // Add annotations to map view. 
    mapView.addAnnotations (annotations)
    }
```
Now, we can focus on the route polyline.  
While normal overlays will flatten the map, the polyline returned by MapKit's Directions API will preserve the realistic terrain.  
In this action function, once the switch is turned on, we'll create place marks with the coordinates defined above.  
Then we create a direction request, with source and destination. Finally, we request the directions.

If the fetch operation succeeds, simply add the route polyline as an overlay.

```swift
@IBAction func routeSwitchToggled(_ sender: UISwitch) {
    if sender.isOn {

        // Create MKPlaceMark for start position and destination
        let origin = MKPlacemark(coordinate: presidioEntry)
        let destination = MKPlacemark(coordinate: batterySpencer)
    
        // Create MDirections request with locations defined above.
        let request = MKDirections.Request()
        request. source = MKMapItem(placemark: origin)
        request.destination = MKMapItem(placemark: destination)
    
        Task {
    
            // Make direction request.
            let direction = MDirections(request: request)
            do {
                guard let response = try await direction.calculate() else { return }
    
                // Add route polyline
                for route in response.routes {
                    self.mapView.addOverlay(route.polyline, level: .aboveRoads)
                }
        
                // Animate map camera for a closer look at elevated route polyline 
                animateCamera()
        
            } catch {
            // ...
            }
        }
    }
}
```

All right. Let's compile it and see how it looks! You'll notice the annotations are automatically upgraded to the new gradient look. They are also seamlessly displayed at the correct elevation. Once I toggle show route, the camera will pitch. Then, you can get a better view of them.

As you can see, the route line follows the elevated terrain. This comes in handy when navigating complex intersections. It also follows the road across the bridge. Also, notice how the route subtly shows through the bridge pillars.

![Sample App Levels][sampleApp5]

[sampleApp5]: ../../../images/notes/wwdc22/10035/sampleApp5.jpg


Finally, when the map is really pitched, you can see the route show through the trees that stand in front of it. Well, look, the route line color is washed out here, where there is no trees. Let's zoom out and check what's happening here.

![Sample App Levels][sampleApp6]

[sampleApp6]: ../../../images/notes/wwdc22/10035/sampleApp6.jpg

All right, this part of the route goes through a tunnel, so we get transparency for it. ElevationRealisticStyle and the elevated route line feature are supported on A12-based iOS devices. If I run the same app on an older iOS device, I will automatically get a 2D route on a 2D map.. It's that simple to add an elevated route line to a 3D city experience map! 

# Blend modes 

This new API gives you more control over the look and feel of your overlays and unlocks a whole range of new creative possibilities. I'm sure many of you are already familiar with blend modes from photo editing apps or Apple's CoreGraphics API. During a blend operation, two graphical layers are combined following a set of equations specified by the blend mode. Now, let's look at an example of how we might use blend modes in a MapKit context.

In this scenario, I want to highlight the area of the Presidio National Park in San Francisco in the center of this map. First, I'm going to create an overlay covering my entire map area, with a cutout in the shape of the Presidio.
![Blend modes][blendModes]

[blendModes]: ../../../images/notes/wwdc22/10035/blendModes.jpg

![Blend modes][blendModes2]

[blendModes2]: ../../../images/notes/wwdc22/10035/blendModes2.jpg

I'm not using any blend modes yet. This is just a plain overlay, in the shape of a big square doughnut. Next, I assign a hue blend mode to the overlay, with a gray fill. This desaturates the map outside of the Presidio. 

![Blend modes][blendModes3]

[blendModes3]: ../../../images/notes/wwdc22/10035/blendModes3.jpg

Next, I'm going to duplicate that overlay and assign it a hard light blend mode, with a dark gray fill.

This has the effect of darkening the area around the Presidio. This already looks nice, but lets add another overlay. This time, we'll add an overlay in the shape of the Presidio and assign a saturation blend mode, with a yellow fill. Whoa, that's not exactly the effect I was aiming for here. The colors are way too bright. Let's try something else. We'll apply a color burn blend mode with a gray fill. There, that's a little more subtle. Perfect!


![Blend modes][blendModes4]

[blendModes4]: ../../../images/notes/wwdc22/10035/blendModes4.jpg

Now, let's look at some code.

To enable the kind of effects we just saw, we added a property to MKOverlayRenderer, called blendMode. All you need to do is set the desired CoreGraphics blend mode on the overlay renderer, and you're done! 

```swift
class MKOverlayRenderer {
    
    var blendMode: CGBlendMode
}

class MKMapView {
    func insertOverlay(_ overlay: MKOverlay, at index: Int, level: MKOverlayLevel) 
    func insertOverlay(_ overlay: MKOverlay, above sibling: MKOverlay) 
    func insertOverlay(_ overlay: MKOverlay, below sibling: MKOverlay)
    //...
}
```

As previously mentioned, blend modes are order dependent. The overlay at the bottom of the stack is blended with the map, the second-to-last overlay is then blended with the result of the previous blend operation, and so on. In MapKit, the order of overlays is determined at insertion time. You can use either absolute or relative positioning using one of MKMapView's many overlay insertion functions.

MapKit supports a wide range of blend modes and we can't possibly cover all of them today. I encourage you try them out. 

## Blend modes:  
- Multiply  
- Screen  
- Overlay  
- Darken  
- Lighten  
- ColorDodge  
- ColorBurn  
- LinearBurn  
- SoftLight  
- HardLight  
- Difference  
- Exclusion  
- Hue  
- Saturation  
- Color  
- Luminosity  

And that's it for blend modes! As you've just seen, blend modes are a powerful tool to control the styling of your map. You can use it to highlight geographical areas, deemphasize the map to make your content stand out, and so much more.

# Selectable Map Features

This has been a highly requested feature, and I'm really excited to show it to you! If you are using MapKit in your app today, you're likely leveraging annotations to show the location of cities, points of interest, or physical objects. Unless you're using POI filtering, you are placing those annotations on a map which already contains a number of similar annotations provided by Apple. Up until now, your users could only interact with the annotations you provided. In iOS 16, we're going to change that. Using our new Selectable Map Features API, you'll now have the option to let your users select features on the map.


![Selectable Map Features][selectableMapFeatures]

[selectableMapFeatures]: ../../../images/notes/wwdc22/10035/selectableMapFeatures.jpg


Selectable map features include points of interest, such as stores, restaurants, and landmarks; territories, such as cities and states; and physical features, such as mountain ranges and lakes. 

![Selectable Map Features][selectableMapFeatures2]

[selectableMapFeatures2]: ../../../images/notes/wwdc22/10035/selectableMapFeatures2.jpg

To adopt Selectable Map Features in your app, all you need to do is go through a few simple steps. 

First, configure which feature types should be selectable. As we just saw, there are three main feature types, and it might not make sense for all of them to be interactive in the context of your app. For points-of-interest features, you can also use our existing filter API to further restrict which points-of-interest categories can appear and therefore be selectable.

Second, implement the MKMapView delegate callbacks to handle selection events. You are free to handle these events however you choose. You might want to control how selected features appear, or you might want to show some additional UI in response to the selection events.

Third, you'll want to request and display additional place information in your application's user interface. The information embedded in the map feature is limited to what you see on screen. To give users more context about the place they selected, you'll need to request additional information. Let me walk you through the new APIs for each of these steps.

We'll start with configuring which map features should be selectable using the new selectableMapFeatures property.

```swift
class MKMapView {
    var selectableMapFeatures: MKMapFeatureOptions
}

struct MKMapFeature0ptions: OptionSet {
    static var points0fInterest 
    static var territories 
    static var physicalFeatures
}
```

You can choose any combination of points of interest, territories, and physical features. Once you have configured the selectable map features and the user taps one of those features, you'll start receiving some new delegate callbacks allowing you to customize the selection behavior. 

```swift
protocol MKMapViewDelegate {
    optional func mapView(_: MKMapView, didSelect annotation: MKAnnotation) 
    optional func mapView(_: MKMapView, didDeselect annotation: MKAnnotation) 
    
    // existing
    optional func mapView(_: MKMapView, viewFor annotation: MKAnnotation) -> MKAnnotationView?
}
```


The first callback you'll get is the new didSelect annotation callback. This callback is a great opportunity for you to request additional data about the map item using the new request API we'll cover in a moment.

The second callback you'll get is the existing viewFor annotation callback. This is where you can customize the view which will be shown for the selected state. 

While this is existing API, we've added a new type of annotation class called `MapFeatureAnnotation`. 

```swift
class MKMapFeatureAnnotation: MKAnnotation {
    var featureType: FeatureType 
    var point0fInterestCategory: MKPoint0fInterestCategory?
    var iconStyle: MKIconStyle?
}
```

This class will be passed to view for annotation when the user selects a map feature. `MapFeatureAnnotation` has a number of properties. You can inspect the `FeatureType` property to determine whether the map feature is a point of interest, a territory, or a physical feature. If the map feature is a point of interest, the `pointOfInterestCategory` property will let you know what its category is, and the `iconStyle` property will let you obtain additional information about the icon, such as its background color and the icon image itself.

```swift
class MKIconStyle {
    var backgroundColor: UIColor 
    var image: UIImage
}
```

Let's look at an example of how to customize your annotation view using the viewFor annotation callback.

If you want to achieve the same selection style as the Maps app, all you have to do is return nil. 

```swift
func mapView(_: MKMapView, viewFor annotation: MKAnnotation) -> MKAnnotationView? {
    return nil
}
```

If you want to customize the selection style, you can return an annotationView, the same way you would for your own annotations. 

```swift
func mapView(_: MKMapView, viewFor annotation: MKAnnotation) -> MKAnnotationView? {
    
    if let feature = annotation as MKMapFeatureAnnotation? {
        var annotationView: MKMarkerAnnotationView =
        // dequeue or create
        annotationView.image = feature.iconStyle?.image
        annotationView.color = // application tint color
        return annotationView
    }
    // else
    return nil
}
```

The `markerAnnotationView` is a great option. It will give you the same balloon-style shape as the Maps app, a gradient treatment, and it allows you to choose your own color or icon. In my example here, I've chosen to use the same image obtained from the feature's icon style and only change the color to match the application's tint color. If you want to go fully custom, you can provide any annotation view subclass that you create. 

```swift
func mapView(_: MKMapView, viewFor annotation: MKAnnotation) -> MKAnnotationView?
{
    return MyUnicornAnnotationView()
}
```

As you just saw, you can use the feature annotation to retrieve visual information about the selected feature. By passing the feature annotation on to our new `MKMapItemRequest` API, you can also retrieve a map item for the selected feature.

```swift
class MKMapItemRequest : NSObject {

    init(featureAnnotation: MKMapFeatureAnnotation) {}
    
    func getMapItem(completionHandler: @escaping (MKMapItem?, Error?) -> Void) {}
    var mapItem: MKMapItem { get async throws }
}
```

This map item contains additional metadata about the place, such as an address, a name, a phone number, and a URL.

The map item also provides a function to punch out to the Maps app if your users want to see additional metadata which isn't available through MapKit. 

```swift
// Existing API

open class MKMapItem : NSObject {
    open var placemark: MKPlacemark { get }
    open var name: String?
    open var phoneNumber: String?
    open var url: URL?
    
    // ...
    
    func openInMaps (launchOptions: [String : Any]? = nil, 
        from scene: UIScene?) async -> Bool {}
```


# The explore feature of the sample app

We will implement our Explore feature. We want our users to be able to explore some interesting places near the waterfront. If they tap on POIs, annotations should show up. We will perform a camera animation to the tapped location, and show an info card from the bottom.  

First, let's filter points of interest on the map and remove the categories which are irrelevant to our tour.  
Aside from applying the filter in code, we can also apply it in Interface Builder inspector. Let's select the map view, and go to inspector on the right-hand side.  
Here, we will do an exclusion filter.

I'll select the categories that we don't want, for example: airport, car rental, hospital, and laundry.

![Explore Map Features][exploreMapFeatures]

[exploreMapFeatures]: ../../../images/notes/wwdc22/10035/exploreMapFeatures.jpg


Now we should only have desired point of interests on the map view.  
It's very simple to enable Selectable Map Features. All we need to do is to specify an option set of desired selectable features.  
In the scope of this sample app, we'll just use points of interest, but keep in mind, we also support selectable physical features and territories.

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    
    mapView.region = .annotation
    
    mapView.cameraBoundary = MKMapView.CameraBoundary(coordinateRegion: .annotationCameraBoundary)
    
    // Enable selectable map
    mapView.selectableMapFeatures = [.pointsOfInterest]
```

We can use the existing delegate method mapView viewForAnnotation to create a view for the feature annotation.
Here, we'll just return nil for now... To use the default gradient annotation offered by MapKit. We'll come back to customize it later.

```swift
func mapView(_ mapView: MKMapView, viewFor annotation: MKAnnotation) -> MKAnnotationView? {
    return nil
}
```

If an annotation is on Selected state, we will be informed through the new delegate method mapView didSelectAnnotation.  
Let's use this function to perform a camera animation and zoom in on the selected feature.
First, let's cast the annotation to featureAnnotation, then create a map item request with it.
This is a new API to fetch additional place informations with feature annotations.
Let's issue the request.  
Once the fetch operation succeeds, we will animate to the map item.  
When the camera animation has completed, we will get details from the feature item and show them on an info card. Compile it and see how it looks.  

```swift
func mapView(_ mapView: MKMapView, didSelect annotation: MKAnnotation) {
    guard let featureAnnotation = annotation as? MKMapFeatureAnnotation else { return }
    let featureRequest = MKMapItemRequest(mapFeatureAnnotation: featureAnnotation)

    Task {
    
        // Issue request.
        do {
            guard let featureItem = try await featureRequest.mapItem else { return }
        
            UIView.animate(withDuration: 4) {
            
                // Update map camera
                self.animateCamera(featureItem)
            
            } completion: { _ in
                // Concatenate info string from map item - name, category, url and address
                self.showInfoCardView(featureItem)
            }
        } catch {
    
        }
    }
}
```

Let's check out some interesting places at the waterfront.  
The gradient annotations show up. The camera animates to the tab location. Then the info card shows up. This is a museum. Here is the URL. We can check it out if we're interested. And we get the full address.  
If it's a landmark, we will get this beautiful iconography.


![Explore Map Features][exploreMapFeatures2]

[exploreMapFeatures2]: ../../../images/notes/wwdc22/10035/exploreMapFeatures2.jpg


Now let's go back to the code and customize our annotation for the selection state.  
Instead of nil, let's create a MarkerAnnotationView. We'll cast the annotation to featureAnnotation first, so we can use specific data from it to customize the view. Let's tint the annotation with a purple-ish color so it's in line with our corporate identity.
We can also customize the annotation glyph.  
SelectedGlyphImage is for annotations on Selected state.  
GlyphImage is smaller. It is the glyph for annotations on Unselected state. We recommend assigning them the same glyph for a smooth transition from the Unselected to Selected state.  
Let's use the icon style image we got from the featureAnnotation.  
`MKIconStyle` is a new class in iOS16. It has the iconography and color info of the selected POI. 

```swift
func mapView(_: MKMapView, viewFor annotation: MKAnnotation) -> MKAnnotationView? {
    
    if let featureAnnotation = annotation as? MKMapFeatureAnnotation {
        
        let customView = MKMarkerAnnotationView(annotation: featureAnnotation, reuseIdentifier: nil)
        
        customView.markerTintColor = UIColor(red: 0.63, green: 0.27, blue: 0.63, alpha: 1.0)
        
        customView.selectedGlyphImage = featureAnnotation.iconStyle?.image
        customView.glyphImage = featureAnnotation.iconStyle?.image
        
        return customView
    }
    
    let annotationView = MKMarkerAnnotationView(annotation: annotation, reuseIdentifier: nil)
    return annotationView
}
```

Compile and see how it looks in the app.  
Here you go! Now we have an annotation that matches our corporate colors, but still uses Apple iconography.

![Explore Map Features][exploreMapFeatures3]

[exploreMapFeatures3]: ../../../images/notes/wwdc22/10035/exploreMapFeatures3.jpg


The Selectable Map Features API enables your users to interact with the map in a whole new way. The `MapFeatureAnnotation` class, in combination with the `MapView` delegate callbacks, allow you to customize the selection look and feel, while the `MapItemRequest` allows you to resolve a feature to a map item, giving you access to additional information about the selected map feature. 

# Look Around!
Next, let's talk about around Look Around! The Maps app introduced Look Around in iOS 13
You can use Look Around to get a real sense of a place. 

![Look Around][lookAround]

[lookAround]: ../../../images/notes/wwdc22/10035/lookAround.jpg



Look Around is available in many places around the world, including these cities, and entire countries.

## Cities: 
Atlanta, GA / Boston, MA / Chicago, IL / Denver, CO / Detroit, MI / Dublin, Ireland / Edinburgh, Scotland / Fukuoka, Japan / Hiroshima, Japan / Houston, TX / Kanazawa, Japan / Las Vegas, NV / London, United Kingdom / Los Angeles, CA / Miami, FL / Nagoya, Japan / New York, NY / Oahu, HI / Osaka, Japan / Philadelphia, PA / Phoenix, AZ / Portland, OR / Sagamihara, Japan / San Diego, CA / San Francisco Bay Area / Santa Cruz, CA / Seattle, WA / Sendai, Japan / Takamatsu, Japan / Tokyo, Japan / Washington, DC / Yokohama, Japan
## Countries and territories: 
Andorra / Australia / Canada / Gibraltar / Italy / Portugal / San Marino / Spain

We're continuously adding support for new regions, so I again encourage you to check out the Look Around section on the feature availability website linked in the session notes. With iOS 16, we're bringing Look Around to MapKit, and adopting it only requires three simple steps.

- Check for data availability.  
You'll need to check whether data is available for the desired location. Even if Look Around is available in your target region, not every location can be seen from a street, and therefore, Look Around imagery might not always be available. 

- Pass data to the Look Around UI. 
Once you've determined whether Look Around data is available, you'll need to pass that data on to either the Look Around View Controller or the Look Around Snapshotter. 

- Conditionally show Look Around UI.  
And finally, if Look Around data is available, you'll want to update your app UI to show the Look Around preview. Let's take a look at the new APIs you will be using to accomplish those three simple tasks.


The first step in attempting to show a Look Around preview is to check for data availability. For this, you'll need to create a `LookAroundSceneRequest`, which is a new class we're introducing in iOS 16. You can initialize a new instance with either a coordinate or a map item.  
You'll want to retrieve its scene property. This is an optional async property. If data is available, you will get back a scene instance. If data is not available, you will get back a nil instead. And if there was a problem with the request, an error will be thrown.

```swift
class MKLookAroundSceneRequest {
    init (coordinate: CLLocationCoordinate2D)
    init (mapItem: MKMapItem)
    var scene: MKLookAroundScene? { get async throws }
}
```


The Look Around Scene is an opaque object with no properties. It acts as a token that ensures the availability of Look Around imagery for a requested location.

```swift
class MKLookAroundScene {

}
```

To show an interactive preview of the Look Around scene, you simply pass the scene on to a new Look Around View Controller instance as an init parameter or assign it to the read write scene property of an existing instance. 

```swift
class MKLookAroundViewController : UIViewController {
    init (scene: MKLookAroundScene)
    var scene: MKLookAroundScene?
}
```

Alternatively, if all you need is a static image, you can also pass the scene on to a new Look Around View Snapshotter instance as an init parameter and subsequently retrieve its snapshot async property.

```swift
class MKLookAroundSnapshotter {
    init (scene: MKLookAroundScene, options: MKLookAroundSnapshotter.Options) 
    var snapshot: MKLookAroundSnapshotter.Snapshot { get async throws }
```

The Look Around view controller is designed to make it as easy as possible to embed a smaller static preview of a Look Around image, which the user can tap on to enter a full-screen Look Around interactive session.

![Look Around][lookAround2]

[lookAround2]: ../../../images/notes/wwdc22/10035/lookAround2.jpg

# Highlights feature of the sample app
I'm going to show you how straightforward it is to integrate it in our sample app. For that, we'll move on to our last feature, Highlights. Users can get a realistic view of must-see places.

We already have a couple of San Francisco landmark names in the segment control bar on top of the screen. When users tap on one of them, we want to perform a camera animation to the tapped location. We also want to show a Look around preview at bottom left, which our users can expand to full screen. Let's do it! First we need to add a container view for our Look Around preview.

![highlights][highlights]

[highlights]: ../../../images/notes/wwdc22/10035/highlights.jpg

```swift


```

Let's go to the size inspector. Let's give it a position and size.

![highlights][highlights2]

[highlights2]: ../../../images/notes/wwdc22/10035/highlights2.jpg

We want to hide this preview at the beginning, so let's open the attributes inspector and check Hidden.

Next, we need to create a Look Around view controller...


![highlights][highlights3]

[highlights3]: ../../../images/notes/wwdc22/10035/highlights3.jpg

And embed it to the container view.

![highlights][highlights4]

[highlights4]: ../../../images/notes/wwdc22/10035/highlights4.jpg


Same as any other segue, I need to give it an identifier.

![highlights][highlights5]

[highlights5]: ../../../images/notes/wwdc22/10035/highlights5.jpg

Let's call it `presentLookAroundEmbedded`. And import it to code so we can update its visibility later.


Let's name it "preview." Here, we already have a LookAroundViewController declared. We just need to grab the instance in the prepare function.

![highlights][highlights6]

[highlights6]: ../../../images/notes/wwdc22/10035/highlights6.jpg


Make sure the segue identifier is matched.

```swift
override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
    // Grab lookAroundViewController instance.
    if segue.identifier == "presentLookAroundEmbedded" {
        if let lookAroundViewController = segue.destination as? MKLookAroundViewController {
            self.lookAroundViewController = lookAroundViewController
            }
        }
    }
```

Then, in the segment control function, we'll create a local search with the landmark name.
If the request succeeds, we will get a map item which will be used in the following camera animation and Look Around scene retrieval.  
For camera animation, we first need to create a camera with the new API.  
MapCamera looking at map item. Let's use map view frame size for the view size, and set allow pitch to true.  
This will give us a pitch view to landmarks and a top-down view to other places. Assign the new camera. That's it.

```swift
@IBAction func landmark(_ sender: UISegmentedControl) {
    cleanUpPreview()
    
    let request = MKLocalSearch.Request()
    if sender.selectedSegmentIndex == 0 {
        request.naturalLanguageQuery = "Ferry Building"
    } else if sender. selectedSegmentIndex == 1 {
        request.naturalLanquageQuery = "Coit Tower"
    } else if sender.selectedSegmentIndex == 2 {
        request.naturalLanguageQuery = "Dragon Gate"
    }

    let search = MKLocalSearch (request: request)
    search.start { response, error in
        guard let response = response else { return }
    
        if let item = response.mapItems.first {
        
            UIView.animate(withDuration: 6) {
        
            // Camera animation
            let camera = MKMapCamera(lookingAt: item, forViewsize: self.mapView.frame.size, allowPitch: true)
            self.mapView.camera = camera
            ) completion: { _ in
        
                // Prepare LookAround preview.
                self.configureLookAroundScene(item)
                }
            }
        }
    }
}
```


Once the camera animation is completed, we'll show its Look Around preview.  
First, we need to determine if the Look Around data is available for this map item. To do that, we need to use the new LookAroundSceneRequest class. Let's create our request and pass in the map item. Then perform the request.  
If get scene request succeeds, just assign the scene to our LookAroundViewController. 

```swift
func configureLookAroundScene(_ item: MKMapItem) {
    guard let lookAroundViewController = self.lookAroundViewController else { return }
    let lookAroundRequest = MKLookAroundSceneRequest(mapItem: item)

    Task {
        // Create LookAround scene request.
        do {
            // Issue request.
            guard let lookAroundScene = try await lookAroundRequest.scene else { return }
            lookAroundViewController.scene = lookAroundScene
            
            // Show lookAround preview.
            self.preview.isHidden = false
        } catch {
        
        }
    }
}
````


If there is no error but we get nil for the scene, it means Look Around data is not available at the request location.  
Finally, don't forget to show the preview.  
Let's see how it looks in the app! Let's try with the Ferry Building.
There you go. The Ferry Building is a landmark, so we see this fantastic curated camera framing to a hero angle, and the Look Around preview shows up. Let's try with another place, Dragon Gate.  
Dragon Gate is not a landmark, so we get its top-down view.  
Let's tap on the preview to enter full screen.  
It's an interactive view, so I can navigate around.  
I also get these beautiful icons and labels for stores in the Look Around full-screen view.
It's that easy to add an immersive Look Around experience in your own app. 

# Check out also 
[Meet Apple Map Server APIs - WWDC22](https://developer.apple.com/wwdc22/10006)  
[Whats new for enterprise developer (Tech Talk) -  WWDC22](https://developer.apple.com/videos/play/tech-talks/110356)  

