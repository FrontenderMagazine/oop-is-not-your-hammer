# Не в ООП сила

Будучи молодым программистом, я выучил первый шаблон проектирования, коим стало наследование. Конечно же, так я познакомился с объектно-ориентированным программированием (ООП). Я был потрясен простой идеей, дополнения или изменения поведение объекта, посредством замены его небольшой части. В конце концов, я продолжил изучать и применять более сложные [шаблоны проектирования, созданные «бандой четырех»][1], каждый из которых расширял простое наследование. Казалось, что наследование и ООП были ответами на все вопросы проектирования, уже известные или *скоро-я-столкнусь-с-этим*.

В 1996, когда я начал работать с JavaScript, я искал способы применения знакомых шаблонов. Вскоре я понял, что JavaScript представляет три препятствия к использовании шаблонов проектирования, основанным на наследовании:

- отсутствие формальных интерфейсов;

- странные цепочки прототипов;

- отсутствие супер-конструктора

Эти препятствия решало бесконечное числом программистов, в том числе и я. К сожалению, попытки применения их в коде делают конечную цель (шаблоны проектирования, основанные на наследовании) более сложной.

Проблемы также были решены предлагаемыми изменениями в языке. Некоторые, вроде Object.create() имеют другие замечательные применения, но двигают нас еще дальше от ООП. В JavaScript  есть свой набор инструментов, который можно применять для создания сложных шаблонов.

## Проблема

Давайте взглянем на простой интернет-магазин. Предположим, у нас есть страница с описанием товара и формой заказа с кнопкой «добавить в корзину», а также виджет, позволяющий пользователю указать количество товаров.

Спроектировать такое приложение достаточно просто. Один с подходов заключается в создании трех составляющих:

- модели, которая олицетворяет предмет в корзине и имеет такие свойства как идентификатор товара и его количество;

- одного или нескольких представлений, которые отображают продукт и форму заказа;

- контроллера, который координирует сделку

Вероятно, вы узнали типичный MVC-подобный шаблон. Существует большое количество вариаций MVC и бесконечный [список реализаций][2].

Тем не менее, MVC — не проблема. Настоящая проблема — отдел по продажам. Теперь они хотят оживленную корзину в правой части страницы. Она должна показывать содержимое корзины, динамически добавлять новый товар и пересчитывать стоимость доставки, налоги и общую сумму заказа.

Вы можете применить ООП в таком случае, добавив новый контроллер, унаследованный от старого. Новый расширенный контроллер может взаимодействовать с новым представлением корзины, посылая ей новый товар и обновленные значения. Тем не менее, в таком подходе есть несколько возможных проблем:

- сложность: взаимодействие требует от расширенного контроллера больше логики;

- тестирование: задача изолирования расширенного контроллера усугубляется увеличением количества заданий возложенных на него и увеличением количества зависимостей;

- получаемые побочные эффекты: обновление оживленной корзины — это побочный эффект относительно главной задачи контроллера, которая заключается в координировании сделки.

Если бы менеджеры по продажам никогда не запрашивали еще одну фичу, влияния этих проблем были бы незначительными, но мы с вами в курсе, что они никогда не перестанут просить новых фич. После нескольких итераций, иерархия наследования контроллера, вероятно станет путаницей перезаписей, условий и разветвлений кода.

Побочные эффекты для меня  — прямой сигнал к тревоге. Так уж отложилось у меня в голове, что объект, модуль или функция должны делать только одну вещь одновременно. В нашем случае контроллер должен убеждаться что пользовательские данные (идентификатор товара и размер заказа) доставлены на сервер. Координация с интерфейсом динамической корзины относительно этой задачи - побочный эффект.

Мой мозг немедленно хочет поручить координацию корзины другому компоненту. В итоге эта идея приводит к другим возможным решениям и имеет некоторые интересные преимущества:

- сложность: она уменьшается вследствие разделения на несколько отдельных компонентов;

- тестирование: меньшее количество задач и зависимостей в каждом компоненте означает, что каждый из них легче тестировать;

- и никаких побочных эффектов!

## Решение 1: события

Одним из наиболее популярных паттернов в JavaScript является «издатель-подписчик» или «наблюдатель». Компоненты запрашивают уведомление про событие («подписываются») или продуцируют события («публикуют»). События обычно состоят из строк определенных пространств имен и соответствующим им данным.

В нашем примере с интернет-магазином, контроллер может публиковать события, вроде «cart.add-item», вместе с данными заказанного товара. Другой компонент —  контроллер корзины, например, может подписываться на событие «cart.add-item» и использовать его для управления корзиной на странице.

Популярность этого паттерна — хорошее доказательство его работоспособности. Мы успешно поручили новому компоненту задачу координирования корзины на странице и инкапсулировали задачу коммуникации между контроллерами на механизм публикации-подписки. Тем не менее, шаблон «подписчик-обозреватель» имеет свои недостатки, такие как:

- больше зависимостей: внедрение механизма «издатель-подписчик» добавляет другую зависимость к компонентам. Тестирование теперь требует, чтобы с механизмом «издатель-подписчик» поизвращались;

