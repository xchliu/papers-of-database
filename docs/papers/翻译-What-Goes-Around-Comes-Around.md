一切皆是命中注定？

摘要

    本论文对过去35年的数据模型总结出9个阶段。我们讨论每一个阶段的报告，并只介绍一些基础的持续较长时间的数据模型。较晚的报告不可避免地和较早的报告会比较相似。于是，研究之前的报告是一件有价值的事情。
    另外，我们提供每一个阶段的探索报告的学习课程。很多当前的研究人员对之前的这些阶段并不熟悉，而且对之前的研究成果的理解是有限的。有个古老的格言是不懂历史的人会去重复历史。通过提供“古代历史”（译注：数据模型发展过程），我们希望未来的研究人员避免重蹈覆辙。
    不幸的是，当前在XML阶段的主要报告和1970s的CODASYL报告非常相似，未成功的原因是他太过复杂。所以，当前阶段是在重演历史，“What Goes Around Comes Around”。期待下个阶段快一些到来。
    
一、介绍
    从1960’后期，数据模型报告开始出现，第一个作者出现。在未来的35年里，报告持续的产出。此外，许多当前的报告的研究人员由于太年轻并没有去学习关于早先报告的讨论。于是，本论文的目的是总结这35年发展过程的价值并指出在这个长时间的探索中什么应该被学习。

    我们将数据库报告分为9个时代：
        Hierarchical (IMS): late 1960’s and 1970’s
        Directed graph (CODASYL): 1970’s 
        Relational: 1970’s and early 1980’s 
        Entity-Relationship: 1970’s
        Extended Relational: 1980’s 
        Semantic: late 1970’s and 1980’s
        Object-oriented: late 1980’s and early 1990’s 
        Object-relational: late 1980’s and early 1990’s
        Semi-structured (XML): late 1990’s to the present
    对于每一个阶段，我们从中立的角度讨论其中的数据模式和相关的查询语言。于是，我们让读者不去关心这些报告特有的细节。我们也会使用一个统一的任期收集，同样也是减少可能出现的疑惑。
    对于大部分报告，我们将使用标准的suppliers和parts来作为样例，来自[CODD70]的关系格式如下：
        Supplier (sno, sname, scity, sstate) 
        Part (pno, pname, psize, pcolor) 
        Supply (sno, pno, qty, price)
    这里，我有supplier，part和供应关系的信息，供应关系标书一个supplier可以供应某个part的任期。
    下面是一些简单的实体数据：
Suppplier
16 General Supply Boston Ma 
24 Special Supply Detroit Mi

Part
27 Power saw 7 silver
42 bolts 12 gray

Supply
16 27 100 $20.00 
16 42 1000 $.10 
24 42 5000 $.08
二、IMS时代
    IMS 在1968年发布，起初基于层次模型。对于一条记录类型的理解为多个命名域和他们相应的数据类型的集合。每个记录类型的实体必须和记录类型的数据定义保持一致。进一步看，一些命名域的子集必须唯一指向一个记录实体，他们将被作为一个key。最后，记录类型必须以属性结构来组织，除了根节点以外，每一个记录类型有一个唯一的父记录类型（父节点）。一个IMS数据库则是衣多个记录类型的集合，于是除了根节点，对于每一个实体，都有一个明确的记录类型作为他的父节点。
    树形结构的组织方式对于我们的样例数据有一些挑战，因为我们必须按照下面的方法来组织模式。
    
    对于第一个模式，我们列出一些简单数据。


    这样的方式由两个场景的不受欢迎的特性（译注：痛点）：
        1）信息容易。在第一个模式中，Part的信息在每一个提供了这个part的supplier中都要存在。在第二个模式中，supplier的信息在每一个他提供的商品中都存在。信息冗余是不受欢迎的，因为存在数据不一致的可能。比如，一个重复的数据元素可能在一些记录中改变，但不是所有他存在的地方，从而导致了数据不一致。
        2） 对父节点的依赖。在第一个模式中，可能存在某个part当前不被任何supplier供应的情况。在第二个模式中，可能存在某个supplier当不供应任何part的情况。在严格的层面结构中，这种极端情况是不支持的。
        IMS悬着使用层次模型的原因是他使用了一个简单的数据操作语言：DL/1。在IMS中的每条记录都有一个层次序列索引（HSK）。基本上，一个HSK通过连接祖辈记录作为起源，然后添加当前记录的索引。HSK定义了IMS数据库中所有记录的自然顺序（译注：记录访问顺序），基本的深度优先，自左到右（译注：前序遍历）。DL/1 频繁的使用HSK来响应查询。例如，命名“get next”返回HSK顺序中的下一个记录。HSK顺序的另一个使用是“get next within parent”命令，这个命令在HSK序列中遍历一个给定记录下面的子树。
        使用上面的第一个模式，通过下面的命令可以获取由supplier 16提供的红色parts：
Get unique Supplier (sno = 16) 
Until no-more {
    Get next within parent (color = red) 
}
        第一个命令查找supplier 16.然后在HSK序列中遍历这条记录的子树，查找红色parts。当子树遍历结束，返回一个错误。
        注意DL/1是“record-at-a-time”语言（译注：一次查询一条记录）。因此程序员编写算法来解决查询问题，然后IMS执行这个算法。通常会有多种方法来解决一个查询问题。下面是另一个解决方法：
Until no-more {
Get next Part (color = red)
}
        尽管有人会认为第二种解决方法比第一种要差；事实上如果数据库中只存在一个supplier（16），第二种解决方法性能比第一个好。DL/1程序员必须做这样的优化权衡。
        另外，IMS的程序员必须关注两个并发情况（记录并发和父节点并发）。由于边缘问题使得问题变得特别复杂。比如，如果一个父节点被一个更新操作少出，那父节点指针应该去哪里？管理并发场景要求显式的错误逻辑处理会很复杂和导致更多的错误。Jim Gray 在报告中提到，一个IMS用户平均需要对一调DL/1语句进行17个测试后才可以成功完成一个Q/A程序。当然，DL/1 程序是非常复杂的，IMS 程序比其他类型的程序的报酬更高。    
        IMS支持四种不同的层次数据的存储格式。root记录存储方式可以是：顺序存储，使用记录的key组织的B-tree,使用记录索引的hash值。
        从root节点查询依赖记录的方法：物理遍历，各种类型的指针。
        DL/1命令会受到一些存储格式的限制。例如自然顺序格式不支持记录的插入。于是，在批处理场景中，对一个有序的HSK序列进行修改，并一次性写入数据库中是比较合适的，变更的数据查询到正确的位置，新数据也完成写入。这种常称为“old-master-new-master”过程。另外，对索引的root记录进行hash处理的存储格式不支持“get next”命令，因为他不太容易返回HSK序列中的hash转换后的记录。
        IMS中这些奇怪的特性（译注：不支持的查询）是为了避免一些效率低效的操作。然而，对应的代价则是：无法自由地改变IMS的存储方式而进行数据库程序调优，因为DL/1程序将不会被允许继续运行。
        数据库程序持续运行的能力，忽略物理层面的调优被称为物理数据独立性。数据的物理独立性非常重要，因为DBMS的应用程序并不是一次性编写的。当新的程序加入到应用中，调优命令可能会改变，通过改变物理组织方式可能带来更好的DBMS性能表现。IMS选择了限制了大量的数据独立性。
        另外，应用逻辑需求可能随时变化。新的记录类型可能添加，来自新的业务需求或者是政府要求。可能需要将一个特定的数据元素移动到另一个记录类型中。IMS支持一定程度上的数据逻辑独立性，因为DL/1其实是在逻辑数据层面定义的，而不是在实际的物理存储。于是，一个DL/1程序一开始可以将逻辑数据和物理数据格式定义相同。然后，记录类型可以添加到物理数据层，逻辑数据可以通过重定义来包含他。于是，一个IMS数据库可以在运行中增加新的记录类型，而对于原始的DL/1程序仍然可以正常运行。一般来说，IMS数据库中逻辑数据层是物理数据层的子树。
        让程序员和逻辑数据交互是非常好的注意，因为这样允许逻辑组织发生改变，而不需要去考虑DL/1程序是否可以运行。相对数据库中的数据来说，DBMS系统是长期运行的服务（经常25年或更久），数据的逻辑独立和物理独立显得非常重要。数据独立性使得数据可以在不影响系统运行的基础上进行数据变更。
        关于IMS的最后一点。很明显，我们前面提到的有很多简单的数据是无法用树形结构来表示的，于是，展示上述提到的简单数据而不考虑独立性问题对于IMS来说是很有压力的。IMS通过扩展数据逻辑概念来解决这个问题。
        
        如上图，假设设计了两个物理数据库，一个只包含Part信息，另一个包含supplier和supply关系信息。当然,DL/1程序是定义在树；于是他们无法直接应用到这两个结构中。IMS允许如下图的方式定义逻辑数据库。在这里面的层次结构中，supply和part的记录类型来自两个不同的数据库通过具有相同的part编号值联接得来（join）。
        根本上来说，上图的结构是实际存储的，而且其中也不存在不好的独立性问题存在。而程序员面向的是下图的结构，他支持标准的DL/1程序。（译注：模式映射问题）
        
        一般来讲，IMS允许两种不同的树形结构的物理数据库“嫁接”成一个逻辑数据库。在使用这一的逻辑数据库时会存在很多限制和额外的复杂性（比如在删除命令），但这是解决非树形结构数据的的解决办法之一。
        复杂性来源于两个方面。第一，用于操作的是数据的视图，而更显需要映射到不同的结构中。即使在关系型数据库中，对视图的支持也是很困难的，而层次结构的复杂性使得这个问题变得更复杂。第二，并发指示器定义在视图中，也必须映射到实际数据库中的其他并发指示器中。这个映射关系异常复杂。
        这种逻辑数据库的复杂性是之后十年IMB决定如何支持关系型数据库的关键考虑因素。
        我们总结下已学到的课程，然后进入CODASYL时代。
        1：物理和逻辑独立性是刚需。
        2：树形数据结构存在很多限制。
        3：提供友好的逻辑树形数据结构是很有挑战的。
        4：record-at-a-time 用户接口要求程序员进行手动查询调优，而且非常困难。

