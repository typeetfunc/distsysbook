# %chapter_number%. Вверх и вниз по уровню абстракции

В этой главе, мы отправимся в путешествие вверх по уровню абстракции - увидим навозможность некотрых вариантов результатов (CAP и FLP результаты) и спустимся обратно вниз ради производительности.

Если вы занимаетесь каким-либо программированием, то идея о разделении уровней абстракции должна быть вам близка. Вы всегда работаете на каком-то определенном уровне абстракции, и в тоже время используете некотрое API(интерфейс) для доступа к более низкому уровню и предоставляете более высокоуровневое API либо некотрый интерфейс пользователя. Семиуровневая модель [OSI](http://en.wikipedia.org/wiki/OSI_model) хороший пример этого.

Распределенное программирование большей своей частью представляет борьбу с последствиями распределенности системы. Это происходит по той причине что между реальностью, где существует несколько узлов системы и нашими желаниями представить систему в виде единого целого огромная пропасть. Поэтому хорошая абстракция должна соблюдать баланс между осуществимостью в реальности и простотой и производительностью.

Что мы понимаем под выражением "X более абстрактно чем Y"? Во-первых, что X не добавляет ничего нового или фундаментально отличающегося от Y. Фактически, X скорее отбрасывает некотрые аспекты Y и представляет из другим способом чтобы сделать их более управляемыми и простыми в использовании.
Во-вторых, X в некотром смысле должна быть более простой в понимании чем Y, предпологается что сущности Y которых нет в X неважны для расмотрения.

Как написал [Ницще](http://oregonstate.edu/instruct/phl201/modules/Philosophers/Nietzsche/Truth_and_Lie_in_an_Extra-Moral_Sense.htm):

> Каждое понятое возникает из предположения одинаковым неодинакового. И как верно то, что один лист никогда не одинаков совершенно с другим, то и понятие “лист” образовано благодаря произвольному опущению этих индивидуальных различий, благодаря забвению того, что различает; так-то получается представление, будто бы в при роде, кроме листьев, есть еще – “лист”, служащий их первообразом, по образцу которого сотканы, нарисованы, размерены, раскрашены и завиты все листья, но это сделано неловкими руками, так что ни один экземпляр не может считаться верным отражением этого первообраза.

Абстракция, фундаментально - ложь. Каждая конкретная ситуация уникальна, как и каждый конкретный узел системы. Но абстрагирование делает мир управляевым: более простая постановка проблемы - свободная от реальности - более поддающаяся для решения и легко осмысляемы. При условии что мы не упускаем ничего фундаметально важного, такие решения могут быть очень широко применены.

Действительно, если мы сохраняем существенные и фундаментальные стороны расмматриваемого обьекта, тогда результаты могут применятся очень широко. Именно поэтому невозможные результаты так важны для нас: они расмматривают проблему в целом и показывают что можно решить в определенных рамках ограничений и предположений.

Все абстракиции игнорируют некотрые особенности в пользу отождествления вещей которые в реальности являются уникальными. Хитрость заключается в том чтобы избавится от вссего что не является необходимым. Как мы определим что считать необходимым? Скорее всего вы не будете знать это наверняка.

Каждый раз исключая некотрый аспект о системе из ее спецификации, мы рискуем внести источник ошибок или проблем с производительностью. Вот почему иногда нам необходимо идти в другом направлении и возвращать в нашу спецификацию выборочно некотрые аспекты реального железа и проблемы реального мира. Этого может быть достаточно чтобы вернуть некотрые специфические особенности обородувания (такие как физическая последовательность - железо работает всегда последовательно) или другие, чтобы получить систему которая работает достаточно хорошо.

Принимая это все во внимание, какое наименьшее количество информации о реальности мы должны сохранить так чтобы наша модель системы была определяема как распределенная? Модель системы это спецификация характеристик которые мы считаем важными; специфицировав их мы можем определить недостижимость некотрых результатов и появление некотрых проблем.

## Модель системы

Ключевое свойство распределенной системы это(простите за каламбур) распределенность. Более подробно, программа в распределенной системе:

- запускается конкурентно на нескольких узлах...
- использует для сообщения сеть что может вносить элемент нон-детерминизма и потерю сообщений ...
- и не имеет общей памяти или общего времени.

Из этих фактов вытекает много следствий:

- каждый узел запускает программу конкурентно(в тоже время когда ее испольняют другие)
- знания каждого узла локальны: узлы имеют очень быстрый доступ только к их локальному состоянию и любая информация о глобальном состоянии может быть потенциально устаревшей
- узлы могут падать и востанавливатся независимо
- доставка сообщений могут быть задержана либо они могут быть совсем потеряны (независимо от падения узла; крайне сложно отличить падение узла от исчезновения сети до этого узла)
- и время(часы) не синхронизированы между узлами (локальное время не согласуется с глобальным порядком операций который не может быть просто отслежен)

Модель системы включает в себя многие допущения связанные с конкретным дизайном системы.

<dl>
  <dt>Модель системы</dt>
  <dd>набор допущений об окружении и возможностях в рамках которых будет осуществлятся распределенная система</dd>
</dl>

Разные модели различаются в допущениях и возможностях. Среди таких допущений может быть:

- что за ресурсы есть у узла и как они могут отказать
- какими коммуникациями они связаны и как они могут отказать
- свойства общие для всей системы, такие как допущения о времени и порядке

Более надежной моделью системы является та что делает более слабые предположения: любой алгоритм написанный для такой системы будет крайне нечувствителен к различным видам окружения, поскольку он делает мало очень слабых допущений.

С другой стороны, мы можем создать модель которая будет крайне проста по причине того что она делает сильные допущения. Для примера, допущение что узел не может отказать означает что алгоритму не надо уметь обрабатывать отказы. Однако, подобная модель системы крайне нереалистична и нежизнеспособна на практике.

Давайте посмотрим на свойства узлов, коммуникации и времени более детально.

### Узлы в нашей модели системы

Узлы это то на чем происходят вычисления и где хранятся данные. У них есть:

- способность выполнять программу
- способность хранить данные во временной памяти(будет потеряна при падении) и в постоянной памяти(которую можно будет прочитать после падения)
- часы(локальный порядок выполнения операци) - которые могут считатся точными или нет

Узлы исполняют детерминированные алгоритмы: локальные вычисления и локальное состояние после вычисления и отправленные сообщения определяются полученными сообщениями и локальным состоянием которые было на узле когда были получены эти сообщения.

Существует много моделей отказов которые описывают варианты при которых узел может отказать. На практике, большинство систем предпологают crash-recovery модель отказа: то есть, узел может отказать из-за какой либо аварии и может быть востановлен после этого до какого то состояния в прошлом.

Другая альтернатива предпологает что узел может отказать в результате неправильного поведения произвольным способом. Она известна как [Byzantine fault tolerance](http://en.wikipedia.org/wiki/Byzantine_fault_tolerance). Ошибки такого типа крайне редко обрабатывааются в реальных системах, так как крайне трудно сделать алгоритмы устойчивыми к произвольным типам ошибок -такие алгоритмы дороже в разработке и их сложнее запускать. В данной книге не будет обсуждатся данная модель.

### Коммуникационные связи в нашей модели системы

Коммуникационные связи соединяют каждый узел с другим, и позволяют отсылать сообщения в обоих направлениях. Многие книги которые рассказывают о распределенных алгоритмах допускают что существуют индивидуальные связи для каждой пары узлов, с FIFO (первым пришёл — первым ушёл) порядком сообщений, и только отправленные сообщения могут быть доставлены но отправленые сообщения могут быть потеряны.

Некотрые алгоритмы допускают что сеть не может отказывать - сообщения не могут быть потеряны или задержаны на неопределенный срок. Это может быть оправданным в предположении некотрых параметров реального мира, но в общем случае предпочтительнее считать, что сеть вещь непостоянная и могут быть как потери сообщений так и их задержка.

Разделение сети происходит когда сеть между узлами падает в то время как сами узлами остаются работоспособными. Когда это происходит, сообщения могу быть потеряны или задержаны до тех пор пока сеть не востановится. Разделенные узлы могут быть доступны для некотрых клиентов и по этой причине должно рассматриваться иначе нежели падение конкретного уззла. Диаграмма ниже показывает разницу между падение сети и падением узла:

<img src="images/system-of-2.png" alt="replication" style="max-height: 100px;">

Редко делаются дальнейшие предположения о коммуникационных связях. Мы можем предположить что связи работают только в одну сторону или мы можем ввести различную "стоимость" коммуникации(например отзывчивость из-за физического растояния). Однако это довольно редкие положения в коммерческих средах, исключением будет только крайне протяженные связи(гео-разнесенные) так как появляется "WAN latency"(задержки изза отправки данных по глобальной сети) и в этой книге не будет идти речь о них; более детальные модели включающие оценку накладных расходов и топоплогии позволяют лучше оптимизировать расходы на передачу данных.

### Время и порядок

Одним из последствий физического разделения является то что каждый узел воспринимает мир уникальным образом. Это неизбежно, потому что информаация распространяется не быстрее скорости света. Если узлы находятся на разных растояниях друг от друга, тогда любые сообщения от одного узла к другим будут приходить в разное время и потенциально в разном порядке.

Временные допущения являются отражением допущений модели о том насколько мы принимаем в расчет реальность. Возможны две альтернативы:

<dl>
  <dt>Синхронная модель системы</dt>
  <dd>Процессы испольняются в некотрых совпадающих шагах(шаг-в-шаг(lock-step)); есть верхняя граница задержки передачи сообщений; каждый процесс имеет точные часы</dd>
  <dt>Асинхронная модель системы</dt>
  <dd>Нет временных допушений - то есть процессы выполняются с независимой скоростью; не существует границы задержки передачи сообщений; несуществует точных часов</dd>
</dl>

Синхронная модель системы навязывает многие ограничения на время и порядок. Она естественно пологает что узлы имеют один и тот же опыт восприятия мира: что отправленные сообщения были получены с конкретной задержкой не превышающей оговоренную, и процессы выполняются шаг-в-шаг. Это довольно удобно, потому что позволяет делать допущения о порядке и времени, что не позволяет асинхронная модель.

Асинхронность это отсуствие допущений: она предпологает что ты не можешь полагатся на время(или некотрый счетчик времени).

Легче решать проблемы в синхронном подходе, потому что в нем существует много допущений о скорости исполнения, границах задержек сообщений и точности часов - все это позволяет нам исключить неудобные сценарии отказов посчитав их невозможными в нашей модели.

Конечно, такая модель нереалистична. Сеть в реальном мире может исчезнуть и несуществует границ на время передачи сообщений. Системы в реальном мире в лучшем случае частично синхронны: они могут большую часть времени работать корректно и предоставлять ограничения времени доставки сообщений, но будут случаи когда сообщения будут задерживаться на неопределенный срок а часы рассинхронизироваться. В этой книге не будут обсуждаться алгоритмы для синхронных систем, но вы вероятно столкнетесь с ними в других вступительных книгах так как они проще для понимания(но нереалистичны).


### Проблема консенсуса

В остальной части этого текста, мы будем вариъировать параметры модели системы. Далее мы увидим как изменяя 2 свойства системы:

- сетевые разделения могут быть включены в модель отказов либо нет, и
- модель времени может быть синхронной или асинхронной

мы можем влиять на выбор дизайна системы и порассуждаем о двух видах невозможных результатов (FLP и CAP).

Конечно, для дисскуссии нам необходима некотрая проблема решение который мы будем обсуждать. Проблема которую мы будем обсуждать известна как [проблема консенсуса](http://en.wikipedia.org/wiki/Consensus_%28computer_science%29).

Несколько компьютеров(узлов) достигают консенсуса если все они согласны насчет некотрого значения. Более формально:

1. Соглашение: Каждый корректный процесс должен быть согласен с одним и тем же значением что и другие.
2. Целостность: Каждый корректный процесс выбрать не более одного значения, и если он выбирает некотрое значение то оно должно быть предложено некотрым процессом.
3. Завершение: Все процессы должны в конечном счете принять решение.
4. Обоснованность: Если все корректные процессы предлагают значение V, тогда все корректные процессы должны принять значение V.

Проблема консенсуса центральная для многих коммерческих распределенных систем. В конце концов мы хотим надежность и производительность для распределенной системы без проблем вызыванных разобщенностью между распределенными узлами (таких как согласие или несогласие между разделенными узлами), и решение проблемы консенсуса позволяет решить несколько других важных проблем таких как атомарная передача или атомарный коммит.

### Два невозможных результата

Первый невозможный результат, известен как FLP результат - этот результат крайне важен для тех кто разрабатывает распределенные алгоритмы. Второй содержится в так называемой CAP теореме - этот результат более интересен для людей занимающейся практикой, которым необходимы выбирать между разными дизайнами системы но которые не разрабатывают новые распределенные алгоритмы.

## Невозможность результата FLP

Я только кратко подытожу [FLP impossibility result](http://en.wikipedia.org/wiki/Consensus_%28computer_science%29#Solvability_results_for_some_agreement_problems), как прнято считать [более важен](http://en.wikipedia.org/wiki/Dijkstra_Prize) в академических кругах. FLP (названный в честь авторов, Фишер, Линч и Паттерсон) рассматривает проблему консенсуса в асинхронной модели времени (технически, проблему согласия, которая является более слабым вариантом проблемы консенсуса). При этом допускается что узел может отказать только в случае падения; что сеть надежна и что обычные допущения асинхронной модели времени оказываются справедливы - например сообщения могут быть задержаны.

Учитывая эти допущения FLP гласит что "несуществует алгоритма(детерминированного) для решения проблемы консенсуса в асинхронной системе с учетом отказов, даже если сообщения никогда не могут быть потеряны, не более одного процесса может отказать и это может случится только в случае его падения(прекращения выполнения)".

Это означает что нет способа решить проблему консенсуса для минимальной системы таким образом что в системе не будет вечных задержек. Доказательством служит то что если такой алгоритм существует то можно разработать выполнение этого алгоритма в котором он будет нерешенным для произвольного количества времени - что конечно позволительно в асинхронной модели времени. Таким образом подобный алгоритм не может существовать.

Невозможность этого хорошо подчеркивает что допущения асинхронной модели приводят нас к компромису: алгоритм который решает проблему консенсуса должен выбирать между корректностью работы и отказоустойчивостью, когда у нас нет гарантии ограничения времени доставки сообщений.

Понимание этого обычно важно для людей который проектируют алгоритмы, потому что это навязывает очень сложные ограничения для разрешения известных нам проблем в асинхронной модели. CAP теорема более важна для практиков: она делает другие допущения (там предпологаются сетевые отказы, а не отказы отдельных узлов), и делаются более четкие выводы для программистов проектирующих системы.

## CAP теорема

CAP теорема изначально была гипотезой высказанной ученым Эриком Брюером. Это популярная и достаточно удобный способ думать о компромиссе в гарантиях которые дает нам модель системы. Существует также [строгое доказательство](https://www.google.com/search?q=Brewer's+conjecture+and+the+feasibility+of+consistent%2C+available%2C+partition-tolerant+web+services), по [Гилберту](http://www.comp.nus.edu.sg/~gilbert/biblio.html) и [Линчу](http://en.wikipedia.org/wiki/Nancy_Lynch) и нет, [Натан Марц](http://nathanmarz.com/) не развенчал его  не смотря на что так могло показатся глядя на [обсуждение на Y Combinator](http://news.ycombinator.com/).

Теорема утверждает, что из трех свойств:

- Согласованность: все узлы видят одни и теже данные в одно и тоже время.
- Доступность: отказавшие узлы не должны прекращать работу не отказавших.
- Устойчивость к разделению: система продолжает работать даже при потери сообщений из-за сети или отказа узла

система может удовлетворять только двум одновременно. Мы можем выразить это в виде диаграммы где каждое свойство по парно пересекается с каждым - пересечения дают нам три вида возможных систем:

![CAP theorem](images/CAP.png)

Обратите внимание что теорема утверждает что средняя часть(где все свойства пересекаются) не достижима. Остаются 3 возможных типа систем:

- CA (согласованность + доступность). Примером может служить строгие протоколы полного кворума, такие как двухфазный коммит.
- CP (согласованность + устойчивость к разделению). Примером может служить алгоритмы мажоритарного коммита когда меньшая часть кворума недоступна (Paxos).
- AP (доступность + устойчивость к разделению). Примером могут являтся протоколы которые используют ращрешение конфликтов, такие как Dynamo.

CA и CP относятся к одной модели согласованности: строгая согласованность. Единственное отличие что CA системы не устойчивы к любым отказам; а CP системы могут устоять при `f` неисправностях на `2f+1` узлах в невизантийской модели отказов(другими словами, при падении менее половины узлов система остается доступной). Причины этого просты:

- CA система не различает отказ узла и сети, и следовательно должна прекратить запись чего либо для избежания расхождений множественных копий. Так как нельзя определить отказал ли узел или исчезла сеть - единственны безопастным решением является прекратить запись чего либо.
- CP система предотвращает расхождения(то есть поддерживает согласованную копию данных на одном узле) путем ассиметричного поведение с обоих сторон разделения. Меньший раздел прекращает запись а что позволяет большему сохранить возможнность записывать данные сохраняя консинтентность.

I'll discuss this in more detail in the chapter on replication when I discuss Paxos. The important thing is that CP systems incorporate network partitions into their failure model and distinguish between a majority partition and a minority partition using an algorithm like Paxos, Raft or viewstamped replication. CA systems are not partition-aware, and are historically more common: they often use the two-phase commit algorithm and are common in traditional distributed relational databases.



Assuming that a partition occurs, the theorem reduces to a binary choice between availability and consistency.

![Based on http://blog.mikiobraun.de/2013/03/misconceptions-about-cap-theorem.html](images/CAP_choice.png)


I think there are four conclusions that should be drawn from the CAP theorem:

First, that *many system designs used in early distributed relational database systems did not take into account partition tolerance* (e.g. they were CA designs). Partition tolerance is an important property for modern systems, since network partitions become much more likely if the system is geographically distributed (as many large systems are).

Second, that *there is a tension between strong consistency and high availability during network partitions*. The CAP theorem is an illustration of the tradeoffs that occur between strong guarantees and distributed computation.

In some sense, it is quite crazy to promise that a distributed system consisting of independent nodes connected by an unpredictable network "behaves in a way that is indistinguishable from a non-distributed system".

![From the Simpsons episode Trash of the Titans](images/news_120.jpg)

Strong consistency guarantees require us to give up availability during a partition. This is because one cannot prevent divergence between two replicas that cannot communicate with each other while continuing to accept writes on both sides of the partition.

How can we work around this? By strengthening the assumptions (assume no partitions) or by weakening the guarantees. Consistency can be traded off against availability (and the related capabilities of offline accessibility and low latency). If "consistency" is defined as something less than "all nodes see the same data at the same time" then we can have both availability and some (weaker) consistency guarantee.

Third, that *there is a tension between strong consistency and performance in normal operation*.

Strong consistency / single-copy consistency requires that nodes communicate and agree on every operation. This results in high latency during normal operation.

If you can live with a consistency model other than the classic one; a consistency model that allows replicas to lag or to diverge, then you can reduce latency during normal operation and maintain availability in the presence of partitions.

When fewer messages and fewer nodes are involved, an operation can complete faster. But the only way to accomplish that is to relax the guarantees: let some of the nodes be contacted less frequently, which means that nodes can contain old data.

This also makes it possible for anomalies to occur. You are no longer guaranteed to get the most recent value. Depending on what kinds of guarantees are made, you might read a value that is older than expected, or even lose some updates.





Fourth - and somewhat indirectly - that *if we do not want to give up availability during a network partition, then we need to explore whether consistency models other than strong consistency are workable for our purposes*.

For example, even if user data is georeplicated to multiple datacenters, and the link between those two datacenters is temporarily out of order, in many cases we'll still want to allow the user to use the website / service. This means reconciling two divergent sets of data later on, which is both a technical challenge and a business risk. But often both the technical challenge and the business risk are manageable, and so it is preferable to provide high availability.

Consistency and availability are not really binary choices, unless you limit yourself to strong consistency. But strong consistency is just one consistency model: the one where you, by necessity, need to give up availability in order to prevent more than a single copy of the data from being active. As [Brewer himself points out](http://www.infoq.com/articles/cap-twelve-years-later-how-the-rules-have-changed), the "2 out of 3" interpretation is misleading.

If you take away just one idea from this discussion, let it be this: "consistency" is not a singular, unambiguous property. Remember:

<blockquote>
  <p>
   [ACID](http://en.wikipedia.org/wiki/ACID) consistency != <br>
   [CAP](http://en.wikipedia.org/wiki/CAP_theorem) consistency != <br>
   [Oatmeal](http://en.wikipedia.org/wiki/Oatmeal) consistency
  </p>
</blockquote>

Instead, a consistency model is a guarantee - any guarantee - that a data store gives to programs that use it.

<dl>
  <dt>Consistency model</dt>
  <dd>a contract between programmer and system, wherein the system guarantees that if the programmer follows some specific rules, the results of operations on the data store will be predictable</dd>
</dl>

The "C" in CAP is "strong consistency", but "consistency" is not a synonym for "strong consistency".

Let's take a look at some alternative consistency models.

## Strong consistency vs. other consistency models

Consistency models can be categorized into two types: strong and weak consistency models:

- Strong consistency models (capable of maintaining a single copy)
  - Linearizable consistency
  - Sequential consistency
- Weak consistency models (not strong)
  - Client-centric consistency models
  - Causal consistency: strongest model available
  - Eventual consistency models

Strong consistency models guarantee that the apparent order and visibility of updates is equivalent to a non-replicated system. Weak consistency models, on the other hand, do not make such guarantees.

Note that this is by no means an exhaustive list. Again, consistency models are just arbitrary contracts between the programmer and system, so they can be almost anything.

### Strong consistency models

Strong consistency models can further be divided into two similar, but slightly different consistency models:

- *Linearizable consistency*: Under linearizable consistency, all operations **appear** to have executed atomically in an order that is consistent with the global real-time ordering of operations. (Herlihy & Wing, 1991)
- *Sequential consistency*: Under sequential consistency, all operations **appear** to have executed atomically in some order that is consistent with the order seen at individual nodes and that is equal at all nodes. (Lamport, 1979)

The key difference is that linearizable consistency requires that the order in which operations take effect is equal to the actual real-time ordering of operations. Sequential consistency allows for operations to be reordered as long as the order observed on each node remains consistent. The only way someone can distinguish between the two is if they can observe all the inputs and timings going into the system; from the perspective of a client interacting with a node, the two are equivalent.

The difference seems immaterial, but it is worth noting that sequential consistency does not compose.

Strong consistency models allow you as a programmer to replace a single server with a cluster of distributed nodes and not run into any problems.

All the other consistency models have anomalies (compared to a system that guarantees strong consistency), because they behave in a way that is distinguishable from a non-replicated system. But often these anomalies are acceptable, either because we don't care about occasional issues or because we've written code that deals with inconsistencies after they have occurred in some way.

Note that there really aren't any universal typologies for weak consistency models, because "not a strong consistency model" (e.g. "is distinguishable from a non-replicated system in some way") can be almost anything.

### Client-centric consistency models

*Client-centric consistency models* are consistency models that involve the notion of a client or session in some way. For example, a client-centric consistency model might guarantee that a client will never see older versions of a data item. This is often implemented by building additional caching into the client library, so that if a client moves to a replica node that contains old data, then the client library returns its cached value rather than the old value from the replica.

Clients may still see older versions of the data, if the replica node they are on does not contain the latest version, but they will never see anomalies where an older version of a value resurfaces (e.g. because they connected to a different replica). Note that there are many kinds of consistency models that are client-centric.

### Eventual consistency

The *eventual consistency* model says that if you stop changing values, then after some undefined amount of time all replicas will agree on the same value. It is implied that before that time results between replicas are inconsistent in some undefined manner. Since it is [trivially satisfiable](http://www.bailis.org/blog/safety-and-liveness-eventual-consistency-is-not-safe/) (liveness property only), it is useless without supplemental information.

Saying something is merely eventually consistent is like saying "people are eventually dead". It's a very weak constraint, and we'd probably want to have at least some more specific characterization of two things:

First, how long is "eventually"? It would be useful to have a strict lower bound, or at least some idea of how long it typically takes for the system to converge to the same value.

Second, how do the replicas agree on a value? A system that always returns "42" is eventually consistent: all replicas agree on the same value. It just doesn't converge to a useful value since it just keeps returning the same fixed value. Instead, we'd like to have a better idea of the method. For example, one way to decide is to have the value with the largest timestamp always win.

So when vendors say "eventual consistency", what they mean is some more precise term, such as "eventually last-writer-wins, and read-the-latest-observed-value in the meantime" consistency. The "how?" matters, because a bad method can lead to writes being lost - for example, if the clock on one node is set incorrectly and timestamps are used.

I will look into these two questions in more detail in the chapter on replication methods for weak consistency models.

---

## Further reading

- [Brewer's Conjecture and the Feasibility of Consistent, Available, Partition-Tolerant Web Services](http://lpd.epfl.ch/sgilbert/pubs/BrewersConjecture-SigAct.pdf) - Gilbert & Lynch, 2002
- [Impossibility of distributed consensus with one faulty process](http://scholar.google.com/scholar?q=Impossibility+of+distributed+consensus+with+one+faulty+process) - Fischer, Lynch and Patterson, 1985
- [Perspectives on the CAP Theorem](http://scholar.google.com/scholar?q=Perspectives+on+the+CAP+Theorem) - Gilbert & Lynch, 2012
- [CAP Twelve Years Later: How the "Rules" Have Changed](http://www.infoq.com/articles/cap-twelve-years-later-how-the-rules-have-changed) - Brewer, 2012
- [Uniform consensus is harder than consensus](http://scholar.google.com/scholar?q=Uniform+consensus+is+harder+than+consensus) - Charron-Bost & Schiper, 2000
- [Replicated Data Consistency Explained Through Baseball](http://pages.cs.wisc.edu/~remzi/Classes/739/Papers/Bart/ConsistencyAndBaseballReport.pdf) - Terry, 2011
- [Life Beyond Distributed Transactions: an Apostate's Opinion](http://scholar.google.com/scholar?q=Life+Beyond+Distributed+Transactions%3A+an+Apostate%27s+Opinion) - Helland, 2007
- [If you have too much data, then 'good enough' is good enough](http://dl.acm.org/citation.cfm?id=1953140) - Helland, 2011
- [Building on Quicksand](http://scholar.google.com/scholar?q=Building+on+Quicksand) - Helland & Campbell, 2009
