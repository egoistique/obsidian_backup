# 1 Cтек потока, память
**

На рис. 4.2 представлен один процесс Microsoft Windows с загруженной в него исполняющей средой CLR. У процесса может быть много потоков. После создания потоку выделяется стек размером в 1 Мбайт. Выделенная память используется для передачи параметров в методы и хранения определенных в пределах методов локальных переменных. На рис. 4.2 справа показана память стека одного потока. Стеки заполняются от области верхней памяти к области нижней памяти (то есть от старших к младшим адресам). На рисунке поток уже выполняет какой-то код, и в его стеке уже есть какие-то данные (отмечены областью более темного оттенка вверху стека). А теперь представим, что поток выполняет код, вызывающий метод M1

**
![](attachments/Pasted%20image%2020251021225533.png)
**

Все методы, кроме самых простых, содержат некоторый входной код (prologue code), инициализирующий метод до начала его работы. Кроме того, эти методы содержат выходной код (epilogue code), выполняющий очистку после того, как метод завершит свою основную работу, чтобы возвратить управление вызывающей программе. В начале выполнения метода M1 его входной код выделяет в стеке потока память для локальной переменной name (рис. 4.3). Далее M1 вызывает метод M2, передавая в качестве аргумента локальную переменную name. При этом адрес локальной переменной name заталкивается в стек (рис. 4.4). Внутри метода M2 местоположение стека хранится в переменной-параметре s. При вызове метода адрес возврата в вызывающий метод также заталкивается в стек (показано на рис. 4.4)

**
![](attachments/Pasted%20image%2020251023233649.png)
**

В начале выполнения метода M2 его входной код выделяет в стеке потока память для локальных переменных length и tally (рис. 4.5). Затем выполняется код метода M2. В конце концов, выполнение M2 доходит до команды возврата, которая записывает в указатель команд процессора адрес возврата из стека, и стековый кадр M2 возвращается в состояние, показанное на рис. 4.3. С этого момента продолжается выполнение кода M1, который следует сразу за вызовом M2, а стековый кадр метода находится в состоянии, необходимом для работы M1. В конечном счете, метод M1 возвращает управление вызывающей программе, устанавливая указатель команд процессора на адрес возврата (на рисунках не показан, но в стеке он находится прямо над аргументом name), и стековый кадр M1 возвращается в состояние, показанное на рис. 4.2. С этого момента продолжается выполнение кода вызвавшего метода, причем начинает выполняться код, непосредственно следующий за вызовом M1, а стековый кадр вызвавшего метода находится в состоянии, необходимом для его работы.

**
![](attachments/Pasted%20image%2020251023233701.png)

**

**все объекты в куче содержат два дополнительных члена: указатель на объект-тип и индекс блока синхронизации.** В объектах типа Employee и Manager оба эти члена присутствуют. При определении типа можно включить 136 Глава 4. Основы типов в него статические поля данных. Байты для этих статических полей выделяются в составе самих объектов-типов. Наконец, у каждого объекта-типа есть таблица методов с входными точками всех методов, определенных в типе. Эта таблица методов уже обсуждалась в главе 1. Так как в типе Employee определены три метода (GetYearsEmployed, GenProgressReport и Lookup), в соответствующей таблице методов есть три записи. В типе Manager определен один метод (переопределенный метод GenProgressReport), который и представлен в таблице методов этого типа.

**
![](attachments/Pasted%20image%2020251023233711.png)

![](attachments/Pasted%20image%2020251023233723.png)
**

Следующая строка метода M3 вызывает статический метод Lookup объекта Employee. При вызове этого метода CLR определяет местонахождение объекта-типа, соответствующего типу, в котором определен статический метод. Затем на основании таблицы методов объекта-типа среда CLR находит точку входа в вызываемый метод, обрабатывает код JIT-компилятором (при необходимости) и передает управление полученному машинному коду. Для нашего обсуждения достаточно предположить, что метод Lookup объекта Employee выполняет запрос к базе данных, чтобы найти сведения о Joe. Допустим также, что в базе данных указано, что Joe занимает должность менеджера, поэтому код метода Lookup создает в куче новый объект Manager, инициализирует его данными Joe и возвращает адрес готового объекта. Адрес размещается в локальной переменной e. Результат этой операции показан на рис. 4.10. Следующая строка метода M3 вызывает виртуальный экземплярный метод GenProgressReport в Employee. При вызове виртуального экземплярного метода CLR приходится выполнять некоторую дополнительную работу. Во-первых, CLR обращается к переменной, используемой для вызова, и затем следует по адресу вызывающего объекта. В данном случае переменная e указывает на объект Joe типа Manager. Во-вторых, CLR проверяет у объекта внутренний указатель на объект-тип. Затем CLR находит в таблице методов объекта-типа запись вызываемого метода, обрабатывает код JIT-компилятором (при необходимости) и вызывает полученный машинный код. В нашем случае вызывается реализация метода GenProgressReport в Manager, потому что e ссылается на объект Manager. Результат этой операции показан на рис. 4.12. Заметьте, если метод Lookup в Employee обнаружит, что Joe — это всего лишь Employee, а не Manager, то Lookup создаст объект Employee, в котором указатель на объект-тип ссылается на объект-тип Employee; это приведет к тому, что выполнится реализация GenProgressReport из Employee, а не из Manager

**

![](attachments/Pasted%20image%2020251023233738.png)

**

# 2 Cсылочные и значимые типы
    
## 1️⃣ Общая информация
Память для ссылочных типов всегда выделяется из управляемой кучи, а оператор C# new возвращает адрес в памяти, где размещается сам объект. При работе со ссылочными типами необходимо учитывать следующие обстоятельства, относящиеся к производительности приложения:

  память для ссылочных типов всегда выделяется из управляемой кучи; 

 каждый объект, размещаемый в куче, содержит дополнительные члены, подлежащие инициализации;

 незанятые полезной информацией байты объекта обнуляются (это касается полей); 

 размещение объекта в управляемой куче со временем инициирует сборку мусора

  

Проектируя свой тип, проверьте, не использовать ли вместо ссылочного типа значимый. Иногда это позволяет повысить эффективность кода. Сказанное особенно справедливо для типа, удовлетворяющего всем перечисленным далее условиям.

  Тип ведет себя подобно примитивному типу. В частности, это означает, что тип достаточно простой и у него нет членов, способных изменить экземплярные поля типа, в этом случае говорят, что тип неизменяемый (immutable). На самом деле, многие значимые типы рекомендуется помечать спецификатором readonly 

 Тип не обязан иметь любой другой тип в качестве базового. 

 Тип не имеет производных от него типов.

  

Также необходимо учитывать размер экземпляров типа, потому что по умолчанию аргументы передаются по значению; при этом поля экземпляров значимого типа копируются, что отрицательно сказывается на производительности. Повторюсь: для метода, возвращающего значимый тип, поля экземпляра копируются в память, выделенную вызывающим кодом в месте возврата из метода, что снижает эффективность работы программы. Поэтому в дополнение к перечисленным условиям следует объявлять тип как значимый, если верно хотя бы одно из следующих условий: 

 Размер экземпляров типа мал (примерно 16 байт или меньше). 

 Размер экземпляров типа велик (более 16 байт), но экземпляры не передаются в качестве параметров метода или не являются возвращаемыми из метода значениями. 

  

Основное достоинство значимых типов в том, что они не размещаются в управляемой куче.

 Конечно, в сравнении со ссылочными типами у значимых типов есть недостатки. Важнейшие отличия между значимыми и ссылочными типы: 

 Объекты значимого типа существуют в двух формах (см. следующий раздел): неупакованной (unboxed) и упакованной (boxed). Ссылочные типы бывают только в упакованной форме.

  Значимые типы являются производными от System.ValueType. Этот тип имеет те же методы, что и System.Object. Однако System.ValueType переопределяет метод Equals, который возвращает true, если значения полей в обоих объектах совпадают. Кроме того, в System.ValueType переопределен метод GetHashCode, который создает хеш-код по алгоритму, учитывающему значения полей экземпляра объекта. Из-за проблем с производительностью в реализации по умолчанию, определяя собственные значимые типы значений, надо переопределить и написать свою реализацию методов Equals и GetHashCode. О методах Equals и GetHashCode рассказано в конце этой главы.  Поскольку в объявлении нового значимого или ссылочного типа нельзя указывать значимый тип в качестве базового класса, создавать в значимом типе новые виртуальные методы нельзя. Методы не могут быть абстрактными и неявно являются запечатанными (то есть их нельзя переопределить). 

 Переменные ссылочного типа содержат адреса объектов в куче. Когда переменная ссылочного типа создается, ей по умолчанию присваивается null, то есть в этот момент она не указывает на действительный объект. Попытка задействовать переменную с таким значением приведет к генерации исключения NullReferenceException. В то же время в переменной значимого типа всегда содержится некое значение соответствующего типа, а при инициализации всем членам этого типа присваивается 0. Поскольку переменная значимого типа не является указателем, при обращении к значимому типу исключение NullReferenceException возникнуть не может. CLR поддерживает понятие значимого типа особого вида, допускающего присваивание null (nullable types). Этот тип обсуждается в главе 19.  Когда переменной значимого типа присваивается другая переменная значимого типа, выполняется копирование всех ее полей. Когда переменной ссылочно- Ссылочные и значимые типы 155 го типа присваивается переменная ссылочного типа, копируется только ее адрес.  Вследствие сказанного в предыдущем пункте несколько переменных ссылочного типа могут ссылаться на один объект в куче, благодаря чему, работая с одной переменной, можно изменить объект, на который ссылается другая переменная. В то же время каждая переменная значимого типа имеет собственную копию данных «объекта», поэтому операции с одной переменной значимого типа не влияют на другую переменную.  Так как неупакованные значимые типы не размещаются в куче, отведенная для них память освобождается сразу при возвращении управления методом, в котором описан экземпляр этого типа (в отличие от ожидания уборки мусора).**

**

## 2️⃣ упаковка
    

Для преобразования значимого типа в ссылочный служит упаковка (boxing).

При упаковке экземпляра значимого типа происходит следующее.

1. В управляемой куче выделяется память. Ее объем определяется длиной значимого типа и двумя дополнительными членами — указателем на типовой объект

и индексом блока синхронизации. Эти члены необходимы для всех объектов

в управляемой куче.

2. Поля значимого типа копируются в память, только что выделенную в куче.

3. Возвращается адрес объекта. Этот адрес яв

  

## 3️⃣ распаковка
    

Сначала извлекается адрес полей Point из упакованного объекта Point. Этот процесс называют распаковкой (unboxing). Затем значения полей копируются из кучи в экземпляр значимого типа, находящийся в стеке.

Распаковка не является точной противоположностью упаковки. Она гораздо менее ресурсозатратна, чем упаковка, и состоит только в получении указателя на исходный значимый тип (поля данных), содержащийся в объекте. В сущности, указатель ссылается на неупакованную часть упакованного экземпляра, и никакого копирования при распаковке (в отличие от упаковки) не требуется. Однако вслед за распаковкой обычно выполняется копирование полей.

## 4️⃣ Equals, HashCode

---

