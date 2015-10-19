title: Mantle with Core Data
date: 2014-11-02 09:03:27
tags: Mantle
---

#Conform to the Protocol
First, on our model we conform to the `MTLManagedObjectSerializing` protocol. This has 2 required methods, but we'll actually need to implement 3 methods.

```Object-c
#pragma mark - MTLManagedObjectSerializing

+ (NSString *)managedObjectEntityName {
    return @"Episode";
}

+ (NSDictionary *)managedObjectKeysByPropertyKey {
    return @{ };
}

+ (NSValueTransformer *)thumbnailImageUrlEntityAttributeTransformer {
    return [MTLValueTransformer reversibleTransformerWithForwardBlock:^NSString *(NSURL *url) {
        return [url description];
    } reverseBlock:^NSURL *(NSString *urlString) {
        return [NSURL URLWithString:urlString];
    }];
}

+ (NSSet *)propertyKeysForManagedObjectUniquing {
    return [NSSet setWithObject:@"episodeNumber"];
}
```

This is all we need to enable the feature for our models, including updating the records if they exist already.

#Serializing to Core Data
Given the we've converted from a JSON document to our model, we also want to convert that model into an `NSManagedObject` for saving. We can do this for each record by using the `MTLManagedObjectAdapter` class:

```Object-c
[self fetchEpisodesWithCompletion:^(NSArray *episodes) {
        NSManagedObjectContext *moc = [self.persistenceController newChildManagedObjectContext];
        [episodes enumerateObjectsUsingBlock:^(Episode *episode, NSUInteger idx, BOOL *stop) {
            NSError *insertError;
            NSManagedObject *mob = [MTLManagedObjectAdapter managedObjectFromModel:episode
                                                              insertingIntoContext:moc
                                                                             error:&insertError];
            
            if (mob) {
                NSLog(@"Mob: %@", mob);
            } else {
                NSLog(@"ERROR: %@", insertError);
            }
        }];
        
        NSError *saveError;
        if ([moc save:&saveError]) {
            [self.persistenceController saveContextAndWait:YES completion:^(NSError *error) {
                NSLog(@"Save completed (%@)", error);
            }];
        } else {
            NSLog(@"Error saving: %@", saveError);
        }
    }];
```
Refer to [http://nsscreencast.com/episodes/120-mantle-with-core-data](http://nsscreencast.com/episodes/120-mantle-with-core-data)