---
layout: post
title:  "Using Swift 5 Result Type With Alamofire 4"
date:   2019-04-26
image:  swift-vs-alamofire.png
---

`TL;DR Try Swift.Result`

<span class="dropcap">I</span>f you are anything like me the second thing after downloading Xcode 10.2 for you would be to try out the new Result type. For me that meant changing this:

```swift
enum Result<T> {
  case success(T)
  case failure(Error)
}

protocol APIClientType {
  func fetchCities(completion: @escaping (Result<Data>) -> Void)
  func fetchTheaters(for cityId: Int, completion: @escaping (Result<Data>) -> Void)
  func fetchMovies(completion: @escaping (Result<Data>) -> Void)
  func fetchScreenings(for theaterId: Int, completion: @escaping (Result<Data>) -> Void)
}
```
to this:

```swift
protocol APIClientType {
  func fetchCities(completion: @escaping (Result<Data, Error>) -> Void)
  func fetchTheaters(for cityId: Int, completion: @escaping (Result<Data, Error>) -> Void)
  func fetchMovies(completion: @escaping (Result<Data, Error>) -> Void)
  func fetchScreenings(for theaterId: Int, completion: @escaping (Result<Data, Error>) -> Void)
}
```

So far so good. `⌘-B`
Nope.

<img src="{{ '/assets/img/conform-protocol-error.png' | prepend: site.baseurl }}" alt="">

Okay, that's easy. I'll just do the same changes in the implementation so my struct conforms to the protocol.
Nope.

<img src="{{ '/assets/img/too-many-type-parameters-error.png' | prepend: site.baseurl }}" alt="">

Wait, what? [Paul Hudson says](https://www.hackingwithswift.com/articles/161/how-to-use-result-in-swift) that this is the correct way to use the Result type. `⌘-click` on the `Result` to investigate further...

<img src="{{ '/assets/img/alamofire-result-type.png' | prepend: site.baseurl }}" alt="">

Ah, I see what's happening here. Alamofire's implementation takes precedence over Swift 5's one. Interesting. Quick googling produced no results, so I started experimenting. First thing I tried was to try with Swift.Result instead of just Result. Funny enough it worked. Problem solved. The final code looks like this:

```swift
  func fetchMovies(completion: @escaping (Swift.Result<Data, Error>) -> Void) {
    let url = endpointUrl(for: .movies)
    let headers = [
      "Authorization": authHeader(),
      "Accept": "application/json"
    ]
    sessionManager.request(url, headers: headers).responseData { response in
      guard let data = response.data else {
        completion(.failure(FetchError.missingData))
        return
      }
      completion(.success(data))
    }
  }
```

No, there's no twist. It's Friday evening after all. Go enjoy your weekend.