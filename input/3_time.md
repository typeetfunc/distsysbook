# %chapter_number%. Время и порядок

Что означет для нас порядок и почему он важен?

Что мы хотим сказать этим вопросом?

Почему мы так фокусируемся на порядке в первую очередь? Почему нам важно знать что A случилось раньше B? Почему мы не беспокоимся о другом свойстве, таком как цвет?

Хорошо, мои сумасшедшие друзья, давайте вернемся назад к определению распределенной системы чтобы ответить на этот вопрос.

Как мы можем помнить, распределенное программирование описывалось как искуство решать тех же задач что вы решаете на одном компьютере на многих компьютерах.

Этот факт - основа нашей одержимости идеей порядка. Любая система, которая может делать только одну операцию в один момент времени создает абсолютный порядок операций. Подобно людям проходящим через одну дверь, каждая операция имеет определенных предшественника и последователя. Это базовая модель для программирования и потрачено много усилий чтобы она была сохранена.

Традиционная модель это: одна программа, один процесс, одно пространство в памяти, один вычислительный процесс. Операционная система абстрагирует, пряча от нас, тот факт что может быть много CPU или много программ, и память на самом деле может быть разделена между программами. Мы не говорим, что несуществует программ использующих множество потоков или событийно-ориентированную парадигму; просто это все специальная абстракция поверх модели "one/one/one"(одна программа, одна память, один CPU). Программы написаны чтобы исполнятся в определенном порядке: исполнение начинается с верхней строчки и переходит вниз до самой нижней.

Порядок, как свойство получает так много внимания, потому что простейший способ доказать "корректность" работы это сказать "это работает также как и работало бы на одной машине". Ичто обычно означает что а) мы запускаем на всегда одни и теже операции и б) мы их запускаем в одинаковом порядке - даже запуская программу на нескольких машинах.

Хорошим свойством для распределенной системы, которая сохраняет порядок операций, является то что она универсальна. Вам можно не думать о том что за операции надо выполнять с ее помощью, потому что они будут выполнены как будто на одной машине. Это великолепно потому что вы можете использовать одну и ту же системы для различных операций.

В реальности, распределенные программы запускаются на распределенных узлах; с многими CPU и многими потоками операций вместе с тем. Вы попрежнему можете назначать абсолютный порядок операций, но это требует либо точных часов или особых форм сообщения. Вы могли бы присвоить каждой операции временную метку используя абсолютно точные часы, чтобы определить глобальный порядок. Или вы можете какуюто систему коммуникаций между узлами которая может сделать возможным назначать глобальные порядковые номера для операций.

## Абсолютный(полный) и частичный порядок