三、CODASYL 时代
        在1969年，CODASYL(Committee on DATA Systems Languages)组织发布了他们的第一个报告[CODA69],接着是1971的[CODA71] 和1973的[CODA73]关于数据语言的说明。CODASYL是一个自发组织的委员会，提出了包含了record-at-a-time操作语言的有向图数据结构。

        这个模型组织了一系列记录类型，每一个都包含索引，存放于一个有向图而不是树形结构中，于是，一个给定的实例可以有多个父节点，而不是IMS中的唯一的父节点。结果是，我们的supplier-parts-supply例子可以用上图来表示。
        这里，我们发现三种记录类型存在于这个有向图中，分别用supplies和supplied_by两条线来连接。一个命名弧线在CODASYL中称为一个结合类型，即使他在技术上并没有描述任何集合。但它指出了对于每一个箭头末端的实体拥有0个或者多个子记录类型（箭头的头部）。因此，在父记录实体和子记录实体之间存在一个1：n的关系。对于每一个父记录，这个结合类型下存在一个实体集合。在这个集合中是父记录和与之关联的所有成员记录。
        下图展示了一些实体样例。

        一个CODASYL有向图由命名记录类型集合和表示连接有向图的命名集合类型的集合组成。一个CODASYL数据库是由记录实体集合和表示有向图关系描述的实体集合组成。
        注意在上面层次数据模型中不支持独立性。比如，可以存一个不由任何supplies供应的part。在这里则是用一个空的supplied_by集合来表示。于是，迁移到有向图数据模型解决了很多层面模型的限制。然而，在CODASYL仍然存在一些复杂的场景。比如，关于婚礼仪式的数据，在新娘，新郎，牧师三者之间存在3条关系。因为CODASYL集合只存在2条关系，如下图：

    
        解决办法需要三组二进制集合来表示三维关系，这样显得不自然。虽然比IMS显得适合一些，CODASYL依然存在一定的限制。
        CODASYL 数据操作语言是一种record-at-a-time的查询语言用户通过查询一个记录类型以及其关联的集合来获取数据。要查询由supplier 16 提供的红色parts，在CODASYL中使用一下的命令：
Find Supplier (SNO = 16) 
Until no-more {
    Find next Supply record in Supplies
    Find owner Part record in Supplied_by Get current record
    -check for red—
}

    通过supplier 16访问数据库，然后迭代获取这个supplier下的供应集合。这样将得出一个supply记录的集合。对于其中的每一个，supplied_by的拥有者是确定的，然后进行判断是否为红色。最终，supplier集合获得，循环结束。
    CODASYL语言建议每个记录类型的记录可以在索引上进行hash或者使用一些距离的拥有者（译注：主键）聚集存储。许多时候，许多父记录和子记录的指针混合可以用于集合的某几种操作中。CODASYL事实上没有提供任何的物理数据独立性。例如，在上面的例子中，如果supplier的索引从sno变为其他列，查询程序将失败。考虑到逻辑独立性，CODASYL提出了逻辑数据库的概念，逻辑数据库由记录类型的子集和物理数据库的集合类型组成。于是，记录类型和集合类型可以被加入到CODASYL数据库，而之前的模式版本定义为一个逻辑数据库。对于更复杂的过程在IMS没有考虑支持。
    使用有向图数据结构的好处对于比如我们的样例数据中的多对多关系支持。然而，CODASYL模型被认为别IMS的数据模型更复杂。在IMS中，一个程序员通过层次空间导航，而CODASYL程序员通过多级关系来导航。在IMS中，程序员只需要关系数据的当前位置和记录的祖先记录的位置（如果他执行命令“get next within parent“）。
    在CODASYL中， 程序员需要关系是：程序最后获取的记录，每种记录类型最近被获取的记录，每种集合类型最近被获取的记录。
    CODASYL的很多命令更新了这些并发执行器。于是，在一个记录被定位前，程序一直在移动CODASYL数据库中的并发执行器 。然后，这条记录被获取到。另外，CODASYL程序可以降低并发移动问题，如果他希望的话。所以，对于一个CODASYL程序员来说，他面对一面钉满了并发指示器图钉的CODASYL有向图地图。在1973年的Turing Award 演讲中，Charlie Bachmann称之为”navigating in hyperspace“[bach73].
    于是，CODASYL面临不断增加的非层次数据的组织复杂问题。相比IMS，CODASYL提供较少的逻辑和物理独立性。
    关于CODASYL还有一些细节问题。例如，在IMS中每一个数据库可以从一个外度数据源进行独立地批量导入 。然而，在CODASYL，所有的数据存在于一张巨大的网络中。这个更大的结构只能一次性全部批量导入，导致需要更多的时间。当然，如果一个CODASYL数据库中断，需要从备份中进去重新全部加载。于是，崩溃恢复相比数据分隔成多个独立的数据库来说更需要解决。
    另外，CODASYL加载程序由于大规模的记录需要关联到集合中，会变得更复杂，同时带来大量的磁盘消耗。因此，考虑加载程序的算法和优化是非常重要的。于是，并没有通用的CODASYL导入工具，每个安装者需要编写他自己的。在IMS这个问题复杂度更小。
    于是，从CODASYL中学到的是：
        5.有向图比层次模式更适合但也更复杂
        6.有向图的加载和恢复比层次模型更复杂。

四、关系时代
    在这样的背景下，Ted Codd 在1970提出了关系模型[CODD70]。在之后和他的一次交谈中，他指出研究关系模型的动力是因为在逻辑或物理（结构）改变时，IMS程序员花费了大量的精力在处理IMS程序上面。于是，他专注于思考如何提供更好的数据独立性（的数据结构）。
    他的三个主要指标：用简单数据结构存储数据（表），通过高级模式的set-at-a-time DML访问数据，不关系物理存储结构。
    使用一个简单的数据结构，更容易提供数据的逻辑独立性。使用高级语言，可以提供更好的数据物理独立性。于是，这样就没必要去指定在IMS和CODASYL中要求的特定的物理存储方式。
    关系模式和供应关系样例数据在论文开头的两张图中。查询供应红色part的供应商的SQL如下：
    
Select S.sno
From Supplier S, Part P, Supply SS
Where S.sno = SS.sno and SS.pno = P.pno and P.pcolor = “red”
    此外，关系模型额外的优势是它更容易表示几乎所有的数据。于是，在IMS中的存在依赖问题（译注：对父节点）在关系模型可以轻松解决。另外，CODASYL中的关于婚礼的三维关系问题在关系模型中可以表示为：
