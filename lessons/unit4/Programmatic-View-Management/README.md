## Programmable View Management Continued

## Objectives 
* Create a TableView and its custom cell in code 
* Create a Custom View and add it to a View Controller
* Create UIBarButtonItems in code add add to the Navigation Bar
* Programmable Auto Layout using LayoutAnchors
* Understand frame and bounds
* Dependency Injection using the View Controller's initializer in code  
* Get a Storyboard instance in code and instantiate a View Controller 
* Create a CollectionView in code along with its default cell and layout

## UIWindow
Normally, Xcode provides your app's main window. New iOS projects use storyboards to define the app’s views. Storyboards require the presence of a window property on the app delegate object, which the Xcode templates automatically provide. 
If your app does not use storyboards, you must create this window yourself.

## AppDelegate Setup for Programmable Views 
```swift 
func application(_ application: UIApplication, 
didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
    // Create a Navigation Controller
    // Same as when we embed a View Controller in a Navigation Controller in Storyboard
    let navController = UINavigationController()

    // Set the View Hierarchy
    // In this case it's a Navigation Controller with a root View Controller
    let mainVC = MainViewController()
    navController.viewControllers = [mainVC]

    // Set the Main Window of the app
    window = UIWindow(frame: UIScreen.main.bounds)
    window?.rootViewController = navController
    window?.makeKeyAndVisible()

    return true
}
```

