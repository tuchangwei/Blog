title: '[Vapor 4.0]Updating field by adding unquie constraint doesn''t work'
date: 2020-04-06 23:06:34

tag: 

Categories: Vapor4.0

---

So today I wanted to add an `unique` constraint to my `ArticleCategory` table, but it doesn't work, my code: 

```swift
struct ModifyNameToCategory: Migration {
    func prepare(on database: Database) -> EventLoopFuture<Void> {
        return database.schema(ArticleCategory.schema)
            .unique(on: ArticleCategory.nameKey)
            .update()

    }
    func revert(on database: Database) -> EventLoopFuture<Void> {
       ...
    }
}
```

The error:

```swift
[ INFO ] query read _fluent_migrations
[ INFO ] query read _fluent_migrations limits=[count(1)]
[ ERROR ] syntax error at end of input (scanner_yyerror)
[ ERROR ] server: syntax error at end of input (scanner_yyerror)
Fatal error: Error raised at top level: server: syntax error at end of input (scanner_yyerror): file /AppleInternal/BuildRoot/Library/Caches/com.apple.xbs/Sources/swiftlang/swiftlang-1103.8.25.8/swift/stdlib/public/core/ErrorType.swift, line 200
```

By reading the source code, I found this because the `fluent-kit` didn't add handling for the `update` action to add constraint. But it does have handing for `create` action to add constraint. Please see [the issue detail](https://github.com/vapor/fluent-kit/issues/235) I submitted.

So what we can do now? Luckly, we can use the old way:

```swift
import Foundation
import Fluent
import FluentSQL
import FluentPostgresDriver
struct ModifyNameToCategory: Migration {
    func prepare(on database: Database) -> EventLoopFuture<Void> {
        /*
        //Because the Fluent kit didn't implement this feature, so we need to run the sql raw.
        return database.schema(ArticleCategory.schema)
            .unique(on: ArticleCategory.nameKey)
            .update()
        */
        let psgl = database as! PostgresDatabase
        let sql = "ALTER TABLE \(ArticleCategory.schema) ADD CONSTRAINT name_unique UNIQUE (\(ArticleCategory.nameKey))"
        print(sql)
        return psgl.query(sql)
            .transform(to: ())

    }
    func revert(on database: Database) -> EventLoopFuture<Void> {
        let psgl = database as! PostgresDatabase
        let sql = "ALTER TABLE \(ArticleCategory.schema) DROP CONSTRAINT name_unique"
        print(sql)
        return psgl.query(sql)
            .transform(to: ())
    }
}
```

Don't forget to import `FluentSQL` and `FluentPostgresDriver`