Ceremony (bride-id, groom-id, minister-id, other-data)
    Codd 在那几年提出了多种关系模式报告（越来越适用），如[CODD79]。此外，他早期关于DML的报告是关于关系微积分（数据语言）[CODD71a]和[CODD72a]中的关系代数。由于他之前是一名数学家（从事细胞自动机），所以他的报告严格且标准，但不便于理解。(译注：A Relational Model of Data for Large Shared Data Banks)
    Codd的报告很快引起了著名的辩论，在1970年代持续了较长时间。辩论在SIGMOD会议上激烈展开（前身是SIGFIDET）。一方是Ted Codd和他的 “跟随者”（多数是研究者和高校老师），主要观点：
        a) CODASYL是非常复杂的，没有之一。
        b) CODASYL不提供可访问的数据独立性
        c) record-at-a-time程序非常难于优化    
        d) CODASYL 和IMS都不便于描述一些常见场景（比如婚礼场景）
    而另一方则是Charlie Bachman 和他的 ”跟随者“（多是DBMS从业者 译注：工业），主要观点：
        a) COBOL程序员不可能理解新的关系语言。
        b) 关系模型无法快速实现。（译注：codd的论文偏数学理论，不能很好地指导数据库的研发）
        c) CODASYL可以提供表，还有更大的主意？
    这场辩论的高潮（或低潮）其实主要是SIGFIDE’74会议中在Codd和Bachman之间以及他们各自的时刻[RUST74]。我们一位同事正好在现场，可以肯定的是双方都没有清晰地论述自己的立场。最终，双方都不能听到对方想表达的观点。
    在接下里的几年里，双方都对自己的立场做出了或多或少的调整：
    关系模型支持方：
        a）Codd是数学家，他的语言不是最佳选择。SQL[CHAM74] 和QUEL[76]更友好。
        b）system R[ASTR76] 和INGRES[STONE76]证明快速实现Codd的关系模型是可行的。此外，查询优化者可以和所有人竞争，但是最好的程序员会设计查询计划。
        c）各系统证明物理数据独立性是可以实现的。此外，关系视图[ston75]相对CODASYL来说提供了更好的数据逻辑独立性。
        d）相比record-at-a-time来说，set-at-a-time 语言大幅提高了程序员的生产效率。
    CODASYL支持方：
        a）提供set-at-a-time网络模型查询语言是可能的，比如LSL[TSIC76]就提供了完整的数据物理独立性和更好的数据逻辑独立性。
        b)   可以对网络模型进行清理[CODA78],它不是那么神秘。（译注：网络模型的难以维护问题）
    于是，双方都回应了对方的批评。这场辩论慢慢平息，而注意力转移到了商业界如何实现和将发生什么。
    微型计算机革命和VAX的兴起，对于关系模型方式一个偶然的机会。我们应重视机器类型的重要性。以前，人们只能在16位机器（PDP 11’s）或者批处理服务器。在内存地址限制在16位的服务器上编程是一件有挑战的事情，如[STON76]所述。批处理明显存在很多缺点。VAX将32位计算放到一个可接受的价位，之后迅速发展流行。
     VAX明显是早期商业关系数据库的目标，比如ORACLE和INGRES。关系模型方高兴的是，主要的CODASYL系统，比如Culinaine Corp的 IDMS。在IBM的编译器中编写，不便于携带。于是，早期关系系统利用VAX市场。这样他们有充分的世界来提高他们产品的性能，VAX市场和关系系统携手一起走向成功。
    在主机上，另一个完全不同的故事在上演。IBM相继推出了基于低端操作系统的system R的派生产品：基于VM/370和VSE。然而，这两个系统都没有被大量商业用户使用。所有的行为都发生在高端操作系统MVS.这里，IBM继续销售IMS,Cullinaine 成功卖出IDMS，而关系系统无处可寻。
    于是，VAX是关系模型的市场，而主机则是非关系模型市场。那时所有的序列数据管理系统都在主机上实现。
    这个情况在1984年IBM宣布即将发布基于MVS的DB/2 数据库。结果是，IBM开始声称IMS是他的DBMS作为两条数据库产品战略之一。由于DB/2采用新技术而且更方便使用，所有都能判断谁能长远发展并取得胜利。
    IBM指出关系系统是一个非常重要的分水岭。首先，它结束了一劳永逸的”著名辩论“。由于当时IBM占领了大量市场，他积极地宣布关系系统获得胜利，而CODASYL和层次系统失败。不久后，Cullinaine 和IDMS也加入了市场竞争。其次，他们很快发现SQL是关系模型的标准语言。其他查询语言（或许有更好的），比如QUEL很快就会死掉。在[DATE84]中对SQL的语义做出了严厉批评。
    

        有一个已知的事实需要讨论一下。IBM在IMS上层加一个关系终端是很自然的事情，如上图。这个结构允许IMS的客户继续使用IMS。新的应用可以和关系模型接口进行交互，如此便提供了一个友好的迁移到的新技术的路径。于是，从DL/1到SQL的转变过程慢慢发生，而这个过程中，保留了IMS高性能的关键特性。
        事实上，IBM趋向于应用这个策略，并成立了一个Eagle的项目。不幸的是，由于语义问题，最后发现在IMS的逻辑数据库之上完善SQL接口非常困难。于是，逻辑数据库的复制问题重新困扰了IBM许多年。最终，IBM强制调整了数据库战略，并声称了一场辩论的胜利者。
        总体来说，CODASYL和关系模型之间的这场辩论以三个事件告终：
        a）VAX的成功
        b）CODASYL机器的不方便携带
        c）IMS逻辑数据库的复杂性
        从关系时代我们学到的总结：
        7.set-at-time 数据查询语言更好，无论是那种数据模型，因为他们推进了物理数据独立性。
        8.逻辑数据独立性在简单数据模型中早于复杂。
        9.技术辩论通常会以商业竞争而结束，而且经常和技术本身相关性不大。
        10.查询优化者可以击败所有人，record-at-a-timeDBMS中最好的应用程序除外。（译注：他们总是要寻找最优的访问路径。）

五、实体-关系时代
    在1970’s中，Peter Chen提出了实体-关系（E-R）数据模型，用于替代关系模型，CODASYL和层次数据模型[CHEN76]。本质上说，他认为一个数据库可以想象成实体实例的集合。简单地说就是存在一些对象，和数据库中其他实体互相独立。在样例数据中，Supplier 和parts可以认为是实体。
    另外，实体拥有属性，这些数据元素用于描述实体本身。在样例数据中，Part的属性是pno，pname，psize 和pcolor。这些属性中的一个或者多个可以被设计为唯一，从而用来作为索引。最后，实体间存在关系。在样例数据中，supply就是实体part和supplier之间的关系。关系可以是1：1，1：n，n:1或者m：n，取决于实体在关系中的形式。在样例数据中，一个supplier可以供应多个part，一个part也可用由多个supplier供应。于是，供应关系是多对多的关系。关系同样也拥有属性，在样例数据中，qty和价格就是关系supply的属性。
    描述E-R模型的通用方法是使用”boxes和arrows“来表示。E-R模型从未从底层DBMS的数据模型中收益，可能是因为在早期没有提出适合它的查询语言。也可能是仅仅被1970’s提出的关系模式给淹没了。也许他更像一个”clean up“ 的CODASYL模型。不管是什么原因，E-R模型在1970’s并没有流行起来。
        
    E-R模型没有在任何领域取得广泛应用和成功，即数据库设计领域。对于关系型数据库的使用者，标准的做法是在初始化的构建一大堆表。然后，他们在初始设计上进行规范化应用。所以，在1970’s一系列的标准格式被提出，包括第二范式（2NF)[CODD71b],第三范式[CODD71b],Boy-Codd范式（BCNF）[CODD72b],第四范式[4NF][fagi77a],以及 project-join 范式[FAGI77b]
    在对现实世界数据库设计中，范式存在两个问题。首先，DBA会立刻问”我如何得到一个初始表集合“，范式对于这个重要问题没有答案。第二个可能更重要，范式理论基于函数依赖，现实世界的DBA并不能理解这个结构。于是，数据库基于范式设计就是”dea in the water“。
    不同的是，E-R模型成为了非常流行的数据库设计工具。Chen的论文中包含了建立初始E-R图的方法。另外，也可以简单地将一个E-R图转换为符合第三范式的初始的表结合[WONG79]。于是，DBA拥有了这样一个自动转换工具。于是DBA仅仅使用方框和箭头在画图工具上面就可以设计E-R模型，最终他会得到一个不错自动生成的关系数据库。本质上，所有的数据库设计工具，例如来自Magna 的Silverrun ，来自Computer Associate的ERwin，以及来自Embaracadero的ER/Studio 在这个事情比较火。
    11：函数依赖对于普通人来说比较复杂，理解起来很困难。另一个KISS的理由（keep it simple and stupid）。

六、R++ 时代
    从1980’s早期开始，大量的论文出现了，可以用下面的模板来概括：考虑一个应用，称之为x，在一个关系型DBMS之上实现这个应用，展示复杂的查询或者比较差的性能表现，给关系模型添加了一个新的特性来解决这个问题。
    这些x包含机器CAD[KATZ86],VLSI CAD[BATO85],文本管理[STON83],时间[SNOD85]和计算机图技术[SPON84]。这些论文共同组成了R++时代，因为他们都是在完善关系模型。我们看来，其中最优秀是的Gem[ZANI83]。Zaniolo 提出了添加以下特性到关系模型中，均是对查询语言的扩展：
    1)set-value特性。在表part中，经常出现这样的场景，像available_colors这样的属性可能包含多个值。添加一个set数据类型让关系模型更好地出现集合问题。
    2)聚合（tuple作为数据类型）。在供应关系中，存在两个外键，sno和pno，在其他表中指向这个元组效率更高。使得supply表支持这样的结构明显会更加清晰。Supply (PT, SR, qty, price)
    这里PT的数据类型是part表中的元组，SR的数据类型是supplier表中的元组。当然，实现支持这种数据类型的方法是某种指针。基于这样的结构，我们可以通过下面的语句来查询供应了红色part的supplier：
