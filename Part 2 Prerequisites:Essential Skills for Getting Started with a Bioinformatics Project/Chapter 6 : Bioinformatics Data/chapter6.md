第六章 生物信息学数据

到目前为止，我们关注了很多生物信息学项目开始之前的东西：组织目录结构，unix, 远程登录机器和版本控控制。然而，我们忽略了生物信息项目中很重要的东西：数据。
数据对任何生物信息学项目来说时必须的。我们后面会通过提炼大量的数据来了解复杂的生物学系统。不幸的是，许多处理小数据和中等数据的任务是对大量的基因组数据是一个挑战。这些挑战包括：
检索数据
不管是下载大量的测序数据还是大量访问某个网站下载特殊的文件，检索数据需要特定的工具和技巧
保证数据的完整性
通过网络转移大量的数据会导致加大数据中断的风险，会导致得到错误分析结果。因此，在下游分析之前，我们需要保证数据的完整性。这个工具也可以用来验证我们在分析中使用到了正确的数据版本。
压缩数据
生物信息学中的数据通常很大，因此我们需要进行数据的压缩。因此，对压缩数据的处理是生信中一个必备的技巧。


检索数据
假设你被告知你的项目的测序已经完成。你已经从你的测序中心下载了6条lane的Illumina数据.通过网页下载这些数据是不可取的。网页不是被设计来下载大量数据集的。此外，你需要将测序数据下载到服务器，而不是你的链接网络的本地设备。为了实现，我们需要用SSH登陆到你的服务器，直接使用命令行来进行数据下载。这一部分我们将看一些这样的工具。

使用wget和curl下载数据
最常用的两个下载数据的命令就是，使用wget和curl.依赖自己的系统，这些命令有可能没有被安装，你可以通过命令行来安装这几个包，比如，Homebrew和apt-get.尽管wget和curl在功能上有一定的相似性，他们的侧重点是不同的，以至于我们两者都用的很频繁。

wget 
wget被用来快速下载文件，例如，下载人类基因组版本GRCh37的22号染色体。
```bash
$ wget http://hgdownload.soe.ucsc.edu/goldenPath/hg19/chromosomes/chr22.fa.gz
```
wget 下载文件到你当前的目录，提供一个可使用的进程指标。注意，这个22号染色体的链接以http（超文本转移协定的简称）开头。wget也能够下载FTP的链接（文件转移协定的简称）。一般情况下，FTP比http更加适合大文件的下载，UCSC推荐使用wget下载。

因为UCSC网站一般用来提供人类的基因组版本，我们不需要授权就能获得。然而，如果你要下载数据来自于LIMS系统，你就需要账号和密码来授权下载。对于简单的HTTP和wget授权，你可以使用 wget --user= 和 --ask-passwd 选项。有些时候你需要更加复杂的授权，这种情况下，你就需要联系系统管理员获取权限。

wget 的一个优势就是可以循环下载数据。当下载时，使用--recursive或者-r，wegt就会递归下载链接。默认情况下，只会下载五个文件，但是可以通过--level -l 进行自定义。
递归下载页面全部数据是非常有用的。例如，一个实验室网站有许多GTF文件，我们需要下载，我们可以使用如下的命令：
```bash
wget --accept "*.gtf" --no-directories --recursive --no-parent \
http://genomics.someuniversity.edu/labsite/annotation.html
```
但是要注意，wget递归下载是激进的。如果不受约束，wget会下载全部的数据，在指定的--level下。在前面的例子中，我们限制wget采取两种方式，--no-parent 阻止wget下载数据上一级目录的数据。--accept "*.gtf"指定只下载这类文件名的数据。

练习注意点，当使用wget递归下载选项时，它会占用很多网络资源，重复加载远程的服务器。在一些情况下，如果你下载的很多很快的话，远程服务器会关掉你的IP。如果你计划下载很多数据，你最好知道下载多少数据不会被屏蔽掉IP，wget --limit-rate 能够实现下载的限速。
wget 是非常好的工具。前面的例子仅仅只展示了他能力的一点点。表格6-1列出来常用的一些参数，man wget 会全部列出。

Curl 

