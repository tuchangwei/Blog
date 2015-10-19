title: How to calculate table cell height using auto-layout
date: 2014-10-13 23:18:31
tags: AutoLayout
---

No time to write it clearly, I just post some code which helps me finish it.

1. Use Xib, not storyboard.
2. Add the following code:

```
- (CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath {
    
    static TBGrowRecordCell *sizingCell = nil;
    
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        sizingCell = [self.tableView dequeueReusableCellWithIdentifier:CellIdentifier];
    });
    TBGrowRecordEntity  *record = self.recordArray[indexPath.row];
    sizingCell.contentLabel.text = record.content;
    [sizingCell layoutIfNeeded];
    
    //Notice that: this is "sizingCell.contentView", not "sizingCell"
    CGFloat height = [sizingCell.contentView systemLayoutSizeFittingSize:UILayoutFittingCompressedSize].height;
    return height;
}

```

Notice that: this is "sizingCell.contentView", not "sizingCell".

3. Choosing the checkbox `constraint to margins`.

![](/2014/10/13/How-to-calculate-table-cell-height-using-auto-layout/QQ20141013-1@2x.png)

4. Set `lines` to 0.
5. choosing the checkbox `explicit`.

![](/2014/10/13/How-to-calculate-table-cell-height-using-auto-layout/QQ20141013-2@2x.png)

Done! The effect is that the UIlabel named `sizingCell.contentLabel` will resize by text's length, and the UITableViewCell will resize by the UILabel.

Refer to:

[Dynamic Table View Cell Height and Auto Layout](http://www.raywenderlich.com/73602/dynamic-table-view-cell-height-auto-layout)
 
 [Using auto-layout to calculate table cell height](http://blog.amyworrall.com/post/66085151655/using-auto-layout-to-calculate-table-cell-height)
 
 http://stackoverflow.com/questions/25265173/how-can-a-get-the-auto-layout-size-of-the-uicollectionviewcells-in-ios-8-syste