Select Supply.SR.sno
From Supply
Where Supply.PT.pcolor = “red”
    通过这种级联点的方式，可以使得查询在supply表时快速地关联其他表。这种级联点的方式和高级网络模型查询语言如LSL中的扩展路径很相似。他允许人们在表之间查询而不需要明确地使用join。
    3）归纳。假设我们的样例中有两种类型的part，称为electrical 和plumbing。对于electrical parts，我们记录电力的消耗和电压。对于plumbing part，我们记录它的直径和制造所使用的材料。如下图所示，存在一个part的root 类到细分领域。每个领域从他的祖先节点集成了数据属性。
    继承制度最早出现在如Simula[DAHL66],Planner[HEWI69]和Conniver[MCDO73]等编程语言中。其相同的组件也包含在现在的编程语言中，如C++。Gem是包含这个组件的早期数据库例子之一。

在Gem中，人们可以在查询中引用一个继承关系。例如，查询红色的electrical parts的名称语句可以是：
Select E.pname
From Electrical E Where E.pcolor = “red”
另外，Gem 对于null的处理也非常优雅。
    扩展这一类特性带来的问题则是，相对传统的关系模型允许更加简单的查询标准，    他们带来了很小的性能提升。比如，在关系模型主键-外键关系可以类比一个元组。此外，外键在本质上还是逻辑指针，这个构建的性能和其他类型的指针差别不大。于是，关于Gem的完善不会比对关系模型的完善速度更快。
    在1980’s早期，人们突出地聚集如何提供事务性能和系统的扩展性，从而让他们可以应用在大规模，可扩展的数据处理应用中。当时这是一个非常大的潜在市场。R++并没有产生较大的冲剂，于是，只有很少的技术从R++转化到商业产品中，而本论文主要关注造成较大冲击的研究。
    12：除非有比较大的性能提升或者函数功能优势，任何组件都得不到推广。

七、语义数据模型时代
    于此同时，另一个学校也提出了类型的观点，但是出于不同的商业战略。他们认为关系模型是”缺乏语义“的，也就说关系模型在加速某些数据类别时候非常乏力。于是，需要出现一个”post relational“的数据模型。
    post relational数据模型一般又称为语义数据模型。在Smith 和 Smith[SMIT77]和Hammer 和Mcleod[HAMM81]中包含了相关的样例。Hammer和McLeod发表的SDM论证了语义模型的紧密型，在本节中我们关注它的构成。
    SDM关注类别概念，一个类别则是同一个模式的记录集合。和Gem一样，SDM提出了聚合和概括的构建方法，同时也包括集合概念。聚合允许类别包含其他类别中记录的属性。然而，SDM 通过允许类别中的一个属性是同一个类别中的记录实体集合来形成聚合。例如，假设存在两个类别，ships 和 country。country类可能包含属性ships_registered_hers,值包好在在ships集合中。相反的，country_of_registrtion也可以在SDM进行定义。
    另外，类别也概括了其他类别。和Gem不一样的是，概括的扩展是图的方式而不是树。比如，在下图中，展示了从Oil_tanker和 American_ship继承属性的American_oil_tankers的图概括。这种结构通常称为多重继承。类别也可以说是唯一的，和其他类别交叉或者不一样。类别同样也可是另一个类的子类，通过声明一个成员关系断言来实现。比如，heavy_ships是ships的子类，包含一个重量超过500吨的条件。最后，一个类也可以是因为某个原因组合在一起的记录集合。例如Allantic_convoy可以是ships中那些一起穿越大西洋的ships的集合。
    最后，类别可以有类别变量，比如ships可以有一个类别变量为类别中的成员数量。
    许多语义数据模型都非常复杂，一般都在论文中。SDM定义后许多年，Univac 实现Hammer和McLeod 的想法。然而，他们很快发现SQL是国际标准，而他们的系统在市场中也没有取得很好的成绩。

    我们看来，SDM 和R++一样也面临两个问题。和R++报告一样，有很多机器可以在关系模型上进行模拟。于是，只有很多的理论被付诸实践。SDM方同时也面对R++的另一个问题，一般都会关系事务的处理性能问题。于是语义模型只流行了很短一段时间。
    
八、OO时代
        从1980’s 中期开始，出现了面向对象数据库OODB潮流。本质上，主要是指出了关系型数据库和面向对象的编程语言直接的不一致问题。
        在实践中，关系型数据库有他们自己的命名系统，自己的数据类型系统和返回查询结果的接口。然而，于之协同运行的编程语言成祥也拥有这些特性。于是，编程语言方式和数据库方式之间的转化需要被缓解。这就像”gluing an apple onto a pancake “，也就是所谓的不一致的原因。
    比如，下面的C++代码定义了part结构体和分配另一个example_part。
Struct Part {
Int number;
Char* name; Char* bigness; Char* color;
} Example_part;

所有包含主机SQL运行系统才能够上面的结构体中加载数据库中的数据。比如，要加载part16的数据，需要下面的代码：
Define cursor P as Select *
From Part Where pno = 16;
Open P into Example_part 
Until no-more{
    Fetch P (Example_part.number = pno, 
                   Example_name = pname
                   Example_part.bigness = psize 
                   Example_part.color = pcolor)
}
    首先定义个用于遍历SQL查询结果的指针。然后，打开游标，遍历查询结构并绑定到代码的变量中，不需要相同的名称和对应的数据类型。如果需要的话，数据类型的转换会由实时接口自动完成。
    程序员这个时候就可以在本地的程序中进行调使用这个结构体。如果查询结构超过一条，则需要像上面一样遍历游标。
    看起来这样使得DBMS和开发语法更加一体化。特别是一些持久化开发语言，比如变量在磁盘，内存，数据库中都是相同的结构体。很多持久化开发语言的协议在1970‘s后期出现，包括Pascal-R[SCHM77]，Rigel[ROWE79]，以及PL/1中的嵌入程序。比如，Rigel中可以这样写上面的查询：
For P in Part where P.pno = 16{ Code_to_manipulate_part
}
    在Rigel中，和在其他持久化语言中一样。变量可以被定义，本例中是pno。在Rigel中它只需要被定义一次，而不是在程序中一次，在数据库中一次。另外，条件p.no=16也会程序代码的一部分。最后，使用标准程序迭代（使用循环）来遍历所有匹配记录。
    一个持久化语言明显要比SQL嵌入要清晰很多。然而，这就要求程序的编译者扩展面向数据库的功能。因为没有世界通用的编程语言，这个扩展需要每个编译者都来一遍。而且，每个都可能是唯一的，因为C++非常特别，和APL不一样。
    不幸的是，编程语言专家拒绝关注I/O和DBMS具体特性。于是，所有我们提到的开发语言在这块都没有内置这些特性。这不仅会使得嵌入了子语言，而且不利于程序编码和错误诊断。最后，语言专家并没有开发面向数据的开发语言，比如这边报告者和所谓的第四代语言。
    所以，在1970’s，没有技术厂商将持久化开发语言应用到商业市场中，也包括嵌入数据语言。
    在1980‘s中期，固定语言出现了复兴，由C++的兴起而驱动。本问主要研究面向对象数据库（OODB）和聚焦在持久化C++。虽然早期出现了一些系统研究如Garden[SKAR86] 和 Exodus [RICH87]，OODB的兴起是因为一群创业公司推动，包括Ontologic, Object Design 和 Versant。他们都使用了持久化C++并建立了商业模式。
    这些系统的标准格式是以C++作为数据模型。于是，任何C++的结构体都可以持久化。因此用关系模型来扩展c++流行起来，直接从十年前提出的实体关系模型中借过来的概率。于是，很多系统通过扩展C++系统来提供这样的特性。
    很多团队决定将定位工程师数据库作为他们的目标市场。典型的历史就是CAD工程师。在一个CAD应用中，一个工程师打开工程设计界面，绘制电子电路图，然后修改工程项目，测试，或者在电路上面进行模拟。当他完成后就关闭这个工程。这一类工程的标准模式就是一开始打开的巨大的工程项目，然后在关闭之前各种运行和调试。
    历史上，这些对象通过一个加载程序读到虚拟内存中。这个程序会将存储在磁盘的对象调配转换到虚拟内存中的C++对象。调配的意思是指任何需要在加载过程中进行的指针修改。在磁盘上，指针指示某种逻辑应用，例如外键，虽然他们也可以是磁盘指针，比如（块号，偏移量）。在虚拟内存中，则指的是是虚拟内存指针。于是，加载器需要将磁盘形式转换位虚拟内存形式。然后，代码可以操作这些对象，经常会使用很久。结束的时候，卸载器会链接c++数据结构到持续化磁盘中。（译注：逆向更新回去）
    为了定位工程师市场，持久化c++完善工作有一下需求：
        1）不需要发起查询。只需要将磁盘上的大规模工程转换到c++中。
        2）不需要事务管理。这个市场很大，但是都是单用户使用这个巨大的工程。当然，一些版本控制还是很好的。
        3）在操作项目时要和传统的c++工程一样。在这个市场中，持久化c++的性能优化算法要和一个用户自定义的程序以及传统c++程序进行竞争。
    自然，OODB厂商瞄准了这些需求。于是，对于事务和查询的支持就非常弱。而他们更关注如何优化持续化c++结构体的性能上面。比如下面的定义:
