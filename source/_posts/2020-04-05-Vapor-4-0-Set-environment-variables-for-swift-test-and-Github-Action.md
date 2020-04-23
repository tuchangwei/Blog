title: '[Vapor 4.0]Set environment variables for swift test and Github Action'
date: 2020-04-05 15:52:58
tags:
categories: Vapor4.0
---

When we run test casts, we don't want to use our development database, we want to use a new database for testing. In Xcode, it is easy to set. Under the `Run` tab, we don't set the database information, but under the `Test` tab, we do.

{%asset_img WX20200405-160058@2x.png %}

{%asset_img WX20200405-160133@2x.png %}

But do you know how to set these variables via command line? Here it is:

```swift
export DATABASE_PORT="5433"; export DATABASE_USERNAME="test"; export DATABASE_PASSWORD="test"; export DATABASE_NAME="test"; swift test
```

If you want to cancel setting, you need to close your terminal window and open it again.

Now, CI(continutal Integration) is popular, today I also try to run test when I push my code to Github reposity. After some failture, here is the right `yml` file.

```swift
name: test
on:
  push:
jobs:
  test:
    container:
      image: vapor/swift:5.2
    services:
      psql:
        image: postgres
        ports:
          - 5432:5432
        env:
          POSTGRES_USER: test
          POSTGRES_DB: test
          POSTGRES_PASSWORD: test
    runs-on: ubuntu-latest 
    
    steps:
      - name: 'Checking out repo'	       
        uses: actions/checkout@v2	
      - name: 'Running Integration Tests'	
				run: swift test --enable-test-discovery --sanitize=thread
        env:
         DATABASE_PORT: 5432
         DATABASE_USERNAME: test
         DATABASE_PASSWORD: test
         DATABASE_NAME: test
         DATABASE_HOST: psql
        
```

I had thought to change the database port to `5433`, but it seems a bug of Github Action. After I changed it back to `5432`, it works.

Thanks to [@0xTim](https://twitter.com/0xTim) for discussion about it.