curl是轻量级的wget.wget 非常适用于下载HTTP和FTP的数据，下载网页数据可以使用递归的参数。curl使用起来类似，默认情况下会将文件输出到标准输出。下载人类22号染色体，像wget那样，我们可以使用如下参数：
```bash
curl http://[...]/goldenPath/hg19/chromosomes/chr22.fa.gz > chr22.fa.gz
```
注意，我们使用了截断的URL，所以这个例子也适用于网页。这个URL地址和我们之前wget使用的一样。
如果你不喜欢重定向curl的输出，可以使用 -O <文件名> 将结果写入到文件中。如果你忽视了文件名参数，curl 将会使用远程服务器使用的文件名。

curl 也有自己的优势，和wget相比，它可以使用更多的协议传输文件。包括SFTP(secure FTP)和SCP（secure copy）。curl一个特别好的特征就是可以使用-L/--location。使用这个参数，curl可以下载最终的文件，而不是重定向网页本身。最后，curl也是一个库，意味着不仅仅作为一个命令行程序，它也可以被其他软件使用，例如RCurl和pycurl.

Rsync and Secure Copy (scp)
wget 和 curl是命令行是快速下载文件的非常好的工具。但是对大量的duty任务不是最佳的方案。例如，假设你的合作者需要你目录中大量的测序文件，这些目录可能被Git忽略。一个好的工具就是使用Rsync下载整个目录。
Rsync是这类任务的最好的选择有如下几个原因。第一，Rsync非常快，因为它只会发送文件版本之间的差异（当文件存在或者部分存在），同时也会在传输中对文件进行压缩。第二，Rysnc有一个文档属性来保护links,改变时间戳，权限，所有者，和其他一些文件属性。Rsync还有一些特征和参数来处理不同情景的脚本。例如对远程文件做一些操作。
rsync的一些基本语法 是 rsync source destination, 这里的源你想拷贝的文件或者目录，目的地就是你想把文件放在那里的目的地。源或者目的能使用 user@host:/path/to/directory/远程指定.
让我们看一个例子，rsync是如何将一个文件夹全部复制到另一台机器。假设你口拷贝你的项目数据 zea-mays/ 到你的合作者的目录 /home/deboran/zea_mays/data 位于 192.168.237.42. rsyns拷贝整个目录最常用的参数组合就-avz。-a表示按照archive模式，-z表示压缩传递，-vb表示可视化，便于看到整个拷贝的进度。因为，你链接远程采用的是SSH模式，我们也需要指定 -e。我们完整拷贝的命令如下所示：
rsync -avz -e ssh zea_mays/data/ viceb@[...]:/home/deboran/zea_mays/data
rsync一个比较重要的点就是斜杆语法（data vs data/）是不一样的。斜杠意味着拷贝源目录下的内容。没有斜杠表示拷贝整个目录。因为，我们想要拷贝zea_mays/data/中的内容到/home/deboran/zea_mays/data，所以我们使用斜杠。如果data/目录在远程机器上不存在，我们就可以使用不带斜杆的语法，进行拷贝。
因为rsync只传递不存在或者已经改变的文件，所以你可以在你的传输完成之后再次进行传输。这个简单的操作能够保证你的数据在两个目录中是同步的。另外一个好的建议就是检查rsync退出的状态，rsync在文件传出问题时会有一个非零的返回状态。最后，rsync也可以使用 SSH中的简写。如果你通过SSH的简写链接远程服务器，你就应该除去-e参数。
有时候，你会需要快速通过SSH拷贝单一的文件。对于这种任务，unix中的cp足够完成，但是需要远程。rsync可以完成，但是有点小题大做。Secure copy(scp) 更加适合这类任务。
```bash
$ scp Zea_mays.AGPv3.20.gtf 192.168.237.42:/home/deborah/zea_mays/data/
```

数据的完整性

