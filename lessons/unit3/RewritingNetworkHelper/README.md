# Re-writing Network Helper

# Objectives

- Rewrite the Network Helper class
- Use coding keys to change the name/organization of custom class properties

# Resources
- [URLSession](https://developer.apple.com/documentation/foundation/urlsession)
- [Medium - Coding Keys](https://medium.com/@sarunw/codable-in-swift-4-0-1a12e38599d8)


# 1. URLSession

A URLSession is a Swift class that facilitates communicating with the internet.  Instantiating a URLSession requires a configuration.  There are four main configuration options:

(from: https://developer.apple.com/documentation/foundation/urlsession)

- The singleton shared session (which has no configuration object) is for basic requests. It’s not as customizable as sessions that you create, but it serves as a good starting point if you have very limited requirements. You access this session by calling the shared class method.
- Default sessions behave much like the shared session (unless you customize them further), but let you obtain data incrementally using a delegate. You can create a default session configuration by calling the default method on the URLSessionConfiguration class.
- Ephemeral sessions are similar to default sessions, but they don’t write caches, cookies, or credentials to disk. You can create an ephemeral session configuration by calling the ephemeral method on the URLSessionConfiguration class.
- Background sessions let you perform uploads and downloads of content in the background while your app isn’t running. You can create a background session configuration by calling the backgroundSessionConfiguration(_:) method on the URLSessionConfiguration class.


### [dataTask](https://developer.apple.com/documentation/foundation/urlsession/1410330-datatask)

```swift
func dataTask(with url: URL, 
completionHandler: @escaping (Data?, URLResponse?, Error?) -> Void) -> URLSessionDataTask
```

When we call this function, we give it a URL and a closure that takes Data? a URLResponse? (e.g 200 OK) and an Error? as input.


# 2. Using URLSession

We can use a URLSession directly to create a network connection inside our app.  Let's return to our NASA app and use URLSession to load the appropriate image for each day.

```swift
    func loadObjectForSelectedDate() {
        let selectedDate = datePicker.date.description
        let formattedYMD = selectedDate.components(separatedBy: " ")[0]
        guard let url = URL(string: "https://api.nasa.gov/planetary/apod?date=\(formattedYMD)&api_key=DEMO_KEY") else {return}
        spinner.isHidden = false
        spinner.startAnimating()
        let mySession = URLSession(configuration: .default)
        let myTask = mySession.dataTask(with: url){(data: Data?, response: URLResponse?, error: Error?) in
            guard let data = data else {return}
            if let error = error {
                print("error: \(error)")
            }
            do {
                let nasaObject = try JSONDecoder().decode(NASAObject.self, from: data)
                DispatchQueue.main.async {
                    self.nasaObject = nasaObject
                }
            }
            catch {
                print(error)
            }
        }
        myTask.resume()
    }
```

# 3. NetworkHelper

We now have all the pieces to understand how our NetworkHelper is created.

```
class NetworkHelper {
	//Make it so we can't make NetworkHelpers outside this class
	private init() {}

	//Create a class property that we will use to get to instance methods
	static let manager = NetworkHelper()
	
	//Create a default URLSession
    private let urlSession = URLSession(configuration: URLSessionConfiguration.default)

	//Give the manager an instance method that takes a URL, completion handler and error handler.  After getting data from the URL, it runs the completion handler.
	func performDataTask(with url: URL, completionHandler: (Data) -> Void, errorHandler: (Error) -> Void) {
		//Create a dataTask
		self.urlSession.dataTask(with: url){(data: Data?, response: URLResponse?, error: Error?) in
			guard let data = data else {return} //Ensure the data is valid
			
			//Handle any errors
			if let error = error {
				errorHandler(error)
			}
			
			//Input the data into the completion handler
			completionHandler(data)
			
		//resume() starts the data task.  Without out, our data task will not run.
		}.resume()
	}
}
```


# 4. Coding Keys

Sometimes when we get data from online, the names of the keys don't match the preferred Swift formatting.  We can use coding keys to tell Swift that we want different names for our properties.

[Universities API](http://universities.hipolabs.com/search?name=middle)