Persistent int I;
And then the code snippet: 
I =: I+1;
    在传统c++中，这个是单独的指令。为了优化，增加一个持久化变量不能调用转换为持续化对象的过程。于是，DBMS必须和程序运行在同一个地址空间。同样的，工程师项目必须缓存在主内存中，并缓慢地写回磁盘。
    于是，OODB的商业产品比如Object Design[LAMB91],通过改变这些结构来达到目标。
    不幸的是。这样的工程应用市场并不是非常大，而同时又有太多的厂商在抢夺这个小众市场。在那段时间，很多OODB厂商都失败了，或者改为别的项目上去了。比如，Object Design 重命名为Exelon销售XML服务。
    在我们看来，这个市场失败的原因有：
        1.缺乏扛杆。OODB厂商为用户提供了不需要变成加载器和卸载器的方便。这不是主服务，用户不会为这个小功能付太多的钱。
        2.没有标准。所有的OODB厂商互相之间是不兼容的。
        3.重连世界。在任何改变中，比如c++中修改某个持久化数据的方法，然后所有使用这个方法的程序需要重新连接。这明显是一个管理问题。
        4.没有世界通用的语言。如果客户的企业有一个需要访问持久化数据的程序，而这个程序不是c++编写的，那么这个客户就不会使用OODB产品。
    当然，OODB产品并不是为商业数据处理应用设计的。他们不仅仅缺乏强壮的事务和查询控制系统，而且他们和应用程序运行在同样的内存地址。这意味着应用可以自由地访问磁盘数据，而数据并没有任何保护措施。保护和认证在商业系统中是非常重要的。另外，OODB的经历和CODASYL的类似，是一个低级的record-at-a-time，由程序员来编写查询优化算法。结果则是这些产品无法进入商业数据处理市场中。
    13：除非是”痛点“，否则用户不会花钱购买软件。
    14：没有开发语言社区的支持，持久化语言无法得到推广。

九、对象-关系时代
    对象-关系时间由一个简单的问题引起。在ingres早期，团队关注于地理系统系统（GIS），并且也建议了他们支持的配置[GO75]。在1982附近，不断出现的GIS简单问题困扰这INGRES研究团队。比如某人想把地理位置信息存储到数据库中。比如，要存储一个交点信息：Intersections (I-id, long, lat, other-data)。
    这里，我们需要存储地理点（long，lat）在数据库中。然后，如果我们找出一个指定矩阵（x0，y0，x1，y1）里面的所有交点，查询SQL为：
    
Select I-id
From Intersections
Where X0 < long < X1 and Y0 < lat < Y1
    不幸的是，这是一个二维查询，在INGRES中的B-tree是一维访问策略。一维访问策略在这样的二维查询中显得很不高效，所有在关系型数据库中无法高效地运行。
    更多的问题是”通知包裹主人“问题。在加利福利亚关于包裹投递的地方法律有一个分歧，在任何时候都要公开，而且所有特定距离内的包裹主人都要被通知到。
    假设某人假设所有的包裹都是矩阵的，他们在表中的存储为：
Parcel (P-id, Xmin, Xmax, Ymin, Ymax)
    于是，他必须使用正确的距离来放大包裹问题，创建一个超大矩阵（x0，x1，y0，y1）来实现。所有在这个超级矩阵中的主人将被通知到，那优化的查询方式为：
Select P-id
From Parcel
Where Xmax > X0 and Ymax > Y0 and Xmin < X1 and Ymax < Y1
    还是一样，在B-tree结构无法快速去执行这个查询。另外，还需要花时间证明这个查询是对的，因为存在多种其他更低效的方法。总的来说，简单的GIS查询在SQL变得很困难，在表中的B-tree结构中性能非常糟糕。
    接下来是OR模型的发展观察。早期关系型系统支持整形，浮点和字符串，以及对应的操作，主要是因为这些是IMS中早已实现的数据类型。IMS选择这些数据类型是因为这些是他们目标商业数据市场需要的。关系系统同样也采用了B-tree是为加速常见商业数据系统的查询速度。后来关系模型系统也随着商业市场数据处理流程扩展了日期，时间和钱等数据类型。最近，decimal和blob类型也添加到其中。
    在其他市场中，比如GIS，这些不是想要的类型，b-tree也不是想要的访问方式。于是，为定位目标市场，数据类型和访问方式也要贴合市场需求。因为还有许多别的市场，最好不要去固化特定的类型和索引策略。比起普遍适配，用户应该添加他需要的，比如根据需求自定义DBMS。通常来说扩展设备也是为了更好地进行商业数据处理，因为每十年都会有更多的新的类型需求出现。
    最总，OR模型在SQL引擎中添加了下面的特性：    
        用户自定义数据类型。
        用户自定义操作。
        用户自定义函数
        用户自定义访问模式
    关于OR模型的主要研究是Postgres[STONE86]。
    将OR模型应用到GIS，用户只需要添加地理位置和地理矩阵作为数据类型。这样，上面的表变成了这样：Intersections (I-id, point, other-data) Parcel (P-id, P-box) 。
    当然，每一种数据类型也需要对应的SQL操作。在我们简单应用中，有！！（矩阵中的点）和##（矩阵中的矩阵）。则可以有这样的查询：
Select I-id
From Intersections
Where point !! “X0, X1, Y0, Y1” and
Select P-id
From Parcel
Where P-box ## “X0, X1, Y0, Y1”
    为了支持用户自定义操作符，必须得先支持用户自定义函数，才可用调用对应的操作。于是，在上面的例子中，我们需要下面的函数。
Point-in-rect (point, box) and
Box-int-box (box, box)
    函数返回的是布尔类型。每当对应的操作被调用，这个函数就要被调用，并传入参数，返回运算后的结果。
    为了满足GIS市场的需求，需要多维空间索引系统，比如Quadtrees[SAME84]或者R-trees[GUTM84]。总的来说，一个高性能的GIS DBMS可以使用用户定义的数据类型，用户自定义操作，用户自定义函数和用户自定义访问方法来组成。
   Postgres最大的贡献就是指出了支持这种扩展性的引擎机制。事实上，之前的关系引擎只能通过硬代码来实现对一些列数据类型，操作和访问方法的支持。 这些硬代码应该由更好的适配机制来实现。更多关于Postgres的详细设计见[STON90]。
    这是对于UDF的另一种理解。在1980’s中期，Sbase宣告在DBMS中支持了存储过程。基本的想法则是提供高兴的TPC-B，包含了下面的代码来模拟现金检查。
Begin transaction
Update account set balance = balance – X Where account_number = Y
Update Teller set cash_drawer = cash_drawer – X Where Teller_number = Z
Update bank set cash – cash – Y
Insert into log (account_number = Y, check = X, Teller= Z) Commit
    这个事情需要应用和DBMS之间进行5-6此信息交互。因为上下文切换消耗在简单处理流程中，应用的性能被上下文的切换时候所限制。
    可以用一个更聪明的办法来减少这个时间：定义个一个存储过程。
Define cash_check (X, Y, Z)
 Begin transaction
