# %chapter_number%. Репликация: протоколы обеспечивающие слабую согласованность

Сейчас, после того как мы расмотрели протоколы которые обеспечивают последовательную согласованность с учетом возрастающего количества обрабатываемых типов отказов, давайте вернемся к пространству возможных опций, которое открывается если мы откажемся от требования последовательной согласованности(без расхождений).

В общем и целом, трудно придумать одно измерение которое будет определять или характиризовать протоколы позволяющие репликам расходится. Большинство таких протоколов обеспечивают высокую доступность и ключевой проблемой является смогут ли конечные пользователи найти гарантии, абстракции, API полезные для их нужд, несмотря на то что реплики могут расходится, когда отказывают узлы или рвется сеть.

Почему слабо согласованные системы менее популярны?

Как я говорил в введении, я думаю что многое в распределенном программировании следует из двух последствий распределенности:

- информация путешествует со скоростью света
- независимые вещи отказывают независимо

Следствием того что скорость распространения информации ограничена, является то что каждый узел воспринимает окружающий мир по-своему. Вычисления на одному узле просты, так как все случается в определенном глобальном и абсолютном порядке. Вычисления на распределенной системе сложны, так как глобального  и абсолютного порядка нет.

Долгое время (десятилетия исследований), мы решали эту проблему введение глобального порядка. Мы обсудили много методов достигающий последовательной согласованности путем создания порядка(с учетом устойчивости к отказам) когда естественного порядка не было.

Конечно, проблема в том что создание порядка крайне затратно. Это зачастую просто невозможно в больших системах работающих в интернете, когда система должна оставатся доступна. Система соблюдающая строгую согласованность ведет себя не как распределенная: она ведет себя как единая система, что плохо для доступности во время разделений.

Кроме того, для каждой операции зачастую большинству узлов необходимо связатся друг с другом - и зачастую не один раз, а дважды(вспомним обсуждение 2PC). Это обычно крайне болезнено для систем которые нуждаются в географической распределенности для обеспечения адекватной производительности для пользователей в разных частях мира.

Так что поведение системы по умолчанию как единого целого возможно не желательно.

Возможно то что мы хотим, это система для которой мы можем писать код без больших затрат на координаци но при это продолжать получать пригодные для наших целей результаты. В замен единственно-верной версии данных, мы принимаем разные реплики с расхождениями между друг другом - это позволит сохранить эффективность и сохранить устойчивость к разделению - и затем мы попробуем найти способ справится с расхождениями каким либо образом.

Согласованность в конечном итоге выражается в следующей идее: узлы которые могут иногда расходится в конечном итоге прийдут к согласию насчет единого значения.

Во всем множестве систем предоставляющем согласованность в конечном итоге, можно выделить два типа архитектур:

*Согласованность в конечном итоге с вероятностными гарантиями*. Этот тип систем может определять конфликты на каком то этапе после их появления, но не дает гарантий что результаты будут эквивалентны результатам корректного порядка исполнения операций. Другими словами, конфликтующие обновления иногда будут перезаписывать новые значения старыми и некотрые аномалии могут проявлятся во время нормального проведения операций(или во время разделения).

В недавние годы,наиболее известная система жертвующая согласованностью на уровне одной копии это Amazon Dynamo, которую я буду обсуждать в качестве системы которая предалагает согласованность в конечном итоге с вероятностными гарантиями.

*Согласованность в конечном итоге со строгими гарантиями*. Этот тип систем гарантирует что результат будет сходится к одному значению эквивалентному полученному при корректной последовательности операций. Другими словами, такие системы не приводят к аномальным результатам; без какой либо координации вы можете создавать реплики сервисов, которые будут коммуницировать любым способом и получать обновления в любом порядке и в конечном итоге они достигнут согласия насчет конечного результата на все время пока они будут видеть одну и туже информацию.

CRDT (сходяющиеся рпелицирующиеся типы данных) это типы данных гарантирующие сходимость к одному и тому же значению несмотря на разделы и задержки в сети и изменение порядка сообщений. Они доказанно сходимы, но ограничены типами данных которые могут быть реализованы как CRDT.

CALM (согласованность как логическая монотонность) гипотеза это альтернативное выражение тех же самых принципов: это уравнивание логической монотонности и сходимости. Если мы можем сделать вывод что чтолибо логически монотонно, тогда мы также можем безопастно запускать это без какой либо координации. Анализ слияний - в частности, применительно к языку программирования Bloom - может быть применен для принятия решений в программировании о том когда и где  использовать координационные техники из строго согласованной системы и когда можно безопастно выполнять операции не пребегая к вычислительно "дорогой" координации.

## Согласование разного порядка операций

Что делает систему не поддерживающей согласованность единной копии? Давайте попробуем сделать более конкретные выводы из нескольких примеров.

Конечно наиболее очевидной характеристикой систем которые не обспечивают согласованности одной копии является, то что они позволяют репликам расходится относительно друг друга. Это означает что нет строгой коммуникации: реплики могут быть отделены друг от друга и даже продолжать быть доступны и записывать данные.

Давайте представим систему из трех реплики, каждая из которых отделена от других. Для примера реплика может быть в другом датацентре или отделена по другим причинам. Каждая реплика остается доступной во время разделения для записи и чтения для некотрого набора клиентов:

    [Клиенты]   - > [A]

    --- Раздел ---

    [Клиенты]   - > [B]

    --- Раздел ---

    [Клиенты]   - > [C]