## Frame vs Bounds 
The [frame](https://developer.apple.com/documentation/uikit/uiview/1622621-frame) of an UIView is the rectangle, expressed as a location (x,y) and size (width,height) relative to the superview it is contained within.

The [bounds](https://developer.apple.com/documentation/uikit/uiview/1622580-bounds) of an UIView is the rectangle, expressed as a location (x,y) and size (width,height) relative to its own coordinate system (0,0).

[What's the difference between the frame and the bounds?](https://stackoverflow.com/questions/1210047/cocoa-whats-the-difference-between-the-frame-and-the-bounds)  

```swift 
private func setupSquare() {
    // frame base layout
    let length: CGFloat = UIScreen.main.bounds.width / 2
    square = UIView(frame: CGRect(x: 0, y: 80, width: length, height: length))
    square.backgroundColor = .blue
    addSubview(square)
}
```

<p align="center">
<img src="https://github.com/C4Q/AC-iOS/blob/master/lessons/unit4/Programmatic-View-Management/Images/frame-vs-bounds.png" width="414" height="736" />
</p>

## Subclassing and Creating a Custom UIView
```swift 
class MainView: UIView {

    // here best practices is to create your views by lazy instantiation
    // the view is not created until it's needed - leads to better processing time 
    lazy var tableView: UITableView = {
        let tv = UITableView()
        tv.frame = bounds // bounds here is the MainView's bounds which is UIScreen.main.bounds (entire screen)
        tv.register(UITableViewCell.self, forCellReuseIdentifier: "NameCell")
        return tv
    }()

    override init(frame: CGRect) {
        super.init(frame: UIScreen.main.bounds)
        commonInit()
    }
    
    required init?(coder aDecoder: NSCoder) {
        super.init(coder: aDecoder)
        commonInit()
    }
    
    private func commonInit() {
        setupViews()
    }
    
    private func setupViews() {
        addSubview(tableView)
    }
}
```

## Creating a TableView and Registering its cell in code 
```swift
lazy var tableView: UITableView = {
    let tv = UITableView()
    tv.frame = bounds // bounds here is the MainView's bounds which is UIScreen.main.bounds (entire screen)
    tv.register(FellowCell.self, forCellReuseIdentifier: "FellowCell")
    return tv
}()
```

## Programmable Constraints
You have three choices when it comes to programmatically creating constraints: You can use layout anchors, you can use the NSLayoutConstraint class, or you can use the Visual Format Language.

## Autolayout with Layout Anchors
```translatesAutoresizingMaskIntoConstraints```
If you want to use Auto Layout to dynamically calculate the size and position of your view, you must set this property to false, and then provide a non ambiguous, nonconflicting set of constraints for the view.
```swift 
private func setupViews() {
    // programmable autolayout

    // profile view
    addSubview(profileView)
    profileView.translatesAutoresizingMaskIntoConstraints = false
    profileView.centerXAnchor.constraint(equalTo: centerXAnchor).isActive = true
    profileView.topAnchor.constraint(equalTo: topAnchor, constant: 100).isActive = true
    profileView.widthAnchor.constraint(equalTo: widthAnchor, multiplier: 0.5).isActive = true
    profileView.heightAnchor.constraint(equalTo: profileView.widthAnchor).isActive = true

    // name label
    addSubview(profileName)
    profileName.translatesAutoresizingMaskIntoConstraints = false 
    profileName.centerXAnchor.constraint(equalTo: profileView.centerXAnchor).isActive = true
    profileName.centerYAnchor.constraint(equalTo: profileView.centerYAnchor).isActive = true
}
```

## Adding a view to your View Controller
```swift
let mainView = MainView()

override func viewDidLoad() {
    super.viewDidLoad()
    configureNavBar()
    mainView.tableView.dataSource = self
    mainView.tableView.delegate = self 
    view.addSubview(mainView)
}
```

## Adding a Blur Effect to a View 
```swift 
private func setupBlurEffectView() {
    let blurEffect = UIBlurEffect(style: UIBlurEffectStyle.dark) // .light, .dark, .prominent, .regular, .extraLight
    let visualEffect = UIVisualEffectView(frame: UIScreen.main.bounds)
    visualEffect.effect = blurEffect
    addSubview(visualEffect)
}
```

## Creating a UIBarButtonItem in code 
```swift 
let shuffleBarButtonItem = UIBarButtonItem(title: "Shuffle", style: .plain, target: self, action: #selector(shuffleFellows))
navigationItem.rightBarButtonItem = shuffleBarButtonItem
```

## Presenting a View Controller Modally over the Current Context
```swift 
func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
    let fellow = fellows[indexPath.row]
    let detailVC = DetailViewController(fellow: fellow)
    detailVC.modalPresentationStyle = .overCurrentContext
    detailVC.modalTransitionStyle = .crossDissolve
    navigationController?.present(detailVC, animated: true, completion: nil)
}
```

## View Controller with Custom Initializer 
```swift 
class DetailViewController: UIViewController {
    let detailView = DetailView()

    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .white
        view.addSubview(detailView)
    }
    
    // Dependency Injection in View Controllers with custom initializers
    // This forces the required properties to be injected by the custom initializer
    // Avoids possible silent injection in prepare: for segue if a conditional optional is used
    // A strong reason why some developers prefer programmable view controllers over storyboard
    init(name: String) {
        super.init(nibName: nil, bundle: nil)
        navigationItem.title = name
        detailView.configureView(name: name)
    }
    
    required init?(coder aDecoder: NSCoder) {
        super.init(coder: aDecoder)
    }
}
```

## Utility Functions / Classes for in class Demo
<details>
<summary>ImageCache</summary>
    
```swift 
class ImageCache {
    private init(){
        
    }
    static let manager = ImageCache()
    
    private var sharedCached = NSCache<NSString, UIImage>()
    
    // get current cached image
    func cachedImage(url: URL) -> UIImage? {
        return sharedCached.object(forKey: url.absoluteString as NSString)
    }
    
    // process image and store in cache
    func processImageInBackground(imageURL: URL, completion: @escaping(Error?, UIImage?) -> Void) {
        DispatchQueue.global().async {
            do {
                let imageData = try Data.init(contentsOf: imageURL)
                let image  = UIImage.init(data: imageData)
                
                // store image in cache
                if let image = image {
                    self.sharedCached.setObject(image, forKey: imageURL.absoluteString as NSString)
                }
                
                completion(nil, image)
            } catch {
                print("ImageCache - image processing error: \(error.localizedDescription)")
                completion(error, nil)
            }
        }
    }
}
```

</details>

<details>
<summary>JSONParsingService</summary>
    
```swift 
class JSONParsingService {
    static func parseJSONFile(filename: String, type: String) -> [Fellow]? {
        var fellows: [Fellow]? = nil
        if let pathname = Bundle.main.path(forResource: filename, ofType: type) {
            guard let jsonData = FileManager.default.contents(atPath: pathname) else { return nil }
            do {
                let decoder = JSONDecoder()
                fellows = try decoder.decode([Fellow].self, from: jsonData)
            } catch {
                print("read json error: \(error.localizedDescription)")
            }
        }
        return fellows
    }
}
```

</details>

<details>
<summary>Fellow Model Object</summary>

```swift 
struct Fellow: Codable {
    let name: String
    let imageURL: URL?
    let bio: String? 
}
```

</details>

<details>
<summary>Shuffle Array</summary>
    
```swift 
@objc func shuffleFellows() {
    fellows = GKRandomSource.sharedRandom().arrayByShufflingObjects(in: fellows) as! [Fellow]
    mainView.tableView.reloadData()
}
```

</details>

<details>
<summary>Present View Modally Over Current Context</summary>

```swift 
func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
    let cell = tableView.cellForRow(at: indexPath) as! FellowCell
    let fellow = fellows[indexPath.row]
    let detailVC = DetailViewController(fellow: fellow, image: cell.profileImage.image)
    detailVC.modalPresentationStyle = .overCurrentContext
    detailVC.modalTransitionStyle = .crossDissolve
    present(detailVC, animated: true, completion: nil)
}
```

</details>

<details>
<summary>Scroll to top of TextView</summary>
    
```swift 
override func viewWillLayoutSubviews() {
    super.viewWillLayoutSubviews()
    // scroll to top of text view
    let range = NSRangeFromString(detailView.bioTextView.text)
    detailView.bioTextView.scrollRangeToVisible(range)
}
```

</details>

<details>
<summary>View Controller Dependency Injection in code</summary>
    
```swift 
// Dependency Injection in View Controllers with custom initializers
// This forces the required properties to be injected by the custom initializer
// Avoids possible silent injection in prepare: for segue if a conditional optional is used
// A strong reason some developers prefer programmable view controllers over storyboard
init(fellow: Fellow, image: UIImage?) {
    super.init(nibName: nil, bundle: nil)
    navigationItem.title = fellow.name
    detailView.configureView(fellow: fellow, image: image)
}
```

</details>

<details>
<summary>Create a TableView and register its Cell</summary>
    
```swift 
// here best practices is to create your views by lazy instantiation
// the view is not created until it's needed leads to better processing time
lazy var tableView: UITableView = {
    let tv = UITableView()
    tv.frame = bounds // bounds here is the MainView's bounds which is UIScreen.main.bounds (entire screen)
    tv.register(FellowCell.self, forCellReuseIdentifier: "FellowCell")
    return tv
}()
```

</details>

<details>
<summary>Add Blur Effect to View</summary>
    
```swift 
private func setupBlurEffectView() {
    let blurEffect = UIBlurEffect(style: UIBlurEffectStyle.dark) // .light, .dark, .prominent, .regular, .extraLight
    let visualEffect = UIVisualEffectView(frame: UIScreen.main.bounds)
    visualEffect.effect = blurEffect
    addSubview(visualEffect)
}
```

</details>

<details>
<summary>Getting a Storyboard and instantiating a View Controller using the Identifier</summary>
    
```swift 
// Get the MainViewController from the "Main" Storyboard and instantiate it
// We set bundle to nil since were looking in the current application
let storyboard = UIStoryboard.init(name: "Main", bundle: nil)
let mainViewController = storyboard.instantiateViewController(withIdentifier: "MainViewController") as! MainViewController
```

</details>


<details>
<summary>Setting up a Collection View in code along with its cell and layout</summary>
    
```swift 
lazy var collectionView: UICollectionView = {
    let layout = UICollectionViewFlowLayout()
    let cv = UICollectionView(frame: self.view.bounds, collectionViewLayout: layout)
    cv.backgroundColor = .white
    cv.register(UICollectionViewCell.self, forCellWithReuseIdentifier: "CollectionCell")
    return cv
}()
```

</details>

## Exercises
## Use LayoutAnchors to achive the following:
**Exercise 1:**
<p align="center">
<img src="https://github.com/C4Q/AC-iOS/blob/master/lessons/unit4/Programmatic-View-Management/Images/exercise-1.png" width="414" height="736" />
</p>

**Exercise 2:**
<p align="center">
<img src="https://github.com/C4Q/AC-iOS/blob/master/lessons/unit4/Programmatic-View-Management/Images/exercise-2.png" width="414" height="736" />
</p>
  
## Resources 
[UIWindow](https://developer.apple.com/documentation/uikit/uiwindow)  
[Programmatically Creating Constraints](https://developer.apple.com/library/content/documentation/UserExperience/Conceptual/AutolayoutPG/ProgrammaticallyCreatingConstraints.html)  
[What are lazy variables](https://www.hackingwithswift.com/example-code/language/what-are-lazy-variables)  
[Dependency Injection in View Controllers](https://medium.com/ios-os-x-development/dependency-injection-in-view-controllers-9fd7d2c77e55)  
[iOS: View Controller Data Injection with Storyboards and Segues in Swift](https://www.natashatherobot.com/ios-view-controller-data-injection-with-storyboards-and-segues-in-swift/)  
[View and Window Architecture](https://developer.apple.com/library/content/documentation/WindowsViews/Conceptual/ViewPG_iPhoneOS/WindowsandViews/WindowsandViews.html)  
[View Hierarchy](https://developer.apple.com/library/content/documentation/General/Conceptual/Devpedia-CocoaApp/View%20Hierarchy.html)  
[How to change Auto Layout constraints after they are set when using constraintEqualToAnchor()?](https://stackoverflow.com/questions/34391732/how-to-change-auto-layout-constraints-after-they-are-set-when-using-constrainteq)
[safeAreaLayoutGuide](https://developer.apple.com/documentation/uikit/uiview/2891102-safearealayoutguide)  
[NSLayoutAnchor](https://developer.apple.com/documentation/uikit/nslayoutanchor)  
[Randomization](https://developer.apple.com/library/content/documentation/General/Conceptual/GameplayKit_Guide/RandomSources.html)  
[Mobile design 101: pixels, points and resolutions](http://blog.fluidui.com/designing-for-mobile-101-pixels-points-and-resolutions/)  
