# %chapter_number%. Литература для дальнейшего чтения и заключение

Если вы зашли так далеко, спасибо вам.

Если вам понравилась эта книга, следите за автором на  [Github](https://github.com/mixu/) (или [Twitter](http://twitter.com/mikitotakada)). Автору приятно осознавать что книга имеет положительное влияние. "Создать большую ценность чем вы поглотили" и это все.

Автор передает благодарности: logpath, alexras, globalcitizen, graue, frankshearar, roryokane, jpfuentes2, eeror, cmeiklejohn, stevenproctor eos2102 и steveloughran за их помощь! Автор также считает что все оставшиеся ошибки и неточности это его вина!

Стоит отметить что глава о согласованности в конечном итоге получилась довольно Berkeley-центричной; Автор хотел бы это изменить. Была упущена одна из важных тем - создание снапшотов. Некотрые темы также стоит раскрыть подробнее: хотелось бы более явного обсуждения свойств живучести и корректности системы и более подробного описания устойчивого хеширования.

Если бы в этой книге была 6 глава, то вероятно она бы была о том каким образом можно использовать и работать большие обьемы данных. Кажется что наиболее распространненым типом вычислений на больших данных является тот в котором [большой набор данных передается через одну простую программу](http://en.wikipedia.org/wiki/SPMD).

## Книги о распределенных системах

#### Distributed Algorithms (Lynch)

Это скоре всего наиболее часто рекомендуемая книга о распределенных алгоритмах. Автор также рекомендует ее, но с оговоркой. Она очень комплексная, но написано для выпускников вузов, так что вы можете потратить много времени читая о синхронных системах и разделяемой памяти прежде чем начнутся вещи интересные для практиков.

#### Introduction to Reliable and Secure Distributed Programming (Cachin, Guerraoui & Rodrigues)

Для практика, книга - сплошное удовольствие. Это коротокое и полное описание реализаций современных алгоритмов|.

#### Replication: Theory and Practice

Если вам интресна репликация, эта книга прекрасна. Глава о репликации во много основана на интересных частях этой книги+более поздних исследованиях.

#### Distributed Systems: An Algorithmic Approach (Ghosh)

#### Introduction to Distributed Algorithms (Tel)

#### Transactional Information Systems: Theory, Algorithms, and the Practice of Concurrency Control and Recovery (Weikum & Vossen)

Эта книга о традиционных транзакционных системах, таких как нераспределенные РСУБД. Она содержит две главы о распределенных транзакциях, но основной упор в книге на традиционные транзакции.

#### Transaction Processing: Concepts and Techniques by Gray and Reuter

Класска. Но автор считает что предыдущая книга более современна.

## Основополагающие публикации

Каждый год, [Приз Эдгара Дейкстры в распределенном программировании](http://en.wikipedia.org/wiki/Dijkstra_Prize) дается выдающим работам в области распределенных вычислений. Ссылка включает полный список этих работ, включая такую классику как:

- "[Time, Clocks and Ordering of Events in a Distributed System](http://research.microsoft.com/users/lamport/pubs/time-clocks.pdf)" - Leslie Lamport
- "[Impossibility of Distributed Consensus With One Faulty Process](http://theory.lcs.mit.edu/tds/papers/Lynch/jacm85.pdf)" - Fisher, Lynch, Patterson
- "[Unreliable failure detectors and reliable distributed systems](http://scholar.google.com/scholar?q=Unreliable+Failure+Detectors+for+Reliable+Distributed+Systems)" - Chandra and Toueg

Microsoft Academic Search имеет список [лучших публикаций в распределенном программировании отсортированный по количеству цитат](http://libra.msra.cn/RankList?entitytype=1&topDomainID=2&subDomainID=16&last=0&start=1&end=100) - это может быть интресно если хочется зацепить больше классики.

Список дополтнительных публикаций:

- [Nancy Lynch рекомендует эти публикации](http://courses.csail.mit.edu/6.852/08/handouts/handout3.pdf).
- [NoSQL Summer список публикаций](http://nosqlsummer.org/papers) - список публикаций связанных с этим термином.
- [Вопрос на Quora о основопологающих публикациях о распределенных системах](http://www.quora.com/What-are-the-seminal-papers-in-distributed-systems-Why).

### Systems

- [The Google File System](http://research.google.com/archive/gfs.html) - Ghemawat, Gobioff and Leung
- [MapReduce: Simplified Data Processing on Large Clusters](http://research.google.com/archive/mapreduce.html) - Dean and Ghemawat
- [Dynamo: Amazon’s Highly Available Key-value Store](http://scholar.google.com/scholar?q=Dynamo%3A+Amazon's+Highly+Available+Key-value+Store) - DeCandia et al.
- [Bigtable: A Distributed Storage System for Structured Data](http://research.google.com/archive/bigtable.html) - Chang et al.
- [The Chubby Lock Service for Loosely-Coupled Distributed Systems](http://research.google.com/archive/chubby.html) - Burrows
- [ZooKeeper: Wait-free coordination for Internet-scale systems](http://research.yahoo.com/pub/3280)
