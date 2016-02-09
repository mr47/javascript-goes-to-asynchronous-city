# Путешествие JavaScript в город асинхронности

Благодаря усилиям [TC39][1] (организации, ответственной за стандартизацию JavaScript
(ECMAScript, если быть точным)) JavaScript прошёл достаточно длинный путь со
своих первых версий и стал современным, широко используемым языком.

Одна из частей стандарта ECMAScript, получившая огромное количество улучшений —
асинхронность ([что такое асинхронное программирование?][2]). Кстати, в новом
браузере Edge из Windows 10 мы добавили поддержку современных возможностей для
работы с асинхронным кодом. Можете посмотреть полный список нововведений:

[https://dev.modern.ie/platform/changelog/desktop/10547/?compareWith=10532][3]

Список довольно длинный, но в этой статье мы сфокусируемся на асинхронных
функциях (ES2016 Async Functions) и рассмотрим их подробно, а также поймём, как
они могут улучшить ваш код.

## Первая остановка: ECMAScript 5 – город коллбеков

Вся работа с асинхронностью в ECMAScript 5 (и предыдущих версиях) сводилась
к коллбекам. Проиллюстрирую это простой задачей, с которой вы, вероятно,
сталкиваетесь довольно часто: выполнение XHR-запроса.

    var displayDiv = document.getElementById("displayDiv");

    // Объявляем функцию для обработки результата
    var processJSON = function (json) {
        var result = JSON.parse(json);

        result.collection.forEach(function(card) {
            var div = document.createElement("div");
            div.innerHTML = card.name + " cost is " + card.price;

            displayDiv.appendChild(div);
        });
    }

    // Объявляем функцию для отображения ошибок
    var displayError = function(error) {
        displayDiv.innerHTML = error;
    }

    // Создаём и настраиваем XHR-объект
    var xhr = new XMLHttpRequest();

    xhr.open('GET', "cards.json");

    // Объявляем коллбеки, которые будут вызваны XHR-объектом
    xhr.onload = function(){
        if (xhr.status === 200) {
            processJSON(xhr.response);
        }
    }

    xhr.onerror = function() {
        displayError("Не получилось загрузить RSS");
    }

    // Отправляем запрос
    xhr.send();

Опытным JavaScript-разработчикам знаком подобный код, потому что XHR-коллбеки
используются повсеместно. Принцип довольно прост: разработчик создаёт
XHR-запрос, а затем задаёт коллбек нужному XHR-объекту.

С другой стороны, коллбеки трудны для понимания из-за внутренней природы
асинхронного кода:

![image][4]

Всё станет ещё сложнее, если внутри коллбека создать ещё один асинхронный
запрос — вы столкнётесь с так называемым «[адом коллбеков][5]».

## Вторая остановка: ECMAScript 6 – Город промисов

ECMAScript 6 набирает обороты, и Edge на данный момент
[лидирует средибраузеров][6] с 88% поддержкой возможностей нового стандарта.

Помимо прочих прекрасных улучшений, в ECMAScript 6 стандартизировали
использование *промисов* (ранее известных как *futures*).

Согласно определению с [MDN][7], промис — это объект, используемый для
выполнения отложенных и асинхронных вычислений. Промис представляет операцию,
которая ещё не была завершена, но её завершение ожидается в будущем. Промисы
позволяют организовать выполнение асинхронных операций так, будто эти операции
синхронны. Как раз то что нужно для нашего примера с XHR-запросом.

> Промисы существовали ещё до появления стандарта ES6, но благодаря их
> стандартизаци больше не придётся использовать сторонние библиотеки — теперь
> промисы поддерживаются современными браузерами «из коробки».

Давайте перепишем предыдущий пример используя промисы и посмотрим, как они
делают код более читабельным и поддерживаемым:

    var displayDiv = document.getElementById("displayDiv");

    // Создаём функцию, возвращающую промис
    function getJsonAsync(url) {
        // Промисам требуется две функции: одна для обработки успешного 
        // завершения операции, вторая для обработки ошибок
        return new Promise(function (resolve, reject) {
            var xhr = new XMLHttpRequest();

            xhr.open('GET', url);

            xhr.onload = () => {
                if (xhr.status === 200) {
                    // Запрос выполнился успешно
                    resolve(xhr.response);
                } else {
                    // Запрос не выполнился, обрабатываем ошибку
                    reject("Не получилось загрузить RSS");
                }
            }

            xhr.onerror = () => {
                // Запрос не выполнился, обрабатываем ошибку
                reject("Не получилось загрузить RSS");
            };

            xhr.send();
        });
    }

    // Функция возвращает промис, поэтому мы можем создавать цепочку
    // вызовов .then и .catch
    getJsonAsync("cards.json").then(json => {
        var result = JSON.parse(json);

        result.collection.forEach(card => {
            var div = document.createElement("div");
            div.innerHTML = `${card.name} cost is ${card.price}`;

            displayDiv.appendChild(div);
        });
    }).catch(error => {
        displayDiv.innerHTML = error;
    });

Наверняка вы заметили множество улучшений. Рассмотрим их подробнее.

### Создание промиса

Чтобы «промисифицировать» (простите, я француз, так что позвольте мне
придумывать новые слова) XHR, вам нужно создать новый объект *Promise*:

![image][8]

### Использование промиса

После создания промиса можно строить элегантные цепочки асинхронных вызовов:

![image][9]

Таким образом код (с точки зрения пользователя) выглядит так:

*   Получаем промис **(1)**
*   Начинаем цепочку с обработки успешного завершения операции **(2 and 3)**
*   Заканчиваем цепочку обработкой ошибки **(4)** (аналогично try/catch)

Что действительно интересно — можно запросто создавать цепочки промисов,
вызывая `.then().then()` и так далее.

*Примечание: вы могли заметить, что помимо промисов я использовал также
некоторый [синтаксический сахар][10] из ECMAScript 6 вроде
[строковых шаблонов][11] или [стрелочных функций][12].*

## Конечная: ECMAScript 7 — город асинхронности

Итак, мы достигли точки назначения! Мы практически в [будущем][13], но благодаря команде разработчиков браузера Edge в одной из его последних сборок появилась поддержка асинхронных функций!

Асинхронные функции — это синтаксический сахар, на уровне языка улучшающий
работу с асинхронным кодом.

> В основе асинхронных функций лежат современные возможности ECMAScript 6 вроде
> генераторов. На самом деле генераторы вместе с промисами позволяют
> сымитировать поведение асинхронных функций, но для потребуется написать
> намного больше кода.

Нам не требуется изменять функцию, генерирующую промис, потому что асинхронные
функции работают напрямую с промисами.

Нам нужно лишь изменить вызов функции:

    // Создаём анонимную асинхронную функцию
    (async function() {
        try {
            // Просто дожидаемся результата из промиса
            var json = await getJsonAsync("cards.json");
            var result = JSON.parse(json);

            result.collection.forEach(card => {
                var div = document.createElement("div");
                div.innerHTML = `${card.name} cost is ${card.price}`;

                displayDiv.appendChild(div);
            });
        } catch (e) {
            displayDiv.innerHTML = e;
        }
    })();

Вот где начинается магия. Этот код написан в синхронном стиле с прямым порядком
выполнения:

![image][14]

Впечатляет, не так ли?

И да, вы можее использовать асинхронные функции вместе со стрелочными функциями или методами классов.
And the good news is that you can even use async functions with arrow functions

## Двигаемся дальше

Если вам интересно узнать, как мы реализовали поддержку асинхронных функций в движке Chakra, то прочитайте [анонс][15] в официальном блоке Edge.

Вы также можете отслеживать поддержку браузерами новых возможностей ECMAScript 6 и 7 на сайте [Kangax][6]. Ну и не стесняйтесь проверять наш [план][16] по реализации новых возможностей в браузере Edge!

Не стесняйтесь давать нам обратную связь и голосовать за те возможности, которые вы в первую очередь хотите увидеть в браузере Edge:

![image][17]

Спасибо за чтение! Будем рады услышать ваши отзывы и идеи.

 [1]: http://www.ecma-international.org/memento/TC39-M.htm
 [2]: https://msdn.microsoft.com/en-us/library/hh191443.aspx

 [3]: https://dev.modern.ie/platform/changelog/desktop/10547/?compareWith=10532 "https://dev.modern.ie/platform/changelog/desktop/10547/?compareWith=10532"
 [4]: img/8816.image_5F00_thumb_5F00_35C48EA5.png "image"
 [5]: http://callbackhell.com/
 [6]: http://kangax.github.io/compat-table/es6/

 [7]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise
 [8]: img/6305.image_5F00_thumb_5F00_56493DA6.png "image"
 [9]: img/6012.image_5F00_thumb_5F00_0D8FC4E2.png "image"
 [10]: https://en.wikipedia.org/wiki/Syntactic_sugar
 [11]: http://tc39wiki.calculist.org/es6/template-strings/
 [12]: http://tc39wiki.calculist.org/es6/arrow-functions/
 [13]: http://kangax.github.io/compat-table/es7/
 [14]: img/1665.image_5F00_thumb_5F00_0BBF6F1B.png "image"

 [15]: http://blogs.windows.com/msedgedev/2015/09/30/asynchronous-code-gets-easier-with-es2016-async-function-support-in-chakra-and-microsoft-edge/ "http://blogs.windows.com/msedgedev/2015/09/30/asynchronous-code-gets-easier-with-es2016-async-function-support-in-chakra-and-microsoft-edge/"
 [16]: https://dev.modern.ie/platform/status/javascript/
 [17]: img/1348.image_5F00_thumb_5F00_49FA5F60.png "image"
 [18]: http://www.twitter.com/deltakosh