После некотрого времени, разделение исчезает и реплики начинают обменниваться информацией. Они получают  различные обновления от разных клиентов что приводит к расхождениям - следовательно необходим некотрый способ согласования. Нам бы хотелось бы чтобы все реплики после этого сошлись к одному результату.

    [A] \
        --> [мерж]
    [B] /     |
              |
    [C] ----[мерж]---> результат


Другой путь размышлять о системах с гарантиями слабой согласованности это представить множество клиентов отсылающих сообщения на узла в некотром порядке. Так как у нас нет координационного протокола чтобы обеспечить глобальный единый порядок, сообщения могут быть доставлены в разном порядке на разные реплики:

    [Клиенты]  --> [A]  1, 2, 3
    [Клиенты]  --> [B]  2, 3, 1

Это в сущности причина по которой мы нуждаемся в протоколах координации. Для примера, предположим что мы пытаемся склеить строку за 3 операции:

    1: { операция: concat('Hello ') }
    2: { операция: concat('World') }
    3: { операция: concat('!') }

Тогда, без координации A получит "Hello World!", а B "World!Hello ".

    A: concat(concat(concat('', 'Hello '), 'World'), '!') = 'Hello World!'
    B: concat(concat(concat('', 'World'), '!'), 'Hello ') = 'World!Hello '


Это конечно же некорректно. Опять же, нам бы хотелось чтобы реплики сходились к одному результату.

Держа в уме оба этих примера, давайте взглянем на Amazon's Dynamo в первую очередь чтобы определить базовый уровень и затем обсудим ряд концепций для построения систем со слабой консистентностью, таких как CRDT the CALM теорема.


## Amazon's Dynamo

Архитектура Amazon's Dynamo (2007) это возможно наиболее известный пример системы представляющей слабые гарантии согласованности и являющейся высоко-доступной системой. Это основа для многих промышленных систем таких как LinkedIn's Voldemort, Facebook's Cassandra и Basho's Riak.

Dynamo это ключ-значение хранилище обеспечивающее согласованность в конечном итоге и высокую доступность. Ключ-значение хранилище напоминает огромную хеш-таблицу: клиент может установить некоторое значение по ключу используя `set(key, value)` и получить некоторое значение по ключу используя `get(key)`. Dynamo кластер содержит N частей узлов; каждый узел отвечает за определенный набор ключей.

Dynamo разработана с приоритетом доступности над согласованностью; согласованность на уровне одной активной копии не гарантируется. Реплики могут расходится относительно друг друга когда значения записываются; во время чтения по ключу, происходит фаза согласования во время которой различия разных реплик обьединяются перед тем как возвратить результат клиенту.

Для многих отраслей бизнесса Amazon, более важно избегать простоев нежели держать данные в полной консистентности, так как отключение может привести к бизнесс-потерям и утрате доверия клиентов. Более того, если данные не особо важные, тогда системы с слабой согласованностью могут предоставлять лучшую производительность и более высокую доступность с более низкими затратами нежели традиционные RDBMS.

Dynamo это полная архитектура системы, в которой необходимо расмотреть много различных частей некоторые из которых выходят за рамки задачи репликации; особенно, как запись диспатчеризируется между узлами и как происходит запись на несколько узлов.

    [ Клиент ]
        |
    ( Отображение ключей в узлы )
        |
        V
    [ Узел A ]
        |     \
    ( Синхронная часть репликации: минимум надежности )
        |        \
    [ Узел B]  [ Узел C ]
        A
        |
    ( Определение конфликта; асинхронная часть репликации:
      обеспечивает востановление разделенных / упавших узлов )
        |
        V
    [ Узел D]

После того как мы взглянем как запись принимается после иницирования клиентов, мы посмотрим как обнаруживаются конфликты и на асинхронную часть репликации. Это часть необходима, из-за дизайна высоко-доступных решений, в которых узлы могут быть временно недоступны(изза отказа или разделения). Синхронизация реплик обеспечивает довольно быстрое приведение реплики к актуальному состоянию даже после отказа.

### Согласованное хэширование(Consistent hashing)

Записываем мы или читаем, первое что должно произойти мы должны определить где данные должны находится в системе. Это требует некотрого отображение ключей в узлы на которых они хранятся.

