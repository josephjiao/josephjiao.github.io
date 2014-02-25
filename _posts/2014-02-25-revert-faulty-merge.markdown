---
layout: post
title: git中如何恢复一个错误的merge
tags: git  
year: 2014
month: 2
day: 25
published: true
summary: git中如何恢复一个错误的merge,其实就是要理解git 的repository history
image: git.jpg
categories: 技术 
---


在实际工作中，创建一个topic分支进行开发之后, 进行了merge操作，之后发现
merge操作失误了，想要重做merge。此时，简单的认为revert 这个merge操作后，
再次merge就可以解决问题了。

实际操作中，发现revert后再次merge的时候，git提示该分支已经merge过，不需要merge。
  
于是查了查git的资料，马上就发现了git文档中自带的这个
[HOWTO](https://www.kernel.org/pub/software/scm/git/docs/howto/revert-a-faulty-merge.txt)
，说的相当的清楚，英文好的可以直接跳到那里了。
 

------------------ 
好了，留下的都是愿意看中文的了。：）

刚才说的情况下，Git库可以简单的看做以下这种状况：

    ---o---o---o---M---x---x---W
                  /
    topic ---A---B
其中： M是merge点，而W看作是”revert M操作“的点。

  
再次merge topic分支，被提示不需要merge的原因在于：  
<font color ="red">虽然 revert了M，此时只是代码的内容被undo掉了，merge这个操作的操作记录在git系统中是认为没有revert掉的。 </font>

用git文档的说法就是：”git revert“仅仅是进行了简单的**data**的revert而没有进行**repository history**的revert。


怎么解决这个问题呢?
----


在我们这里，其实简单重写整个topic分支，人为的让git以为这是和原来topic分支毫无关系的一个分支，就可以了。
  
执行

    git rebase -i --no-ff P

得到如下的结果：

      A'---B'---C'------------------
     /                              \
    P---o---o---M---x---x---W---x---M2
     \         /
      A---B---C
 这里一定要加上”--no-ff“选项，不然，在rebase中进行”pick“动作的时候，
git会默认采用fast-forward，ABC节点的SHA ID值都会不变，就如果没有任何效果
一样。
 
-------------
  
 好了，这个问题再次延伸一下：如果我们当时不是想要重做merge操作，而是
想先回复主干为merge前的可用状态，接着在分支上进行修补，等修补好了以后再进行
merge，又应该怎么办呢？
 
    ---o---o---o---M---x---W-------x-------N
                  /                       /
          ---A---B-------------------C---D
 
如果这么做的话，我们会发现在N点上，A B两次提交的修改不会在N中出现。

原因自然还是那句：”git revert“仅仅是进行了简单的**data**的revert而没有
进行**repository history**的revert。

两种解决方法
----

- 进行一次revert的revert

如下图

    ---o---o---o---M---x---x---W---x---Y----N
                  /                        /
          ---A---B-------------------C---D
 
 此处的Y点就是 git revert W产生的

- 放弃原有分支，重新建立新的分支。

这个没有什么特殊的，就是简单的rebase + merge的操作了。

    ---o---o---o---M---x---x---W---x---x---x ---*
                  /                 \         /
          ---A---B                   A'--B'--C'

这里需要注意的是，不要进行revert的revert，不然在merge A'B'C'的过程中会产
生大量的冲突。