- не нативная абстракция: события публикуются с помощью продуцирования строчек (с данными) в прозрачный механизм. Инструменты проверки качества кода, включая линтеры, не смогут убедиться, что события приходят к нужным получателям а средства отладки не смогут отслеживать вызовы от издателей к подписчикам.

## Решение 2: АОП

Что такое АОП? [Аспектно-ориентированное программирование][3] (AOP) — это способ не-агрессивного изменения или дополнения поведения методов и функций (в том числе конструкторов). Добавленное поведение называют «советом», его можно добавлять до, после или вместо функции, где следует применить совет.

В JavaScrit АОП особенно легок, так как совет инкапсулируется в функциях, а функции являются объектами первого уровня(first-class objects - никогда не встречал такого термина, помогите перевести). Возможно, вы уже использовали АОП, даже не подозревая этого. Следующий код — очень простая форма АОП:
 
    var origMethod = someObject.method;
    someObject.method = function myAdvice () {
        // сделать что-то до выполнения
        var result = origMethod.apply(this, arguments);
        // сделать что-то после выполнения
        return result;
    };
 
В этом примере функция замены myAdvise советует первоначальному методу. Такой тип совета называется советом «вокруг» («arround» advice), потому как он полностью окружает первоначальный метод и может добавлять поведение по всему методу. Вы можете переделать его в форму, которая добавляла бы новое поведение только перед или после первоначального метода.

На самом деле, вы можете легко создать свою собственную вспомогательную функцию для добавления советов «до» и «после».
 
    function before (f, advice) {
        return function () {
            advice.apply(this, arguments);
            return f.apply(this, arguments);
        };
    }
    function after (f, advice) {
        return function () {
            var result = f.apply(this, arguments);
            advice.call(this, result);
            return result;
        };
    }
 
АОП имеет несколько применений. В функциональных языках, вроде JavaScript, одним из них может быть не-агрессивное связывание компонентов. Попробуем использовать эти простые вспомогательные функции в нашем примере с интернет-магазином, чтобы посмотреть как это работает.

Во-первых, предположим, что в нашем контроллере есть метод `saveItem`, который отвечает при пользовательском клике на кнопке «Добавить в корзину» и отправляет детали заказа на сервер. Давайте также предположим, что у контроллера интерфейса корзины есть метод `addItem`, который принимает детали товара. Мы можем запрограммировать взаимодействие между этими двумя контроллерами с помощью следующего кода:
 
    controller.saveItem = after(controller.saveItem, function () {
        onscreenCart.addItem(this.product);
    });
 
Что в этом коде хорошего, так это немногим большая декларативность и очевидность происходящего.

Давайте немного расширим наш сценарий. Как известно, запрос к серверу будет асинхронным, что потребует небольшого интервала между нажатиям пользователя на кнопку «Добавить в корзину» и ответа сервера с результатом. В этом интервале, отдел по продажам хотел бы показывать счетчик в интерфейсе корзины (ну еще бы!).

Если метод `saveItem` контроллера возвращает `promise`, мы можем использовать его для получения времени, когда нужно спрятать счетчик или удалить товар, если сервер отклонит запрос.
 
    controller.saveItem = after(controller.saveItem, function (promise) {
        var product = this.product;
        onscreenCart.showSpinner(); // показать счетчик
        onscreenCart.addItem(product);
        promise.then(
            function () {
                onscreenCart.hideSpinner(); // спрятать счетчик
            },
            function (error) {
                onscreenCart.removeItem(product); // удалить товар
            }
        );
    });
 
Этот код действительно легко тестировать. `onScreenCart` не имеет зависимостей с `controller` или механизмом публикации-подписки. Равно как и `controller` не имеет зависимостей ни с `onScreenCart`,  ни с механизмом публикации-подписки. Вышеуказанная функция совета может быть легко протестирована так как не взаимодействуем ни с чем кроме `onScreenCart`.

## Больше информации

АОП имеет много других применений и должна быть частью инструментария каждого JavaScript разработчика.

Хотите опробовать АОП? Изучите этот великолепный набор ресурсов про АОП и его реализаций.

- [AOP in 50 LOC][4]

- [cujoJS][5] – [meld][6] ([документация][7])

- [Dojo][8] – dojo/aspect

- [Flight][9] – flight/advice

- [Hooker][10]

- [dcl][11]

[1]: http://ru.wikipedia.org/wiki/Design_Patterns
[2]: http://todomvc.com/
[3]: http://ru.wikipedia.org/wiki/Аспектно-ориентированное_программирование
[4]: https://github.com/briancavalier/aop-jsconf-2013/blob/master/src/aop-simple.js
[5]: http://cujojs.com/
[6]: https://github.com/cujojs/meld
[7]: https://github.com/cujojs/meld/blob/master/docs/TOC.md
[8]: http://dojotoolkit.org/
[9]: http://twitter.github.io/flight/
[10]: https://github.com/cowboy/javascript-hooker
[11]: https://github.com/uhop/dcl