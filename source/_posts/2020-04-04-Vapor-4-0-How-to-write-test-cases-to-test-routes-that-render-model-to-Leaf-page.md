title: >-
  [Vapor 4.0]How to write test cases to test routes that render model to Leaf
  page
date: 2020-04-04 22:45:06

Categories: Vapor4.0
---

You want to write test cases for every route right? Yes, I want too. But today, I met a question - How to write test cases to test routes that render model to Leaf page?

We all know testing html is complicated, I want to test the model which will be rendered to the html page, but how?

Let's see the answer.

First, we need to write our `ViewRender`, we want to use the custom ViewRender to get the model before it is rendered to html page.

```swift
import Vapor
class CapturingViewRenderer: ViewRenderer {
    var shouldCache = false
    var eventLoop: EventLoop
    init(eventLoop: EventLoop) {
        self.eventLoop = eventLoop
    }
    
    func `for`(_ request: Request) -> ViewRenderer {
        return self
    }

    private(set) var capturedContext: Encodable?
    private(set) var templatePath: String?
    func render<E>(_ name: String, _ context: E) -> EventLoopFuture<View> where E : Encodable {
        self.capturedContext = context
        self.templatePath = name
        let string = "I just want to get the context and the templatePath, I don't care what the view will look like"
        var byteBuffer = ByteBufferAllocator().buffer(capacity: string.count)
        byteBuffer.writeString(string)
        let view = View(data: byteBuffer)
        return eventLoop.future(view)
    }
}
```

You see, we get the rendering destination `templatePath`, and the context `capturedContext`. Maybe you don't know what context is, just see the route hander, you will undertand.

```swift
func getArticlesHandler(req: Request) throws -> EventLoopFuture<View> {
        return Article.query(on: req.db).all().flatMap {
            return req.view.render("Home",  HomeContext(articles: $0))
        }
    }
```

The HomeContext is the Context the Leaf framework needs, it wraps our model.

Second, we need to register the ViewRender to our app.

```Swift
final class ArticleRoutesTests: XCTestCase {
    var app: Application!
    let url = "/articles"
    let articleName = "Swift"
    let articleBody = "This is my first post"
    var capturingViewRender: CapturingViewRenderer!
    
    override func setUpWithError() throws {
        app = Application(.testing)
        try configure(app)
        capturingViewRender = CapturingViewRenderer(eventLoop: app.eventLoopGroup.next())
        app.views.use {_ in self.capturingViewRender }
    }
		...
}
```

Third, write the test case to compare the route path and model.

```Swift
  func testGetArticles() throws {
        try Utils.cleanDatabase(db: app.db)
        let article = Article(name: articleName, body: articleBody)
        try article.save(on: app.db).wait()
        try app.test(.GET, url) { res in
            let context = try XCTUnwrap(capturingViewRender.capturedContext as? ArticleController.HomeContext)
            XCTAssert(capturingViewRender.templatePath == "Home", "render to wrong page")
            XCTAssertTrue(context.articles?.count == 1, "GET /articles api failed.")
            XCTAssertTrue(context.articles?.first?.name == articleName, "GET /articles api failed.")
            XCTAssertTrue(context.articles?.first?.body == articleBody, "GET /articles api failed.")
        }
    }
```

Done! Enjoy testing.

Thanks to [@0xTim](https://twitter.com/0xTim) for discussion about it.