Естественное состояние для распредленной системы это [частичный порядок](http://en.wikipedia.org/wiki/Partially_ordered_set). Ни сеть ни независимость различных узлов не позволяют намм говорит о каких либо гарантиях относительно порядка; но для каждого узла вы можете соблюдать локальный порядок.

[Абсолютный(полный) порядок](http://en.wikipedia.org/wiki/Total_order) это бинарное отношение, которое определяет порядок для каждого элемента в некотром множестве.

Два уникальных элемента являются **сравниваемыми** когда один из них больше другого. В частично упорядоченном множестве, некотрые пары элементов не являются сравниваемыми и следовательно частичный порядок не указывает точный порядок следования для каждого элемента.

И абсолютный и частичный порядок [транзитивны](http://en.wikipedia.org/wiki/Transitive_relation) и [антисиметричны](http://en.wikipedia.org/wiki/Antisymmetric_relation). Следующие утверждения верны и для частичного и абсолютного порядка для всех a, b, c из X:

    Если a ≤ b и b ≤ a тогда a = b (антисиметричность);
    Если a ≤ b и b ≤ c тогда a ≤ c (транзитивность);

Однако, абсолютный порядок также является [полным отношением](http://en.wikipedia.org/wiki/Total_relation):

    a ≤ b или b ≤ a (полнота) для всех a, b в X

в то время как частичный порядок обладает только [свойством рефлексивности](http://en.wikipedia.org/wiki/Reflexive_relation):

    a ≤ a (рефлексивность) для всех a в X

Заметим что полнота включает в себя рефлексивность; то есть частичный порядок более слабый вариант полного порядка.
Для некотрых элементов в частичном порядке, свойство полноты не будет выполнено - иными словами, некотрые элементы оказываются не сравнимыми.

Ветки Git это пример частично упорядоченного порядка. Как вы возможно знаете, система контроля ревизий git позволяет создавать множество ветвей из одной базовой ветки - например из мастер ветки. Каждая ветвь представляет собой отдельную историю изменений исходного кода основанную на общем предке:

    [ branch A (1,2,0)]  [ master (3,0,0) ]  [ branch B (1,0,2) ]
    [ branch A (1,1,0)]  [ master (2,0,0) ]  [ branch B (1,0,1) ]
                      \  [ master (1,0,0) ]  /

Ветви A и B были получены из общего предка, но нигде неопределенн порядок между ними: они представляют разные истории и не могут быть сведены к одной последовательной истории без дополнительной работы(мерджа). Конечно можно попытатся расположить коммиты в определенном порядке (скажем сортирую сначала по порядку в предке а затем расположить все измения из А и только потом из Б - или наоброт) - но это будет сопровождатся потерей информации так как вы пытаетесь имитировать порядок там где его нету.

В системе, состоящей из одного узла, абсолютный порядок возникает неизбежно: так как инструкции исполняются а сообщения отправляются в определенном порядке. Мы можем положится на абсолютный порядок вычислений - он делает выполнение программ предсказуемым. Такой порядок может поддерживаться и в распределенной системе, но это будет иметь свою цену: коммуникации крайне дороги, а синхронизация времени крайне сложна и хрупка.

# Что такое время?

Время это источник порядка - оно позволяет нам опрделить порядок выполнения операций - который по случайному совпадению может пониматься людьми (секунды, минуты и.т.д).

В некотром смысле, время похоже на любой другой счетчик. Достаточно важно, что большинство компьютеров имеет встроенный сенсор времени, известный как часы. Это настолько важно что человечество научилось синтезировать такой счетчик с некотрой погрешностью используя некотрые физические обьекты (от восковой свечи до атома цезия). Под "синтезировать" понимается что мы можем получить приблизительное значение счетчика в двух физически отдаленных местах используя некотрое физическое свойство без прямой коммуникации.

Временная метка это сокращенное значение для представления состояния мира с появления вселенной до текущего момента - если чтото происходит в определленый момент времени, тогда на это могло повлиять все что произощло до этого момента. Эта идея может быть обобщена до часов на основе причинных связей, которые явно отслеживают зависимости событиями, а не просто предпологают что все произошедшее до определнного события связано с ним. Конечно обычно предпологают, что мы должны беспокоится только о состоянии конкретной системы а не о состоянии всего мира.

Если предположить, что время движется везде с одинаковой скоростью - и это крайне большое предположение к оторому мы еще вернемся -  время и временные метки могут интерпритироватся несколькими полезными нам способами, как:

- Порядок
- Продолжительность
- Интерпритация

*Порядок*. Когда мы говорили, что время это источник порядка имелось ввиду что:

- мы можем присваивать временные метки на неупорядоченные события и тем самым упорядочить их
- мы можем использовать временные метки для обеспечения конкретного порядка операций или доставки сообщений(например путем задержки операции если она поступила не в свою очередь)
- мы можем использовать значение времени чтобы определить, что предшествовало в хронологическом порядке какому либо событию

*Интерпритация* - время это универсальное сравниваемое значение. Абсолютное значение времени может интерпретироватся как дата, что может полезно для людей. Учитывая временную метку начала даунтайма из логов мы можете сказать, что он произошел в прошлую субботу во время  [thunderstorm](https://twitter.com/AWSFail/statuses/218915147060752384).

*Продолжительность* - продолжительность измеряемая во времени имеет отношение к реальному миру. Алгоритмы как правило не беспокоятся о абсолютном значении часов или интерпритации их как даты, но продолжительность может помочь сделать программе несколько полезных выводов. К примеру, по времени потраченому на ожидание ответа можно предположить является система разделенной или просто велики задержки коммуникации.

Природа компонентов распределенных систем такова что они не могут вести себя предсказуемым способом. Они не гарантируют какой либо определенный порядок, скорость или отсутсвие задержек. Каждый узел имеет некотрый локальный порядок - приблизительно последовательное выолнение - но этот локальный порядок независим от порядка других узлов.

Установление (или допущение) порядка это единственный способ сократить пространство возможных путей исполнения и путей возникновения. Людям тяжело рассуждать о вещах который могут происходить в любом порядке - так как в таком случае число возможных перстановок крайне велико - больше чем может вместить человеческий мозг.

## Везде ли одинаково течет время?

Все мы имеем интуитивное понимание времени, основывающееся на нашем личном опыте. К сожалению, это интуитивное понимание облегчает представление абсолютного(глобального) порядка, но не частичного. Гораздо проще представить последовательность, в которой события происходят одно за другим, а не одновременно. Гораздо проще рассуждать о сообщениях, поступаемых в единой последовательности, чем о сообщениях, поступающих в разном порядке и с разными задержками.

Однако, разрабатывая распределенную систему, хочется обойтись без сильных допущений касательно времени и порядка, потому что, чем сильнее допущения, тем более уязвима становится система к проблемам с временем. Более того, наложение порядка влечет за собой дополнительные расходы. Чем больше непоследовательных действий мы можем обрабатывать, тем больше возможность мы сможем получить от распределенной стистемы.

Есть три основных ответа на вопрос "везде ли время течет одинаково?":

- "Глобальные часы": да
- "Локальные часы": нет, но...
- "В отсутствии часов": нет!

Они примерно соответствуют трем временным моделям, о которых говорилось во второй главе: модель синхронной системы работает с глобальными часами, частчно синхронная система - с локальными часами, ассинхронная система вообще не может использовать часы. Давайте разберем эти предположения подробнее.

### Время в модели "глобальных часов"

Модель глобальных часов подразумевает существование абсолютно точных мировых часов, к которым есть доступ у каждого. Это то, как мы привыкли думать о времени, потому что в повседневной жизни небольшие различия во времени не имеют значения.

![Global clock](images/global-clock.png)

Глобальные часы по факту являются источником общего порядка(точный порядок на каждом узле, не смотря на то, что они даже не взаимодействуют друг с другом).

Однако, это идеализированное предствление мира: на самом деле синхронизация часов возможно только с ограниченной точностью. Ограничение обусловлено отсутствием точности в часах обычных компьютеров, а так же задержкой, если используется протокол синхронизации, например такой как [NTP](http://en.wikipedia.org/wiki/Network_Time_Protocol) и кроме того принципиально ограничено [природой пространства-времени](http://en.wikipedia.org/wiki/Time_dilation).

Предположение, что часы в распределенной системе идеально синхронизированы, подразумевает собой предположение, что отсчет времени начался одновременни, а так же, что часы никогда не смогут разойтись. Это полезное допущение, потому что, с ним появляется возможность использовать временные метки без ограничений, чтобы определять глобальный общий порядок.  It's a nice assumption because you can use timestamps freely to determine a global total order - bound by clock drift rather than latency - но это [не тривиальная](http://queue.acm.org/detail.cfm?id=1773943) задача и потенциальное место для аномалий. Существует множество различных сценариев, когда небольшая неполадка - например, случайное изменение пользователем локального времени на компьютере, или добавление в кластер машины с опозданием, или синхронизированные часы..  There are many different scenarios where a simple failure - such as a user accidentally changing the local time on a machine, or an out-of-date machine joining a cluster, or synchronized clocks drifting at slightly different rates and so on that can cause hard-to-trace anomalies.

Nevertheless, there are some real-world systems that make this assumption. Facebook's [Cassandra](http://en.wikipedia.org/wiki/Apache_Cassandra) is an example of a system that assumes clocks are synchronized. It uses timestamps to resolve conflicts between writes - the write with the newer timestamp wins. This means that if clocks drift, new data may be ignored or overwritten by old data; again, this is an operational challenge (and from what I've heard, one that people are acutely aware of). Another interesting example is Google's [Spanner](http://research.google.com/archive/spanner.html): the paper describes their TrueTime API, which synchronizes time but also estimates worst-case clock drift.

### Time with a "Local-clock" assumption

The second, and perhaps more plausible assumption is that each machine has its own clock, but there is no global clock. It means that you cannot use the local clock in order to determine whether a remote timestamp occurred before or after a local timestamp; in other words, you cannot meaningfully compare timestamps from two different machines.

![Local clock](images/local-clock.png)

The local clock assumption corresponds more closely to the real world. It assigns a partial order: events on each system are ordered but events cannot be ordered across systems by only using a clock.

However, you can use timestamps to order events on a single machine; and you can use timeouts on a single machine as long as you are careful not to allow the clock to jump around. Of course, on a machine controlled by an end-user this is probably assuming too much: for example, a user might accidentally change their date to a different value while looking up a date using the operating system's date control.


### Time with a "No-clock" assumption

Finally, there is the notion of logical time. Here, we don't use clocks at all and instead track causality in some other way. Remember, a timestamp is simply a shorthand for the state of the world up to that point - so we can use counters and communication to determine whether something happened before, after or concurrently with something else.

This way, we can determine the order of events between different machines, but cannot say anything about intervals and cannot use timeouts (since we assume that there is no "time sensor"). This is a partial order: events can be ordered on a single system using a counter and no communication, but ordering events across systems requires a message exchange.

One of the most cited papers in distributed systems is Lamport's paper on [time, clocks and the ordering of events](http://research.microsoft.com/users/lamport/pubs/time-clocks.pdf). Vector clocks, a generalization of that concept (which I will cover in more detail), are a way to track causality without using clocks. Cassandra's cousins Riak (Basho) and Voldemort (Linkedin) use vector clocks rather than assuming that nodes have access to a global clock of perfect accuracy. This allows those systems to avoid the clock accuracy issues mentioned earlier.

When clocks are not used, the maximum precision at which events can be ordered across distant machines is bound by communication latency.

## How is time used in a distributed system?

What is the benefit of time?

1. Time can define order across a system (without communication)
2. Time can define boundary conditions for algorithms

The order of events is important in distributed systems, because many properties of distributed systems are defined in terms of the order of operations/events:

- where correctness depends on (agreement on) correct event ordering, for example serializability in a distributed database
- order can be used as a tie breaker when resource contention occurs, for example if there are two orders for a widget, fulfill the first and cancel the second one

A global clock would allow operations on two different machines to be ordered without the two machines communicating directly. Without a global clock, we need to communicate in order to determine order.

Time can also be used to define boundary conditions for algorithms - specifically, to distinguish between "high latency" and "server or network link is down". This is a very important use case; in most real-world systems timeouts are used to determine whether a remote machine has failed, or whether it is simply experiencing high network latency. Algorithms that make this determination are called failure detectors; and I will discuss them fairly soon.

## Vector clocks (time for causal order)

Earlier, we discussed the different assumptions about the rate of progress of time across a distributed system. Assuming that we cannot achieve accurate clock synchronization - or starting with the goal that our system should not be sensitive to issues with time synchronization, how can we order things?

Lamport clocks and vector clocks are replacements for physical clocks which rely on counters and communication to determine the order of events across a distributed system. These clocks provide a counter that is comparable across different nodes.

*A Lamport clock* is simple. Each process maintains a counter using the following rules:

- Whenever a process does work, increment the counter
- Whenever a process sends a message, include the counter
- When a message is received, set the counter to `max(local_counter, received_counter) + 1`

Expressed as code:

    function LamportClock() {
      this.value = 1;
    }

    LamportClock.prototype.get = function() {
      return this.value;
    }

    LamportClock.prototype.increment = function() {
      this.value++;
    }

    LamportClock.prototype.merge = function(other) {
      this.value = Math.max(this.value, other.value) + 1;
    }

A [Lamport clock](http://en.wikipedia.org/wiki/Lamport_timestamps) allows counters to be compared across systems, with a caveat: Lamport clocks define a partial order. If `timestamp(a) < timestamp(b)`:

- `a` may have happened before `b` or
- `a` may be incomparable with `b`

This is known as clock consistency condition: if one event comes before another, then that event's logical clock comes before the others. If `a` and `b` are from the same causal history, e.g. either both timestamp values were produced on the same process; or `b` is a response to the message sent in `a` then we know that `a` happened before `b`.

Intuitively, this is because a Lamport clock can only carry information about one timeline / history; hence, comparing Lamport timestamps from systems that never communicate with each other may cause concurrent events to appear to be ordered when they are not.

Imagine a system that after an initial period divides into two independent subsystems which never communicate with each other.

For all events in each independent system, if a happened before b, then `ts(a) < ts(b)`; but if you take two events from the different independent systems (e.g. events that are not causally related) then you cannot say anything meaningful about their relative order.  While each part of the system has assigned timestamps to events, those timestamps have no relation to each other. Two events may appear to be ordered even though they are unrelated.

However - and this is still a useful property - from the perspective of a single machine, any message sent with `ts(a)` will receive a response with `ts(b)` which is `> ts(a)`.

*A vector clock* is an extension of Lamport clock, which maintains an array `[ t1, t2, ... ]` of N logical clocks - one per each node. Rather than incrementing a common counter, each node increments its own logical clock in the vector by one on each internal event. Hence the update rules are:

- Whenever a process does work, increment the logical clock value of the node in the vector
- Whenever a process sends a message, include the full vector of logical clocks
- When a message is received:
  - update each element in the vector to be `max(local, received)`
  - increment the logical clock value representing the current node in the vector

Again, expressed as code:

    function VectorClock(value) {
      // expressed as a hash keyed by node id: e.g. { node1: 1, node2: 3 }
      this.value = value || {};
    }

    VectorClock.prototype.get = function() {
      return this.value;
    };

    VectorClock.prototype.increment = function(nodeId) {
      if(typeof this.value[nodeId] == 'undefined') {
        this.value[nodeId] = 1;
      } else {
        this.value[nodeId]++;
      }
    };

    VectorClock.prototype.merge = function(other) {
      var result = {}, last,
          a = this.value,
          b = other.value;
      // This filters out duplicate keys in the hash
      (Object.keys(a)
        .concat(b))
        .sort()
        .filter(function(key) {
          var isDuplicate = (key == last);
          last = key;
          return !isDuplicate;
        }).forEach(function(key) {
          result[key] = Math.max(a[key] || 0, b[key] || 0);
        });
      this.value = result;
    };

This illustration ([source](http://en.wikipedia.org/wiki/Vector_clock)) shows a vector clock:

![from http://en.wikipedia.org/wiki/Vector_clock](images/vector_clock.svg.png)

Each of the three nodes (A, B, C) keeps track of the vector clock. As events occur, they are timestamped with the current value of the vector clock. Examining a vector clock such as `{ A: 2, B: 4, C: 1 }` lets us accurately identify the messages that (potentially) influenced that event.

The issue with vector clocks is mainly that they require one entry per node, which means that they can potentially become very large for large systems. A variety of techniques have been applied to reduce the size of vector clocks (either by performing periodic garbage collection, or by reducing accuracy by limiting the size).

We've looked at how order and causality can be tracked without physical clocks. Now, let's look at how time durations can be used for cutoff.

## Failure detectors (time for cutoff)

As I stated earlier, the amount of time spent waiting can provide clues about whether a system is partitioned or merely experiencing high latency. In this case, we don't need to assume a global clock of perfect accuracy - it is simply enough that there is a reliable-enough local clock.

Given a program running on one node, how can it tell that a remote node has failed? In the absence of accurate information, we can infer that an unresponsive remote node has failed after some reasonable amount of time has passed.

But what is a "reasonable amount"? This depends on the latency between the local and remote nodes. Rather than explicitly specifying algorithms with specific values (which would inevitably be wrong in some cases), it would be nicer to deal with a suitable abstraction.

A failure detector is a way to abstract away the exact timing assumptions. Failure detectors are implemented using heartbeat messages and timers. Processes exchange heartbeat messages. If a message response is not received before the timeout occurs, then the process suspects the other process.

A failure detector based on a timeout will carry the risk of being either overly aggressive (declaring a node to have failed) or being overly conservative (taking a long time to detect a crash). How accurate do failure detectors need to be for them to be usable?

[Chandra et al.](http://www.google.com/search?q=Unreliable%20Failure%20Detectors%20for%20Reliable%20Distributed%20Systems) (1996) discuss failure detectors in the context of solving consensus - a problem that is particularly relevant since it underlies most replication problems where the replicas need to agree in environments with latency and network partitions.

They characterize failure detectors using two properties, completeness and accuracy:

<dl>
  <dt>Strong completeness.</dt>
  <dd>Every crashed process is eventually suspected by every correct process.</dd>
  <dt>Weak completeness.</dt>
  <dd>Every crashed process is eventually suspected by some correct process.</dd>
  <dt>Strong accuracy.</dt>
  <dd>No correct process is suspected ever.</dd>
  <dt>Weak accuracy.</dt>
  <dd>Some correct process is never suspected.</dd>
</dl>

Completeness is easier to achieve than accuracy; indeed, all failure detectors of importance achieve it - all you need to do is not to wait forever to suspect someone. Chandra et al. note that a failure detector with weak completeness can be transformed to one with strong completeness (by broadcasting information about suspected processes), allowing us to concentrate on the spectrum of accuracy properties.

Avoiding incorrectly suspecting non-faulty processes is hard unless you are able to assume that there is a hard maximum on the message delay. That assumption can be made in a synchronous system model - and hence failure detectors can be strongly accurate in such a system. Under system models that do not impose hard bounds on message delay, failure detection can at best be eventually accurate.

Chandra et al. show that even a very weak failure detector - the eventually weak failure detector ⋄W (eventually weak accuracy + weak completeness) - can be used to solve the consensus problem. The diagram below (from the paper) illustrates the relationship between system models and problem solvability:

![From Chandra and Toueg. Unreliable failure detectors for reliable distributed systems. JACM 43(2):225–267, 1996.](images/chandra_failure_detectors.png)

As you can see above, certain problems are not solvable without a failure detector in asynchronous systems. This is because without a failure detector (or strong assumptions about time bounds e.g. the synchronous system model), it is not possible to tell whether a remote node has crashed, or is simply experiencing high latency. That distinction is important for any system that aims for single-copy consistency: failed nodes can be ignored because they cannot cause divergence, but partitioned nodes cannot be safely ignored.

How can one implement a failure detector? Conceptually, there isn't much to a simple failure detector, which simply detects failure when a timeout expires. The most interesting part relates to how the judgments are made about whether a remote node has failed.

Ideally, we'd prefer the failure detector to be able to adjust to changing network conditions and to avoid hardcoding timeout values into it. For example, Cassandra uses an [accrual failure detector](https://www.google.com/search?q=The+Phi+accrual+failure+detector), which is a failure detector that outputs a suspicion level (a value between 0 and 1) rather than a binary "up" or "down" judgment. This allows the application using the failure detector to make its own decisions about the tradeoff between accurate detection and early detection.

## Time, order and performance

Earlier, I alluded to having to pay the cost for order. What did I mean?

If you're writing a distributed system, you presumably own more than one computer. The natural (and realistic) view of the world is a partial order, not a total order. You can transform a partial order into a total order, but this requires communication, waiting and imposes restrictions that limit how many computers can do work at any particular point in time.

All clocks are mere approximations bound by either network latency (logical time) or by physics. Even keeping a simple integer counter in sync across multiple nodes is a challenge.

While time and order are often discussed together, time itself is not such a useful property. Algorithms don't really care about time as much as they care about more abstract properties:

- the causal ordering of events
- failure detection (e.g. approximations of upper bounds on message delivery)
- consistent snapshots (e.g. the ability to examine the state of a system at some point in time; not discussed here)

Imposing a total order is possible, but expensive. It requires you to proceed at the common (lowest) speed. Often the easiest way to ensure that events are delivered in some defined order is to nominate a single (bottleneck) node through which all operations are passed.

Is time / order / synchronicity really necessary? It depends. In some use cases, we want each intermediate operation to move the system from one consistent state to another. For example, in many cases we want the responses from a database to represent all of the available information, and we want to avoid dealing with the issues that might occur if the system could return an inconsistent result.

But in other cases, we might not need that much time / order / synchronization. For example, if you are running a long running computation, and don't really care about what the system does until the very end - then you don't really need much synchronization as long as you can guarantee that the answer is correct.

Synchronization is often applied as a blunt tool across all operations, when only a subset of cases actually matter for the final outcome. When is order needed to guarantee correctness? The CALM theorem - which I will discuss in the last chapter - provides one answer.

In other cases, it is acceptable to give an answer that only represents the best known estimate - that is, is based on only a subset of the total information contained in the system. In particular, during a network partition one may need to answer queries with only a part of the system being accessible. In other use cases, the end user cannot really distinguish between a relatively recent answer that can be obtained cheaply and one that is guaranteed to be correct and is expensive to calculate. For example, is the Twitter follower count for some user X, or X+1? Or are movies A, B and C the absolutely best answers for some query? Doing a cheaper, mostly correct "best effort" can be acceptable.

In the next two chapters we'll examine replication for fault-tolerant strongly consistent systems - systems which provide strong guarantees while being increasingly resilient to failures. These systems provide solutions for the first case: when you need to guarantee correctness and are willing to pay for it. Then, we'll discuss systems with weak consistency guarantees, which can remain available in the face of partitions, but that can only give you a "best effort" answer.

---

## Further reading

### Lamport clocks, vector clocks

- [Time, Clocks and Ordering of Events in a Distributed System](http://research.microsoft.com/users/lamport/pubs/time-clocks.pdf) - Leslie Lamport, 1978

### Failure detection

- [Unreliable failure detectors and reliable distributed systems](http://scholar.google.com/scholar?q=Unreliable+Failure+Detectors+for+Reliable+Distributed+Systems) - Chandra and Toueg
- [Latency- and Bandwidth-Minimizing Optimal Failure Detectors](http://www.cs.cornell.edu/people/egs/sqrt-s/doc/TR2006-2025.pdf) - So & Sirer, 2007
- [The failure detector abstraction](http://scholar.google.com/scholar?q=The+failure+detector+abstraction), Freiling, Guerraoui & Kuznetsov, 2011

### Snapshots

- [Consistent global states of distributed systems: Fundamental concepts and mechanisms](http://scholar.google.com/scholar?q=Consistent+global+states+of+distributed+systems%3A+Fundamental+concepts+and+mechanisms), Ozalp Babaogly and Keith Marzullo, 1993
- [Distributed snapshots: Determining global states of distributed systems](http://scholar.google.com/scholar?q=Distributed+snapshots%3A+Determining+global+states+of+distributed+systems), K. Mani Chandy and Leslie Lamport, 1985

### Causality

- [Detecting Causal Relationships in Distributed Computations: In Search of the Holy Grail](http://www.vs.inf.ethz.ch/publ/papers/holygrail.pdf) - Schwarz & Mattern, 1994
- [Understanding the Limitations of Causally and Totally Ordered Communication](http://scholar.google.com/scholar?q=Understanding+the+limitations+of+causally+and+totally+ordered+communication) - Cheriton & Skeen, 1993
