ezoixx130 的博客 <https://www.luogu.com.cn/blog/ezoixx130/codeforces-advanced-tutorial>

应@[ComeIntoPower](https://www.luogu.org/space/show?uid=11751)大大的建议，创作进阶版。

**前排提醒**：未食用游玩攻略的请先食用[Codeforces游玩攻略](https://www.luogu.org/blog/ezoixx130/codeforces-tutorial)

------------

Codeforces游玩攻略进阶版 —— **Codeforces快速精通**

![](https://cdn.luogu.com.cn/upload/pic/24981.png)

## 1. 社区相关

### (1) 语法支持

Codeforces的社区系统支持Markdown和 $\LaTeX$ ，同时还有许多Codeforces的独特语法。

下面我们给一个例子：

![](https://cdn.luogu.com.cn/upload/pic/26354.png)

可见Codeforces支持Markdown的基本语法，例如标题、加粗、斜体等。

单击格式栏里最右侧的一个按钮，您可以看到Codeforces论坛特有的元件，从上往下分别是插入用户、提交记录、题目、比赛、比赛排名、折叠器。这里主要讲讲折叠器的作用，当您插入代码时，由于代码的长度很长，容易占据很多空间，这是可以用折叠器把代码折叠起来，这样就不会占据太多的页面空间了。当有人需要查看代码的时候，可以将其展开。

### (2) 评论

在每条评论的最右侧，您都能看到一个正三角形和一个倒三角形夹着一个数字，这就是对评论的评价系统。单击正三角形为“赞”，倒三角形为“踩”。一名用户收到的“赞”和“踩”的数目会影响一个用户的Contribution。（Contribution拥有一个极其玄学的计算方式，貌似rating或者Contribution高的人“赞”和“踩”对别人的Contribution影响更大？如果一段时间不发言，Contribution会逐渐降至0？）

您有可能会发现某些评论的颜色变淡或者只有一句“The comment is hidden because of too negative feedback, click here to view it”。这是因为该评论收到了较多的“踩”，从而导致系统自动隐藏了这些评论（就是把颜色弄浅），如果收到的“踩”太多了，那么评论就会变为“The comment is hidden because of too negative feedback, click here to view it”，只能单击“here”查看。

如果一条评论的底色为黄色，则表示这条评论是由您发出的。

如果一条评论的底色为蓝色，则表示这条评论是新的评论，即您上次浏览该博客后发出的评论。

### (3) 浏览

在浏览博客时，页面右侧中间可能会出现一个灰色小框，含有一个上箭头，一个下箭头和一个数字。这个数字代表的是新评论的数量，两个箭头可以帮助您定位到上一条或者下一条的新评论上。

当您将鼠标指针移至页面最左侧时，会显示出一条灰色的暗条，单击这个条会回到页面最顶端。


------------


## 2. 做题相关

### (1) 题目

单击首页最上方的PROBLEMSET按钮，您将会看到Codeforces的题库。这些题目均为曾经的比赛题目。

![PROBLEMSET页面的样子](https://cdn.luogu.com.cn/upload/pic/32763.png)

在这个页面上您可以看到每道题目的编号、名字、标签、自己是否通过、通过人数的信息。若您知道一道题目的编号，那么您可以输入 `http://codeforces.com/problemset/problem/题目编号中的数字/题目编号中的英文` 来进入该题，例如，我们知道`The Child and Binary Tree (小朋友和二叉树)`的题号为`CF438E`，那么我们只要输入[http://codeforces.com/problemset/problem/438/E](http://codeforces.com/problemset/problem/438/E)就可以进入该题了。

#### Q&A

**Q**：我怎么看到我有没有通过一道题目啊？

**A**：您可以点开该题，然后在右侧找到Last submissions一栏查看自己是否通过。或者在Problemset中找到该题（不要点开），看看底色是否为绿色（绿色为已通过，红色为已提交但未通过，白色为未提交）。

**Q**: 我怎么看别人在这题上的提交记录啊？

**A**：在Problemset中找到该题（不要点开），点击右侧的通过人数即可。

**Q**: 我怎么查看一道题目的最优解，或者最短解呢？

**A**：在Problemset中找到该题（不要点开），点击右侧的通过人数，然后在页面最下方选择排序方式。

排序方式有：默认排序，提交时间排序，评测时间排序，代码长度排序和运行时间排序。

**Q**：我能否筛选一道题某种语言的提交呢？

**A**：能！在右侧的Status filter中选择条件，然后按Apply即可。

**Q**：`You have submitted exactly the same code before`是啥？为什么我点提交就出现这句话，然后提交不上去呢？

**A**: Codeforces不允许重复交同一份代码，请对代码做些改动再提交。

### (2) 评测

Codeforces上大多数的评测结果都与其它OJ没有区别，遇到Judgement failed等等的情况一般都是服务器的故障，稍等片刻，等待管理员修复，重交一遍一般就好了。

Codeforces上的数据是（半）公开的，每个数据都能在你的提交页面中看到，但是过大的数据只能显示出前一部分。

**注意**：Codeforces在评测程序时，会在第一个错误的测试点上马上停止，不会评测后面的测试点。

由于在比赛中的部分hack数据会在比赛结束后加入这道题目的测试点中，所以部分题目可能有上百个测试点，遇到这种毒瘤题请耐心等待，Codeforces的神机还是很快的。

### (3) GYM

gym中有许多ACM套题可供练习，可以直接做，也可以Virtual participation。但是紫名以及以下的用户在gym中无法看到数据，也无法看到别人的代码。

gym中的比赛均为ACM-ICPC赛制。

------------

## 3. 比赛相关

正题来了。

### (1) 赛制

*$\color{red}\text{若您已经大致了解Codeforces的赛制，则建议您跳过此部分}$*

第一部分：Div.1和Div.2比赛的赛制——CF赛制

Codeforces最出名的当然是CF赛制啦！在一场采用CF赛制的比赛中，每道题拥有一个满分，一般来说，满分与题目难度成正比，题目难度按顺序递增，例如，一场比赛的满分分布可能是`500-1000-1500-1500-2250-3000`。

然而，每道题的分数不是不变的，随着比赛时间的流逝，分数会逐渐减少，例如，一道500满分的题目，在00:01通过pretest的分数一般为498，在00:02通过pretest的分数一般为496。并且，每一次错误的提交还会扣除您50分的得分。

举一个例子，一道500分的题目，在第三分钟通过pretest，但是有一次错误的提交，那么得分为494-50=444分（494为这道题在00:03时的分数，50分为一次错误提交的罚分）。

重点：上面为什么说的是通过pretest而不是AC呢，因为CF赛制的题目会有两套数据，一套称为Pretest，另一套称为System Test，当比赛进行时，您的提交将会用Pretest测评，若通过所有Pretest，则会显示$\color{green}\text{Pretests passed}$，否则显示错误的Pretest编号和错误类型，例如$\color{red}\text{Wrong answer on pretest 3}$，并且还会被罚50分（如果编译错误则不会罚分，错在第一个测试点也不会被罚分）。

当您的一道题目$\color{green}\text{Pretests passed}$后，您可以单击题目列表中那到题目后面的“锁”的符号，这称为锁题，当您锁了一道题后，您就不能再次提交该题了，但是您可以查看同一个房间中其它人本题的代码，若找到了其他人代码中的错误，您可以向他发起hack，即提交一组测试数据使得他的代码错误（例如Wrong answer等等）。一次成功的hack可以使您获得100分，不成功则扣掉50分。

房间：参加比赛的所有用户大约每40各人组成一个房间，只有在房间里的用户才能互相hack。

发起hack的方式是：双击您的房间的排行榜中的任何一个绿色数字，再单击通过的提交的编号查看代码，如果找到了错误，那么您可以单击hack it!，然后输入数据或者上传数据生成器，单击hack即可。

**注意**：锁了的题不能再提交，也就是说如果您锁了一道题，但是您的程序被hack了，那么您就没有补救的机会了。所以，~~叉人有风险，锁题需谨慎~~。

比赛结束后不久就会进行System Test，就是将您已经通过Pretest的程序再测试一套数据，这套数据包括所有比赛中成功hack的数据（所以在Codeforces上有些题目可能拥有上百个测试点），只有您的程序通过了System Test，您的程序才是真正通过了，否则称为Failed System Test(FST)，这道题也就不得分了。

（摘自Codeforces游玩攻略）

第二部分：Div.3和Educational比赛的赛制——拓展ACM-ICPC赛制

拓展ACM-ICPC赛制是指，每次提交立即评测出结果，排名按照通过题数排（这意味着每道题权重相同），题数相同则按总时间排，总时间指的是每道题第一次通过的时间之和+错误的提交次数×10分钟。

当然Codeforces的核心——hack还是会出现的，每场拓展ACM-ICPC赛制的比赛结束后，会有12个小时的时间，让您随意查看、hack每个人的提交，这12个小时结束后，所有程序还会测试一遍成功的hack的数据，得到的结果才构成最终的排名。

### (2) 做题技巧

下分大佬请自行左转【洛谷日报#38】[OwenOwl]浅谈如何在Codeforces下分。

我们可以发现，分值大的题目分数随时间减少的速度比分值小的题目更快，所以，如果您确保自己能够很快的通过C、D、E甚至F题，您可以选择倒序做题。

例如，若您认为您的水平已经达到能够一眼看出A、B、C题的解法，您可以选择按照C-B-A或着C-A-B的顺序做题。若您是轻松AK的大佬，您可以按照F-E-D-C-B-A的顺序做。

![](https://cdn.luogu.com.cn/upload/pic/32771.png)

我们可以看到，同样是做出5题的选手，上面这张图片的rank1就是采用了按照E-C-B-A-D的做法，拿到了较高的分数。

另外，若您对一道题思考许久没有思路，您可以切换到下一题（因为经常有毒瘤出题人出的题目，难度不按顺序递增）。

### (3) hack技巧

CF赛制最刺激的部分是什么？Hack！System Test！

Hack其实没有多大的技巧，但是关键是快、准、狠。因为，比赛的时间不长、hack失败要扣分、hack成功有不错的回报。

给些hack的tips:仔细看题目数据范围的边界情况，例如$1<=n<=10000$的题目可能出现需要特判$n=1$的情况，而没有特判的程序也有可能通过pretest，这时候就大胆hack吧。

另外：有些暴力也是能过pretest的，这时候也去写个generator大胆hack吧。

### (4) 插件推荐

推荐插件CF-Predictor。

安装完这个插件后，您的Codeforces的比赛排名榜页面会变成这样子的：

![](https://cdn.luogu.com.cn/upload/pic/32806.png)

有没有发现，最右侧多了一栏数字，这些数字代表的是rating的变化。

**Q**: 那这个插件有什么用呢？

**A**: 它可以告诉你rating的变化啊。

**Q**: 它可以在比赛结束时告诉我rating将要变化多少吗？

**A**：当然可以。比赛结束时打开排名榜，找到自己，然后看自己最后一栏的数字是什么就代表您的rating将怎样变化。由于System Test的存在，这个数字在System Test前是不准的（也不会有多大误差）
，在System Test结束后，这个就是准确的rating变化值了（误差不超过1）。

**Q**: 比赛时CF-Predictor有用吗？

**A**: 有用！CF-Predictor在比赛是给出的值是如果您保持当前排名不变，比赛结束后您的rating变化值。

**Q**: 为什么我显示不出来啊。

**A**: 首先，请确保正确安装并启用了这个插件。打开比赛排名榜页面后，如果没有显示出来，请先稍等大约30秒。如果您访问的是https版的Codeforces，试试把https改为http，然后再尝试一次。

**Q**: CF-Predictor怎么安装啊？

**A**: Firefox用户可以直接在浏览器的“附加组件”中搜索CF-Predictor安装。如果您不是Firefox用户，您可以点击[https://tampermonkey.net/](https://tampermonkey.net/)来安装tampermonkey，再访问[https://greasyfork.org/zh-CN/scripts/38050-cf-predictor](https://greasyfork.org/zh-CN/scripts/38050-cf-predictor)安装CF-Predictor。

这个方法对Microsoft Edge、Chrome等浏览器均有效。

### (5) 作弊处罚

如果在比赛结束很久后你的rating都没有更新、比赛的提交状态都显示为Skipped，并且收到一条这样的信息`Attention! Your solution (你的提交编号) for the problem （题目编号） significantly coincides with solutions (用户名)/(提交编号). Such a coincidence is a clear rules violation. Note that unintentional leakage is also a violation. For example, do not use ideone.com with the default settings (public access to your code). If you have conclusive evidence that a coincidence has occurred due to the use of a common source published before the competition, write a comment to post about the round with all the details. More information can be found at http://codeforces.com/blog/entry/8790. Such violation of the rules may be the reason for blocking your account or other penalties. `，那么这就说明你的代码与其他选手有雷同，所以这场比赛对你Unrated。

这是Codeforces对比赛作弊的处罚，如果你多次作弊，那么将会被封号。

### (6) Virtual participation

如果您觉得大部分比赛时间太晚，而赛后做题又没有比赛的感觉，您可以尝试Virtual participation，即参加一场虚拟比赛。

发起Virtual participation的方式：在CONTESTS中找到您要发起Virtual participation的比赛，然后单击比赛名称下方的`Virtual participation » `，设置好时间和参赛人员即可。

Virtual participation具有以下特点：

- Unrated

- ACM-ICPC赛制

- 可自定义开始时间以及参加选手

- 提交直接测system test，而不是pretest

如果想要和同学参加同一场Virtual participation的话，可以加入同一个Team，然后再选择Take part：as a team member即可。

------------

由于学识尚浅，就只讲这么多了，如有错漏请指出。最后祝大家high rating，早日LGM!