我们项目下载的数据使我们未来分析和结论的基础。尽管看起来不可能，但是在传输大数据数，数据在传输中的奔溃的风险是我们需要关心的。大文件需要更多的时间来传输，就会容易大致网络的中断和数据的丢失。此外保证传输没有报错，验证数据的完整性也很重要。Checksums 浓缩数据，只要数据有一个bit的差别他都会计算出来，checksum就会不一样。
数据的完整性检查对数据版本的追溯非常有用。在一个合作的项目中，我们的分析依赖于合作者的中间结果，一旦中间结果改变，下游依赖这些结果的分析都得重新来过。许多的中间文件，哪些数据改变了，哪些步骤需要重复都是不清楚的。如果数据改变，哪怕是很小的差别checksum都会检测出来。因为我们可以用checksum计算我们数据的版本。校验和也有助于再现性，因为我们可以将特定的分析和结果集与由数据的校验和值汇总的数据的精确版本相关联。
SHA and MD5 Checksums
最常用的检测数据完整性的算法就是 MD5和SHA-1.我们在第四章中遇到过SHA-1，是Git用来commit ID。SHA也是一种比较老的校验算法。但是它也经常被使用。MD5和SHA-1表现类似，但是SHA-1是比较新和受欢迎的。然而，MD5更为常见；如果服务器已在一组文件上预先计算校验和，那么您可能会遇到这种情况。
我们先使用SHA-1试一试。我们打印一字符通过标准输出传给shasum
```bash 
$ echo "bioinformatics is fun" | shasum
f9b70d0d1b0a55263f1b012adab6abf572e3030b -
$ echo "bioinformatic is fun" | shasum
e7f33eedcfdc9aef8a9b4fec07e58f0cf292aa67
```
一长串数据和字符就是shasum的输出结果。Checksums 使用的是十六进制的格式，每个字节是16个字符中的一个，数字：0--9，字符:1,b,c,d,e,f。最后的破折号表示SHA-1的输入来自于标准输入。注意但我们省略掉bioinformatics中的s来计算SHA-1的校验值，会发现和之前的完全不一样。使用检验的优势在于，一旦数据有微小的变化，校验值是绝对确认性的。意味着不管系统的时间咋样，校验的值只和输入的数据有关。
我们也可以检查输入文件的校验值（注意这个文件是假的，为了演示而产生的）
```bash 
$ shasum Csyrichta_TAGGACT_L008_R1_001.fastq
fea7d7a582cdfb64915d486ca39da9ebf7ef1d83 Csyrichta_TAGGACT_L008_R1_001.fastq
```
如果我们的测序中心说数据的校验值是“069bf5894783db241e26f4e44201bd12f2d5aa42”，但是我们本地的校验值是“fea7d7a582cdfb64915d486ca39da9ebf7ef1d83,” 我们就知道我们的数据和原始的数据有差异。
当我们下载很多数据的时候，一个个检查文件的校验值是比较麻烦的。shasum程序有一个比较简单的方式，它会产生和验证文件中包含很多校验值。我们可以验证如下的：
```bash
$ shasum data/*fastq > fastq_checksums.sha
$ cat fastq_checksums.sha
524d9a057c51b1[...]d8b1cbe2eaf92c96a9 data/Csyrichta_TAGGACT_L008_R1_001.fastq
d2940f444f00c7[...]4f9c9314ab7e1a1b16 data/Csyrichta_TAGGACT_L008_R1_002.fastq
623a4ca571d572[...]1ec51b9ecd53d3aef6 data/Csyrichta_TAGGACT_L008_R1_003.fastq
f0b3a4302daf7a[...]7bf1628dfcb07535bb data/Csyrichta_TAGGACT_L008_R1_004.fastq
53e2410863c36a[...]4c4c219966dd9a2fe5 data/Csyrichta_TAGGACT_L008_R1_005.fastq
e4d0ccf541e90c[...]5db75a3bef8c88ede7 data/Csyrichta_TAGGACT_L008_R1_006.fastq
```
我们也可以用shasum的检查选项 -c 会验证这些文件和原来的校验值是否相同。
```bash
$ shasum -c fastq_checksums.sha
data/Csyrichta_TAGGACT_L008_R1_001.fastq: OK
data/Csyrichta_TAGGACT_L008_R1_002.fastq: OK
data/Csyrichta_TAGGACT_L008_R1_003.fastq: OK
data/Csyrichta_TAGGACT_L008_R1_004.fastq: OK
data/Csyrichta_TAGGACT_L008_R1_005.fastq: OK
data/Csyrichta_TAGGACT_L008_R1_006.fastq: FAILED
shasum: WARNING: 1 computed checksum did NOT match
```
只要碰到检测的校验值不一致，shasum就会告诉你文件失败了，以非零的状态退出。
md5sum 就会计算MD5哈希值，使用和shasum一致。然而，在OX操作系统上，md5没有-c选项，所以你要自己安装。此外一些服务器会使用一些比较过时的校验程序例如sum或者chsum。使用这些过时的程序和我们使用shasum和md5sum的方式一致。
最后，你可能会好奇所有文件是如何被一个40字符长的SHA-1的校验值统计的。只有16的40次方个不同的校验值。然而，16的40次方是一个非常大的数，以至于校验值相同的概率是非常小的。检查数据完整性的目的就是使得数据冲突的风险降到最低。

查看数据之间的差异