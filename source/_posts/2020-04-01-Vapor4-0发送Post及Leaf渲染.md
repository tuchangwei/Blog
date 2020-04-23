title: Vapor4.0发送Post及Leaf渲染
date: 2020-04-01 22:12:36
tags:
Categories: Vapor4.0
---

今天填了两个坑。

1,  Post 创建文章。`Article` database Model结构为:

```swift
final class Article: Model, Content {
    static var schema: String = "articles"
    static var idKey: FieldKey = .id
    static var nameKey: FieldKey = "name"
    static var slugKey: FieldKey = "slug"
    static var bodyKey: FieldKey = "body"
    static var categoryIDKey: FieldKey = "category_id"
    static var createdKey: FieldKey = "created_at"
    static var updatedKey: FieldKey = "updated_at"
    
    @ID(key: idKey)
    var id: UUID?
    
    @Field(key: nameKey)
    var name: String
    
    @Field(key: slugKey)
    var slug: String?
    
    @Field(key: bodyKey)
    var body: String
   
    @OptionalParent(key: categoryIDKey)
    var category: ArticleCategory?
    
    @Timestamp(key: createdKey, on: .create)
    var createdAt: Date?
    
    @Timestamp(key: updatedKey, on: .update)
    var updatedAt: Date?
    
    init() {
    }
    init(name: String, body:String, categoryID: UUID? = nil) {
        self.name = name
        self.body = body
        self.$category.id = categoryID
        self.slug = "\(Date().formatedString())\(name.split(separator: " ").joined(separator: "-"))"
    }
    
}
```

在`Article` database model中， 我创建了一个父关系`category`, 但这个项目中，我并不打算让每篇文章都有一个类别，所以我把父关系设置为可选类型。但是post的时候并不能把`category`这个字段省略，请求的格式必须这样：

```json
{
  "body": "Hello, my first blog.",
  "category": {},//或者 "category": { "id": null },
  "name": "my first blog"
}
```

是不是很不合理？前端拿到这个API会比较迷惑，组建json的时候，这个字段没有值，不能不写这个key-value pair, 也不能写成`category:null` 。社区中大神给出的方案是用一个不含`category`字段的结构体先接收，然后再转成`database mode`的样子。具体解释请参考[这里](https://github.com/vapor/vapor/issues/2280)

2, Leaf渲染。Vapor4的文档还没有出，Leaf的定义也与Vapor3有很大的改变，这就难为死我了。这种情况下，只能去扒一扒Test里面写的测试用例了。坑如下：

```leaf
<p>#(title)</p>
#if(!articles.isEmpty):
    <p>#(title)</p>
    #for(article in articles):
        <p>#(article)</p>
    #endfor
#else:
<p>no artcile</p>
#endif

```

遍历打印文章是打印不到的。必须是打印文章的属性，如下写法是正确的：

```leaf
<p>#(title)</p>
#if(!articles.isEmpty):
    <p>#(title)</p>
    #for(article in articles):
        <p>#(article.name)</p>//<--注意这里
    #endfor
#else:
<p>no artcile</p>
#endif
```

填坑完毕。







