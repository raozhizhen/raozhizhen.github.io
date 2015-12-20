---
date: 2015-12-19 12:00
status: public
title: 'UICollectionView实现一个选择标签页'
---


	这算是第一篇技术博客，《Effective Objective-C 2.0》已经看完啦，本来想做笔记记录点什么，但书中都总结的很好，看笔记不如直接看书。so，开始，分享一下如何使用UICollectionView制作一个选择标签页
	
	

先上图	
![](https://github.com/raozhizhen/raozhizhen.github.io/blob/master/blogImage/LPSelectTagImage.png?raw=true) 


gif录制用的是[licecap](http://www.cockos.com/licecap/)不是很清晰啊，有更好的推荐的留言哈

![](https://github.com/raozhizhen/raozhizhen.github.io/blob/master/blogImage/LPSelectTagGif.gif?raw=true) 

首先封装了一个通用一点的选择标签的collectionView

我比较喜欢先写.h，这样更能让我理清思路，就像写小说先写大纲一样

![](https://github.com/raozhizhen/raozhizhen.github.io/blob/master/blogImage/LPTagCollectionViewImage.png?raw=true)

LPTagModel里一共三个属性 

	@property (nonatomic, strong) NSNumber *identifier;/**<活动标签id*/
	@property (nonatomic, copy) NSString *name;/**<标签名称*/
	@property (nonatomic, assign) BOOL isChoose;/**<是否被选择*/

用来储存标签数据，来控制item的显示，

cell的宽度是根据文本自适应的，用下面这个方法计算string的宽度

	- (CGSize)sizeWithAttributes:(nullable NSDictionary<NSString *, id> *)attrs NS_AVAILABLE(10_0, 7_0);

当一行放不下下一个item的时候,UICollectionView会将它放置下一行，但放满的这一行会自动左右对齐，这样显得item的间距不一样，为了解决这一问题，在GitHub上找到这个[UICollectionViewLeftAlignedLayout](https://github.com/mokagio/UICollectionViewLeftAlignedLayout),只要将你使用的UICollectionViewFlowLayout替换成UICollectionViewLeftAlignedLayout，这样item就会向左靠齐了。

UICollectionView封装完毕（细节就不说了，有兴趣可以去[我的GitHub](https://github.com/raozhizhen)上看看）

选择标签页面

这个页面主要是两个UICollectionView,和一个按钮，像排列三个label一样用自动布局？可惜UICollectionView里并没有能把它撑起来的东西，所以我们需要知道UICollectionView的内容高度，也就是collectionView.contentSize.height。UICollectionView在布局完成之后contentSize里才有实际的值，那就可以在updateViewConstraints方法里，布局的时候去获取获取collectionView的高度，或者在viewDidLayoutSubviews里，布局完成之后去获取它的高度再更新布局。

然后上面的collectionView，就是在最后一个item里放了一个UITextField,这样就可以输入添加新的标签了，输入标签的时候做了两个for循环，第一个for循环检查是否输入的是重复的标签，如果是的话，不添加新的标签，将重复的标签移动到最后一个，让用户知道添加了一个重复的标签（是的，我就是这么爱为用户考虑），实现是先移动数据源里的数据，再移动item，移动完成之后还需要更新一下collectionView的高度

	[_selectedArray moveObjectFromIndex:i toIndex:_selectedArray.count - 1];
            [_selectedCollectionView performBatchUpdates:^{
                [_selectedCollectionView moveItemAtIndexPath:fromIndexPath toIndexPath:toIndexPath];
            } completion:^(BOOL finished) {
                if (finished) {
                    [self updateSelectedCollectionView];
                }
            }];

如果是删除的话，首先是添加了长按删除，以及直接按键盘上的delete键删除最后一个item，实现方式是，在输入框的开头放了一个空格，当要删除这个空格的时候，不执行删除，而删除最后一个item，虽然说显示方式看起来有点扭曲，但效果上还是很完美的。

	- (BOOL)textField:(UITextField *)textField shouldChangeCharactersInRange:(NSRange)range replacementString:(NSString *)string {
    if (range.length == 1 && range.location == 0) {
        if (self.delegate && [self.delegate respondsToSelector:@selector(deleteTag)]) {
            [self.delegate deleteTag];
        }
        return NO;
    }
    return YES;
}


好像也没有什么干货，重要是是分享一下自己解决问题的一些思路。如果有更好的实现方式，也希望能分享给我

以上