Update account set balance = balance – X Where account_number = Y
Update Teller set cash_drawer = cash_drawer – X Where Teller_number = Z
Update bank set cash – cash – Y
Insert into log (account_number = Y, check = X, Teller= Z) Commit
End cash_check
    然后，应用直接调用这个存储过程，传入必要的参数，比如：Execute cash_check ($100, 79246, 15) 。
    这样只需要应用程序和DBMS直接进行一次交互而不是5-6次，极高地提升了TPC-B。为了在基准测试比如TPC-B上跑的更快，所有的厂商都支持了存储过程。当然，这需要他们定义专门的开发语言来处理错误和请求控制。当一个账户中出现余额不足这种情况是，需要存储过程可以正确处理。
    存储过程是用用户使用专门的语言开发的UDF，这种情况下他只能按照定义的参数来调用来运行。Postgres UDTs和UDFs允许用户使用通用的开发语言来编写来实现这个概念，被称为sql通用SQL查询。
    Postgres 为UDTs，UDFs和用户定义访问方式定义了一个灵活的机制。另外，Postgres 也对继承，指针的类型构造，集合和数组的少量支持。这些特性使得Postgres成为在OO热潮中的面向关系的高度。
    后来通过Buck[CAR97]在基础测试上面的努力证明了Postgres的主要成功是UDTs和UDFs;OO的构建在传统关系系统中非常模拟和高效。关于R++和SDM几年前已经发现的示范工作再一次出现；内置聚合和概括提供非常少量的性能效益。换句话说，OR最大的贡献是提出了一个更好的存储过程和用户自定义访问方法的机制。
    OR模型也获得了一些商业成功。Postgres 由Illustra公司运营。经过头几年的市场奋斗，Illustra 赶上了互联网潮流，并成为了通信数据库。如果用户想存储文本和图片到数据库中，并和传统数据类型混合起来，Illustra则是其中的存储引擎。随着互联网热潮的兴起，Illustra 被Informix收购。从Illustra的角度来看，有两个加入Informix的原因：
    a）在每一个OR应用中，存在一个处理事务的子应用。为了在OR中成功，也就是在OLTP场景下获得更好的性能。Postgres从未关注过OLTP性能，而且Illustra添加这个特性的花费也非常大。
    b）为了成功，Illustra也说服了不少厂商将他们的应用适配到UDTs和UDFs。这并不是一个琐细的企业，而且很多外部厂商都被夸大了，至少在Illustra 可以拿下OR市场机会大量份额之前。于是，Illustra面临一个鸡和蛋的问题。为了获取市场共享，他们需要UDTs和UDFs;为了UDTs和UDFs他们需要市场共享。
    Informix 提供了一箭双雕的解决办法，联合公司在GIS领域OR技术里面推进中获取了很大的成功，以及大规模数据仓库的市场（比如CNN和BBC）。然而，商业数据处理对OR的大规模采用还是难以抓住的。当然，Informix财务方面的困难使得他在销售类型OR这类产品时非常困难。这个因素阻碍了广泛接受。
    OR技术逐渐获得了市场认可。例如，将数据挖掘算法使用UDFS来实现会非常有价值，有Red Brick提出，近期由Oracle添加。与其移动大规模的数据到数据挖掘中间件层，更好的方式是将代码到DBMS中，从而避免消息溢出。OR技术也被用于执行XML处理，正如我们现在看到的。
    OR技术在更大的商业市场没能获得广泛认可的原因是标准的缺失。每个厂商在定义和调用UDFs时都有自己的方法，另外，很多厂商支持java UDFs,但是微软不支持。除非大部分厂商愿意接受标准定义和调用方法 否则OR技术不会得到广泛使用。
    14：OR的主要效益是两个方面：将代码放进数据库中（缩短代码和数据之间的距离）；一个允许OR DBMS 快速响应市场需求的通用的扩展机制。
    15：新技术的广泛应用需要标准或者大力推广。

十、半结构化模型
    在最近五年关于半结构化的研究工作出现一波热潮。早一点的例子则是Lore[MCHU97]。最近，各种XML-based的论文是一样的味道。现在，XML模式和Xquery是XML-based 数据的标准。
    这一类的工作有两个基本点：
        1）后模式
        2）面向图的复杂数据模型
    在这一节里面我们分别讨论这两点。
    10.1 后模式
        关于这个概念有两种理解。第一种，模式在一开始不需要。在模式为先的系统中，模式是需要指定的。这个模式下的数据记录的实体才可以顺利加载。于是，数据库总是由提前存在的模式来组成，因为DBMS检测所有数据是否和模式一致。所有之前的数据模型要求DBA先指定一个模式。
        另一种对后模式的理解是最近出现的。他建议模式应该是不定的且可灵活改动的。一个模式先定义，但是当模式中的数据发送改变时，这个模式也要随之改变。我们称之为轻模式进化，来区分第一种看法。我们依次来讨论这两种看法。
    10.1.1 后模式
        在这个观点中，模式不需要提前指定。可以之后再指定，或者不指定。在一个后模式系统中，数据实体必须可以自描述，因为不需要模式来对未来的数据做出解释。没有自描述的记录，可以视为一堆字节。
        为了使得记录自解释，则需要对每一个数据增加元数据定义属性来描述字段的意义。一下是一些记录样例，使用了一些人员标记系统：
Person:
    Name: Joe Jones
    Wages: 14.75
    Employer: My_accounting
    Hobbies: skiing, bicycling
    Works for: ref (Fred Smith)    
    Favorite joke: Why did the chicken cross the road? To get to the other side Office number: 247
    Major skill: accountant
End Person
Person:
    Name: Smith, Vanessa
    Wages: 2000
    Favorite coffee: Arabian
     Pastimes: sewing, swimming 
    Works_for: Between jobs 
    Favorite restaurant: Panera
     Number of children: 3
End Person:

如上，两个对人员描述的记录。此外，每个属性有下面三个特点：
    1）他只出现在这两条记录中的一条中，而且在其他记录中不存在相同的意义。
    2）他只出现在这两天记录的一条中，但是有其他记录存在相同意义的属性。（比如pasties 和hobbies）
    3）他存在于两条记录中，但是格式和意义不相同（比如work for和wages）
很明显，比较这两个人是一个挑战。这是一个意义多相性的例子。一个常见对象的信息并没有以常见方式呈现。语义多相性使得查询面临巨大的挑战，因为没有能力为基础索引决策和查询执行策略的数据结构。
 对这种后模式的接受是因为允许用户可以自由地写入数据，或者做一些处理（可能为文本添加一些元数据来描述文档结构）。这种情况下，用户添加数据之前就没有必要要求模式先存在。后模式支持使用上面的结构或者半结构记录来自动或者半自动地标记未来写入的数据   。
  不同的是，如果一个业务表格是用于数据写入，（比如上面Person数据 ），则一个模式优先的技术将被采用，因为设计这个表格的人事实上也定义允许写入的数据格式。最终，后模式大部分应用在数据写入主要是自由文本的应用中。
    为了关键后模式的实用性，我们列举出下面四个模式来进行应用分类。
        使用固定结构化数据
        使用固定结构胡数据和少量文本字段
        使用半结构化数据
        使用文本
    固定结构化数据中包含的数据必须和模式一致。一般来说，这里面包含了所有业务流程操作的数据。比如，一个公司的工资单数据库，这个数据必须固定结构，或者支票打印程序将错误运行。业务流程依赖的数据不能丢失或者出现错误格式。对于固定结构胡数据，应该坚持结构优先。
    大型公司的员工系统则是典型的第二种应用。包含了一些固定数据，比如每个员工的健康计划，员工的福利待遇。另外，也存在一些文本字段，比如经理在员工访谈的备注信息。员工访谈表格是固定格式；于是唯一的文本输入是备注字段。模式优先仍然是正确的选择，这一类应用可以定位成包含一个text数据类型的对象-关系型DBMS。
    第三种类型是半结构化。能想到最好的例子是广告和简历。在每种场景中，有一些结构化数据，但是数据实体在表现和组织上的不同的。此外，对于实体没有需要的数据模式。半结构化实体经常以文本文档的方式写入，然后通过解析来获得需要的数据，在存储引擎中分解成多个字段。在这里，后模式是个好主意。
    第四种是纯文本。比如没有特定结构的文档。在这点上，没有具体的结构来展示。数据恢复系统（IR）专注这一类的数据很多年。一些IR研究者对半结构化数据也有一些兴趣；虽然他们关注基于文档文本内容的文档恢复。于是，在这一点上没有模式，也成为”无模式“。
    最后，我们的传统系统中后模式只能应用在第三种场景中。除了广告和简历外，很难想出很多这一类的例子。倡导者（主要是高校）经常建议大学课程说明适用于这类。然而，我们知道每个大学的课程描述都有固定表格，其中包含了一个或者多个文本字段。大部分对于数据写入有标准格式，而一个反解课程描述的系统（手动或者自动）并不适合这个格式。于是，课程描述是第二种类别的例子，而不是第三种。在我们看来，对于第三类应用实体认领的考虑往往会产生很少的第三类实体。此外，专业研究简历最大网站（monster.com）最近的一个业务表格采取了数据写入的格式。于是，他们从第三类转化为第二类，大概是为了和他们的数据库保持一致。（这样具备更好的比较性）。
    语义多相性和企业共存了很长时间。他们在数据仓库项目上马花费了大量聚合来设计标准的模式，然后转化运营数据到标准格式。另外，在大部分组织中，语义多相性用一个数据集合库来处理；使用不同模式的数据集将被转换。传统的数据仓库项目超出预算的，因为模式转换是非常困难的。任何后模式应用在记录接记录的数据库中将面临语义多相性问题。这样将会等导致需要更多的预算来解决。所以，如果可能，尽量避免后模式。
    总的来说，后模式只有在第三种应用场景才是最适合的。另外，很难找到这一类有说服力的例子。如果有，更趋向于转换为第二种，可以更容易解决语义多相性问题。最后，第三种应用经常会存在大量的数据。因为这些原因，我们任务后模式数据库是一个有前景的市场。
    10.1.2 模式进化
    当前的关系型数据模式有相当的原始和固定点可以用于模式进化。一个alter table 命令可以修改一个表的定义。之前的表定义可以定义为新表定义之上的一个视图。类似的方法，一个表可以拆分开，旧表可以定义为带join的视图。这种情况下新表必须从已经存在的表映射组成。至少存在两种方法来使得这个结构非常有用。
     首先，如果一个表被标记为可进化的是一件非常好的事情。这样，用户可以简单的写入数据而不去确认模式，系统会自动地执行一个ALTER Table命令。另外，并不是立马将全部数据转化为新的模式（在新模式是从就记录中拆分出来的），更好的方式是在后台执行一个缓慢的批量拷贝甚至不做操在。在这个场景中，系统需要处理数据问题，拆分旧数据和新数据直接的模式。这样的机制使得模式进化更加高效和易用。
     另一个方法是预留未来的扩展。在科学数据库社区，维持数据家族常常是很重要的。于是，不仅需要一个数据集合中的数据模式，也需要系统跟踪之前发生在数据上的操作。这可能包含数据清理的算法，任何数据库过滤转换操作。数据家族和卫星图形关系特别紧密。这里，一行数据常常是卫星经过的目标区域的图像集合，包括一些被云遮挡的部分。存储许多算法来选择每个张图片的区域从而合成一张完整的区域地图。
    科虚家使用这些源数据集合需要知道图片合成的算法，才能决定是否符合他的目的。于是，他需要执行数据集合中的家族关系。当前的DBMS没有支持数据家族的能力，而只有有一个社区是希望有更好的这方面的支持能力。
    我们相信模式进化是已存在的数据库非常缺乏的领域。更好的支持能力是可能的，我们希望商业产品可以调整方向。类型的评论可以通过版本控制来实现。

