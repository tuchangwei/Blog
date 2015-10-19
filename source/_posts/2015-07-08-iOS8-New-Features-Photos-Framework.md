title: "iOS8 New Features: Photos Framework"
date: 2015-07-08 21:09:08
tags: Photos Framework
categories: iOS8
---

最近有个项目要请求系统相册，就先把iOS8中的新特性`Photos Framework`研究一下，参考 iOS_8_by_Tutorials。

###模型对象
`Photos Framework` 中的模型对象是继承于一个基类`PHObject`，它们是轻量级的对象，仅仅包含了一些原数据的描叙，但并不包含其内容。这些具体的模型对象是：

1. `PHAsset`：表示一个单独的asset，它可能是一张图片或者视频。它包含了元数据比如是否是喜爱的，是否是隐藏的和媒介的类型。

2. `PHAssetCollection`： 表示一组有序的asset的列表。

3. `PHCollectionList`: 表示一组有序的`PHAssetCollection `的列表，或者`PHCollectionList `的列表。

### 获取(Fetching)
获取(Fetching)在`Photos framework`中是一个关键的特性，你可以获取任何的模型对象通过使用它所对应的类方法。另外当你获取的时候，你也可以使用`PHFetchOptions`选项。`PHFetchOptions`允许你指定哪些是你想要的，获取的结果应该如何排列，还有当结果发生改变时，Photos应该怎样提醒你。这些可以通过设置`PHFetchOptions`对象中的`predicate`，`sortDescriptors`,`includeAllBurstAssets`,`includeHiddenAssets`和`wantsIncrementalChangeDetails`属性，然后把它传给获取(Fetch)方法得到。

获取(Fetching)会一直返回一个`PHFetchResult`对象，一个`PHFetchResult`对象是一组有序的模型对象集合,它有一个接口和`NSArray`类似。这里最大的不同应该能让你站起来欢呼——`PHFetchResult`能替你管理内存并且实现对象的懒加载。这意味着获取和遍历上千的asset,理论上讲，不会比遍历几百个asset消耗更多的内存。十分惊奇。

记住，模型对象并不hold任何的内容，如果你想retrieve某个asset collection中的所有assets， 你不的不fetch他们。类似的，如果你想去展示和这些asset绑定的图片，你不的不加载它们。

```
- (void)fetchCollections {
  
  //获取喜爱的asset集合
  self.userFavorites = [PHAssetCollection fetchAssetCollectionsWithType:PHAssetCollectionTypeSmartAlbum subtype:PHAssetCollectionSubtypeSmartAlbumFavorites options:nil];
  
  //获取相册的集合（即是asset的集合的集合）
  self.userAlbums = [PHCollectionList fetchTopLevelUserCollectionsWithOptions:nil];
  
}
```
###加载内容
当你想展示属于一个asset的图片或视频内容的话，你需要通过一个`PHImageManager`的实例去加载它。这个默认的图片加载器是异步工作的，处理对asset的加载，缓存和调整。它的一个叫`PHCachingImageManager`的子类能预缓存asset的内容到内存中，并确保当你需要用到它们的时候，它们是可用的。

```
// 1let reuseCount = ++cell.reuseCountlet asset = currentAssetAtIndex(indexPath.item)// 2let options = PHImageRequestOptions() options.networkAccessAllowed = true// 3 PHImageManager.defaultManager().requestImageForAsset(asset,targetSize: assetThumbnailSize, contentMode: .AspectFill, options: options) { result, info inif reuseCount == cell.reuseCount { cell.imageView.image = result} }
```
注意这里的`requestImageForAsset`方法，这个方法可能会被回调多次。`Photos Framework`提供了可以立即使用的低分辨率的asset版本，随后也可能提供一个高质量的asset版本。

###预加载
```
- (void)updateCache {
  
  CGFloat currentFrameCenter = CGRectGetMidY(self.collectionView.bounds);
  NSLog(@"%@",NSStringFromCGRect(self.collectionView.bounds) );
  if (fabs(currentFrameCenter - self.lastCacheFrameCenter) < CGRectGetHeight(self.collectionView.bounds)/3) {
    // Haven't scrolled far enough yet
    return;
  }
  self.lastCacheFrameCenter = currentFrameCenter;
  
  static NSInteger numOffscreenAssetsToCache = 60;
  
  NSArray * visibleIndexes = [self.collectionView.indexPathsForVisibleItems sortedArrayUsingSelector:@selector(compare:)];
  
  if (!visibleIndexes.count) {
    [self.imageManager stopCachingImagesForAllAssets];
    return;
  }
  
  NSInteger firstItemToCache = ((NSIndexPath *)visibleIndexes[0]).item - numOffscreenAssetsToCache/2;
  firstItemToCache = MAX(firstItemToCache, 0);
  
  NSInteger lastItemToCache = ((NSIndexPath *)[visibleIndexes lastObject]).item + numOffscreenAssetsToCache/2;
  if (self.assetsFetchResults) {
    lastItemToCache = MIN(lastItemToCache, self.assetsFetchResults.count - 1);
  }
  else {
    lastItemToCache = MIN(lastItemToCache, self.selectedAssets.count - 1);
  }
  
  NSMutableArray * indexesToStopCaching = [[NSMutableArray alloc] init];
  NSMutableArray * indexesToStartCaching = [[NSMutableArray alloc] init];
  
  // Stop caching items we scrolled past
  for (NSIndexPath * index in self.cachingIndexes) {
    if (index.item < firstItemToCache || index.item > lastItemToCache) {
      [indexesToStopCaching addObject:index];
    }
  }
  [self.cachingIndexes removeObjectsInArray:indexesToStopCaching];
  
  [self.imageManager stopCachingImagesForAssets:[self assetsAtIndexPaths:indexesToStopCaching]
                                     targetSize:self.assetThumbnailSize
                                    contentMode:PHImageContentModeAspectFill
                                        options:nil];
  
  // Start caching new items in range
  for (NSInteger i = firstItemToCache; i < lastItemToCache; i++) {
    NSIndexPath * index = [NSIndexPath indexPathForItem:i inSection:0];
    if (![self.cachingIndexes containsObject:index]) {
      [indexesToStartCaching addObject:index];
      [self.cachingIndexes addObject:index];
    }
  }
  
  [self.imageManager startCachingImagesForAssets:[self assetsAtIndexPaths:indexesToStartCaching]
                                      targetSize:self.assetThumbnailSize
                                     contentMode:PHImageContentModeAspectFill
                                         options:nil];
  
}
```

