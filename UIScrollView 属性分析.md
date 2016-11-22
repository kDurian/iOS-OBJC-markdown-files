# UIScrollView



## contentSize

```objective-c
- (void)viewDidLoad {

	UIScrollView *scrollView = [UIScrollView new];
  	scrollView.delegate = self;
  
    scrollView.alwaysBounceVertical = NO;  // default NO
    scrollView.alwaysBounceHorizontal = NO; // default NO

	scrollView.frame = self.view.frame;

	scrollView.contentSize = CGSizeMake(self.view.width, self.view.height);
  
  	[self.view addSubview:scrollView];
  
}

- (void)scrollViewDidScroll:(UIScrollView *)scrollView {
	...... // 1
}
```



现象1：

> scrollView.width  **>=**  scrollView.contentSize.width  **&&**  
>
> scrollView.height  **>=**  scrollView.contentSize.height
>
> 不会调用 1

现象2：

> scrollView.width  **<=**  scrollView.contentSize.width **||** 
>
> scrollView.height  **<=**  scrollView.contentSize.height
>
> 会调用 1



结论：

> 当 alwaysBounceVertical 和 alwaysBounceHorizontal 都为 NO 时，scrollView 能滚动的前提是它的 contentSize 的 width 或 height  **至少要有一个** 大于 scrollView.frame.size  的 width 或 height。



## contentOffset && contentInset

contentOffset 表示的是 contentSize 大小的可滚动区域的左上角 在 x 和 y 方向与scrollView 可显示窗口的左上角的距离。

![](https://c4.staticflickr.com/6/5472/30575107035_f6d1ed6efb_b.jpg)



* **contentOffset 与 contentInset 的关系：**

  * 当 contentInset = { 0,  0,  0,  0 } 时，初始 contentOffset = { 0,  0 }
  * 当 contentInset = { top,  left,  bottom,  right } 时，初始 contentOffset = { -left,  -top }

* **contentOffset 与 可滚动区域位置的关系：**

  * 当可滚动区域的左上角顶点 与 scrollView 左上角顶点重合时，contentOffset = { 0,  0 }

  * 当可滚动区域的左上角顶点 在 scrollView 左上角顶点的 上 或 左 时，contentOffset.x 或 contentOffset.y 大于 0 

  * 当可滚动区域的左上角顶点 在 scrollView 左上角顶点的 下 或 右 时，contentOffset.x 或 contentOffset.y 小于 0 

    ​

***图解*** ***contentOffset 与 可滚动区域位置的关系：***

* contentOffset.x  &&  contentOffset.y   ==   0

![](https://c2.staticflickr.com/6/5679/29941275553_510a5e21e4_b.jpg)

​										*Show image detail*	

* contentOffset.x  &&  contentOffset.y   >   0

![](https://c2.staticflickr.com/6/5580/29941215473_0f67f1ed9b_b.jpg)

​										*Show image detail*



* contentOffset.x  &&  contentOffset.y   <    0

![](https://c3.staticflickr.com/6/5503/30458143562_f9b4de692b_b.jpg)

​										*Show image detail*



### One more thing

From iOS7, if vc.navigationController != nil, vc.**automaticallyAdjustSrollViewInsets** will influence  the tableView's initial contentOffset.

```objective-c
- (void)viewDidLoad {
  	self.tableView = [[UITableView alloc] initWithFrame:self.frame    style:UITableViewStylePlain];
  	self.tableView.delegate = self;
  	self.tableView.dataSource = self;
  	[self.view addSubview:self.tableView];
}
```

* **self.navigationController = nil**

  > **self.tableView.contentOffset = {0, 0}**

  ​

  ![](https://c5.staticflickr.com/6/5466/30482394532_48e18c08ec_b.jpg)

  ​

* **self.navigationController != nil**

  * **self.automaticallyAdjustScrollViewInsets = YES;**

    > **self.tableView.contentOffset = {0, -64}**

    ![](https://c3.staticflickr.com/9/8649/30482394242_cc19ebe326_b.jpg)

    ​

  * **self.automaticallyAdjustScrollViewInsets = NO;**

    > **self.tableView.contentOffset = {0, 0}**

    ![](https://c6.staticflickr.com/6/5623/29965440453_a8040ab4a9_b.jpg)