10.2 XML 数据模式
    我们再来看XML数据模型。过去，描述一个模式的方式是文档类型定义(DTDs),在未来，数据模型可以定义成XML模式。DTDs和XML模式都是面向格式化文档的结构定义（也就是DTDs中的文档）。最终，他们看起来像文档标记语言，作为SGML的具体子集。由于文档的结构可能会非常复杂，文档定义标准也需要非常复杂。作为文档说明系统，我们对这个标准没有争议。
    在DTDs和XML模式的固定，DBMS研究社区的成员决定尝试使用他们来描述结构化数据。作为结构化数据的数据模型，我们相信两个标准都是有缺陷的。比如，这些标准包含之前报告中已经指出的信息。另外，他们也包含额外的复杂功能，这些在DBMS社区之前的数据模型研究没有提及。
    例如，在XML模式中有下面的特性：
    1）XML记录可以继承，如同IMS。
    2）XML记录可以链接到其他记录，如同CODASYL，Gem和SDM
    3）XML记录可以包含集合层属性，如同SDM
    4）XML记录可以用多种方法从其他记录继承，如同SDM
    另外，XML模式也有一些在DBMS社区中非常出门但是由于过于复杂从未尝试的特性。比如union type，则是一条记录的属性可以是一个类型的集合。比如，在一个人员数据库中，列 work-for可以是一个企业中的部门编号，也可以是员工贷款的外部机构。这种情况下 works-for 可以是string类型或者整形来表达不同的意思。
    注意在B-tree中对于复合类型的索引是非常复杂的。结果就是需要对复合类型中的每个数据类型创建一个索引。另外，对于每一个访问复合类型的查询都必须有不同的查询计划。如果两个复合类型分别包含m和n个基础类型，在join的时候最多会出现Max（m，n）中执行计划。因为这些原因，复合类型从未被考虑加入到DBMS中。
    当然，XML模式离已经提出的复杂数据类型还远。很明显他和关系模型的KISS原因是另一个极端。很难想象某个复杂的类型被用于结构化数据。我们可以看到未来的三种情况。
        剧本1：XML模式由于太复杂导致了失败。
        剧本2：一个面向数据的XML模式子集更简单被提出。
        剧本3：XML模式成为流行。就像困扰IMS和CODASYL多年的问题促使Codd提出了关系的情况再次重现。那时候一些企业研究员，假设是Y，会重温Codd的论文，著名的讨论将再次出现。大概也会和之前一样结束。
另外，Codd 在1981[CODD82]因为他的贡献获得图灵奖。在这个场景中，Y也会在2015年获得图灵奖。
        对于”X stuff”倡导者的公证来看，他们从历史上学到了一些东西。他们提出了set-at-a-time的查询语言，Xquery，提供了一个程度的数据独立性。正如之前在CODASYL领域发现的，为图数据模型提供视图将会是一个挑战。（而且会比关系模型更难）。
    10.3 总结
    总的来说，XML,XML模式，Xquery是一个挑战，因为还有许多问题要处理。很明显，XML会成为在线网络数据传输的流行格式。原因很简单：XML穿过防火墙，其他格式不行。因为防火墙处于两个企业的服务器之间，他和跨企业数据移动一起移动。因为传统企业希望数据在内部移动和外部移动方式一样，这是为什么XML会成为未来数据移动标准的原因。
    最终，所有的系统和应用都必须做好使用XML来接受和传输消息。将关系型数据库产生的元组数据转换为XML是很简单的事情。如果有OR引擎，存在一个用户自定义的函数。类似的，他可以接受XML输入并转换为元组存储到另一个用户自定义的函数。于是OR技术加速了必要的格式转换效率。其他系统软件可能都需要一个转换工具。
    另外，更高级别的数据传输工具在XML顶层设计，比如SOAP，也会流行起来。很明显，可以穿过防火墙的远程存储过程调用比现在的更有用。于是，SOAP将会统治其他RPC协议。
    原生的XML DBMS可能也会流行起来，但是我们不讨论它。XML DBMS至少还要花费十多年才能成为一个可以和当下的数据库竞争的高性能引擎。另外，后模式只会应用到有限的市场中，而过于复杂的图结构数据模型和KISS原则形成对立。XML模式处出现各种子集，一个干净的XML模式子集可能有具备他从当前的关系型数据库中参考的特性。在哪种情况下，什么才是实现一个新引起的关键点？于是，我们期待原生XML DBMS成为了个广阔的市场。
    现在聊聊Xquery。许多厂商乐意将某个子集对应到OR SQL系统中。比如，Imformix用用户自定义函数 实现Xquery 操作’//‘。于是，在很多已有引擎的顶层实现Xquery的子集是很容易的事情。结果是，现在这些主流的数据库会同时支持SQL和一部分XML模式和异步Xquery。后续的接口再转换为SQL。
    XML有时候也用来解决前面提到的语义多相性问题。显而易见。仅仅是因为两个人员有一个标签是薪水并不表示两个数据元素是可以比较的。一个可以是包含了午餐补贴的法国的税后收入，单位是法郎。另一个则可以税前的美元。进一步说，如果你称之为为橡胶手套，而我称之为乳胶护手，XML在判断他们是不是同一个概念时没有用处。于是，XML的角色在提供常见模式提供的结构词汇方面是有限的。
    另外，我们相信企业之间的数据共享采用通用的数据模式即将到来，因为语义多相性问题难以解决。虽然W3C在这个方面有一个项目，所谓的语义网页，我们不看到这个功能的未来。总之，AI社区已经在知识展示方面做几十年的工作，取得了很少的结果。语义网页承受了类似这样的打击。因为web服务依赖于在不同的系统中传输数据，别过早的在这个概念上获得成功。
    更准确地说，我们相信跨企业的数据共享将会受到以下限制：
    合作可以拥有高经济价值。比如，航公公司已经和很多不同的预定系统分享数据。
    语义简单的应用（比如邮件），他的主要数据类型是text，而且也不包含复杂的语义映射逻辑。
    拥有控制了市场的应用。例如WalMart 和Dell这样的企业，他们和供应商直接的数据共享没有难度。他们只需要说：如果你想卖给我，这是和我的信息系统交互的方式。当有绝对的能力来制定标准，跨企业信息分享可以很容易实现。
    最后一个极端的例子。许多年前，OLE-DB由微软推出，现在他是“x stuff”。OLE-DB来自微软，因为他没有控制ODBC，而且觉察对OLE-DB的竞争优势。现在微软发现来自java巨大的威胁和他多种跨平台的扩展，比如J2EE。于是，通过模仿java的成功来推广XML和Soap来是非常困难的。
    无论那个理由我们都详细未来几年后，微软会面临来做其他面向DBMS的标准的竞争。和OLE-DB早早夭折一样，我们希望微软将x stuff早早处理，市场战略做出一些调整。
    少一些顾忌，我们坚信技术发展在不断改变规则。比如，很明显微型传感器技术在未来几年内会出现在市场，会对系统软件带来巨大改变，我们期待DBMS和相关的接口在某些方面（待指出）也受到影响。
    于是，我们期待新一代DBMS标准获得成功。不管下一个大事件是什么，这个数据库具备很强的适应性，从而改变世界。或者DBMS拥有某些特性，原生XML DBMS没有。
    16：后模式可能是一个潜在市场
    17： Xquery 相对OR SQL是一个更好的不同语法的查询语言
    18： XML 不能解决无论是企业内部还是企业外部的语言多相性问题