В Dynamo, ключи отображаются в узлы используя технику хеширования известную как [согласованное хеширование](https://github.com/mixu/vnodehash) (которое я не буду обсуждать детально). Главная идея это что ключи могут быть отображены в набор узлов при помощи простых вычислений на клиенте. Это значит что клиент может обнаружить ключи без запроса к системе для определения расположения ключа; это экономит системные ресурсы так как хеширование во много раз быстрее чем вызов удаленной процедуры.

### Частичный кворум

После того как мы разобрались как ключ должен хранится, мы должны понять как хранится значение. Это синхронная задача; причина по которой нам надо записывать немедленно значение на несколько узлов это предоставление высокого уровня надежности (например для защиты от немедленного отказа узла).

Так же как и Paxos или Raft, Dynamo использует кворумы для репликации. Однако Dynamo кворумы нестрогие(основанные только на часте узлов) в отличии от строгих(основанных на большинстве) кворумов.

Неформально, строгие системы кворумов это системы кворумов с таким свойством что любые два кворума в системе пересекаются. Требование большинства голосов для обновления перед его принятием гарантирует что только одна версия истории изменений будет признаной для каждого мажоритарного кворума так как каждый такой кворум будет пересекатся с другим хотя бы в одном узле. На это свойство опирается к примеру Paxos.

Частичный кворум не удовлетворяет этому свойству; это означает что большинство не требуется и различные подмножества кворума могут содержать различные версии одних и тех же данных. Юзер может выбирать число узлов для записи и чтения:

- пользователь может выбрать некотрое число W-из-N узлов требуемое для того чтобы запись была успешна; и
- пользователь может определить число узлов (R-из-N) которым необходимо контактировать при чтении.

`W` и `R` определяет число узлов которые должны быть вовлечены в процесс записи и чтения. Запись с использованием большего числа узлов будет более медленной но повысит вероятность того что значение не будет потеряно; чтение с большего числа узлов повышает вероятность того что прочитанное значение будет актуально.

Типичная рекомендация это `R + W > N`, потому что это означает что кворумы для чтения и записи пересекаются хотя бы в одном узле - что делает менее вероятным что устаревшее значение будет прочитано. Обычная конфигурация это `N = 3` (то есть всего 3 рпелики для каждого значения); это означает что юзер может выбирать между:

     R = 1, W = 3;
     R = 2, W = 2 или
     R = 3, W = 1

В более общем плане `R + W > N`:

- `R = 1`, `W = N`: быстрое чтение, медленная запись
- `R = N`, `W = 1`: быстрая запись, медленное чтение
- `R = N/2` and `W = N/2 + 1`: хорошо и для того и для того

N редко больше 3, так как хранение большого числа копий может быть дорогостояще для большого количества данных!

Как я упомянал ранее, публикация Dynamo  вдохновила многие подобные системы. Они все используют репликацию на основе частичных кворумов, но с другими N, W и R по умолчанию:

- Basho's Riak (N = 3, R = 2, W = 2 по умолчанию)
- Linkedin's Voldemort (N = 2 or 3, R = 1, W = 1)
- Apache's Cassandra (N = 3, R = 1, W = 1)

Так же есть другой ньюанс: когда отправляется запрос на запись или чтение, все N узлов опрашиваются (Riak), или только некоторое число узлов - минимальный кворум (то есть R или W; Voldemort). Отправка всем более быстра и менее чувствительна к задержкам(так как можно ждать ответ только R или W узлов из N) но менее эффективен, Отправка запроса только минимуму узлов более чувствительна к задержкам(так как задержка в общении с одним узлом приведет к задержке всей операции) но более эффективно (меньше сообщений / соединений в целом)

Что случится когда кворумы записи и чтения перекрываются то есть (`R + W > N`)? В частности, зачастую говорят что в  результате мы получим "строгую согласованность".

### R + W > N это тоже самое что и "строгая согласованность"?

Нет.

Для этого нет оснований: система где `R + W > N` может определять конфликты записи и чтения, так как кворумы для записи и чтения пересекаются. Например хотя бы один узел будет обоих кворумах:

       1     2   N/2+1     N/2+2    N
      [...] [R]  [R + W]   [W]    [...]

Это гарантирует что предыдущая запись будет видна для последующих чтений. Однако, это так только если число узлов N никогда не будет менятся. Следовательно, Dynamo не сдерживает этих гарантий, потому что в Dynamo кластер может быть изменен если узлы откажут.

Dynamo разработана чтобы быть всегда доступной для записи. Она содержит который обрабатывает отказ узлов путем добавления другого не связанного сервера в набор узлов отвественных за хранение определенных ключей пока другой узел не работает. Это означает что кворум не гарантирует пересечений всегда. Даже `R = W = N` не будет этого гарантировать, так как пока размер кворума N узлы в кворуме могут менятся изза отказов. В частности, во время раздела, если достаточное число узлов не может быть достигнуто, Dynamo будет добавлять новые узлы из несвязанных с исходными но доступных узлов.

Кроме того, Dynamo не обрабатывает разделы так же как система с строгой согласованностью, а именно: она позволяет записывать данные по обе стороны раздела сети, это означает что система не действует так как если бы она не была распределенной. Поэтому называть `R + W > N` "строго согласованным" ошибочно; гарантии только лишь вероятностные - что не подходит для строго согласованной системы.

### Определение конфликтов и чтение исправленых записей

Системы которые которые позволяют репликам расходится должны иметь способ в конечном итоге согласовать два различных значения. Как кратко упоминалось в ходе обсуждения подходов основанных на частичном кворуме, один из способов это использовать определение конфликтов во время чтения а затем применить какой-либо алгоритм разрешения конфликтов. Но как это сделать?

В общем случае, Iэто можно сделать путем отслеживание причинноследственных связей между кусочками данных через сохранненые в них метаданные. Клиенты должны получить метаданные когда они читают данные из системы и они должны вернуть назад значение метаданных когда они записывают данные в базу данных.

Мы уже встречались с методом который делает это: векторные часы могут быть использованы для представления истории некотрого значения. В самом деле, именно их использует оригинальная архитектура Dynamo для определения конфликтов.

Однако, использование векторные часы не единственная альтернатива. Если взглянуть на архитектуру многих промышленных систем, можно понять немного о том как они работают глядя на метаданные которые они отслеживают.

*Без метаданных*. Когда система не отслеживает метаданные, и всегда возвращает только значение(с помощью клиентского API), у нее нет возможности как либо специально обрабатывать конкурентные записи. Общее правило в таких системах "last writer wins": другими словами, если два записывающих клиента пишут в одно и тоже время, только значение от более медленного клиента будут сохранятся.

*Временные метки*.  Формально, значение с более поздней временной меткой "выигрывает". Однако, если время не синхронизированно точно могут происходить многие странные случаи когда новое значение(из части системы с отстающим временем) будет затерто более старым. Facebook Cassandra это реализация архитектуры Dynamo которая использует временные метки вместо векторных часов.

*Номера версий*. Номера версий могут помочь избежать могих проблем использования временных меток. Однако для отслеживания нескольких причинно-следственных связей недостаточно номеров версий и необходимы векторные часы.

*Векторные часы*. Используя векторные часы,  конкурентные и устаревшие обновления могут быть обнаружены. Выполнение исправления при чтении также становится возможным, хотя в некотрых случаях(конкурентные изменения) необходимо спрашивать у клиента значение. Это так потому что если изменения паралельные и мы не знаем ничего больше о данных(как в нашем случае когда мы работаем с хранилищем ключ-значение), тогда лучшей политикой будет спросить нежели отбросить данные произвольно.

Когда происходит чтение значения, клиент контактирует с `R` из `N` узлов и опрашивает их о последнем значении для ключа. Далее берет все их ответы и отбрасывает значения которые строго более старые(для этого используются векторные часы). Если остается только одна уникальная пара векторные часы + значение, тогда возвращается она. Если остается несколько таких пар(которые редактировались конкурентно), тогда возвращаются все они.

Из этого очевидно что чтение исправленных записей может возврашать множество значений. Это означает что клиент / приложение должно уметь время от времени обрабатывать случаи выбирая значение на основе конкретного критерия в каждом отдельном взятом случае.

В дополнение, ключевой компонент промышленных систем с векторными часами это то что часы не могут расти вечно - так что необходимо иногда собирать мусор безопастным способом чтобы не нарушить требования отказоустойчивости хранилища.

### Синхронизация реплик: gossip и деревья Меркла

При условии что Dynamo-архитектура устойчива к падениям узлов и разделениям сети, она должна уметь пересоединять кластер после разделения или заменять упавший узел или присоединять востановившийся.

Синхронизация реплик используется для обновления реплик до актуального состояния после отказа и для периодической синхронизации реплик между друг другом.

Gossip это вероятностный способ для синхронизации реплик. Это паттерн коммуникации (то есть способ которым узел общается с узлом) не детерминирован заранее. Вместо этого, узлы имеют некотрую вероятность `p` того что узел попытается синхронизироватся с другими. Каждые `t` секунд, каждый узел выбирает узел с которым будет коммуницировать. This provides an additional mechanism beyond the synchronous task (e.g. the partial quorum writes) which brings the replicas up to date.

Gossip is scalable, and has no single point of failure, but can only provide probabilistic guarantees.

In order to make the information exchange during replica synchronization efficient, Dynamo uses a technique called Merkle trees, which I will not cover in detail. The key idea is that a data store can be hashed at multiple different level of granularity: a hash representing the whole content, half the keys, a quarter of the keys and so on.

By maintaining this fairly granular hashing, nodes can compare their data store content much more efficiently than a naive technique. Once the nodes have identified which keys have different values, they exchange the necessary information to bring the replicas up to date.

### Dynamo in practice: probabilistically bounded staleness (PBS)

And that pretty much covers the Dynamo system design:

- consistent hashing to determine key placement
- partial quorums for reading and writing
- conflict detection and read repair via vector clocks and
- gossip for replica synchronization

How might we characterize the behavior of such a system? A fairly recent paper from Bailis et al. (2012) describes an approach called [PBS](http://pbs.cs.berkeley.edu/) (probabilistically bounded staleness) uses simulation and data collected from a real world system to characterize the expected behavior of such a system.

PBS estimates the degree of inconsistency by using information about the anti-entropy (gossip) rate, the network latency and local processing delay to estimate the expected level of consistency of reads. It has been implemented in Cassandra, where timing information is piggybacked on other messages and an estimate is calculated based on a sample of this information in a Monte Carlo simulation.

Based on the paper, during normal operation eventually consistent data stores are often faster and can read a consistent state within tens or hundreds of milliseconds. The table below illustrates amount of time required from a 99.9% probability of consistent reads given different `R` and `W` settings on empirical timing data from LinkedIn (SSD and 15k RPM disks) and Yammer:

![from the PBS paper](./images/pbs.png)

For example, going from `R=1`, `W=1` to `R=2`, `W=1` in the Yammer case reduces the inconsistency window from 1352 ms to 202 ms - while keeping the read latencies lower (32.6 ms) than the fastest strict quorum (`R=3`, `W=1`; 219.27 ms).

For more details, have a look at the [PBS website](http://pbs.cs.berkeley.edu/)  and the associated paper.

## Disorderly programming

Let's look back at the examples of the kinds of situations that we'd like to resolve. The first scenario consisted of three different servers behind partitions; after the partitions healed, we wanted the servers to converge to the same value. Amazon's Dynamo made this possible by reading from `R` out of `N` nodes and then performing read reconciliation.

In the second example, we considered a more specific operation: string concatenation. It turns out that there is no known technique for making string concatenation resolve to the same value without imposing an order on the operations (e.g. without expensive coordination). However, there are operations which can be applied safely in any order, where a simple register would not be able to do so. As Pat Helland wrote:

> ... operation-centric work can be made commutative (with the right operations and the right semantics) where a simple READ/WRITE semantic does not lend itself to commutativity.

For example, consider a system that implements a simple accounting system with the `debit` and `credit` operations in two different ways:

- using a register with `read` and `write` operations, and
- using a integer data type with native `debit` and `credit` operations

The latter implementation knows more about the internals of the data type, and so it can preserve the intent of the operations in spite of the operations being reordered. Debiting or crediting can be applied in any order, and the end result is the same:

    100 + credit(10) + credit(20) = 130 and
    100 + credit(20) + credit(10) = 130

 However, writing a fixed value cannot be done in any order: if writes are reordered, the one of the writes will overwrite the other:

    100 + write(110) + write(130) = 130 but
    100 + write(130) + write(110) = 110

Let's take the example from the beginning of this chapter, but use a different operation. In this scenario, clients are sending messages to two nodes, which see the operations in different orders:

    [Clients]  --> [A]  1, 2, 3
    [Clients]  --> [B]  2, 3, 1

Instead of string concatenation, assume that we are looking to find the largest value (e.g. MAX()) for a set of integers. The messages 1, 2 and 3 are:

    1: { operation: max(previous, 3) }
    2: { operation: max(previous, 5) }
    3: { operation: max(previous, 7) }

Then, without coordination, both A and B will converge to 7, e.g.:

    A: max(max(max(0, 3), 5), 7) = 7
    B: max(max(max(0, 5), 7), 3) = 7

In both cases, two replicas see updates in different order, but we are able to merge the results in a way that has the same result in spite of what the order is. The result converges to the same answer in both cases because of the merge procedure (`max`) we used.

It is likely not possible to write a merge procedure that works for all data types. In Dynamo, a value is a binary blob, so the best that can be done is to expose it and ask the application to handle each conflict.

However, if we know that the data is of a more specific type, handling these kinds of conflicts becomes possible. CRDT's are data structures designed to provide data types that will always converge, as long as they see the same set of operations (in any order).

## CRDTs: Convergent replicated data types

CRDTs (convergent replicated datatypes) exploit knowledge regarding the commutativity and associativity of specific operations on specific datatypes.

In order for a set of operations to converge on the same value in an environment where replicas only communicate occasionally, the operations need to be order-independent and insensitive to (message) duplication/redelivery. Thus, their operations need to be:

- Associative (`a+(b+c)=(a+b)+c`), so that grouping doesn't matter
- Commutative (`a+b=b+a`), so that order of application doesn't matter
- Idempotent (`a+a=a`), so that duplication does not matter

It turns out that these structures are already known in mathematics; they are known as join or meet [semilattices](http://en.wikipedia.org/wiki/Semilattice).

A [lattice](http://en.wikipedia.org/wiki/Lattice_%28order%29) is a partially ordered set with a distinct top (least upper bound) and a distinct bottom (greatest lower bound). A semilattice is like a lattice, but one that only has a distinct top or bottom. A join semilattice is one with a distinct top (least upper bound) and a meet semilattice is one with a distinct bottom (greatest lower bound).

Any data type that be expressed as a semilattice can be implemented as a data structure which guarantees convergence. For example, calculating the `max()` of a set of values will always return the same result regardless of the order in which the values were received, as long as all values are eventually received, because the `max()` operation is associative, commutative and idempotent.

For example, here are two lattices: one drawn for a set, where the merge operator is `union(items)` and one drawn for a strictly increasing integer counter, where the merge operator is `max(values)`:

       { a, b, c }              7
      /      |    \            /  \
    {a, b} {b,c} {a,c}        5    7
      |  \  /  | /           /   |  \
      {a} {b} {c}            3   5   7

With data types that can be expressed as semilattices, you can have replicas communicate in any pattern and receive the updates in any order, and they will eventually agree on the end result as long as they all see the same information. That is a powerful property that can be guaranteed as long as the prerequisites hold.

However, expressing a data type as a semilattice often requires some level of interpretation. Many data types have operations which are not in fact order-independent. For example, adding items to a set is associative, commutative and idempotent. However, if we also allow items to be removed from a set, then we need some way to resolve conflicting operations, such as `add(A)` and `remove(A)`. What does it mean to remove an element if the local replica never added it? This resolution has to be specified in a manner that is order-independent, and there are several different choices with different tradeoffs.

This means that several familiar data types have more specialized implementations as CRDT's which make a different tradeoff in order to resolve conflicts in an order-independent manner. Unlike a key-value store which simply deals with registers (e.g. values that are opaque blobs from the perspective of the system), someone using CRDTs must use the right data type to avoid anomalies.

Some examples of the different data types specified as CRDT's include:

- Counters
  - Grow-only counter (merge = max(values); payload = single integer)
  - Positive-negative counter (consists of two grow counters, one for increments and another for decrements)
- Registers
  - Last Write Wins -register (timestamps or version numbers; merge = max(ts); payload = blob)
  - Multi-valued -register (vector clocks; merge = take both)
- Sets
  - Grow-only set (merge = union(items); payload = set; no removal)
  - Two-phase set (consists of two sets, one for adding, and another for removing; elements can be added once and removed once)
  - Unique set (an optimized version of the two-phase set)
  - Last write wins set (merge = max(ts); payload = set)
  - Positive-negative set (consists of one PN-counter per set item)
  - Observed-remove set
- Graphs and text sequences (see the paper)

To ensure anomaly-free operation, you need to find the right data type for your specific application - for example, if you know that you will only remove an item once, then a two-phase set works; if you will only ever add items to a set and never remove them, then a grow-only set works.

Not all data structures have known implementations as CRDTs, but there are CRDT implementations for booleans, counters, sets, registers and graphs in the recent (2011) [survey paper from Shapiro et al](http://hal.inria.fr/docs/00/55/55/88/PDF/techreport.pdf).

Interestingly, the register implementations correspond directly with the implementations that key value stores use: a last-write-wins register uses timestamps or some equivalent and simply converges to the largest timestamp value; a multi-valued register corresponds to the Dynamo strategy of retaining, exposing and reconciling concurrent changes. For the details, I recommend that you take a look at the papers in the further reading section of this chapter.

## The CALM theorem

The CRDT data structures were based on the recognition that data structures expressible as semilattices are convergent. But programming is about more than just evolving state, unless you are just implementing a data store.

Clearly, order-independence is an important property of any computation that converges: if the order in which data items are received influences the result of the computation, then there is no way to execute a computation without guaranteeing order.

However, there are many programming models in which the order of statements does not play a significant role. For example, in the [MapReduce model](http://en.wikipedia.org/wiki/MapReduce), both the Map and the Reduce tasks are specified as stateless tuple-processing tasks that need to be run on a dataset. Concrete decisions about how and in what order data is routed to the tasks is not specified explicitly, instead, the batch job scheduler is responsible for scheduling the tasks to run on the cluster.

Similarly, in SQL one specifies the query, but not how the query is executed. The query is simply a declarative description of the task, and it is the job of the query optimizer to figure out an efficient way to execute the query (across multiple machines, databases and tables).

Of course, these programming models are not as permissive as a general purpose programming language. MapReduce tasks need to be expressible as stateless tasks in an acyclic dataflow program; SQL statements can execute fairly sophisticated computations but many things are hard to express in it.

However, it should be clear from these two examples that there are many kinds of data processing tasks which are amenable to being expressed in a declarative language where the order of execution is not explicitly specified. Programming models which express a desired result while leaving the exact order of statements up to an optimizer to decide often have semantics that are order-independent. This means that such programs may be possible to execute without coordination, since they depend on the inputs they receive but not necessarily the specific order in which the inputs are received.

The key point is that such programs *may be* safe to execute without coordination. Without a clear rule that characterizes what is safe to execute without coordination, and what is not, we cannot implement a program while remaining certain that the result is correct.

This is what the CALM theorem is about. The CALM theorem is based on a recognition of the link between logical monotonicity and useful forms of eventual consistency (e.g. confluence / convergence). It states that logically monotonic programs are guaranteed to be eventually consistent.

Then, if we know that some computation is logically monotonic, then we know that it is also safe to execute without coordination.

To better understand this, we need to contrast monotonic logic (or monotonic computations) with [non-monotonic logic](http://plato.stanford.edu/entries/logic-nonmonotonic/) (or non-monotonic computations).

<dl>
  <dt>Monotony</dt>
  <dd>if sentence `φ` is a consequence of a set of premises `Γ`, then it can also be inferred from any set `Δ` of premises extending `Γ`</dd>
</dl>

Most standard logical frameworks are monotonic: any inferences made within a framework such as first-order logic, once deductively valid, cannot be invalidated by new information. A non-monotonic logic is a system in which that property does not hold - in other words, if some conclusions can be invalidated by learning new knowledge.

Within the artificial intelligence community, non-monotonic logics are associated with [defeasible reasoning](http://plato.stanford.edu/entries/reasoning-defeasible/) - reasoning, in which assertions made utilizing partial information can be invalidated by new knowledge. For example, if we learn that Tweety is a bird, we'll assume that Tweety can fly; but if we later learn that Tweety is a penguin, then we'll have to revise our conclusion.

Monotonicity concerns the relationship between premises (or facts about the world) and conclusions (or assertions about the world). Within a monotonic logic, we know that our results are retraction-free: [monotone](http://en.wikipedia.org/wiki/Monotonicity_of_entailment) computations do not need to be recomputed or coordinated; the answer gets more accurate over time. Once we know that Tweety is a bird (and that we're reasoning using monotonic logic), we can safely conclude that Tweety can fly and that nothing we learn can invalidate that conclusion.

While any computation that produces a human-facing result can be interpreted as an assertion about the world (e.g. the value of "foo" is "bar"), determining whether a computation in a von Neumann -machine based programming model is monotonic difficult because it is not exactly clear what the relationship between facts and assertions are and whether those relationships are monotonic.

However, there are a number of programming models for which determining monotonicity is possible. In particular, [relational algebra](http://en.wikipedia.org/wiki/Relational_algebra) (e.g. the theoretical underpinnings of SQL) and [Datalog](http://en.wikipedia.org/wiki/Datalog) provide highly expressive languages that have well-understood interpretations.

Both basic Datalog and relational algebra (even with recursion) are known to be monotonic. More specifically, computations expressed using a certain set of basic operators are known to be monotonic (selection, projection, natural join, cross product, union and recursive Datalog without negation), and non-monotonicity is introduced by using more advanced operators (negation, set difference, division, universal quantification, aggregation).

This means that computations expressed using a significant number of operators (e.g. map, filter, join, union, intersection) in those systems are logically monotonic; any computations using those operators are also monotonic and thus safe to run without coordination. Expressions that make use of negation and aggregation, on the other hand, are not safe to run without coordination.

It is important to realize the connection between non-monotonicity and operations that are expensive to perform in a distributed system. Specifically, both *distributed aggregation* and *coordination protocols* can be considered to be a form of negation. As Joe Hellerstein [writes](http://www.eecs.berkeley.edu/Pubs/TechRpts/2010/EECS-2010-90.pdf):

> To establish the veracity of a negated predicate in a distributed setting, an evaluation strategy has to start "counting to 0" to determine emptiness, and wait until the distributed counting process has definitely terminated. Aggregation is the generalization of this idea.

and:

> This idea can be seen from the other direction as well. Coordination protocols are themselves aggregations, since they entail voting: Two-Phase Commit requires unanimous votes, Paxos consensus requires majority votes, and Byzantine protocols require a 2/3 majority. Waiting requires counting.



If, then we can express our computation in a manner in which it is possible to test for monotonicity, then we can perform a whole-program static analysis that detects which parts of the program are eventually consistent and safe to run without coordination (the monotonic parts) - and which parts are not (the non-monotonic ones).

Note that this requires a different kind of language, since these inferences are hard to make for traditional programming languages where sequence, selection and iteration are at the core. Which is why the Bloom language was designed.


## What is non-mononicity good for?

The difference between monotonicity and non-monotonicity is interesting. For example, adding two numbers is monotonic, but calculating an aggregation over two nodes containing numbers is not. What's the difference? One of these is a computation (adding two numbers), while the other is an assertion (calculating an aggregate).

How does a computation differ from an assertion? Let's consider the query "is pizza a vegetable?". To answer that, we need to get at the core: when is it acceptable to infer that something is (or is not) true?

There are several acceptable answers, each corresponding to a different set of assumptions regarding the information that we have and the way we ought to act upon it - and we've come to accept different answers in different contexts.

In everyday reasoning, we make what is known as the [open-world assumption](http://en.wikipedia.org/wiki/Open_world_assumption): we assume that we do not know everything, and hence cannot make conclusions from a lack of knowledge. That is, any sentence may be true, false or unknown.

                                    OWA +             |  OWA +
                                    Monotonic logic   |  Non-monotonic logic
    Can derive P(true)      |   Can assert P(true)    |  Cannot assert P(true)
    Can derive P(false)     |   Can assert P(false)   |  Cannot assert P(true)
    Cannot derive P(true)   |   Unknown               |  Unknown
    or P(false)

When making the open world assumption, we can only safely assert something we can deduce from what is known. Our information about the world is assumed to be incomplete.

Let's first look at the case where we know our reasoning is monotonic. In this case, any (potentially incomplete) knowledge that we have cannot be invalidated by learning new knowledge. So if we can infer that a sentence is true based on some deduction, such as "things that contain two tablespoons of tomato paste are vegetables" and "pizza contains two tablespoons of tomato paste", then we can conclude that "pizza is a vegetable". The same goes for if we can deduce that a sentence is false.

However, if we cannot deduce anything - for example, the set of knowledge we have contains customer information and nothing about pizza or vegetables - then under the open world assumption we have to say that we cannot conclude anything.

With non-monotonic knowledge, anything we know right now can potentially be invalidated. Hence, we cannot safely conclude anything, even if we can deduce true or false from what we currently know.

However, within the database context, and within many computer science applications we prefer to make more definite conclusions. This means assuming what is known as the [closed-world assumption](http://en.wikipedia.org/wiki/Closed_world_assumption): that anything that cannot be shown to be true is false. This means that no explicit declaration of falsehood is needed. In other words, the database of facts that we have is assumed to be complete (minimal), so that anything not in it can be assumed to be false.

For example, under the CWA, if our database does not have an entry for a flight between San Francisco and Helsinki, then we can safely conclude that no such flight exists.

We need one more thing to be able to make definite assertions: [logical circumscription](http://en.wikipedia.org/wiki/Circumscription_%28logic%29). Circumscription is a formalized rule of conjecture. Domain circumscription conjectures that the known entities are all there are. We need to be able to assume that the known entities are all there are in order to reach a definite conclusion.

                                    CWA +             |  CWA +
                                    Circumscription + |  Circumscription +
                                    Monotonic logic   |  Non-monotonic logic
    Can derive P(true)      |   Can assert P(true)    |  Can assert P(true)
    Can derive P(false)     |   Can assert P(false)   |  Can assert P(false)
    Cannot derive P(true)   |   Can assert P(false)   |  Can assert P(false)
    or P(false)

In particular, non-monotonic inferences need this assumption. We can only make a confident assertion if we assume that we have complete information, since additional information may otherwise invalidate our assertion.

What does this mean in practice? First, monotonic logic can reach definite conclusions as soon as it can derive that a sentence is true (or false). Second, nonmonotonic logic requires an additional assumption: that the known entities are all there is.

So why are two operations that are on the surface equivalent different? Why is adding two numbers monotonic, but calculating an aggregation over two nodes not? Because the aggregation does not only calculate a sum but also asserts that it has seen all of the values. And the only way to guarantee that is to coordinate across nodes and ensure that the node performing the calculation has really seen all of the values within the system.

Thus, in order to handle nonmonotonicity one needs to either use distributed coordination to ensure that assertions are made only after all the information is known or make assertions with the caveat that the conclusion can be invalidated later on.

Handling non-monotonicity is important for reasons of expressiveness. This comes down to being able to express non-monotone things; for example, it is nice to be able to say that the total of some column is X. The system must detect that this kind of computation  requires a global coordination boundary to ensure that we have seen all the entities.

Purely monotone systems are rare. It seems that most applications operate under the closed-world assumption even when they have incomplete data, and we humans are fine with that. When a database tells you that a direct flight between San Francisco and Helsinki does not exist, you will probably treat this as "according to this database, there is no direct flight", but you do not rule out the possibility that that in reality such a flight might still exist.

Really, this issue only becomes interesting when replicas can diverge (e.g. during a partition or due to delays during normal operation). Then there is a need for a more specific consideration: whether the answer is based on just the current node, or the totality of the system.

Further, since nonmonotonicity is caused by making an assertion, it seems plausible that many computations can proceed for a long time and only apply coordination at the point where some result or assertion is passed to a 3rd party system or end user. Certainly it is not necessary for every single read and write operation within a system to enforce a total order, if those reads and writes are simply a part of a long running computation.

## The Bloom language

The [Bloom language](http://www.bloom-lang.net/) is a language designed to make use of the CALM theorem. It is a Ruby DSL which has its formal basis in a temporal logic programming language called Dedalus.

In Bloom, each node has a database consisting of collections and lattices. Programs are expressed as sets of unordered statements which interact with collections (sets of facts) and lattices (CRDTs). Statements are order-independent by default, but one can also write non-monotonic functions.


Have a look at the [Bloom website](http://www.bloom-lang.net/) and [tutorials](https://github.com/bloom-lang/bud/tree/master/docs) to learn more about Bloom.

---

## Further reading

#### The CALM theorem, confluence analysis and Bloom

[Joe Hellerstein's talk @RICON 2012](http://vimeo.com/53904989) is a good introduction to the topic, as is [Neil Conway's talk @Basho](http://vimeo.com/45111940). For Bloom in particular, see [Peter Alvaro's talk@Microsoft](http://channel9.msdn.com/Events/Lang-NEXT/Lang-NEXT-2012/Bloom-Disorderly-Programming-for-a-Distributed-World).

- [The Declarative Imperative: Experiences and Conjectures in Distributed Logic](http://www.eecs.berkeley.edu/Pubs/TechRpts/2010/EECS-2010-90.pdf) - Hellerstein, 2010
- [Consistency Analysis in Bloom: a CALM and Collected Approach](http://db.cs.berkeley.edu/papers/cidr11-bloom.pdf) - Alvaro et al., 2011
- [Logic and Lattices for Distributed Programming](http://db.cs.berkeley.edu/papers/UCB-lattice-tr.pdf) - Conway et al., 2012
- [Dedalus: Datalog in Time and Space](http://db.cs.berkeley.edu/papers/datalog2011-dedalus.pdf) - Alvaro et al., 2011

#### CRDTs

[Marc Shapiro's talk @ Microsoft](http://research.microsoft.com/apps/video/dl.aspx?id=153540) is a good starting point for understanding CRDT's.

- [CRDTs: Consistency Without Concurrency Control](http://hal.archives-ouvertes.fr/docs/00/39/79/81/PDF/RR-6956.pdf) - Letitia et al., 2009
- [A comprehensive study of Convergent and Commutative Replicated Data Types](http://hal.inria.fr/docs/00/55/55/88/PDF/techreport.pdf), Shapiro et al., 2011
- [An Optimized conflict-free Replicated Set](http://arxiv.org/pdf/1210.3368v1.pdf) - Bieniusa et al., 2012

#### Dynamo; PBS; optimistic replication

- [Dynamo: Amazon’s Highly Available Key-value Store](http://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf) - DeCandia et al., 2007
- [PNUTS: Yahoo!'s Hosted Data Serving Platform](http://scholar.google.com/scholar?q=PNUTS:+Yahoo!'s+Hosted+Data+Serving+Platform) - Cooper et al., 2008
- [The Bayou Architecture: Support for Data Sharing among Mobile Users](http://scholar.google.com/scholar?q=The+Bayou+Architecture%3A+Support+for+Data+Sharing+among+Mobile+Users) - Demers et al. 1994
- [Probabilistically Bound Staleness for Practical Partial Quorums](http://pbs.cs.berkeley.edu/pbs-vldb2012.pdf) - Bailis et al., 2012
- [Eventual Consistency Today: Limitations, Extensions, and Beyond](https://queue.acm.org/detail.cfm?id=2462076) - Bailis & Ghodsi, 2013
- [Optimistic replication](http://www.ysaito.com/survey.pdf) - Saito & Shapiro, 2005
