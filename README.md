# How to use weak self in Swift
https://www.youtube.com/watch?v=TPHp9kR0Go8

 * Often overlooked, some courses don't even touch on it
 * Helps to understand Automatic Reference Counting
 * Everytime you make an object or (screen!), they system is holding a reference to that object
 * The system does a lot of cleanup (init/deinit) itself behind the scenes
 * But, sometimes WE write code that doesn't allow a screen to deinitialize when we want b/c we created a strong reference to it
 * Especially important when usign time-intensive network requests
 
 ```swift
 class WeakSelfSecondScreenViewModel: ObservableObject {
    
    @Published var data: String? = nil
    
    init() {
        print("INITIALIZE NOW")
        let currentCount = UserDefaults.standard.integer(forKey: "count")
        UserDefaults.standard.set(currentCount + 1, forKey: "count")
        getData()
    }
    
    deinit {
        print("DEINITIALIZE NOW")
        let currentCount = UserDefaults.standard.integer(forKey: "count")
        UserDefaults.standard.set(currentCount - 1, forKey: "count")
    }
    
    func getData() {
        // First part of tutorial before simulating network request
        // ...just so we can see the screen and count working
        // Count stays at 1 even when navigating back and forth
        // Immediately inits a new and deinits the old one
        // Everything's good!
        self.data = "Instantaneous data for UI testing!"
        
        // THE PROBLEM:
        // When running asynchronous tasks!
        // So, simulating a long background task...
        DispatchQueue.main.asyncAfter(deadline: .now() + 20) {
            self.data = "New data after long request."
            /* HAD TO add self: 'Reference to property in closure requires explicit use of 'self' to make...
             self.data
             - creates a strong reference to this class, WeakSelfSecondScreenViewModel
             - tells the system, that while these tasks are running, this class, this self absolutely needs to stay alive b/c we need that self when we come back
             - user could navigate away and come back over and over
             - deinit doesn't get called until closure completes
             - until it completes, this self will stay alive
             - now you have multiple instances of the WeakSelfSecondScreenViewModel class
             */
        }
        
        // THE FIX - create a weak reference instead - [weak self]
        DispatchQueue.main.asyncAfter(deadline: .now() + 20) { [weak self] in
            // Adding [weak self] makes self Optional
            self?.data = "New data after long request."
        }
    }
}
```
