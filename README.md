# software-competition
摘要：近几年来，计算机和信息技术迅猛发展并且普及应用，行业应用系统的规模迅速扩大，所产生的数据呈爆炸性增长，因此提高对数据的提取、处理和分析能力，通过进一步加工实现数据的“增值”已经成为现在世界的迫切需求。本项目通过编写python网络爬虫程序，使用requests库、bs4库等爬取互联网中近期与计算机相关专业的招聘信息，收集各专业学科学习课程内容。通过数据清洗、转化、可用信息提取等技术手段获得可用的原始数据。对原始数据进行初步的分析后，利用jieba、gensim等python第三方机器学习库，把项目要求的最终结果通过可视化的手段展现出来。
 
    关键词:机器学习;网络爬虫;jieba分词;gensim;知识图谱
    
正文：

1、数据处理过程
整个数据处理流程可以概括为三步，分别是采集、清洗和转化，具体步骤如下：
1.1数据采集
本项目数据采集利用Mysql数据库来接收本小组从前程无忧、齐鲁人才网、拉勾网等多个网页上爬取的大约43万的数据。 
具体采集过程中，首先,getHtmltext()函数中采用requests库中的requests等方法进行网页总体源代码信息的爬取，requests是用python语言基于urllib编写的，采用的是Apache2 Licensed开源协议的HTTP库，Requests比urllib更加方便，节约大量的工作。Beautiful Soup借助网页的结构和属性等特性来简易的解析网页， 然后再调用getInfo()函数，该函数使用bs4库中的BeautifulSoup类对已爬取的网页源代码进行初步的处理，通过对BeautifulSoup类对象使用soup.find_all()函数对网页所需信息进行一些初步的查找和提取并将其写入列表中，对列表操作进行初步数据处理，再根据该列表中标明工作具体要求网址的项进入下层网页，最后采用类似的操作对具体工作信息再进行数据的初步爬取与处理。
 爬取的网页内容最终都存在于一个多维列表之中，
 
1.2数据清洗
1.2.1初步处理过程：
初步处理主要是通过调用字符串处理工具正则表达式（re）库，对信息进行正则匹配及处理，通过观察可以发现具体处理的地方是：      
1、工资：由一个整体分割为最高工资和最低工资      
2、关键词：使用jieba分词根据具体信息（info）字段内容将分词结果取词频较高的词集作为关键词，其中关键词分词的规则进行了一定的人工干涉
1.2.2数据清洗过程
在获取网页数据时，我们发现，在诸如职位名、具体信息等数据项中因不明原因出现了参差不齐的数据，其中不仅有格式混乱，字符混用的情况，更有内容掺杂None或乱码或缺少部分数据项的数据。通过将获取的数据转化为字典并检测相关键是否存在完成了对缺少数据项数据的清洗，通过将文件重新编码解决乱码问题，通过正则表达式替换解决了None和中英文字符混用的问题，最后将清洗后的数据放入字典设置数据库表项主键的方式完成了数据的去重。
1.3数据转化
我们使用了MySQL数据库,在Navicate For MySQL中创建数据库和对应表后设置表项主键为(职位名,公司名)，设置表项为职位名，公司名，工作地点(省)，工作地点(县市区)，最低工资，最高工资，具体信息，关键词，各表项均为varchar(num)类型并设置非空。通过python的MySQLdb库连接数据库后将清洗后的数据产生的字典按键写入数据库完成了对数据的转化。
2.	数据分析与挖掘方法
对数据进行分析和挖掘主要目的在于发现数据隐藏的信息，对上一步产生的数据库文件进行分析知，隐藏的信息可能位于职位名，工作城市，具体位置，最低工资，最高工资，岗位职责，任职要求上。通过分析相同或相似职位名可得到职业需求量，通过分析工作地点进行相关统计可以得到工作的城市分布图，通过分析最低工资最高工资可以得到部分省县市区的最低最高薪资分布，通过分析具体信息和提取的关键词可以得到企业对招聘人员的具体要求。
由上可知我们可以从已导入的数据获取的隐藏信息有:企业对大数据要求最迫切的前十名招聘职位（通过计算职位名或具体信息出现大数据关键词的职位出现的次数获得）、大数据职位需求量最高的前十名城市（通过计算职位名或具体信息出现大数据关键词的城市出现的次数获得）,计算机相关专业需求量前十名（通过计算职位名或具体信息或关键词中计算机相关专业出现的次数获得），计算机专业薪水最高的前十名招聘职位（通过计算职位名或具体信息或关键词中涉及计算机的职位按薪水排序获得），提供简历由已有数据中获得最匹配职业前十名（通过分词产生词向量后由机器学习进行相似度匹配获得）。
由上可知，为了精确提取职位名和具体信息，我们使用了jieba分词进行关键词的提取，并将分词后的数据进行词频计算选择前几的词当做该招聘信息的关键词写入数据库。剩余隐藏信息一部分可由sql语句查询对应表项获得，另一部分可由sql语句查询生成的关键词获得。为了保证生成关键词的准确性，我们使用了用生成的关键词产生词云人工筛选的方式，通过数十次的人工筛选确保了关键词的粒度和精确性。
在获取了较精确的关键词后，我们可以通过gensim库提供的机器学习的内容进行训练，之后通过计算相似度来获得提供简历与数据库中较为匹配的数据。