**1.** Поскольку **неупакованные значимые типы** не имеют индекса блока синхронизации, то не может быть и нескольких потоков, синхронизирующих свой доступ к экземпляру через методы типа `System.Threading.Monitor` (или инструкция `lock` языка C#).

---

**2.** Определяя собственный тип и приняв решение переопределить `Equals`, обеспечьте поддержку **четырех характеристик**, присущих равенству:

1️⃣ **Рефлексивность:** `x.Equals(x)` должно возвращать `true`.  
2️⃣ **Симметричность:** `x.Equals(y)` и `y.Equals(x)` должны возвращать одно и то же значение.  
3️⃣ **Транзитивность:** если `x.Equals(y)` возвращает `true` и `y.Equals(z)` возвращает `true`, то `x.Equals(z)` также должно возвращать `true`.  
4️⃣ **Постоянство:** если в двух сравниваемых значениях не произошло изменений, результат сравнения тоже не должен измениться.

---

**3.** Если вы определяете тип и переопределяете метод `Equals`, вы должны переопределить и метод `GetHashCode`.

---

**4.** Причина, по которой в типе должны быть определены оба метода — `Equals` и `GetHashCode`, — состоит в том, что реализация типов  
`System.Collections.Hashtable`, `System.Collections.Generic.Dictionary` и любых других коллекций требует, чтобы **два равных объекта имели одинаковые значения хеш-кодов**.

👉 Поэтому, переопределяя `Equals`, нужно переопределить и `GetHashCode`, обеспечив **соответствие алгоритмов** вычисления равенства и хеш-кода объекта.

---

**5.** По сути, когда вы добавляете пару **«ключ-значение»** в коллекцию:

- сначала вычисляется **хеш-код ключа**, чтобы определить, в каком сегменте будет храниться пара;
    
- при поиске коллекция вычисляет хеш-код и ищет в том же сегменте.
    

⚠️ Если вы измените ключ объекта, хранящийся в коллекции, — коллекция **не сможет** его найти.  
➡️ Чтобы изменить ключ:  
удалите пару → измените ключ → добавьте новую пару в коллекцию.

---

**6.** Рекомендации при выборе алгоритма вычисления хеш-кодов:

1️⃣ Используйте **случайное распределение** для повышения производительности хеш-таблицы.  
2️⃣ Можно вызывать `GetHashCode` базового типа, но **лучше не использовать** встроенные реализации `Object` или `ValueType` — они неэффективны.  
3️⃣ В алгоритме должно использоваться **как минимум одно экземплярное поле**.  
4️⃣ Поля, участвующие в алгоритме, **не должны изменяться** после создания объекта.  
5️⃣ Алгоритм должен быть **максимально быстрым**.  
6️⃣ Объекты с одинаковым значением должны возвращать **одинаковый хеш-код** (например, два `String` с одинаковым текстом).

---

**7.** Для упрощения разработки с помощью **отражений** или **взаимодействия с другими компонентами**, компилятор C# предлагает помечать типы как **динамические (`dynamic`)**.

# 3️⃣ Основные сведения о членах и типах

---

### 🔹 Константы

**Константа** — идентификатор, определяющий постоянную величину.  
Используется для упрощения чтения и сопровождения кода.

- Константы **всегда связаны с типом**, а не с экземпляром.
    
- На логическом уровне — **всегда статические члены**.
    
- Подробнее о константах — в главе 7.
    

💡 **Summary:** Константы — неизменяемые статические значения, хранящиеся в метаданных и встраиваемые компилятором напрямую в IL-код.

---

### 🔹 Поля

**Поле** — это значение данных (только для чтения или для чтения/записи).  
Может быть:

- **статическим** — часть состояния типа;
    
- **экземплярным** — часть состояния объекта.
    

⚠️ Рекомендуется ограничивать доступ к полям, чтобы защитить состояние типа или объекта.  
Подробнее — в главе 7.

💡 **Summary:** Поля хранят состояние объекта или типа; прямой доступ к ним должен быть ограничен.

---

### 🔹 Конструкторы

**Конструктор экземпляров** — метод, инициализирующий поля экземпляра при создании.  
**Конструктор типа** — метод, инициализирующий статические поля.  
Подробнее — в главе 8.

💡 **Summary:** Конструкторы гарантируют корректную инициализацию экземпляров и типов перед использованием.

---
### 🔹 Методы

**Метод** — функция, выполняющая операции над состоянием типа или объекта.  
Может быть:

- **статическим** (работает с типом),
    
- **экземплярным** (работает с объектом).  
    Подробнее — в главе 8.
    

💡 **Summary:** Методы реализуют поведение типа, читая и изменяя его состояние.

---

### 🔹 Перегруженные операторы

**Перегруженный оператор** определяет, как объект должен реагировать на использование конкретного оператора (`+`, `-`, `==` и т. д.).

- **Не входит в спецификацию CLS**, так как не все языки поддерживают перегрузку операторов.  
    Подробнее — в главе 8.
    

💡 **Summary:** Перегрузка операторов расширяет синтаксическую выразительность, но снижает совместимость между языками.

---

### 🔹 Операторы преобразования

Методы, задающие правила **явного или неявного преобразования** типа.  
Как и перегруженные операторы, **не входят в CLS**.  
Подробнее — в главе 8.

💡 **Summary:** Позволяют управлять преобразованием между типами, повышая безопасность и читаемость кода.

---

### 🔹 Свойства

**Свойство** — механизм, предоставляющий простой синтаксис (как у полей) для управления состоянием объекта с проверкой его целостности.

- Бывают **необобщенные** (часто) и **обобщенные** (редко, обычно в коллекциях).  
    Подробнее — в главе 10.
    

💡 **Summary:** Свойства — контролируемый интерфейс доступа к данным объекта.

---

### 🔹 События

**Событие** — механизм уведомления о произошедших изменениях.

- **Статические события** уведомляют статические или экземплярные методы.
    
- **Экземплярные события** уведомляют методы конкретных объектов.
    
- Событие состоит из двух методов (регистрация и отмена регистрации) и **поля-делегата**.  
    Подробнее — в главе 11.
    

💡 **Summary:** События реализуют шаблон подписки и уведомления, обеспечивая реакцию на изменение состояния объекта.

---

### 🔹 Вложенные типы

Тип может **содержать другие типы** для логического разбиения кода.  
💡 **Summary:** Вложенные типы помогают структурировать большие классы, разделяя функциональность.

---

## ⚙️ Метаданные

Метаданные создаются компилятором **для всех членов типа**.

- Единый формат метаданных обеспечивает **совместимость между языками**.
    
- CLR использует метаданные для управления константами, полями, конструкторами, методами, свойствами и событиями во время выполнения.
    

💡 **Summary:** Метаданные — ключевой механизм, обеспечивающий кросс-языковую интеграцию и работу CLR.

---

## 🔐 Модификаторы доступа к членам

>![](attachments/Pasted%20image%2020251023235349.png)

- Все **члены интерфейсов** по требованию CLR **должны быть открытыми**.
    
- Компилятор C# автоматически делает их открытыми и **запрещает явно указывать модификаторы доступа**.
    

💡 **Summary:** Интерфейсные члены всегда публичные, независимо от указания программиста.

## 🧱 Статические классы

Ограничения для статических классов:

- Должен быть **потомком System.Object**. — наследование любому другому базовому классу лишено смысла, поскольку наследование применимо только к объектам, а создать экземпляр статического класса невозможно.
    
- **Не может реализовывать интерфейсы** **поскольку методы интерфейса можно вызывать только через экземпляры класса.**
    
- Может содержать **только статические члены**.**  (поля, методы, свойства и события). Любые экземплярные члены вызовут ошибку компиляции.

    
- Нельзя использовать как **поле, параметр или локальную переменную**.  поскольку это подразумевает существование переменной, ссылающейся на экземпляр, что запрещено. Обнаружив подобное обращение со статическим классом, компилятор вернет сообщение об ошибке
    

💡 **Summary:** Статические классы — контейнеры для функций и данных, не требующих создания экземпляров.

## 🧩 Частичные классы, структуры и интерфейсы

Ключевое слово `partial` позволяет **разбивать определение типа на несколько файлов**.  
CLR объединяет все части при компиляции.

Причины использования:  
1️⃣ **Управление версиями** — несколько разработчиков могут работать над одним типом.  
2️⃣ **Разделение функционала** — логическое группирование методов и свойств.  
3️⃣ **Автоматически генерируемый код** — Visual Studio разделяет пользовательский и машинный код.

💡 **Summary:** Частичные типы упрощают сопровождение, позволяют разделять ответственность и повышают удобство работы в IDE.

---

## 🔒 Модификаторы доступа

> ![](attachments/Pasted%20image%2020251023235713.png)

- **Невиртуальные методы** вызываются быстрее, чем виртуальные,  
    так как CLR при вызове виртуальных методов проверяет тип объекта, чтобы выяснить, где находится метод.

💡 **Summary:** Использование невиртуальных методов повышает производительность при вызове.

##  🧩 **Константы**

  

**Константа (constant) — это идентификатор, значение которого никогда не меняется. Значение, связанное с именем константы, должно определяться во время компиляции.** Затем компилятор сохраняет значение константы в метаданных модуля. Это значит, что константы можно определять только для таких типов, которые компилятор считает примитивными. Тем не менее C# позволяет определить константную переменную, не относящуюся к элементарному типу, если присвоить ей значение null.

  
  

Так как значение констант никогда не меняется, константы всегда считаются частью типа. Иначе говоря, константы считаются статическими, а не экземплярными членами. Определение константы приводит в конечном итоге к созданию метаданных.

  

Встретив в исходном тексте имя константы, компилятор просматривает метаданные модуля, в котором она определена, извлекает значение константы и внедряет его в генерируемый им IL-код. Поскольку значение константы внедряется прямо в код, в период выполнения память для констант не выделяется. Кроме того, **нельзя получать адрес константы и передавать ее по ссылке.**

## 🧩 Поля

  
  

![[Pasted image 20251028125621.png]]

  

**

  

Динамическая память для хранения поля типа выделяется в пределах объекта типа, который создается при загрузке типа в домен приложений (см. главу 22), что обычно происходит при JIT-компиляции любого метода, ссылающегося на этот тип. Динамическая память для хранения экземплярных полей выделяется при создании экземпляра данного типа. Поскольку поля хранятся в динамической памяти, их значения можно получить лишь в период выполнения. 

  

Данные же в неизменяемые (readonly) поля можно записывать только при исполнении конструктора (который вызывается лишь раз — при создании объекта).

  

Неизменность.поля.ссылочного.типа.означает.неизменность.ссылки,.которую.этот. тип содержит, а вовсе не объекта, на которую указывает ссылка



## 🧩 Методы

### ⚙️ **Конструкторы**

Конструкторы — это специальные методы, позволяющие корректно инициализировать новый экземпляр типа. В таблице определений, входящих в метаданные, методы-конструкторы всегда отмечают сочетанием .ctor.

 При создании экземпляра объекта ссылочного типа выделяется память для полей данных экземпляра и инициализируются служебные поля (указатель на объект-тип и индекс блока синхронизации), после чего вызывается конструктор экземпляра, устанавливающий исходное состояние нового объекта.

Невозможность наследования означает, что к конструктору экземпляров нельзя применять модификаторы virtual, new, override, sealed и abstract.

C# не позволяет определять для значимого типа конструкторы без параметров.


### ⚙️ Конструкторы типов

 **CLR поддерживает конструкторы типов (так же известные как статические конструкторы, конструкторы классов и инициализаторы типов).

 конструкторы типов служат для установки первоначального состояния типа. По умолчанию у типа не определено конструктора. У типа не может быть более одного конструктора; кроме того, у конструкторов типов никогда не бывает параметров. 


 Конструкторы типов всегда должны быть закрытыми, чтобы код разработчика не смог их вызвать, напротив, в то же время среда CLR всегда способна вызвать конструктор типа.


У вызова конструктора типа есть некоторые особенности. При компиляции метода JIT-компилятор обнаруживает типы, на которые есть ссылки из кода. Если в каком-либо из типов определен конструктор, JIT-компилятор проверяет, был ли исполнен конструктор типа в данном домене приложений. Если нет, JIT компилятор создает в IL-коде вызов конструктора типа. Если же код уже исполнялся, JIT-компилятор вызова конструктора типа не создает, так как «знает», что тип уже инициализирован.


Поскольку.CLR.гарантирует,.что.конструктор.типа.выполняется.только.однажды. в.каждом.домене.приложений,.а.также.обеспечивает.его.безопасность.по.отношению.к.потокам,.конструктор.типа.лучше.всего.подходит.для.инициализации.всех. объектов-одиночек.(singleton), необходимых для существования типа 

  
Код конструктора типа может обращаться только к статическим полям типа; обычно это делается, чтобы их инициализировать.


Конструктор типа не должен вызывать конструктор базового класса. Этот вызов не обязателен, так как ни одно статическое поле типа не используется совместно с базовым типом и не наследуется от него.

### ⚙️ Методы перегруженных операторов


Спецификация CLR требует, чтобы перегруженные операторные методы были открытыми и статическими. Дополнительно C# (и многие другие языки) требует, чтобы у операторного метода тип, по крайней мере, одного из параметров или возвращаемого значения совпадал с типом, в котором определен операторный метод. Причина этого ограничения в том, что оно позволяет компилятору C# в разумное время находить кандидатуры операторных методов для привязки.

###  ⚙️ Методы операторов преобразования

 **C# требует, чтобы у метода преобразования тип, по крайней мере, одного из параметров или возвращаемого значения совпадал с типом, в котором определен операторный метод. Причина этого ограничения в том, что оно позволяет компилятору C# в разумное время находить кандидатуры операторных методов для привязки.


Ключевое слово implicit указывает компилятору C#, что наличие в исходном тексте явного приведения типов не обязательно для генерации кода, вызывающего метод оператора преобразования. Ключевое слово explicit позволяет компилятору вызывать метод только тогда, когда в исходном тексте происходит явное приведение типов.


С#.генерирует.код.вызова.операторов.неявного.преобразования.в.случае,.когда. используется.выражение.приведения.типов .Однако.операторы.неявного.преобразования.никогда.не.вызываются,.если.используется.оператор.as или is

### ⚙️ Методы расширения 

Методы расширения позволяют определить статический метод, который вызывается посредством синтаксиса экземплярного метода. 

 Для того чтобы превратить метод IndexOf в метод расширения, мы просто добавим ключевое слово this перед первым аргументом

**![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAlQAAACYCAYAAAAiLXfPAAAAAXNSR0IArs4c6QAAIABJREFUeF7t3Qv8rlOVB/CnyzDIpMukiEjRSKRxGUIh5JYTucxxS2gopjCpIUyXEyLGaCqXmjK5jPvQhSMjTUI1Exkq5c7IVIOmSebCfL770/7Pc57zvu+z38v/+q71+ZyPOud5n2fv3957rd9ea+29nvH0008/XYUEAoFAIBAIBAKBQCAQCAyMwDOCUA2MXfwwEAgEAoFAIBAIBAKBhEAQqpgIgUAgEAgEAoFAIBAIDIlAEKohAYyfBwKBQCAQCAQCgUAgEIQq5kAgEAgEAoFAIBAIBAJDIhCEakgA4+eBQCAQCAQCgUAgEAgEoYo5EAgEAoFAIBAIBAKBwJAIBKEaEsD4eSAQCAQCgUAgEAgEAtNKqH71q19Vv/u7v1s9+9nP7joSv/71r6unnnoq/fuznvWsaqmllur67C9+8Yvq+c9/fvWMZzxjrEb25z//efXCF75w2vrsKrP//M//nPj+EkssUfkT8v8I/Md//Ee15JJLzhpc/vu//7uy9p773OcuNoz9rMn84yeffDKt417rN+bLogjMtjmj9T/72c+qf/7nf65WXnnl6g/+4A9iSAOBsUJgWgkV4vO3f/u31V577dUV9Je+9KXVQw89lP79D//wD6vvfve7HZ/92te+Vm2zzTbVe9/73uoTn/jEtA4ipfLv//7v1RprrDFUO3784x9Xyy67bPXiF7+463uOPfbY6qMf/Wh1+eWXV295y1uG+t6gP/7JT35SvfKVr5z4+RFHHFGdfPLJg75uyn531113Vbfeemsy9AzA2muvnQh+lv/93/+t/umf/ql63ete15P0tzXYXFhttdWql7/85el9kyF33HFH9ctf/jK9eumll65WWmml6nnPe97An9pss82q733ve9WPfvSjaoUVVljkPaVrsv6jAw44oPrhD39YffOb3xy4TZ1+aI3YSDVl3XXXTQS2HylZb/28b5hnp2LODNO+Tr+96KKLqre//e3V8ssvX73//e+v/uRP/mTUn4j3BQIzGoEZT6geffTRimH7wAc+UN1yyy1dCdXNN9+cCJWF/Od//ufTCvprX/va6k1vetNQpOKxxx5LBvHKK6+sdthhh679+au/+qvqmGOOSYRq8803n5Z+IyQMANl0002r7bfffqi+T3YneEso/gsvvDCRVZ7Pn/70p9WnPvWpiuHPAtO3vvWtFU/Bc57znIGb5ffmBIL9la98ZeD39PrhJptsUlkDSMQTTzyRHn3b295WnXPOOX0TC7/ddddd0/uQqhe84AWLfLp0TU4FodJH41Qnwr6LYCLJpVK63krfN+xzUzFnhm1j8/cve9nL0pyb7g3tqPsV7wsEShGY8YQqd+SQQw6pbrrppq6EqrTDU/HcVBKqqehPP99Ya621qje/+c0zmlB9+MMfrv7yL/+yuv7666vXvOY1qXu/+c1v0p/llltu5ISqH/wGfRahesUrXlF9/vOfr4TrLrvssmqPPfaoPvnJT1bvete7Bn1tz9/1syYny0PFgEsduOqqq4bq40wjVEN1Zhp+bJOC1P7DP/zDtG3spqHb8clAYBEEBiJU++yzT/VHf/RH1X/9139Vp512WnK52/GfeuqpabdPPGPH8pGPfGTig5SfsF32IAn5fe5zn6u+/e1vV9zFv/M7v1N97GMfq/bbb7/Fhqmb8r7//vurrbfeeuL5d7zjHdWRRx652O//7d/+rRIe++pXv5ryfbTtr//6r1M/SoXS0B+7fu9bZpllqj333LPiJSKvetWr0n/vvffe9G+///u/P/FqZJCxZuxg4/8/8sgjKfdpl112qU488cSUXwML/87rIwQhvOJdZPXVV6+uuOKK9L8vvvji6oMf/ODE+z/zmc9Ub3zjGxfrCg/DX/zFX6TvPfOZz0xt/Pu///vq937v90q7Xf3P//xP9fGPf7w699xzq4cffjj168/+7M+qAw88cLF3dCNUwj3CsbyMdt/wP+ywwxZ5Rxu+Pvav//qvaXwZUIZUzpy29QobNxspNAp7HphOgoT44/3CzXDPeXm+k3F/z3vek7DYcMMNU5uM+1ZbbVWdffbZE/gaE94vAhvjVhfjbc5eeuml1cEHH5zaxJN15plnprWShRdGaFcIzpzJ4awPfehD1e67717VCVX+jecQmRNOOGFifp500knVjjvumP6/XCjhTOtgyy23TH+nH3mO+f/yYYQQO0kvQvUv//Iv1XHHHZfmHezoCjjWQ35f/vKXk+f57rvvrlZZZZU0T3nGspTg20aojKP+nXfeeZUwILnxxhsT5vAUGi1Zb37X1l7v2XjjjVPoFWG3buiHU045ZWL+lMzxtjmjLf/4j/+Y1uDtt9+e8tzokOOPP35CV5S0xXukXNA9xkAuq7n3ne98p1g35AetFekJ2mUuhgQC44jAQITKgmfsEQSLmHF697vfnbwSf/qnf5pw9IwdM+OSZb311kuLjbIhFO2LXvSiRGooT+Et/8borrPOOouMRzflzSh84xvfSM8edNBByeXczN+hxChTSp2SZ8yFBHiS+iFUfnvGGWckErjqqqsmIim8woiSL33pS+m/MPBuSjuLcCTCyAvCaDFgK664YsXw8CAwLAifd1L4SB8Pg79bf/3102uQILkt5MEHH0y/pbwZ1E6hQcZXv1//+tenNkkI9u5DDz10EU9M28TXD4RXW5AHpAbhmz9//mI/7Uao5L4Jd8FbKPOSSy5JivzrX/969YY3vCG9pw1fz3jWt/3W/IODnI3mfOnVJ6RCaFio1Fhkwpp/c+edd1b+IAMLFixIfc8hJeP+6le/Oj1qrjFojz/+eCJ12oNwIY7IAdE/Yy6cCLdmDqB/F6o1F4wLg/y+972vMmeRYeK/5oB57f3mmTE56qijEgnz2yahQpp33nnn6rrrrpuYM82cxWwEebPmzZuXvvX9738/tRPx0fde4c5uaxLplpCMuNocGS8k1GYrEyqEWDgV2REm1ie5d+YID2cpvm2EypqHjXUKe22xNvPGonS9lbSXzrNxoF9gZ/3BCBneaaediud425y54YYb0jqwphEp3zFnEGUeSlLSFnODHvr0pz89gZF8SHqnXxEGtrmxQcj6qt93xPOBwGxHYGBCRWnaTecQCeNql8MI5QVdQqh222236oILLkjkymkxO0ZGI++qM8Al4YVuxpwhYjx/8IMfpJ3woPLHf/zHSXnpN3LUTfoN+Xmvk3rXXHPNxCtLQxB+x0vSiVDJvbrvvvuSQe51krIXHowQBWmMELc26Sfk95KXvCQZnKOPPjq9tgRfv2FEeB4GFUYVgUXekSneVbt9xKQubTlUjPm1116bSEImWTBChjO5bpu/mVAh6dkzy5uy9957JxLAa2nTor/5cIZ3IpBbbLFF8goTpOG2226r4INk84ohnQxtlhJClZ/lSeMtGoRQ2dgg7jZG2bPXDPkh5uZVfRwRMDrAXCYl+HqG5wipzmJMEd0s99xzT9pYSJLmNULmrYl60n7beitpLxLjQAr9kHPueOXMieypL5njbXPGuPMGIc1ZzB8YI8PyAkvaYnOIlMGnefCgdG1ZS3TQZz/72UT46Zth8g1LvxvPBQIzEYGBCVWTLNmRUY6IFin1UDVP+fH2CFWcf/75i+A1DKGSJG03LmdmGGE47Z7txChGbvVsSOvvbSNUvCoICuXDaHqvPtfDIW0KPn+vF6FiWBAFYaFBhTeHJwRJKLkKoRehyl4qSp9h4/nhlRF6ISX4IhA8GUJVxgD5rhvTfvqpHUJrvEc8QkhE9jZ6Twmh4n2qkyehYGHdJjnrNn8zoXrggQdSeJd4H2+DXb8NC8PHg8vwIUzmhhODPIa8YQShMj4MJELl1OsXv/jFSij4ne98Z3pmqgiVOcDo51C4b9cJlUMmwoi8I05WZkFOrVMbH4IsteHrGbjU57jNQ/Zy5XcjUYiNjRDMEaS69Fpvpe0t0Xklczy3q9ucQQQRNP+exYlVOpnXSTtK2kJ3bLTRRinCYJMiPGlM+rl2Rn94GJEoujx7OvtZh/FsIDBXEBgZoRLioLxzvkjJgu50bYLQGIWBcNRlGEK13XbbpVeN4oQVL4HdoBNiOVwnbFGXXoTK7hU2FD4jKBeGR0J+06gJFaNlXDrllJVOYKEsxqp+z1Sv33YjVIwrbwnDKk9Df72X1yATKu8twZd3wRgwkkgHUsSjM6jwwsjDM3cZ5yyDEKpubRiGUCF+TqwhjkI9vD+8EAhIzm/qlEPFSPJaCV9OJaFyhQaiw7OWpU6okFfhZxudZqjWxiKTxE6EqolvW8gvP49oOHnLsyMvDCEtJVSl7S3ReaVz3HOd5gwPHuxsLIR7s9ik8e7z1tF3pW2RMoF4/93f/V0imhtssEG1cOHChFOJ8FDZRPD2Ch3aGPSTn1nyjXgmEJgtCIyMUFGOdnL5tA3C4Li1RGbiWD1vjl1iPYeq7qGyG2U4eFWaJGAYQkVBUxhCkqO8WFBCvncLA3LtZ5EnxIPC89EUuS88NXJVsuy7776VHWadUPFcMZbaLSzaTXp5qBA7IU7EYFDJpAKJ8b426UaoGDAhyLrXQtt4mOqEqv7+bvjmZ+THIFISYZGOYUSImSeOdykfrDCXt91227SDl+vXlBKDn38zDKFyiMP9VUKjclxgiVghpVk6ESoeVPk25haRB3b66adPHAT41re+lfLr6jlU+X3DhPwYc4Qvb4p4neRP1r2wvHHSBBD2blKCbwmhQtKtR9+z7uRN6Xs92b5tvZW0t5TElM7xbnPGyVThUpuK+njRrQiN8N0gbUE0vRcxyl7N0jUFU+MLV16vkEBgHBEYmFAhJha0MANSJAzDAOWQiRNcwnZ2PXZVDJ9EcAu1TqgYCQTKTonhsDuyo7ZDYtyExQhvhpwMyp9I/OS+F96wQyJ2ZpK25cdw++d8KYmiQguMI48NZS+sYMffKWTXbSI4Eiwp2e+QP7tEBsHusH6iz+k3YRttlfzK2Nu1ay+PjByRrNCRBrkuSFjz0kO/ocjPOuusFGZk2PPtw/43zwoPjV0lL43kZrjlEJjx2X///VNCMNJmrOS4waF0B2p3ToHDExlCmPQHQbbjJyVjwCtlPHj2hIWNNbJm/DOhasNXW8wnY8mLKV8FnjxLjEGpOCgBx3wHFS+OQxVO1eX55V1OkCIvwpLmlHls3PO8ajP4JfO3JOSHhPNGWVPmmVCueVj37iBUDBosrSUkE8buBNJ+Io8oe3+tJblj1g7ylEM1TioKx1rLfufiU8QDoeR5KOmTbzpgwFvCuPNOOcQCyzzHtdOJXvMW0RYyQgaQRGubtOGbnzGfml7iNddcM8137UUuzFEbGXPG/KEreGbq0mu9lbS3hMS0zfESfPXVXJC3BDv5YvSr9UhXkJK2WDMw4tmyvrVN8vwglwTnAw455Fi6FuO5QGAuITAwobLzpXgpQMZSrg0vVRbKUfgOcbEzlu9BoVm8mVA55SPXw2IU/kMI5IvIBSCIULfyBRS054QZ667v/H05LIhOFt9B+oTpuKkZFoZErkepIGsMVRaGlfFwmqouCIf8Hs8ywpQWo++bvAWUoL7BTiIuQyMU2CRUTi8yptnDYJefjzQ7icN71RR5ENlQ+La8NoQtJzRrM0Xazy3avCJycyhcpEaYEsF1Uo6UjAGCyWOCACLj3mc+IJKZULXh67fan28E9225MAxLP8SYVwahzZKvvzCWzYRaxgvpc5KPIP/mejbmzRyf+niUzN8SQmUeMJD6aEz139xGpBEt7UeorElibK0bp0fNhywOPfBiwBGZ/Ju/+Zv07/DLhMrfI+tNYahz2K5tTfL2mJ+uXzD35X9pE4KV5zjvIp1gQwFDQk/wjiB6JfjmZ4R+myJ0a674tvnp9FkmoEh99pT79yy91ltJe0tITNscL5kzIgFyqGzmYG2jZhMnKTx74EvaYqOF1JpTBGFG1PIJ1cVA7fEXQaj6QSuenasIDEyokBkGiMLsFUZDJCzUbnfZAJaxohR6PTOqAUAI8v1F/SRf5u9LXtVe/ZYg3EsYCs8KfTZP2fEMICadaqU13+kddq7eM0ibvY83C5Es+V63Pvk91z7jOMipQb/nTdCP5s3WpfhS/rwMuc5cP8Sw3i+/9x7t4H1pw9WzxnwY/Aadw4iUkBVPcN34C/s56dVPySHG2BggToOMYT99sPYR1F6nvhAVz8GWh22y21TS/l7rbRTt7UeH9GovXZbvshtUd1oHvM3Wpk1oDneX4FR/Bi7WUv3ai37fEc8HArMdgaEIVf2OqdkORLQ/EJipCNiQCH/xJGXix+siTMnjGkVoZ+rIjVe7pDfk3K5e18qMFyrR23FCIAjVOI129HVWIiAM7oQkjyYvAm8ZL6uQTxSgnZVDOicbLfwshMwbLmxYvwNtTnY4OhUINBAYiFA51ca9m5NHA9VAIBCYXASEZpxQY6zcTeWgQLew6eS2JN4eCHRHQGpCLr016GWhgW8gMFsRGIhQzdbORrsDgUAgEAgEAoFAIBCYDASCUE0GqvHOQCAQCAQCgUAgEBgrBIJQjdVwR2cDgUAgEAgEAoFAYDIQCEI1GajGOwOBQCAQCAQCgUBgrBCY9YTK/ScuGO11D4t/z5cHGl33ZvW6b0U5F3fijJNIdnaKrKQA8jjhMh19zfO11/1NU9UupwnNDXd1dVoz7i/6whe+kC7jbLvLq95mF1K62NacaxOJzvnesbZnZ8u/z7Y+9WqvsTEPiDlSUt7LHVp+U/LsVI9piU2Zqja5N85acXlvP+trqto36HcGmTODfuucc85JF22P6ioPp6xdjq1MXvPKmllPqBzPdTO427y7ifuy3CScRekXt5V3Erc3u7l7kPILgw54t9+5DV6JGJcwDiomrtvqlVXpJi72W2211dKN7WrGTYYYH0S1KU6Ktl2Q2vyNCa3NbujvJC5NNeGRAbdl63tTGbkg1K3zbvR3SW2zSG8JBr5B+TbFCTwKcFAxn5USYcRGfdElQ+a2cpdLKsGi3EoncWO3ckVuGCc2LJ0KZB900EHVV7/61UqtR+WR6tJr/na6ybsbXm4X936niyfz5JiKBErtMPSUJXzqJykZN+vDJavDjstk90mZr1xRwNipUDHoJbjGpVd7lcfKlRisNRUQ2qReLLvt2an+9xKbMuo2uaAVbvQSfeROL9KpksKovz0d7xtkzgzSTptB61h1DBcjD0tKL7roorR5VN5NpZDmtTVjQagYEcBmI9yLUKlXd8wxxyRCpRTOdAmDRwH2amtJ2xRvZkB7KTnYKHyMoLjpeDLEJFTqh1HiLaTkTW6lL5Sh6Ue0Vd0ypTY6CdzsRlyIqRakXYTSQ7nAsb+zY2Gk3drOGOyyyy6pzmC94HBbm5BQirC5o1VySN28QWWyCJXyRi4IVXqG90vb1dZU7qXZbwVueRvOO++8ROiVukG466JWpLFTH1KNx7q0zd9+CNWuu+6aSjPl8RoU126/ox/MT+Of6zu6vV1xc4Y/Sy4Ubr0M6z2c7D4pRQQzHkAeDmLs7dZLvIJNrHq113yyrtW7VCMyCFX5DIXb4YcfnmpRIu/WofmltJfN/VwlVIPMmXJUF32SQ0GJsg9/+MMVeziMcABYR6rEdJKxIFS54zwkissOS1KGGZDS37YZpNL3lBCq0neN4jk1/XbcccfkGbJLGUTaCBWC5AJMoqYbjwIPpYswCS+EGnKKWyslg+gxGIp5q0HXr6jvx9j28pL2887JIFTmk10vLPQXoeW5RajUe1T3LwuC4d+RKTXvOonNCYJll6YIc1Pa5m8/hKof7AZ5lqJVX/T6669P93sRpN8fd35lGSWhGqSd/fwGoeLp+PznP588nYp+m9vNse7nnW3PHnLIIYlcB6FqQ+r//13tRMSdPqKjbGLoLJsddU7nKqHKCPQzZ8pRXfxJBIjnEbka1O7Qi0ivmrbdnC3TSqgoeLdAX3DBBRVXGq8Cj8V+++2XEBFuUFCXouNiIyaYMINwjXAYkPyWQqTc77zzzmrjjTdOxqCZB9WLUDEyioVmUdqD0m+KXbJvUhx2E/qgnhpvSKnYOdrNCR/Il7Er5hnyrtxnYQchE4Ofw0err756KjhLhGSQJTtCOxrM2eLMoU24eZfwlx1q/RJWXritt946vUcf7cYJLwMcmqKNyMi5556b6sAhpUhEPYxa2vdehCrPB7to7RBCYuyEv0h2g+eLA7Uji/GoG796e+xOeN+8t5vwanHfnnDCCaVdmXiuG6GCv+LWl156aSrgbdy148wzz1wkBIuc2I0KnRlLeSX333//IiE/c8G8904kEBFCZKwZ5GXbbbdNxZPr5aD22muvFGY1P81n7UQ2s6dOB7bccsv0+3qo13wQhtWeN7/5zR3xMB94CBHjusejdP6ad8bTnEZsefmMc93reOSRR07Md40QYm3mSlJyCgUbW3M9F7o2x0tFLUQGzPh0EiTEHyFk+GlzDh3AOOsN60PBcv1hKBTg5h1VhDrnWrT1qXTOIHdSE4RB6bk8BjwbSHCdUOU+eY7HLc9xOoJegF8Wu29hu0yS29pbx6uXcVQi6bjjjkt6E3bGG471gvC95rjvlOBbMuaK1+vXVVddNVHX1Xw2lqQfm1LyvW7PaAf8tWXBggUdH8uEitfbON14440JN8XK615w/2bu0Y3WyKabbprmodA1ga2NLE89G8vm8s7r91vf+taibmijeWde1/ORFBjn9fdvpK0tJXPGeuP4uPrqqyce11b9tznIouA6W3r33Xen9A5jZ3PcFJsK/84T3Q3rNhByAXBjYX11kmklVBaWnIj11lsvVYUHIGKEJMhp6cTOs0HmMmRAAYjdS6Y2kDwTjJd3MmR16UWohDQsenkHFFInLxblZRKLxwp1MHwG+NBDD+1qzJugM14GVsiJgYQBz4YBYugUH/ZO+Sp2lAyt+lgEaZPLQOwCkTC7GGTgkksuqU488cSEmaK53uFdFp58CosrC2xyXpbn7cTtkizwTrtLpABp1RbkxHMI3/z589vm4GL/3otQwQJx3mabbRJhsLsW97ZY/b3fEtjzUmlXFr/plHRIgdtNWITdyIGFkpWLedivdCNUef6ak+YIoq8cBxKAmBM4+nv/hS8SqWafcFrOoWLskQ8bDaTPLgs++p+No/7tvPPOyegYF783vyx+Y8ZQGjfKsC6UFEOHdFt7FK72+e8GG2yQyDNhZBnr+hwyjxC1upTOX4TqtttuS2vh+OOPTwqfQa4Xe3YzvDYzupRgp1CbttuU6e+qq66a5ry+bLXVVsXDeNJJJ6XNmFA/49bMf7NJ88dc0g5rIedW+SYiSxgBGPk342Qdmk/WrblA2vpUMmfMHToB+UQCrAtz4aijjkq6z3xrEiq4mh/XXXfdhA7p5CU0pn5rLpS0tw5yN0JlE4ZQIq7msLmOhPLGZEJVMsdL8C0ZdOOiDfQlG0L30y85j7Ifm1LyvW7P0G/woP+lD3SSPB+sQ3OJ/qdv6AYbjCxsH48xr6Q14Fl5c7wphG3jfbVp9l/zxOaL3tD/EocAJwadgDjttNNO6b3WJFti3msXaWtLyZzplMfm/fQSfkAQYmRQuBSBtA6OOOKIrgW62Q3PIF+DCM5BP9v0ZJvcfM+0E6rddtstsWXGVEKeSUBJMBSlhMrCsDBz4jUPgBCGqvF15VgS8uv1jER2xp1CGzQpFSljHCllRq6btIVMOv2OV4FSO/rooyf+uTTk100ZIlgmjzHqFv7pZ3K2ESq7ZqfGCGPKOFhEPClZ2kJ+njMXLAA7Nkayl+eJIWVALLS616u0X22EirHPXleeU54YRt8mAK52Tdz8OTm/GfJj7JFnxjh7Rk455ZS0mciJx9pKYfgtzxKCiWzwWhKbAO9veiARVpjz7ng/skZJMs7WUPas2Enb4RLKnEeEQfDbTtI2fxlz30T8edyIBHmbiLrHxN/nkGwnQmVjYk3xLg16iodxtcs1B+gL42FMc9g4968t5MfgI84wzd4O+WlINNJel259yjqv15xBQH0rJ4J7LzKwxRZbTGyckCKE1ZibI7yO9GS9vl4Jocpt7jUG+ZluOgSxN84MYZ6/zaT0kjneD7691i5M5Ex6XyexrkptSqmO6PQcEmKu29B2syd5PrBpOSJg02Td+V23OW+TQR94BuaZUHmHdxFrfc0110zRHkS6RMwzpM0GnuSNG1JW93zX39VsS/3fus2ZEkJlo8g21ccRaccjOESaIkeSDcMLSghk/j39gBfY6NrE4ADdciinnVDVlY8O2FlyTctnKSVUzVN+N9xwQ9plCYvVT4INS6goW4q2vlMvmYT1Zwy2xWzHKEzDgPj/zePDbQbJO7OXCvEQlkPSKHShgCzDEiq7d8aVx2wUVyq0Ear6fMguVruoefPmTfSphFAhFdpMiQv/MJKdwkAIDYKD6HRyFZeMbxuhqueLNT2sFMo3vvGNRGayNAkVQ4l81E+m8ujwwDKUORwujMLb5V3mljmWBSGlBOp/598oCcbNyckcIm8L+emPUMI111yTDgd0krb5OypjbiPF82jniFwxNNljVDJ29WesI8aGt5aXDomoe7pKCBViVidPSCDvCyNUlzZC1WvOMFA8EJmEw5p3goczJ93Sf9Yro4pQ2ZSYVzyKvJtkVGOQ+9XNOPKKmMP19dckVCVznOEsxbfX2PPW82TIJzRnkN+8hvyukzHvZlP6nWP15214ECqbq24HBUpsoHfSdWwmkmS8eb38Nnu5M6Gqkyc2w4bTxqH0+gpjADtrhVeftw+RYnuytLWljsGghEoyv9AmveYUXxYFuq0/ODQlj+Htt9+eiGSp0DE8YPQn+1S3Rc13zDhCZZFzITJ0JZOp0+S3G2JYgJrzbnR8WEJlAH2Px2NYwaDtNPxXyI6hRRSytBkkysluk2JCGuVzIXpCDaMkVEJC3tvpyPwgGEwVoaq3LbvWhR7qV1AgJBT1EclfAAAgAElEQVQqLJvHX/vp2zCEyo6RsjNnszQJlblMSdh9NcXOXi4hsZOSG0epUFSnn376xOOuQEC0cqgx/wNvJiXJm5dP+rURKgQBjnXXf7NdbfN3lMacp8ZO2Q4UljxrwgCDChIERzggLVkGIVTd2jAMoWLMEFokIIf3jYdxzzlmnXKobNx4rYQvySjHwPu6GUeeR954nrUsTUJVMsc7EapBx9g6MGd4Wsx9RDrnavZjUwb9vt/l63ykZDTvM8rvLbGB2s9Tw+tiU0gfIE7yCnsRqkHabl3z8PHUbL/99onIZ4+495W0pf7dQQmVDQ8SqA3NK29sDDud5hPuRtxFIoTqS4VetaHlweZxttnp5uGaUYQKU6coMGA7ETkLjm9TAPm+HAl1jEA9h6rpoZLRL5kZgaq7RIclVAiPnI+cfFc6IL2e0yZ9tCB4SbLAgnKUaCgs2hQTmceivuvTNrutOqGCo0R2i7aXdJvY2YhQQHXCN2jfR0Go5I3ZXfIklAhyjjgxRDmslskU0pF37CXv6vTMMISK8qbMkYJ6ojOynZWhMbUG8p1Q3dppXSAScgrl1FCoe+65Z3rc3wn/2bnm6w/s8uzUKCRkJEsboeJlpcyERbrlnLXN31Eb89x2/aRMhQEl7w4qQsQ8s8YgX2gqZ4PnD6HsFN7ox+APQ6gkjDtEYLzzeCJW9asvOhEq3ju7dPdtEZ49XgZzjTgcwbsnLJJzqDJ+w4T86ptk77M5EGJi+HIOVckc7wff0nHPhyHkGtIPpBOh6mZTbNJ4LegX6Sr9CLIud+rd7373Ipuf+jtKCJV1bsPElhhP4iAEHTBqQuXdiGdOcpeji6DkuVfSlnr/utkd649uzmFtOsfao5tyDpUDW/JF8+ntNuylktjA8uDV75dr+13+d7lp5qw8VDa7k0w7oZKcyCDpJAWG7TL+FJYJwgAiWHbiBsvzFmSdUHGBm3hclxaoxYlI5CQ5CtCu028k1TFgEpUx+ezqLXnGjmb//fdPbTCBudSRPgOdvQRtgyP+auFS9kKIDCViZAcnL6YuSKRJY9IKaWhj3snwSiFQDKFFTckiP5RsnVDleLtFT4HCmevSxLDYtIfwQpmo+QSFnBlk1E5AEqMYP/LGfa/9lG+3cE8nDHwH/hSXMITTG8bYmOW8JYSiJORnUSBm2soDqT2w0l6eNLsJXj+XmjI8voeA5bg/gmm3znVrHOvChVzq/uY6hqEQgt1PPoFpbLyjRBlKLJXvhQggzsYO2TOmWRlml7M1gMCYs0J0/uT7n+DBgwU/JIr3EuG0+BEm2Jtz8DaPJeMKEXHV283W3eZthApe2223XWqfsF836TV/SwiVcISwBCIjlO26C5sM8ybvECXd2m3CRL6IsaBg5XSU5sPxFFhX+Q4qXhxGzrjUTxQ5eYmMagv9Qsn7pnVISgx+W59K5gxDyRuFIOsjPQKD+k4dobLGzSfrwLqjIxAD7Sd+n1Mr9IWhpHttMDKhamtviQ7xTeEtJ7IcQuKdYpBhmQlVyRwvwbdN/9JnMDbf6QhriHfanM9J3ghVm03J37GGhO2sB/3rV3ybPUIsEFltMgZIvEMEJfPBGCIWCxcuTJEZa5rNpKMng1A5dW/9cn6whzDIUtKWkjkjNYF+ZtNsoEWF9IsOy4TK3OZggR8byq7xHiF39VPtuW05f4o+GURyCgpd3+kGAO+cdkLFUFOayAklYIddD9NhqgiM3TSjKIFYZn8mVEJmlFv2wCAenrdLzTt+eUo8PU1heHi3SMkzlI5FbWeemTNlaiGW3kKMMJoo+YZt/WfcLeCcnJvbKbeG8sw7Srs6xo8woHaccGC8kQY5VSZfnVDBjYJERuEME5NVgrEcs26uZgov533YBXs/40UhifcjYAx8qXTDV45WJr6lhAqBsiNkJIyJuUMhGoPs1cztsmODofZmoiRJFt6dxK4xG8i2vjHASG5Tcp5CiTL0W95Uf4wVFzZjnglLTlY1TylJZJFQGpQZZaLNjL98nzzPzS9Jmwi0eQEbXkZYZIWE+Flv9YR/7y4hVL5j/ZhDzfygkvlbQqi64WuDkS/cdOrJPMhi7BhwxqhUJOzXTz/mqxe8p5l8Ci+bFomtxNw1h0mJwW/rU8mcoRfgx5tk/htjBNJmEdHSfrqUN4oYe+tcKDR7LP094+PwgtQIO3aG0VzhacmEqq29JTqEt9L6t+GwVm0KtAkBqV+b0GuOl+LbNub0pTlSP8xhncglzLl3JTYlf8eG2IaTp6ktCtCpbda8tW/zkytJIMlCavR7yXyw1s33TOicRKcbkLTJIFT6QXewCzbK9XudStpSMmfgok/mDLtgs6lfCFvWX75lztqM5tJy5rGwnJzZutBpxkgOYQ7tts2V5r/PCkLF0DMeyES3kw4WpIXQq+yEicMLZaEOe718CdC+JbbaJEElv0VsGEYTws66zSNCeesfclDvm+/zZPj7NhemCWrB2tm3fa9bH3yP2xPGg55yLMGn9BmLCDb6X2+PXafJr5/wnYr5UNrmbs/JTTAvetWQZDyRSf/1XNuYd/sWfMyHYUoamQvc3pR/vs6i2/e6zd9hMcu/h51vMNb9ljHK7+DFgQtMecDa5oxnfW+Q9T9svxl+m0veyCxIlrBf/dqJ0u8wNvRCr3qope/q9ZzvIKi9bpkf1Rzv1Q7fMH65PmS3zXCJTbEO2CXef7mag4o2sQm87M0STqXvRBLZynqCfelvR/3cqNrC7sOmFybsqLllPdKLnWwTgmWjy8s9qO3yHfrBietuV/BMu4eqecpv1AMb7wsEAoHJQUCSO1Il5KF8RsjUIID8SBHgVcnEz86bZ1NSfjev89S0bny+Ik+J/eLNd6hk0Bu4xwex6ekpp40oDU92t/ujSlsmeuYd0iY6XVkRhKoUyXguEAgEFkPAbk0Yifs/J8QGTJOLgHC1HDmhdzlx2SMrd2yY06qT2+q593ahy3yRdGkO7dxDYWb3SCRJfqH830Euom72TkidvvNeuXb1e908O62ESrxeYuKgLs6ZPZTRukBgPBAQBg4yNbVjLVzFQ0ixO2Agj2fQEPDUtjy+FghMLQJSXXqlUvTbGqHgXP6smYY0rYSq347E84FAIBAIBAKBQCAQCMxEBIJQzcRRiTYFAoFAIBAIBAKBwKxCIAjVrBquaGwgEAgEAoFAIBAIzEQEglDNxFGJNgUCgUAgEAgEAoHArEJgzhEq94K4k8Kpi+k+eaEtrrt3B0bbvTazatb0aKwEZYcMRtFf94+4+dx9Urn8x1zBaTL7McokzHEYA/fLuANssu9hmswxH+Td7ixyQWy3wryDvLPtN6PUD74lOd+9QqMo3N7W9kH+XUUI1QycBgyZ+wjMKULlPhy3mOdCvo4RN481TuWQupHbzehuqM4nGV3epvZRU1wu5/bZfsTljEqruOSv02VllI2TlAimSxzdtN7JaLhHxYkhFxq6ab3T/Rol7VLR3s3Lbql3y/Sw4vZntzYrb9JPmZthvzvq3+eSO4yXquWTeRmkm4PdlK9kQ6diyv32ba6MQa9+d6rd1i9OnkdQXBzo4s3p3sxpT5t+6HRb/SD9Lv3NqPWD77ql3y3s9eoQpe2ZiufcF+ZmcHcglVbTmIp2xTcmB4E5Q6jsBNxe6qI1kxiZoFBK63mNGl4XfynXohRKrrfmGy7jywVlET+7Q2RIPUC1ivqRXLjY0elOtw8r0ePeDDfnulEdmVLPjlEnMNIWbXUpnXI6aiUpMTPILdo333xzIlRK0qgbNqzMBWMOb/UCKdNM9P2dWpKTIe5bUZ7J3BjFN+bCGLThPCpC5VJN1xcwnkq/TLe06YepJlSj1g+zgVBZ8+aC2nLGI2RuIzBnCJX6WorKqks13aIopfu1ehGLXBdomJvi2xSmMiVKcQi/Wdhbb711KmHz7W9/O0GEiLlBlkHJtZl4sez43Lw83TLbjbn2KxCMXPIcKZWjBiHDoj7joCWApnJcZtIYqJlpziqxMkoJQnX2KOGc0ndNtodKKFi6Qa8UhrZnRAnUWVRoe6eddppSfOJjU4vAnCFUwmuKJvIIdRJFZBWcZCBUyVamIVdd9/y5556bCsuqVq+opGKpSjgoL9BvKQehRt9SfLRbfkIvQqXIKuPB06b4Ks+RIrDa4oIy/+6Pd2SvUl7we+21VyoO3UkQPMUl77///q6zjFeLN6sbjp1+6H3IWhYFeFUHr4tCnyqhq/Gk8Col5MbZU045ZUJZyTmDmyKp2sCrIxxZD/kxqEKKlBNRZVx4MXsi4aKA7XnnnZeKaRJlIbRJWEAh6qkS31OoEz45B4z3TzFiZLiuXIWC805W1fh+c9AuvvjiRcYdIW5WRFc2wS3bSjF4XhhaYdZ6sdBRjAGPnJvTeY2z51QBU95Z46ZIab+ivd7L63v88cf39XMbC/NRlXlrRr+tUWuFIFS8w+alNaLgrblqDvVzIWCJh0pOmmLK5qmNlzIWvrvOOuukttjsmKs2hwcffHAi3zY79JcC2FnMH/PrRz/6UWpj1jOKgLs5vUQ/mB/mBK80vSeXzFywBpsifKx/119/fQrD9yMl+qFN5+XvqVWofcZIWJX+UzQ8h/za8O1HP9BR0h/MDbq4k5Q843eK+2prLljdD37x7OxBYFYTKkaI8iG33XZbMtYqx2dhaCmjE044IdUbo0QZWYp1wYIFScHnqtQWGmXktmHlG+yChcPUzVLxux/h5fGnl5enF6Gi4LVHW3LFe8aEsqP4KBN/kB79sODzLcmu2c9V0+ttfuSRR1Koj2L42Mc+1rU78JNLlQlLSb+FDhVnJYitbzSVMuUtl4DrW5sZAsS1TiwYPcbeeFH0jIsxqBMq7dNXRMx3GVghU6SJMAqIiXwWO0ME4bWvfW16H2VcKvLehIybQomX5rpRojyCGcsLLrggzS2kEOlFZrIo3GkXq2+SypdZZpnSpqbnbBQYPGtg9913r6688spENuuCpAn/CsvyTCrmykOqYnwuqDqKMXCLsHW26667JiLAa+z/I0KHHXZYX/3ysLVrzlgTBxxwQN+/t5bNA/mVbhWHlf5mEmO9IXwSm30rlxOxhhGbUikhVLyViBrdw3tsnfiGjYN1R9cI1WqDDR9iJw/0ySefTLmYxH8RMb9FChWnpgePOuqoRMJ4o0v0gzVJb66yyippbLTBmmwWWIYd3MxbKQRbbLFFKSTpuRL90KbzvOfLX/5ytfPOO6eQtjbQ//vvv3/qcyZUbfj2ox9KyFLJM9pujOnFe+65J+nAkLmJwKwmVBa4U3SEd4mitFvJQqEzgBQVRVlX5jwmDO4dd9yRHqesKbB6CI53gTJDfkqFAbVbZKx8o5u0ESq7WLvTnH/FADJM8jOytIX8PIfgUNKMnIRzZLJbiQpJo1tttVWqpi2naxDRXrlsnQiVnbM+Za+FnTHjb2x483gujCMvGWmGmyT4Iwk8DpkA8OTZuRvLvIOntBhw5IXCk7PECPWTFCqHrNO4M3A8SCWC/MmpQf4QQ543hJinUVs/9alPLfIaHh2eNocMBhVkzDu6ESpzMq8ZODLcxp3XbJRjkA0I48zYwbMfQqv/QqT77bdfMuL13L9+sUFceCat8U7CmCNb3/zmNyfmECL4rne9q3r88ceLyW0boXI4QamKnOepLQy8v0ME6K5MqOQ16jtBwGymbBKQPuRHX3g8siCHSIb5VaofECqkxGYkH5QQohbytybr4tCK+bH99tv3C/8iz3fTD8agTefZFNkMIL9Z6iG/Enz9rlQ/lJClkmd8E850VpOsDgVm/HjGITCrCVUdzW4hP2FAxtVOrJkc7jeUFA8HBfWBD3xgESOKsPHsWLSlQukIKbadTGsjVIyvsEkWoT+GT39KFabneHkYTicLEUThDm77JqlC1oQVEZxuhqcEg16ECo5nn/3/+Ro8ALxJQh5COghwPlbtW01CxaODqDF0Wezc/Z73R9uzMMD+P5c9I7XhhhuWNH+kz/gmEmdO8HZdeOGF1dprr53+IJ08pqOWNkJV3zDkOciDNm/evJGPgQ2KECPC0C+hRTTMR8bPHB5mV49kICuIqhw2Xp1MyOHfKYdKaMbc5FUt9Ui2ESpeXN4yRJ/HJwuCzfNifWZCZc44KEJ4oHbccccKYfA7mCLosEEWH3vssZSzKU9POLxUP3RKSq+vyVHPTe/rRah66bw8r61rWGWpE6oSfNv0g+/YeGaR62gzkD3GyCwPYtszdFldhCLpXHOxrr8mA+N45/QhMOcJld0XrwJPVD0XyuK16+Med/qtE6EaZFgQMAuwmSPTfNdUEar6d3mphATlUdmFZhEeoCT8EXJzvH9QGZRQyWFj6Hglcj5Ik1AJaciPque+5XYKryEqWa677rp01QIPJS8Wg9OPIDuMelOQ5V6ex/rzcqSuuOKKNM8Yy6yUFRI+7rjjUrhy1DIMoRr1GAihIyu8cQhKv/cdwU6eHfycXux0NUgpfggdrw+DjJjwQOXcsU6EKusNGw2ekRJpI1Q8mzws8jfrhMq7hfDMlxJCZYNkHiKFCJq20jk8efVrUdo82LOJUPHGIZg87DDsRKhK8G3TDzZovLtEqNOmDBnPBArhRGLbnhFGrYsNlfl/1llnTXgeS+ZUPDO7EJjzhCobGGGOuiGUK0AB5VOBoyJUdiJObwkh5OsROk2JURAqykVoDokT1mwT91FRBnVCJZcJkUI+hCmHIVO+PyihyoakbsB4sw488MAJb5//b3cndNjrLifKlzdi/vz56X4tXi+HDPq5uDGHWJqYIuXNcEg33OWCIRXc/fleIiFbnhdGsL7LNW94Huy4h7lzaxhCNcoxYNzMTX2SL8bDwpj0K8bPb4Wi5Ar2E7bt9K2ceG0cEBPSiVDJXeQRhWf9XjbrBU7IfzPPrY1Q8Tjrh1yg7bbbriMUJYRKeND9c042e6fNAmLVXLtt+qEfQiXc7qCIMGS/BybqHR3UQyW0xsNDT4ssEFEHeYc5Kb0EX78r1Q8l4bySZ3xTniKSNUgOWr9rJp6fPgTmPKECLfer3B25CBK2udDlJDHQdsBkVITKuyhLuVTCfk1h3LnzhRp5ziTbeh5B4D3qpuA7hfycnqFMeWyEKxllHp68O6L85dPYaVE2DJvE09tvvz3lMQkDyvNhpBj/ukKWyJ1DDm3TUyI00pD7jjBoD49Cbkub8rYz9D27T8bMDhBx0cccPpXPIs+KEvUM48TDqJ/+jsDdt7RJXhXyxXMFY17JqRREVxudgJT3ASMkD6l12q8uPKjmphwZCr+f02Xe41tC1LwvsOCBQZQRuRzeYgh7hfxGNQbGDKGVC4dEIRBy37QJQe5X9E1I0qk480LuXanoE5JiDpjn5oOQm80Fz2Veb4if5+SUyaVCmMxh664uxtNachrQmqlLJlRCeNZPlvralqPIU0z35DC0Ncnj5LkSQiUBnWGWE2p9I3Z0R06yz99t0w9ta7LeNxs22PEaIrj9SIl+6ERqmzrPd4U5tYEOtUGm78ypnJTehm8/+qGELJU8AyseavMJhjP1Vvd+xjSe7YzAWBAqJIab3Q4XuaC4nHSrx7JHSagkViNq8i+a+VdO/jmV0hTJnoheVvAlOVSeFQKzU0U2CCUkuZPY0TEo+X9TSHKQKG8i70hOSSdhFPIJyLbF061PjJOwHSlR3nbBXOyIgTALHJEDuGSvDZLkpJcbqbN4NnsaeQW1R95NNjByl7xXrlUvr2FbPwf5d/lqDCDyitDIneItbV44a146hcmTpm/18GXJd4Vw4dUU8zATyTZC5bfDjkE+RWXN6Uc+gGCtCbnJc8nkt6Rf+Rnz2IkuYRMn5EoFwUTqGfUsiIx35NOwSBpjlw+oyDO0+ZCP1PTGIIYIYj1pPL83E6peaxuRs/6RAmNOfA+J4/0sIVQwtJ603yZK36wzuCJadc9ZL/1QsiZzX3jFEEgbg35D1SX6oYRQIVPWjxQFmDnUQU8Jw2VC1YZvP/rB+MhLoyM7nZyGTckzxsjYWNP9zN3SOR7PzRwE5gyhKoGU4kE8eAiGycdo+5ZFttFGGyWjmUlS22+G/fe886mHwvTXrt4dSHIsBi0pM2zb+vm9HSdjLJG5V2hBKIZ3imKdCWU+2voovCTk2Mydqf+Op1EOG09Wv/lGbd/v599n8hjIRem3riODZn04vm99dAsb8l4g8/6929zzffOzJMTeC3Mhf+9BOOXU9SOMOw8gb2OWnJDd7RRZJ/3Qzzflnjn23wxV9/OOUTxLt+ZTvr302TD4jqKd9XdIpUCiRQb6zeUcdVvifZOLwFgRqsmFctG3y/1AqoStDj/88Kn8dHxrliHASMlrEwYSFssXTs6ybkRzpwgBd+MhN7wdmfjxAvHC85L1exFxr2YLG/IC8Qblk4hT1M058RnjIa2CHejXszcnABizTgShmsQBd5+TkIuci353oZPYrHj1DEPA6Tr5PBJ+61d7zLBmRnNmCALyvdyPx4sprM77xCPjZKr8sFGKeYnkuym9WyL9KL83194lVUHenRyqkLmPQBCqSR5jbv0gU5MMcrw+EBgzBIQvecGFKIWRXSDb7cLeMYNmRnVXykWvEPKMamw0ZmgEglANDWG8IBAIBAKBQCAQCATGHYEgVOM+A6L/gUAgEAgEAoFAIDA0AkGohoYwXhAIBAKBQCAQCAQC445AEKpxnwHR/0AgEAgEAoFAIBAYGoEgVB0gdMdRyU3VTtZICnUnTb934+TP+pYyGG6V7kfctO5m87b7itx94ibmLK5yqBeGzX+vH941qlt8XcTozhhleEYp8HIJYhaXNtZvpR7lt2bKu9yj5M6tLMZoVOPUqY/uwnKZ4Wy4t2ymjFGzHaU6pKT9buN2N1i+JLXkN/FMIBAITD0CQagamLsZ1627Cos6KtxJ3IC+7777ptu4iQsb6wavdBid1HE7totG+70AtNMtx52+6+Z0x57XXHPN9M9KtrgXpS5Ooijp4tI5NcJGIW4zh1Od/IzivY5x5xvulZzZZZdd0v1Nc1lyjbLcR7XL3Hg/GYJYu9HZHHFx5DB120bRPmvELfgKLM8WKdEh/fQlF3J38ehkXkjcT5vi2UAgEFgcgSBUDUxUtT/mmGMSoVILrZPw8vBIqQ3oBnJlHwa5AVctqu9973up3Ei/t333Q6iUa1HSopsworw8a6yxRuXurFHIZBGqetuUAYH/XCdUPH1IL0HAlSmaLELlGwiM8iyKAyu/Mp3i+0i5OTxbpESH9NOXIFT9oBXPBgLTh0AQqj6xF8rikUKm1IcbVBTHdekbY6FIcr8ySkLV77dLng9CVYJS/8+4+FM9s8kkVFrFk6m+GnJVWiS7/960/2I2Eqr2XvX3RBCq/vCKpwOB6UIgCNVvkb/44otTQdQsSjkgLU1RfFOIThFZhm1QceuwHBW1tzrJzTffnAq2CsHJoeCJ4T1SCJjk4qiKL9sRu0BUMeNTTz11kXwuIb9eHirv0SfCWMOhLopGC7MpdeFdDz30ULXZZpulchT1PDMlFo477rjqpptuSmEiRXJXX331RUJ+isrqk0LBcp8Y7F133TV9TvhUGPWwww6r3vnOd6a/U9h2m222SZ7CE088cTGYZrqHaqeddkr5dWedddZE23mb4Gf8jzrqqPT3ylIobHzvvfcmss4LZRxzEet6x7sRKt7Ol73sZeldWYyZUJnCtkQulsLZxpTHa/31108Fo3MR6fp3hJeMkTm1YMGCvqa5nD2FuL/2ta+lIuSKcLvc1k3e9VzBXvNBf8wJdQ3lC+pbFvN96623Tv/XetAn3yNChOrcKQq85ZZbpr/jUc7kEK7mJsKfsSqd420glOgQ7XXTuTw1z6tFyRO49957T7ze2Agb0jG8x/IQlYCph/xgfNBBByWM1Ce0ZowznfLYY49V2267bSroW/feKmkkt4vOqedeWlvGBnHzu5BAIBAYDIEgVL/FTdgOKVBQmOdJBXoGOwvDw0DwUPmv6uGKHxNG60Mf+lDxCCAKEsMp1T322GOx31GIjJnEYEoTQZFHs8kmm0wUZUWEeA+QmuOPP7565JFHUi0vnot6zag2QiUUKOFVrS5FR5uhFYaIwXELM6+FxFjEhwFgnMjDDz+c6ofJOVM+BWlAToVFcw7VVVddlYwpQ4cwyBmTC4QkZmLKG4F83HLLLalcA6Oq6KuwaKeCtoMSKljCuJOoXE98UyHcpgjNCo2WCEN/9NFHJ1wz+bz22muTZ/I73/lOlb/FsAsZI8eI8Xve855qpZVWqngxm9KNUHXyWHq/OWPuEgaXZ1UNOJsCc+XSSy9NhxY6Ffs1j4wTAtyPWEdu7tZW/1UYlmG/7LLLUnhcLbq2+aC+ISzMhzvuuGNirmmHftlgEGtDrleuf+igiDHyrXnz5qVnYK+/SARyaN0g7tddd13amJTM8ZL+t+mQ3F5r30YBCVI4V/vvu+++pBOsHR5r/0WqEC7j9bnPfW6CUFnriJm1ptSM/njXO97xjuqEE05ITdV/tfeUNZo/f376PV3iAIxwbl2EkK1DqQ7IXUggEAgMhkAQqgZudnCIUpNQMYQUlx0jz4LE6FyE1O4575BLhgEh22qrrZIS7eSFkGxNYV500UXJM9RJGAJEhidL6QlCcTJ+vERZ2ghVfu6QQw5Jv+tEqBAD38l15ihvWFDOhKJmABGhnMTcDPlR4jwiyFkWBIzXBNaE50AytF32kUceWe22224dDUD+/aCEilfM7r+TPP7444kIMtiMc1MYu4ULF5YMc/KuCJchooceemj6DZKoZAisugkPBjKD6DaTwgclVEj8CiuskIx3nlPw9neIFnLblAsvvDBtLmCCBJVKJlQHHnhgOhBBzBcHIzKRLJkPftcW8islVNbZ9ddfP5GriLyoh2d9IFRtc7y0757rpkP8m/aaA1/4whfSK5Ft9fjoBB413jxewXvuuSeRQJ5PyhAAABJfSURBVNIM+b3//e9PBMg8yvPjlFNOSf2wIcxiDvmtuY7A8SAjkk3hoUYubXiifE0/Ix3PBgKLIhCEqjEjeilDj44i5MeoUaoMWqej6UiG02tc81zwvFj+f/0Kgk4eCbtvhAXRyjIKQkUp8+rU38kAZFLAyG+xxRYp9JilTqh4eoSyGAwnyLIgqUgLY5sFIRSyEboRiuhkAPKzgxKqqVQCyItQHqKqr4iafvEmZnFC9Pzzz084MIiw5jnsdKprUELF0/eGN7wheTQyAfd9oVuejE7FW2+44Ybk4XL1Rj4lWoJdJlR1L5ywNRyQB1c+lM6HUREqoWZrIYvNDE8WDxAi0jbHS/qdn2kjVL08ajY2xgpZytIkVNYasl73oPMy8jbST/laFPrFBsC76JFu6QX99C2eDQQCge4IBKGaBkIlXwZJEnaS/9BNeG647P1XyEv4Jd+51IlQCf2ddtppEzlR3jsVhEp4DnHw/U6ESpgUGRRaaObrCMfUT5IxCOuuu27qgxwTRLKbDEqo4F+/m6v+fiE6u345JQxSU3g6kOFSEdpizJAMpEr4i1cikxqeI547HiCeM0YeEREOHiWh4lXjpUDk6oRKP3xfvldTeC0YbyR31VVXLe1y6qtQX51Q1X/cz3yYLEJVb89MIlS8evDj8e1GqJAk5LzTtS68V/nEsLChXDMbF0Tt9NNPLx7DeDAQCAT6RyAI1TQQKsqSUuTh6ZQQ3BxGO16JtQyfHBjSiVAJO/IG1a8+mApCpS08L8IVhLKX54Is5RwqoS8hSUSlm2g7LxbvFOOPHEq6zYn4zd8NSqjkmXQ7hi/M5cJUOWKSoZsizFtP/G5bcoya/CgE2gEDYRxkKYv/LSfNGEvcJrxG8mH6IVTy0PweAScSmyUlC9nJocp3WUkElxBfIsJSDDyvWT+hoDZC5dsl88FzPEtXXHFFyqPqJNqFKGgncQWJsHEzh6rpoaq/ayYRKm0RJnX4I4fz5IcZ1zwf/P8777xz4h68bmNpcyBn0Toyn8y1Pffcc7HHYSssCMN+QrslcyieCQTGCYEgVL8dbYme8qN4DCScU2pOl9nt1W8WH0XIzw6dYZW3Quk1RTiCF8MpuWWWWSYpT+SBF0iuBEFiGBPJpsInlKXwGLKm/Vl6ESoK2reIpHoEjyEi8sKEI0uMjRwh+RmMtZwc4T4eIEQiEyqXpTrxBVd9kdz+wAMPJPKST3DJaZEcLyHcRaPCTbx4SFWne7oGJVRTvcBhK7naHOMtcMovi1AfosmDhGTLmzMvEKJsQBEaIR6CDPk9guCSR4cXiLH3LqFCIWOEkKGU75aT0uXtCbtJEM+JycaJ162TpzTnT/Gy9SMlhKpkPvgmIsEjKEyGNMLC3MlJ/ryZvLfIvPkr/whWvJv1pPSpIFQlOqQt58uJWodckCA5hHASyhPGz/PBmnKwgzfKAQb66Wc/+1n6k/McebN5sOCGROW1hXA2N3G+Y94dfvjhKd8vJBAIBAZDIAjVb3HjQRAKagplJIchyygIlXcJ/chDYvSaeVR2i4xBDjkxnEJfkpWz4aNo7T7zlQfCbvKn8nHyEkLlJvOcWN/sN0Pr1FkJoeLJgR9PgvwYSp6RQ7AyodIXp5YYCsnWBCGUu8MI5lNJCBVsyF133ZVCnPp0ySWXLDY2s4VQIY6IDzwdOKgLXOQwwYogCPvvv38K0WQD6gqPjEn9t/KRnCwjvsGrJw8LrrDmhXNaMBMqc8V7jBPPGZFDZIya88CzSK1v14/0l6iZEkLVNh/yd3gtkUIbBnlYCIlcoHwI5JprrkleOBshXlLePWvWybipJlQlOqSNUOm3k3b+6LswuTAtIl33WNJJiHcm2jYm5o0Ni4R2pAyBzjoN3kg0Qmpe1E/NXn311YmsSkpH3kICgUBgMASCUA2G29C/4oFixBxVrifL5hczHpQlRWgH2q0mnvAYRdstF6s05Dd0h36bsM970KvmmP4w1ogXL8MwpTRmC6EqwZahQ0w71Vks+X2d8Avb8Fp2EyFZVxIYpxxmbD6L5MqBuvXWW4cao7a2l84H5EJYVN+aa8G/8eAgVMPMp7a2TuW/88zSAb1qivJE0iP+67l+wrL1vpgL+foVnryQQCAQGAyBIFSD4TaSXwm9OE4vDDTIbekljZhKQlXSnlE+M5cI1ShxGfZdvEG8Qq7FkLcXMjcRQNjkn7kLy1UhTnWOupj53EQuehUIdEYgCNU0zww5VC5wrJ/qGWWTECpKM19CKtE43yA9yu9M1bvglG9Xt7MW3pnrtfymClvfkUfoRJ8rMOR2hcxdBITeXaMhid8loZ2ucJm7vY+eBQKjRyAI1egx7fuNEpDlskyGyLHJeVbeL9eq11UNk9GGUb6Twa/nISGK9bIko/zWuL5LaK1XqGlccYl+BwKBQCDQC4EgVDE/AoFAIBAIBAKBQCAQGBKBIFRDAhg/DwQCgUAgEAgEAoFAIAhVzIFAIBAIBAKBQCAQCASGRCAI1ZAAxs8DgUAgEAgEAoFAIBAIQhVzIBAIBAKBQCAQCAQCgSERCEJVA9AFgeqtve51r5szFwTqnpNxyre4hXuyThMOOQ/j54FAIBAIBAKBwKxGIAhVbfguv/zyVH4BAel12/dsGXE1A5V2UUNPnS9lOt70pjfNluZHOwOBQCAQCAQCgVmDQBCqOUqolBdRKFU5EzeKK6IchGrWrMtoaCAQCAQCgcAsQ2DWEypFfBVKVWiYJ0bVefXJPv7xjydvUxbFZ9Wpuvvuu1OhWkV/843bigr7g4Q89NBD1eqrr56KsJK99tqr+uAHP5j+96te9arqpJNOmijMqo6e8KAixfn28Y022ii9W5HaU089NdXiO+CAA6qPfOQj6R2+o6L82972tlTDz/c222yzVNR11JcpuhVd4V1hzM0337wroVIzcJNNNkkXZC5cuHCi77NsLkdzA4FAIBAIBAKBaUNg1hOqXNmeN+Y1r3lNdfDBB6dSJJdddlnllnDFVK+66qpErhCfTTfdtPrSl76UKrV/5StfSVXW77zzzvTnpptuqhYsWFBddNFFE4VGleF49atfnQaopFI8UvSSl7ykWnLJJdO7EKvDDjusuu6666o3vvGNiWwhVQqZfuITn0ihxX333bfae++9EwGbDPn617/ek1ApgrvBBhukNrkle5lllpmMZsQ7A4FAIBAIBAKBOYvAnCFUBx54YHXmmWemgUJi1lxzzQpRWG+99aoNN9wwFXlFZLK85S1vSVXar7zyyom/a8uhKiVUK6+8cnX99ddXyy67bHr38ssvX73vfe9LHimEipfr5ptvTiE5ohCtNitGOxnSRqh88+qrr071/njcQgKBQCAQCAQCgUCgPwTmDKHK5En3VVG/9957qxVXXLFaYoklqqWXXjqF5NZee+0JdK699toU4kNksoyKUAktIk9Z7rvvvkSunLBDqL74xS9WP/nJTyb+3bNClrfccktfo7fGGmtUjz/++MRvhA4vvPDCxd5RQqj6+nA8HAgEAoFAIBAIBAKLIDAnCVW9h066LbXUUtX2229frbPOOot0Xnjuve9976QTqvpHR0mozjvvvJSjlWWFFVaott566yBUscgDgUAgEAgEAoEpRmDOEyp4vvSlL63mz5+fEtV7iVyrbbfdtnrkkUeqF73oRYs9Ksfo9NNPr4QXybe+9a3q9a9/fcrXmjdvXvo7JK3poZosQlU6V9o8VEKfZ5xxRvWKV7wirlUoBTWeCwQCgUAgEAgEagiMBaH66Ec/mk4AyrFyhYBE8AceeKB65jOfmU62Zbn//vurl7/85dWhhx6aSBGi8Zvf/CadCiQuxnze856XThIKz7397W+vnJC7+OKLZyShuuGGG9K1CbfeemsKQSKU+qDPr3zlKyf6fccdd6TEe+FRpw5HfdowVlwgEAgEAoFAIDDXERgLQiUsduyxx1annXZaIkiEt8mll0hRXZwEPProoydykxARSeTEPU6777579eijj1YvfvGL01UHe+65Z/XZz352RhIqRFD+VlOccDz55JMn/vqpp55Kpx953JCveq7ZXF8A0b9AIBAIBAKBQGAUCMx6QtUPCIjVT3/60+SJ4YV59rOf3fXnbhb33HOf+9xFnlGe5uGHH06Eqtfv+2nXTHiWV+6cc85JHjdXPoQEAoFAIBAIBAKBQDkCY0WoymEZnydvvPHG6vzzz093dwmJusg0JBAIBAKBQCAQCAT6QyAIVX94zbmnzz333HRz+3777TdxL9ac62R0KBAIBAKBQCAQmGQEglBNMsDx+kAgEAgEAoFAIBCY+wgEoZr7Yxw9DAQCgUAgEAgEAoFJRiAI1SQDHK8PBAKBQCAQCAQCgbmPQBCquT/G0cNAIBAIBAKBQCAQmGQEglBNMsDx+kAgEAgEAoFAIBCY+whMK6FSINjt5GuttVbHUi9zH/7oYSAQCAQCgUAgEAjMBQSmlVAp7usG8wcffDDdgbTPPvvMBUyjD4FAIBAIBAKBQCAwZghMK6HKWB911FGJUP385z8fM/iju4FAIBAIBAKBQCAwFxCYEYTKbd0bb7xx9Ytf/KJ6/vOfPxdwjT4EAoFAIBAIBAKBwBghMCMI1Xe/+91q/fXXn6iRN0b4R1cDgUAgEAgEAoFAYA4gMCMI1W233Vatvfba1X333VetvPLKcwDW6EIgEAgEAoFAIBAIjBMCM4JQPfHEE4lI7bnnntWRRx5ZLb/88tWznvWscRqH6GsgEAgEAoFAIBAIzGIEZgShgt/ChQur3XbbrXr88cerK6+8stphhx1mMazR9EAgEAgEAoFAIBAYJwRmBKF68sknq1VXXbXaf//9q0MOOaR64QtfGB6qcZqF0ddAIBAIBAKBQGCWIzAjCNUPfvCDas0110yXfK600kqzHNJofiAQCAQCgUAgEAiMGwIzglDdcsst1brrrhun/MZt9kV/A4FAIBAIBAKBOYJAEKo5MpDRjUAgEAgEAoFAIBCYPgRmBKG66aabqo022qh69NFHq+WWW2760IgvBwKBQCAQCAQCgUAgMAAC006onnrqqerQQw+tLr/88uqhhx4aoAvxk0AgEAgEAoFAIBAIBKYXgWklVGeccUZ1xBFHVEsuuWR1zjnnVNtvv/30ohFfDwQCgUAgEAgEAoFAYAAEppVQPfLII+neqVVWWaVaYoklBmh+/CQQCAQCgUAgEAgEAoHpR2BaCdX0dz9aEAgEAoFAIBAIBAKBwPAIBKEaHsN4QyAQCAQCgUAgEAiMOQJBqMZ8AkT3A4FAIBAIBAKBQGB4BIJQDY9hvCEQCAQCgUAgEAgExhyBgQjVY489Vt14443V6quvXq222mpjDmF0PxAIBAKBQCAQCATGHYGBCNUPf/jDarfddqvU4DvggAOqT3/60+OOY/Q/EAgEAoFAIBAIBMYYgYEIVcZr4cKF1TbbbFPddttt1VprrTXGMEbXA4FAIBAIBAKBQGCcERiKUD399NPVUkstVZ133nnVzjvvPM44Rt8DgUAgEAgEAoFAYIwRGIpQwe05z3lOdfbZZ1d77LHHGMMYXQ8EAoFAIBAIBAKBcUZgaEL1ghe8oDr11FOrffbZZ5xxjL4HAoFAIBAIBAKBwBgjMDSh2mGHHapf/epXlbp8SsioyxcSCAQCgUAgEAgEAoHAOCEwNKF68MEHqy222KL68Y9/nAodn3zyyeOEX/Q1EAgEAoFAIBAIBAKBamhCtcsuu1RPPPFE9clPfrJaccUVw0MVkyoQCAQCgUAgEAgExg6BoQnV8ssvn7xSe++999iBFx0OBAKBQCAQCAQCgUAAAkMTquWWW676zGc+E6f8Yj4FAoFAIBAIBAKBwNgiEIRqbIc+Oh4IBAKBQCAQCAQCo0JgaEK19NJLp4s9582bN6o2xXsCgUAgEAgEAoFAIBCYVQgMRaguuOCCav78+ZXafgolhwQCgUAgEAgEAoFAIDCOCAxEqL7//e9Xm2++efXLX/6yOvbYY6tjjjlmHLGLPgcCgUAgEAgEAoFAIJAQGIhQ/frXv67uuuuudJHnsssuG1AGAoFAIBAIBAKBQCAw1ggMRKjGGrHofCAQCAQCgUAgEAgEAg0EglDFlAgEAoFAIBAIBAKBQGBIBIJQDQlg/DwQCAQCgUAgEAgEAoEgVDEHAoFAIBAIBAKBQCAQGBKBIFRDAhg/DwQCgUAgEAgEAoFAIAhVzIFAIBAIBAKBQCAQCASGRCAI1ZAAxs8DgUAgEAgEAoFAIBAIQhVzIBAIBAKBQCAQCAQCgSER+D/fHtK9rd8/KwAAAABJRU5ErkJggg==)**

#### Правила и рекомендации по методам расширения в C#

- Поддержка: C# поддерживает только методы расширения — никаких свойств, событий или операторов расширения.    
    
- Объявление: Метод расширения:

	- должен быть в статическом необобщённом классе;
    
	- иметь хотя бы один параметр, перед которым стоит ключевое слово this.  
      

- Поиск компилятором:  
    Методы расширения ищутся только в статических классах верхнего уровня, находящихся в области видимости файла. Вложенные классы не допускаются
    

### ⚙️ Частичные методы

1 🧩 Место объявления:  
Частичные методы можно объявлять только внутри частичных классов или структур.  
  

2 🔁 Возврат и параметры:

- Возвращаемый тип — только void.
    
- out-параметры запрещены, так как метода может не существовать во время выполнения.
    
- Разрешены ref, универсальные, статические, экземплярные и даже unsafe параметры.
    

3 ⚖️ Сигнатуры:  
Объявление и реализация частичного метода должны иметь идентичные сигнатуры.  
Атрибуты метода и его параметров объединяются при компиляции.  
  

4 🚫 Отсутствие реализации:  
Если реализация отсутствует —

- метод не включается в итоговый код,
    
- делегаты к нему создавать нельзя → ошибка CS0762.
    

5 🔒 Модификаторы доступа:  
Частичные методы всегда private (неявно).  
Ключевое слово private писать нельзя — компилятор не позволит.


### ⚙️ Правила использования параметров в C#

1. 🧩 Где можно задавать значения по умолчанию:  
      
	- Методы, конструкторы, индексаторы и делегаты.
    
	- Делегаты с такими параметрами позволяют опускать аргументы при вызове.  
      

2. 📏 Порядок параметров:  
      

	- Параметры со значениями по умолчанию идут после всех обязательных.
    
	- Исключение — params, оно всегда в конце и не может иметь значение по умолчанию.  
      

3. 🔒 Ограничения на значения:  
      

	- Значения по умолчанию должны быть известны на этапе компиляции.
    
	- Разрешено использовать:  
      
		- примитивы (int, bool, char и т. д.);
    
		- enum;
    
		- ссылочные типы (null);
    
		- default или new() для структур.  
      
4. 🚫 Что нельзя делать:  

	- ref и out параметры не могут иметь значения по умолчанию.
    
	- Нельзя переименовывать параметры, если используется именованный вызов → ошибка CS1739.  
    
5. ⚠️ Осторожно с изменением значений по умолчанию:  
    

	- Изменение значения без перекомпиляции вызывающего кода приведёт к тому, что старое значение сохранится.  
      
    

	Лучше использовать null и оператор ?? для безопасной подстановки:  
  
```
private static string MakePath(string filename = null) =>

    $"C:\\{filename ?? "Untitled"}.txt";
```

    

6. 🔠 Именованные и необязательные параметры:  
      

	- Можно передавать аргументы в любом порядке, но именованные — в конце.
    
	- Именованные аргументы можно использовать и для обязательных параметров.
    
	- Между запятыми нельзя пропускать аргументы → M(1, ,DateTime.Now) ❌
    
	- Для ref и out параметры тоже можно вызывать по имени:  
	    `M(x: ref a);`
    
**

### ⚙️ Передача параметров в метод по ссылке

По умолчанию CLR предполагает, что все параметры методов передаются по значению. При передаче объекта ссылочного типа методу передается ссылка (или указатель) на этот объект. То есть метод может изменить переданный объект, влияя на состояние вызывающего кода. Если параметром является экземпляр значимого типа, методу передается его копия. В этом случае метод получает собственную копию объекта, а исходный экземпляр сохраняется неизменным.


CLR также позволяет передавать параметры по ссылке, а не по значению. В C# это делается с помощью ключевых слов `out` и `ref`.


Если параметр метода помечен ключевым словом `out`, вызывающий код может не инициализировать его, пока не вызван сам метод. 

В этом случае вызванный метод не может прочитать значение параметра и должен записать его, прежде чем вернуть управление. 


Если же параметр помечен ключевым словом `ref`, вызывающий код должен инициализировать его перед вызовом метода, а вызванный метод может как читать, так и записывать значение параметра.


В случае ссылочных типов вызывающий код выделяет память для указателя на передаваемый объект, а вызванный код управляет этим указателем. В силу этих особенностей использование ключевых слов out и ref со ссылочными типами полезно, лишь когда метод собирается «вернуть» ссылку на известный ему объект.


### ⚙️Передача переменного количества аргументов

Метод, принимающий переменное число аргументов, объявляют так: 
```
static Int32 Add(params Int32[] values) 
{

}
```

 Ключевым словом params может быть помечен только последний параметр метода `(ParamArrayAttribute)`. Он должен указывать на одномерный массив произвольного типа.

  
Вызов.метода,.принимающего.переменное.число.аргументов,.снижает.производи тельность,.если,.конечно,.не.передавать.в.явном.виде.значение.null .



### ⚙️Типы параметров и возвращаемых значений

 Объявляя тип параметров метода, нужно по возможности указывать «минимальные» типы, предпочитая интерфейсы базовым классам. Например, при написании метода, работающего с набором элементов, лучше всего объявить параметр метода, используя интерфейс `IEnumerable<T>` вместо сильного типа данных, например `List<T>`, или еще более сильного интерфейсного типа `ICollection<T>` или `IList<T>`.

  
```
// Рекомендуется в этом методе использовать параметр слабого типа 

public void ManipulateItems(IEnumerable<T>  collection) { ... } 

  

// Не рекомендуется в этом методе использовать параметр сильного типа 

public void ManipulateItems(List<T>  collection) { ... }
```


 Причина, конечно же, в том, что первый метод можно вызывать, передав в него массив, объект `List<T>` , объект `String` и т. п., то есть любой объект, тип которого реализует интерфейс `IEnumerable<T>` . Второй метод принимает только объекты `List<T>` , с массивами или объектами `String` он работать уже не может. Ясно, что первый метод предпочтительнее, так как он гибче и может использоваться в более разнообразных ситуациях.



В то же время, объявляя тип возвращаемого методом объекта, желательно выбирать самый сильный из доступных вариантов (пытаясь не ограничиваться конкретным типом). Например, лучше объявлять метод, возвращающий объект `FileStream`, а не `Stream`:

 ```
 // Рекомендуется в этом методе использовать сильный тип возвращаемого объекта 

 public FileStream OpenFile() { ... }


 // Не рекомендуется в этом методе использовать  слабый тип возвращаемого объекта 

 public Stream OpenFile() { ... } 
 ```
  

Здесь предпочтительнее первый метод, так как он позволяет вызывающему коду обращаться с возвращаемым объектом как с объектом FileStream или Stream. А вот второму методу требуется, чтобы вызывающий код рассчитывал только на объект Stream, то есть область его применения более ограничена.

*

## 🧩Свойства

Одним из краеугольных камней объектно-ориентированного программирования и разработки является инкапсуляция данных. Инкапсуляция данных означает, что поля типа ни в коем случае не следует открывать для общего доступа, так как в этом случае слишком просто написать код, способный испортить сведения о состоянии объекта путем ненадлежащего использования полей. 


Методы, выполняющие функции оболочки для доступа к полю, обычно называют методами доступа (`accessor`).


Можно считать свойства «умными» полями, то есть полями с дополнительной логикой. CLR поддерживает статические, экземплярные, абстрактные и виртуальные свойства.

**

## 🧩Анонимные типы

Кортежный тип (tuple type) — это тип, который содержит коллекцию свойств, каким-то образом связанных друг с другом.

```
// Определение типа, создание сущности и инициализация свойств
var o1 = new { Name = "Jeff", Year = 1964 }; 


// Вывод свойств на консоль 
Console.WriteLine("Name={0}, Year={1}", o1.Name, o1.Year);
```

  

## 🧩Свойства с параметрами (индексаторы)

индексатор можно представить как средство, позволяющее разработчику на C# перегружать оператор []. 

```
 // Индексатор (свойство с параметрами)
 public Boolean this[Int32 bitPos] { 

	// Метод доступа get индексатора 

	get {..... }

 }
```
  

## 🧩Cобытия

Тип, в котором определено событие (или экземпляры этого типа), может уведомлять другие объекты о некоторых особых ситуациях, которые могут случиться. 

Например, если в классе Button (кнопка) определить событие Click (щелчок), то в приложение можно использовать объекты, которые будут получать уведомление о щелчке объекта Button, а получив такое уведомление — исполнять некоторые действия. 

События — это члены типа, обеспечивающие такого рода взаимодействие.

 А именно определения события в типе означает, что тип поддерживает следующие возможности: 

1 регистрация своей заинтересованности в событии; 

2 отмена регистрации своей заинтересованности в событии; 

3 оповещение зарегистрированных методов о произошедшем событии.

  

Модель событий CLR основана на *делегатах* (`delegate`). 

Делегаты обеспечивают реализацию механизма обратного вызова, безопасную по отношению к типам. Методы обратного вызова (callback methods) позволяют объекту получать уведомления, на которые он подписался.

**

### ⚙️ Разработка типа, поддерживающего событие

#### 🧩 Этап 1. Создание класса для передачи данных события

1. При возникновении события объект-источник должен передать обработчикам дополнительные данные.
    
2. Эти данные помещаются в класс-наследник `System.EventArgs`, имя которого оканчивается на `EventArgs`.
    
3. Класс содержит *закрытые поля* и *публичные свойства только для чтения*.   

```
 internal class NewMailEventArgs : EventArgs {

    public string From { get; }

    public string To { get; }

    public string Subject { get; }

    public NewMailEventArgs(string from, string to, string subject) =>

        (From, To, Subject) = (from, to, subject);

}
```

4. Если событие не передает данных, можно использовать готовый объект  
    `EventArgs.Empty`.
    
---

#### 🔔 Этап 2. Объявление события в типе

1. Событие объявляется с ключевым словом event и типом делегата.  
      
    ```
    public event EventHandler<NewMailEventArgs> NewMail;
    ```
    
2. Делегат `EventHandler<T>` имеет сигнатуру:  
      
    ```
    void MethodName(object sender, T e);
    ```
    
3. Параметр `sender` всегда имеет тип `object`,  
    чтобы событие можно было использовать в наследуемых типах  
    (например, `MailManager` и `SmtpMailManager`).
    
4. Параметр данных события традиционно называется e,  
    для единообразия во всех событиях .NET.
    
5. Все обработчики событий должны возвращать `void`,  
    потому что при возникновении события вызывается *несколько обработчиков*,  
    и вернуть общее значение невозможно.
    

---

#### 📤 Этап 3. Метод уведомления подписчиков

1. В классе определяется *защищённый виртуальный метод* `OnEventName`,  
    который вызывает событие и уведомляет подписчиков.
    
2. Используется для *вызова события внутри класса или его потомков*.
    
3. Рекомендуется сохранять делегат во временной переменной для  
    *безопасности при многопоточности*.  
      
    ```
    protected virtual void OnNewMail(NewMailEventArgs e) {


	    var temp = Volatile.Read(ref NewMail);
	
	    temp?.Invoke(this, e);

	}
    ```

4. Если класс не предназначен для наследования, метод можно сделать `private` или невиртуальным.  
      

---

#### ✅ Итоговая структура типа с событием:

1. Класс данных события → `...EventArgs`.  
      
    
2. Делегат (обычно` EventHandler<T>`).  
      
    
3. Событие с ключевым словом ``event``.  
      
    
4. Метод `OnEventName()` для вызова события.
    

**

### ⚙️ Реализация событий компилятором (C#)

Когда ты пишешь:

```csharp
public event EventHandler<NewMailEventArgs> NewMail;
```

🔽  
Компилятор автоматически генерирует три элемента 👇

---

#### 1️⃣ Закрытое поле делегата

```csharp
private EventHandler<NewMailEventArgs> NewMail = null;
```


- Хранит список подписчиков (делегатов).   
    
- Всегда private, чтобы внешний код не мог:      

	- очистить список;        
    
	- напрямую вызывать или изменять событие.
    

---

#### 2️⃣ Метод add_Xxx — добавление обработчика

```csharp
public void add_NewMail(EventHandler<NewMailEventArgs> value) {

    EventHandler<NewMailEventArgs> prevHandler, newMail;

    do {

        prevHandler = newMail = this.NewMail;

        var newHandler = (EventHandler<NewMailEventArgs>)

            Delegate.Combine(prevHandler, value);

        newMail = Interlocked.CompareExchange(

            ref this.NewMail, newHandler, prevHandler);

    } while (newMail != prevHandler);

}
```


- Позволяет подписаться на событие.        
    
- Использует `Delegate.Combine` для добавления делегата.        
    
- Реализовано безопасно для многопоточности (через `CompareExchange`).
    

---

#### 3️⃣ Метод remove_Xxx — удаление обработчика

``` csharp
public void remove_NewMail(EventHandler<NewMailEventArgs> value) {

    EventHandler<NewMailEventArgs> prevHandler, newMail;

    do {

        prevHandler = newMail = this.NewMail;

        var newHandler = (EventHandler<NewMailEventArgs>)

            Delegate.Remove(prevHandler, value);

        newMail = Interlocked.CompareExchange(

            ref this.NewMail, newHandler, prevHandler);

    } while (newMail != prevHandler);

}
```

- Позволяет отписаться от события.        
    
- Использует `Delegate.Remove.`        
    
- Если обработчик не найден → ошибки не будет.
    

---

#### ⚙️ Дополнительно

- Модификаторы `add/remove` наследуют доступность события  
    (`public`, `protected`, `static`, `virtual` и т. д.).        
    
- В метаданные модуля добавляется запись события (тип делегата + ссылки на `add` и `remove`).        
    
- CLR не использует эти метаданные в рантайме — они нужны лишь инструментам и Reflection.    

---

#### 🧠 Итого:

Событие в C# — это **инкапсулированный делегат**  
с автоматически создаваемыми методами `add` и `remove`,  
обеспечивающими безопасную, управляемую подписку и отписку.


**

## 🧩Обобщения

### ⚙️ Преимущества обобщений (Generics) в C#

---

#### 🛡️ 1. Защита исходного кода

- Пользователь не видит исходник обобщённого алгоритма.        
    
- Достаточно использовать скомпилированный тип (в отличие от шаблонов C++).    

---

#### ✅ 2. Безопасность типов

- Компилятор и CLR контролируют соответствие типов.        
    
- Невозможно передать несовместимый объект → ошибка на этапе компиляции.  
          
- Меньше runtime-ошибок, больше надёжности.    

---

#### 🧠 3. Простой и читаемый код

- Без приведения типов (кастов).        
    
- Код становится чище, короче и безопаснее.        
    
- Легче сопровождать и отлаживать.    

---

#### ⚡ 4. Повышение производительности

- Без упаковки (boxing) и распаковки (unboxing) значимых типов.        
    
- Меньше выделений памяти → меньше работы GC → быстрее исполнение.        
    
- CLR не тратит ресурсы на проверки преобразований типов.    

---

#### 🧩 Итого:

**Generics** = безопасный, быстрый и универсальный способ  
писать переиспользуемый код без потери производительности и сильной типизацией.

  

### ⚙️Контравариантные и ковариантные аргументы-типы в делегатах и интерфейсах

 Каждый из параметров-типов обобщенного делегата должен быть помечен как ковариантный или контравариантный. Это позволяет вам осуществлять приведение типа переменной обобщенного делегата к тому же типу делегата с другим параметром-типом. 

Параметры-типы могут быть: 

- 1 Инвариантными. Параметр-тип не может изменяться. Пока в этой главе при водились только инвариантные параметры-типы.

- 2 Контравариантными. Параметр-тип может быть преобразован от класса к классу, производному от него. В языке C# контравариантный тип обозначается ключевым словом in. Контравариантный параметр-тип может появляться только во входной позиции, например, в качестве аргументов метода. 

- 3 Ковариантными. Аргумент-тип может быть преобразован от класса к одному из его базовых классов. В языке С# ковариантный тип обозначается ключевым словом out. Ковариантный параметр обобщенного типа может появляться только в выходной позиции, например, в качестве возвращаемого значения метода.
  

Предположим, что существует следующий тип делегата: 

```cs
public delegate TResult Func(T arg); 
```

Здесь параметр-тип T помечен словом `in`, делающим его контравариантным, а параметр-тип `TResult` помечен словом `out`, делающим его ковариантным.


**

### 📘 Обобщённые методы (Generics Methods)

#### 🧩 1. Что это такое

- Обобщённые методы — это методы, у которых есть собственные параметр-типы,  
    независимые от параметров типа класса или структуры.        
    
- Такие методы можно применять для любых типов данных без потери типобезопасности  
    

---

#### ⚙️ 2. Зачем нужны

- Повышают гибкость — позволяют использовать разные типы в одном методе.  
      
    
- Обеспечивают типобезопасность: компилятор контролирует совместимость типов.  
      
    
- Избавляют от приведения типов и упаковки/распаковки, что ускоряет работу кода.  
      
    

---

#### 💡 3. Пример идеи

Класс может иметь свой параметр типа T,  
а метод внутри — дополнительный собственный параметр типа, например TOutput.  
Это даёт возможность, например, преобразовывать значения из одного типа в другой,  
не завязываясь на конкретные типы.

🧠 Пример (в словах):

Метод `Converter<TOutput>()` может брать значение типа `T` и возвращать его в виде `TOutput`.  
То есть `GenericType<int>` может вызывать `Converter<string>()`.

---

#### 🔁 4. Пример практического применения — метод `Swap<T>`

- Меняет местами два значения любого типа.       
    
- Благодаря `T` работает одинаково для `int`, `string`, `DateTime` и т. д.        
    
- Вызывается как` Swap<int>(ref x, ref y)` или просто `Swap(ref x, ref y) `— компилятор сам выведет тип.       
    

---

#### 🧱 5. Особенности с ref и out

- При использовании `ref/out` параметры должны быть строго одного типа,  
    что обеспечивает дополнительную безопасность типов.        
    
- Именно поэтому методы` Interlocked.Exchange<T>` и `CompareExchange<T> ` 
    (в многопоточном коде) реализованы как обобщённые —  
    чтобы избежать ошибок при передаче разных типов.       
    

---

#### ✅ 6. Главное запомнить

|                      |                                                                               |
| -------------------- | ----------------------------------------------------------------------------- |
| Преимущество         | Описание                                                                      |
| 🔒 Типобезопасность  | Ошибки ловятся на этапе компиляции, без приведения типов                      |
| ⚡ Производительность | Без упаковки/распаковки и кастов                                              |
| 🧠 Универсальность   | Один метод — для любых типов                                                  |
| 🧩 Независимость     | Метод может иметь свой собственный параметр типа, не связанный с типом класса |


### ⚙️Ограничения

Ограничение сужает перечень типов, которые можно передать в обобщенном аргументе, и расширяет возможности по работе с этими типами. Вот новый вариант метода Min, который задает ограничение (выделено полужирным шрифтом): 


```cs
public static T Min(T o1, T o2) where T : IComparable

 {

  if (o1.CompareTo(o2) < 0) 

return o1; return o2;

 }
```

  
 Маркер where в C# сообщает компилятору, что указанный в `T` тип должен реализовывать обобщенный интерфейс `IComparable` того же типа `(T)`.

  
 При переопределении виртуального обобщенного метода в переопределяющем методе должно быть задано то же число параметров-типов, а они, в свою очередь, наследуют ограничения, заданные для них методом базового класса. Собственно, переопределяемый метод вообще не вправе задавать ограничения для своих параметров-типов, но может переименовывать параметры-типы.

**

## 🧩 Интерфейсы

интерфейс представляет собой именованный набор сигнатур методов. Обратите внимание, что в интерфейсах можно также определять события, свойства — без параметров или с ними (последние в C# называют индексаторами), поскольку все это просто упрощенные средства синтаксиса, которые в конечном итоге все равно соответствуют методам. Однако в интерфейсе нельзя определять ни конструкторы, ни экземплярные поля.


C# не позволяет определять в интерфейсе статические члены.


Компилятор C# требует, чтобы метод, реализующий интерфейс, отмечался модификатором public. CLR требует, чтобы интерфейсные методы были виртуальными. Если метод явно не определен в коде как виртуальный, компилятор сделает его таковым и, вдобавок, запечатанным. Это не позволяет производному классу переопределять интерфейсные методы. Если явно задать метод как виртуальный, компилятор сделает его таковым и оставит незапечатанным, что предоставит производному классу возможность переопределять интерфейсные методы.

  
###  **⚙️**Явные и неявные реализации интерфейсных методов (что происходит за кулисами)

  
Когда тип загружается в CLR, для него создается и инициализируется таблица методов (см. главу 1). Она содержит по одной записи для каждого нового, представляемого только этим типом метода, а также записи для всех виртуальных методов, унаследованных типом. Унаследованные виртуальные методы включают методы, определенные в базовых типах иерархии наследования, а также все методы, определенные интерфейсными типами. 



 Если в C# перед именем метода указано имя интерфейса, в котором определен этот метод (в нашем примере — IDisposable.Dispose), то вы создаете явную реализацию интерфейсного метода (Explicit Interface Method Implementation, EIMI). Заметьте: при явной реализации интерфейсного метода в C# нельзя указывать уровень доступа (открытый или закрытый). Однако когда компилятор создает метаданные для метода, он назначает ему закрытый уровень доступа (private), что запрещает любому коду использовать экземпляр класса простым вызовом интерфейсного метода. Единственный способ вызвать интерфейсный метод — обратиться через переменную этого интерфейсного типа.


```cs 
internal sealed class SimpleType : IDisposable 
{ 
	public void Dispose() 
	{ 

		Console.WriteLine("public Dispose");
	} 

	void IDisposable.Dispose() 	
	{ 	
		Console.WriteLine("IDisposable Dispose");	
	}
 }
```
  

### **⚙️** Обобщенные интерфейсы

Преимущества: 

- 1 обобщенные интерфейсы обеспечивают безопасность типов на стадии компиляции. 

- 2 при работе со значимыми типами требуется меньше операций упаковки. 

- 3 класс может реализовать один интерфейс многократно, просто используя параметры различного типа.  

### **⚙️** Обобщения и ограничения интерфейса
  

- 1  параметр-тип можно ограничить несколькими интерфейсами. В этом случае тип передаваемого параметра должен реализовывать все ограничения.

 ```cs
 // Параметр T типа M ограничивается только теми типами, 
// которые реализуют оба интерфейса: IComparable И IConvertible
 private static Int32 M(T t) where T : IComparable, IConvertible { ... }
 ```

- 2  избавление от упаковки при передаче экземпляров значимых типов. В предыдущем фрагменте кода методу M передавался аргумент x (экземпляр типа `Int32`, то есть значимого типа). При передаче x в M упаковка не выполнялась. Если код метода M вызовет `t.CompareTo(...)`, то упаковка при вызове также не будет выполняться (упаковка может выполняться для аргументов, передаваемых `CompareTo`).


### 🧩 EIMI — явная реализация интерфейсных методов

#### 💡 1. Проблема

Иногда приходится реализовывать необобщённые интерфейсы (например, IComparable),  
где методы принимают или возвращают Object.  
Это приводит к двум проблемам:

1. ⚙️ Упаковка (boxing) значимых типов при передаче в Object.  
      
    
2. ❌ Потеря типобезопасности — возможны ошибки приведения (InvalidCastException).
    

📉 Пример:

Int32 n = v.CompareTo(v); // упаковка

n = v.CompareTo(o);       // InvalidCastException

  

---

#### 🛠️ 2. Решение — явная реализация (EIMI)

Можно реализовать два метода:

1. ✅ Типобезопасный — с параметром конкретного типа (CompareTo(SomeValueType other)).  
      
    
2. ⚙️ Интерфейсный — явно реализующий контракт (Int32 IComparable.CompareTo(Object other)).
    

🧠 Смысл:

- Первый метод используется напрямую — без упаковки и безопасно.
    
- Второй нужен только для совместимости с интерфейсом.
    

---

#### 🚀 3. Что это даёт

|   |   |
|---|---|
|Преимущество|Описание|
|🔒 Типобезопасность|Ошибки приводятся на этапе компиляции.|
|⚡ Без упаковки|При прямом вызове метода.|
|🧩 Совместимость|Сохраняется контракт интерфейса (IComparable).|

---

#### ⚠️ 4. Но есть нюанс

Если вызвать метод через переменную интерфейсного типа,  
всё равно произойдёт упаковка (требование CLR):

IComparable c = v; // упаковка

c.CompareTo(v);    // снова упаковка

  

---

#### 🧠 5. Где применяется EIMI

Используется при реализации интерфейсов:

- IComparable
    
- IConvertible
    
- ICollection
    
- IList
    
- IDictionary
    

---

#### 📘 Главное запомнить

EIMI позволяет сочетать типобезопасность и совместимость с интерфейсами.  
Прямой вызов — безопасен и без упаковки.  
Через интерфейс — упаковка неизбежна, но контракт сохраняется.




### ⚠️ Опасности явной реализации интерфейсных методов (EIMI)

 💡 1. Когда используют EIMI

EIMI применяют, когда:

- нужно реализовать два интерфейса с одинаковыми методами;  
      
    
- интерфейс не имеет обобщённой версии, и избежать Object невозможно.  
      
    

Однако злоупотреблять EIMI не стоит — у него есть серьёзные недостатки.

---

 🚨 2. Основные проблемы EIMI

#### 🧩 1. Нет документации и поддержки IntelliSense

- В справке по типу (например, Int32) не видно явно реализованных методов.  
      
- Visual Studio не подсказывает их при вводе.  
      
- Разработчик может не догадаться, что метод вообще существует.  
      

📉 Пример:

```cs
int x = 5;

x.ToSingle(null); // Ошибка: int не содержит ToSingle
```

  
Чтобы вызвать метод, нужно привести:

```cs
float s = ((IConvertible)x).ToSingle(null);
```
  
— неочевидно и неудобно.

---

#### 📦 2. Упаковка значимых типов

При приведении struct к интерфейсу CLR упаковывает объект:

```cs
IConvertible c = x; // упаковка int
```

Это:

- тратит память и процессорное время;        
    
- снижает производительность;        
    
- делает EIMI неэффективным для частых вызовов.       
    

---

#### 🚫 3. Нельзя вызвать EIMI из производного класса

- EIMI — это не `public` и не `protected` метод,  
    поэтому `base.CompareTo(o)` в наследнике не скомпилируется.        
    
- Вызов через `IComparable c = this; c.CompareTo(o); ` 
    создаёт бесконечную рекурсию.  
      

---

#### 🧠 3. Как правильно исправить

✅ Решение: добавить виртуальный метод  

```cs
//В базовом классе:
class Base : IComparable {

    int IComparable.CompareTo(object o) {

        Console.WriteLine("Base’s IComparable CompareTo");

        return CompareTo(o); // вызов виртуального метода]
    }  

    public virtual int CompareTo(object o) {

        Console.WriteLine("Base’s virtual CompareTo");

        return 0;
    }
}

//В производном:

sealed class Derived : Base, IComparable {

    public override int CompareTo(object o) {

        Console.WriteLine("Derived’s CompareTo");

        return base.CompareTo(o);
    }
}
```

🔹 Теперь:

- EIMI вызывает виртуальный метод;        
    
- наследники могут безопасно переопределять поведение;        
    
- код остаётся читаемым и предсказуемым.        
    

---

#### 📘 4. Итог

|   |   |
|---|---|
|Проблема|Последствие|
|Нет IntelliSense и документации|Методы "прячутся" от разработчика|
|Упаковка значимых типов|Потери производительности|
|Невозможность вызова из наследников|Ошибки и рекурсия|

---

#### ✅ Вывод

Явную реализацию интерфейсных методов нужно использовать только при необходимости  
(например, при конфликте сигнатур).  
В остальных случаях — предпочитайте обобщённые интерфейсы и открытые виртуальные методы.


**

### ⚖️ Дилемма разработчика: базовый класс или интерфейс

#### 💡 Основной вопрос

Когда проектируешь новый тип — что выбрать:  
📘 базовый класс или 🔷 интерфейс?  
Оба подхода полезны, но применяются в разных случаях.

---

#### 🧭 1. Связь потомка с предком

- Базовый класс — это отношение «является частным случаем» (is-a).  
    → Один тип может наследовать только один базовый класс.  
      
    
- Интерфейс — это отношение «поддерживает функциональность» (supports).  
    → Можно реализовать несколько интерфейсов.  
      
    

📌 Пример:

- IConvertible — тип поддерживает преобразование.  
      
    
- ISerializable — тип умеет сериализоваться.  
      
    
- ValueType не может иметь базовый класс, кроме System.ValueType,  
    поэтому для структур всегда используют интерфейсы.  
      
    

---

#### ⚙️ 2. Простота использования

- Базовый класс может предоставлять готовую реализацию.  
    → Достаточно переопределить только нужные части.  
      
    
- Интерфейс требует реализации всех методов вручную.  
      
    

📈 Поэтому наследование от базового класса удобнее,  
если можно переиспользовать код.

---

#### 📚 3. Четкость и надежность реализации

- Интерфейс описывает контракт, но не гарантирует корректную реализацию.  
      
    
- Базовый класс с продуманной реализацией служит надежным фундаментом.  
      
    

🧩 Пример: в COM часто были ошибки из-за неправильных реализаций интерфейсов.  
В .NET лучше использовать базовые классы, если есть общий код.

---

#### 🔄 4. Управление версиями

- Добавление метода в базовый класс не ломает существующий код —  
    наследники просто получают новую реализацию.       
        
- Добавление метода в интерфейс требует:  -      

	- изменения исходного кода всех реализаций;        
    
	- их перекомпиляции.          

✅ Поэтому базовые классы проще поддерживать в будущем.

---

#### 🧱 5. Примеры из .NET Framework

##### 📂 Когда выбрали базовый класс

- `System.IO.Stream` — базовый абстрактный класс для потоков.  
    → Потомки (`FileStream`, `MemoryStream`, `NetworkStream`) наследуют  
    общие методы (`Read`, `Write`, `async`-операции).  
      
    
- `System.Windows.Forms.Control` — базовый класс всех UI-элементов  
    (`Button`, `ListBox`, `TextBox`) с огромным объёмом общей логики.  
      

##### 🔗 Когда выбрали интерфейсы

- Коллекции в `System.Collections.Generic`:  
    `IEnumerable<T>`, `ICollection<T>`, `IList<T>`, `IDictionary<TKey, TValue>`.  
    → Реализуются разными классами (`List<T>`, `Dictionary<K,V>`,` Queue<T>`).  
    → Общего кода мало, поэтому удобнее интерфейсы.        
    

📊 Интерфейсы позволяют писать универсальный код,  
не зная конкретный тип коллекции (например, работать через `IList<T>`).

---

#### 🔧 6. Комбинированный подход

Иногда используют и интерфейс, и базовый класс, реализующий его.

📘 Пример:

- Интерфейс `IComparer<T>`      
    
- Абстрактный класс `Comparer<T>` реализует этот интерфейс частично  
    и даёт реализацию по умолчанию.        
    
Такой подход сочетает гибкость интерфейсов и удобство наследования.

---

#### ✅ Вывод

| Критерий                | Базовый класс      | Интерфейс                |
| ----------------------- | ------------------ | ------------------------ |
| Отношение               | «является»         | «поддерживает»           |
| Наследование            | Только одно        | Несколько                |
| Реализация по умолчанию | Есть               | Нет                      |
| Простота расширения     | Легко              | Требует новой реализации |
| Версионность            | Без перекомпиляции | Требует перекомпиляции   |
| Примеры                 | Stream, Control    | IList<T>, IComparer<T>   |

---

Итог:

🔹 Используй базовый класс, если есть общий код и поведение.  
🔹 Используй интерфейс, если важна гибкость и нет общей реализации.  
🔹 Идеальный вариант — интерфейс + абстрактный базовый класс,  
реализующий этот интерфейс частично.


**

## 🧩 Тип System.String

`System.String` — это неизменяемый ссылочный тип, представляющий упорядоченный набор символов Unicode.  
Он наследуется от `System.Object` и реализует интерфейсы:

- `IComparable`, `IComparable<string>` — для сравнения строк        
    
- `ICloneable` — для копирования        
    
- `IConvertible` — для преобразований типов        
    
- `IEnumerable<char>` — для перебора символов  
    
- `IEquatable<string>` — для быстрого сравнения на равенство        
    

Строки всегда размещаются в куче, даже если они выглядят как примитивы.

---

### 🏗 Создание строк

#### 🔹 Литералы и загрузка в метаданные

Компилятор C# позволяет вставлять строковые литералы прямо в исходный код:  
  
string s = "Hi there.";

-   
    
- Эти строки сохраняются в метаданных сборки и загружаются во время выполнения.  
      
    
- В IL для создания строк используется инструкция ldstr, а не newobj.  
    Это значит: CLR создаёт строку особым образом, используя встроенный пул строк (см. интернирование).  
      
    

#### 🔹 Конструкторы

- Можно создавать строки через конструкторы, принимающие:           

	- массив char[],        
    
	- указатель char* или sbyte* (в unsafe коде).            

- Но нельзя использовать `new String("...")` для литералов — это синтаксическая ошибка.  
---

### 🧾 Специальные формы строк

#### 🔹 Управляющие последовательности

В обычных строках спецсимволы (например \n, \r, \t) обозначаются обратной косой чертой:

string s = "Hi\r\nthere.";

  

⚠️ Лучше использовать Environment.NewLine, чтобы строка корректно работала на всех платформах:

string s = "Hi" + Environment.NewLine + "there.";

  

---

#### 🔹 Конкатенация строк

Строки можно объединять с помощью оператора +:  
  
string s = "Hi" + " " + "there.";

-   
    
- Если все части — литералы, компилятор объединяет их на этапе компиляции.  
      
    
- Если части — переменные, строки конкатенируются во время выполнения, что создаёт новые объекты в куче.  
    Это неэффективно при множестве операций → используй StringBuilder (System.Text).  
      
    

---

#### 🔹 Буквальные строки (verbatim strings)

Перед строкой ставится @, и все символы воспринимаются буквально:

string path = @"C:\Windows\System32\Notepad.exe";

  

- @ отключает экранирование — \n и \\ трактуются как обычные символы.  
      
    
- Особенно удобно для путей и регулярных выражений.  
      
    

---

### 🧱 Неизменяемость строк

#### ❗ Ключевой принцип:

Строки в .NET неизменяемы — после создания их содержимое нельзя изменить.

Это значит:

- нельзя заменить символ внутри строки;        
    
- любые операции вроде ToUpper(), Substring(), Replace() создают новую строку.  
    
Пример:

```cs
s.ToUpperInvariant().Substring(10, 21).EndsWith("EXE");
```
  

Каждый вызов создаёт новый объект String, старые собираются GC.

---

#### ⚙️ Почему строки неизменяемы

 ✅ Плюсы:

1. Безопасность и потокобезопасность.  
    Одну строку можно использовать из разных потоков без синхронизации.  
      
    
2. Интернирование (String Interning).  
    CLR может хранить одну копию идентичных строк — экономия памяти.  
      
    
3. Предсказуемость поведения.  
    Нет побочных эффектов при передаче строк как параметров.  
      
    
4. Оптимизация под CLR.  
    CLR знает внутреннюю структуру String и напрямую работает с её полями.  
      
    

---

### 🔒 Почему String запечатан (sealed)

- CLR жёстко полагается на структуру и свойства System.String.  
    Если бы его можно было наследовать, пользовательский подкласс мог бы:  
      
    

- нарушить неизменность,  
      
    
- добавить поля, нарушающие внутреннюю память CLR,  
      
    
- изменить методы, критичные для производительности (например Equals, GetHashCode).  
      
    

Поэтому String sealed — нельзя наследовать и расширять.

---

### 🧠 Итог: суть типа System.String

| Свойство             | Описание                                                                    |
| -------------------- | --------------------------------------------------------------------------- |
| Тип                  | Ссылочный, sealed                                                           |
| Хранение             | Только в куче                                                               |
| Представление        | Набор символов Unicode                                                      |
| Создание             | Через литералы или конструкторы (в особых случаях)                          |
| Неизменяемость       | Да — любое изменение создаёт новую строку                                   |
| Интернирование       | CLR хранит одну копию одинаковых строк                                      |
| Безопасность потоков | Полная                                                                      |
| Производительность   | Оптимизирован через прямую интеграцию с CLR                                 |
| Расширяемость        | Только через вспомогательные типы (StringBuilder, ReadOnlySpan<char> и др.) |

---

### ⚡ Вывод

- String — фундаментальный тип .NET, сочетающий удобство, безопасность и эффективность.  
      
    
- Его неизменяемость — сознательный дизайн для:  
      
    

- потокобезопасности,  
      
    
- экономии памяти,  
      
    
- стабильности работы CLR.  
      
    

- Для частых изменений текста — использовать StringBuilder.  
      
    
- Для кроссплатформенных строк — Environment.NewLine и Unicode.  
      
    
- Для удобных литералов — @ (verbatim strings).
    

  
  

### 🧾 Сравнение строк

  

#### 🔹 Основные цели сравнения строк

Сравнение используется для:

1. Проверки равенства строк.  
      
    
2. Сортировки (особенно при отображении пользователю).  
      
    

---

#### 🔹 Рекомендуемые методы класса String

Использовать следует методы, где явно задаётся тип сравнения:

- Equals(string value, StringComparison comparisonType)  
      
    
- Compare(string a, string b, StringComparison comparisonType)  
      
    
- StartsWith, EndsWith с аргументом StringComparison или CultureInfo.  
      
    

Использование операторов ==, !=, CompareTo, CompareOrdinal и других неявных методов не рекомендуется, т.к. они не фиксируют способ сравнения и ведут к неоднозначности.

---

#### 🔹 Виды сравнений (StringComparison)

|   |   |
|---|---|
|Значение|Описание|
|Ordinal / OrdinalIgnoreCase|Побайтовое сравнение, быстрое, не учитывает культуру. Подходит для внутренних данных (путь, URL, ключи реестра и т. д.).|
|CurrentCulture / CurrentCultureIgnoreCase|Лингвистически корректное, с учётом текущей культуры пользователя. Использовать при выводе данных пользователю.|
|InvariantCulture / InvariantCultureIgnoreCase|Универсальное, но медленное; не рекомендуется.|

---

#### 🔹 Сравнение и культура (CultureInfo)

- Каждая культура (en-US, de-DE, ja-JP и т.д.) задаёт правила сортировки и сравнения символов.  
      
    
- В .NET у каждого потока есть свойства:  
      
    

- CurrentCulture — для чисел, дат и строковых операций.  
      
    
- CurrentUICulture — для выбора локализованных ресурсов интерфейса.  
      
    

- Эти значения можно переопределить через CultureInfo.DefaultThreadCurrentCulture и ...UICulture.  
      
    

---

#### 🔹 Производительность и точность

- Для внутренних сравнений: StringComparison.Ordinal или OrdinalIgnoreCase.  
      
    
- Для пользовательских интерфейсов: CurrentCulture или CurrentCultureIgnoreCase.  
      
    
- Приведение регистра перед сравнением — через ToUpperInvariant (оптимизировано в FCL).  
      
    

---

#### 🔹 Объект CompareInfo

- Каждая культура имеет свой CompareInfo, в котором хранятся таблицы сравнения.  
      
    
- Через него можно делать расширенные сравнения, например, игнорировать тип каны (CompareOptions.IgnoreKanaType) в японском.  
      
    
- Флаги CompareOptions дают тонкий контроль (например, IgnoreCase, IgnoreWidth, StringSort и др.).  
      
    

---

#### 🔹 Итоговые рекомендации

✅ Явно указывайте тип сравнения — повышает читаемость и предсказуемость кода.  
✅ Используйте Ordinal для логики, CurrentCulture для UI.  
⚠️ Избегайте InvariantCulture, если строка отображается пользователю.  
⚠️ Не используйте CompareTo, ==, !=, если важен конкретный способ сравнения.

  
  

### 🧱Интернирование строк 

#### 🔹 Проблема

Сравнение и хранение строк — ресурсоёмкие операции:

- При порядковом сравнении (Ordinal) CLR проверяет длину, а затем символы.  
      
    
- При культурном сравнении приходится сравнивать посимвольно, т.к. строки разной длины могут считаться равными.  
      
    
- Так как строки неизменяемы, хранение множества одинаковых экземпляров тратит память впустую.  
      
    

---

#### 🔹 Решение: интернирование строк

Интернирование (string interning) — механизм CLR, позволяющий хранить только один экземпляр каждой уникальной строки.

CLR при запуске создаёт внутреннюю хеш-таблицу, где:

- ключ — строка,  
      
    
- значение — ссылка на объект String в управляемой куче.  
      
    

---

#### 🔹 Методы для работы

string String.Intern(string str)

string String.IsInterned(string str)

  

- Intern(str) — ищет строку в таблице.  
      
    

- Если найдена — возвращает ссылку.  
      
    
- Если нет — добавляет копию строки и возвращает ссылку на неё.  
      
    

- IsInterned(str) — только ищет, не добавляя строку; возвращает null, если не найдена.  
      
    

Интернированные строки живут, пока не выгружен AppDomain — GC их не очищает.

---

#### 🔹 Интернирование литералов

- По умолчанию все строковые литералы сборки интернируются.  
      
    

Это можно отключить, если сборка помечена атрибутом:  
  
[CompilationRelaxations(CompilationRelaxations.NoStringInterning)]

-   
    
- Компилятор C# всегда добавляет этот атрибут для оптимизации.  
      
    
- Однако некоторые версии CLR (например, 4.5) всё равно интернируют строки при загрузке сборки.  
    ➡️ Нельзя полагаться на автоматическое интернирование — только на явный вызов String.Intern.  
      
    

---

#### 🔹 Пример

string s1 = "Hello";

string s2 = "Hello";

Object.ReferenceEquals(s1, s2); // может быть True или False в зависимости от CLR

s1 = String.Intern(s1);

s2 = String.Intern(s2);

Object.ReferenceEquals(s1, s2); // всегда True

  

После интернирования обе переменные ссылаются на один объект в куче.

---

#### 🔹 Практическое применение

Без интернирования:

- При каждом сравнении строк выполняется посимвольная проверка.  
      
    
- В куче множество копий одинаковых строк.  
      
    

С интернированием:

- Строки ссылаются на общий объект → экономия памяти.  
      
    
- Сравнение выполняется по указателям (ReferenceEquals) → быстрее.  
      
    

---

#### 🔹 Но осторожно!

- Интернирование само по себе требует времени при вставке в таблицу.        
    
- Эффект ощутим только при множественном повторном использовании одних и тех же строк.  
      
- Поэтому не стоит интернировать всё подряд — это может ухудшить производительность.        
    

---

#### ⚡ Вывод

| Плюсы                                     | Минусы                                                  |
| ----------------------------------------- | ------------------------------------------------------- |
| Быстрее сравнение (по ссылке)             | Интернирование требует времени                          |
| Экономия памяти при повторяющихся строках | Интернированные строки не собираются GC                 |
| Гарантированная уникальность строк        | Потенциально увеличивает нагрузку при массовых вставках |

📌 Использовать интернирование вручную стоит только там, где строки часто повторяются и активно сравниваются (например, в анализаторах текста, парсерах, компиляторах и т.п.).


### 📌Создание пулов строк разобрать отдельно!!


### Методы копирования строк