十一、 周而复始
    本论文调查了三十年来数据模式的思考。很明显我们又回到了原点。我们从一个复杂的数据模型开始，紧接着是关于复杂模型和简单者的著名辩论。简单者更便于理解并提供了数据独立性支持。
    然后一些列显著的功能加入其中，然而没有一个获得了显著的市场成功，大部分是因为他们没有能处理好功能和复杂度增长的平衡问题。唯一获得市场认可的是OR DBMS的扩展，而且拥有其他数据模型没有的性能构建。现在很多报告是之前所有报告的合并的超集。我们又回到了原点。
    关于XML支持者和关系型模型之间的讨论经历了类似25年前的著名辩论。一个简单的数据模型和一个复杂的进行比较。关系型模型被比喻成CODASYL 二代。唯一的区别是CODASYL 二代有更高级别的查询语言。CODASYL二代中的逻辑数据独立性比第一代要难，因为二代比第一代更加复杂。
    我们可以看到，历史在重演。如果源生XML数据库获得发现，则用户将在逻辑独立性和复杂性上遇到问题。
    为了避免历史重演，最好的建议是站在巨人的肩膀上，而不是站在他们的脚上。在某个领域，如果我们不去学习他的历史，我们可能会重复他的历史。
    更抽象地说，我们看到少数新的数据模型。绝大部分是在再现过去的成果。比较新的概念有：在数据库中编码（来自OR阵营）；最后模式（来自半结构数据模型阵营）
    最后模式可能是一个理想市场，我们并没有看到划时代的观点出现。在数据库中编码是真的好观点。另外，我们注意到设计一个数据和代码对等的数据库系统对于用户来说非常有用的。如是，数据库系统的附加特点如存储过程，触发器和告警器等将成为第一批用户。OR模型某部分也如此，也许现在是时候结束这个努力了。
    

References
[ASTR76] Morton M. Astrahan, Mike W. Blasgen, Donald D. Chamberlin, Kapali P. Eswaran, Jim Gray, Patricia P. Griffiths, W. Frank King, Raymond A. Lorie, Paul R. McJones, James W. Mehl, Gianfranco R. Putzolu, Irving L. Traiger, Bradford W. Wade, Vera Watson: System R: Relational Approach to Database Management. TODS 1(2): 97- 137 (1976)
[BACH73] Charles W. Bachman: The Programmer as Navigator. CACM 16(11): 635- 658 (1973)
[BATO85] Don S. Batory, Won Kim: Modeling Concepts for VLSI CAD Objects. TODS 10(3): 322-346 (1985)
[CARE97] Michael J. Carey, David J. DeWitt, Jeffrey F. Naughton, Mohammad Asgarian, Paul Brown, Johannes Gehrke, Dhaval Shah: The BUCKY Object-Relational Benchmark (Experience Paper). SIGMOD Conference 1997: 135-146
[CHAM74] Donald D. Chamberlin, Raymond F. Boyce: SEQUEL: A Structured English Query Language. SIGMOD Workshop, Vol. 1 1974: 249-264
[CHEN76] Peter P. Chen: The Entity-Relationship Model - Toward a Unified View of Data. TODS 1(1): 9-36 (1976)
[CODA69] CODASYL: Data Base Task Group Report. ACM, New York, N.Y., October 1969
[CODA71] CODASYL: Feature Analysis of Generalized Data Base Management Systems. ACM, New York, N.Y., May 1971
[CODA73] CODASYL: Data Description Language, Journal of Development. National Bureau of Standards, NBS Handbook 113, June 1973
[CODA78] CODASYL: Data Description Language, Journal of Development. Information Systems, January 1978
[CODD70] E. F. Codd: A Relational Model of Data for Large Shared Data Banks. CACM 13(6): 377-387 (1970)
[CODD71a] E. F. Codd: A Database Sublanguage Founded on the Relational Calculus. SIGFIDET Workshop 1971: 35-68
[CODD71b] E. F. Codd: Normalized Data Structure: A Brief Tutorial. SIGFIDET Workshop 1971: 1-17
[CODD72a] E. F. Codd: Relational Completeness of Data Base Sublanguages. IBM Research Report RJ 987, San Jose, California: (1972)
CODD72b] E.F. Codd: Further Normalization of the Data Base Relational Model. In Data Base Systems ed. Randall Rustin, Prentice-Hall 1972
[CODD79] E. F. Codd: Extending the Database Relational Model to Capture More Meaning. TODS 4(4): 397-434 (1979)
[CODD82] E. F. Codd: Relational Database: A Practical Foundation for Productivity. CACM 25(2): 109-117 (1982)
[DAHL66] Dahl, O. and Nygard, K: 'SIMULA; An ALGOL-based Simulation Language: CACM 9(9) 671-678 (1966).
[DATE76] C. J. Date: An Architecture for High-Level Language Database Extensions. SIGMOD Conference 1976: 101-122
[DATE84] C. J. Date: A Critique of the SQL Database Language. SIGMOD Record 14(3): 8-54 (1984)
[FAGI77a] Ronald Fagin: Multivalued Dependencies and a New Normal Form for Relational Databases. TODS 2(3): 262-278 (1977)
[FAGI77b] Ronald Fagin: Normal Forms and Relational Database Operators. SIGMOD Conference 1977: 153-160
[GO75] Angela Go, Michael Stonebraker, Carol Williams: An Approach to Implementing a Geo-Data System. Data Bases for Interactive Design 1975: 67-77
[GUTM84] Antonin Guttman: R-Trees: A Dynamic Index Structure for Spatial Searching. SIGMOD Conference 1984: 47-57
[HAMM81] Michael Hammer, Dennis McLeod: Database Description with SDM: A Semantic Database Model. TODS 6(3): 351-386 (1981)
[HEWI69] Carl Hewit: PLANNER: A Language for Proving Theorems in Robots. Proceedings of' IJCAI-69, IJCAI, Washington D.C.: May, 1969.
[KATZ86] Randy H. Katz, Ellis E. Chang, Rajiv Bhateja: Version Modeling Concepts for Computer-Aided Design Databases. SIGMOD Conference 1986: 379-386
[LAMB91] Charles Lamb, Gordon Landis, Jack A. Orenstein, Danel Weinreb: The ObjectStore System. CACM 34(10): 50-63 (1991)
[MCDO73] D. McDermott & GJ Sussman: The CONNIVER Reference Manual. AI Memo 259, MIT AI Lab, 1973.
[MCHU97] Jason McHugh, Serge Abiteboul, Roy Goldman, Dallan Quass, Jennifer Widom: Lore: A Database Management System for Semistructured Data. SIGMOD Record 26(3): 54-66 (1997)
[RICH87] Joel E. Richardson, Michael J. Carey: Programming Constructs for Database System Implementation in EXODUS. SIGMOD Conference 1987: 208-219
[ROWE79] Lawrence A. Rowe, Kurt A. Shoens: Data Abstractions, Views and Updates in RIGEL. SIGMOD Conference 1979: 71-81
[RUST74] Randall Rustin (ed): Data Models: Data-Structure-Set versus Relational. ACM SIGFIDET 1974
[SAME84] Hanan Samet: The Quadtree and related Hierarchical Data Structures. Computing Surveys 16(2): 187-260 (1984)
[SCHM77] Joachim W. Schmidt: Some High Level Language Constructs for Data of Type Relation. TODS 2(3): 247-261 (1977)
[SKAR86] Andrea H. Skarra, Stanley B. Zdonik, Stephen P. Reiss: An Object Server for an Object-Oriented Database System. OODBS 1986: 196-204
[SMIT77] John Miles Smith, Diane C. P. Smith: Database Abstractions: Aggregation and Generalization. TODS 2(2): 105-133 (1977)
[SNOD85] Richard T. Snodgrass, Ilsoo Ahn: A Taxonomy of Time in Databases. SIGMOD Conference 1985: 236-246
[SPON84] David L. Spooner: Database Support for Interactive Computer Graphics. SIGMOD Conference 1984: 90-99
[STON75] Michael Stonebraker: Implementation of Integrity Constraints and Views by Query Modification. SIGMOD Conference 1975: 65-78
[STON76] Michael Stonebraker, Eugene Wong, Peter Kreps, Gerald Held: The Design and Implementation of INGRES. TODS 1(3): 189-222 (1976)
[STON83] Michael Stonebraker, Heidi Stettner, Nadene Lynn, Joseph Kalash, Antonin Guttman: Document Processing in a Relational Database System. TOIS 1(2): 143-158 (1983)
[STON86] Michael Stonebraker, Lawrence A. Rowe: The Design of Postgres. SIGMOD Conference 1986: 340-355
[STON90] Michael Stonebraker, Lawrence A. Rowe, Michael Hirohama: The Implementation of Postgres. TKDE 2(1): 125-142 (1990)
[TSIC76] Dennis Tsichritzis: LSL: A Link and Selector Language. SIGMOD Conference 1976: 123-133
[WONG79] Eugene Wong, R. H. Katz: Logical Design and Schema Conversion for Relational and DBTG Databases. ER 1979: 311-322
[ZANI83] Carlo Zaniolo: The Database Language GEM. SIGMOD Conference 1983: 207-218








