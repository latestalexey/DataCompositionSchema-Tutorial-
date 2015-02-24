# DataCompositionSchema-Tutorial-
Dedicated to beginners in 1C programming who want to master the basics of "Data composition schema" object

Language: Russian

Данный туториал будет полезен тем, кто уже имеет навыки работы с СКД и хотел бы далее совершенствовать свои знания и навыки. Материал, который содержится в туториале ,  описывает процесс создания и настройки СКД посредством встроенного языка 1С. Если вы никогда не работали с СКД советую ознакомится с ее основами: Е.Ю. Хрусталева «Разработка сложных отчетов в 1С Предприятие 8»

Итак начнем.

Для начала работы создадим:

1.	Справочник «Номенклатура» (иерархия групп и элементов) и справочник «Единицы измерения». 

2.	Перечисление «Типы цен» со значениями «Оптовая» и «Розничная».

3.	Независимый периодический регистр сведений «Цены номенклатуры»,  периодичность: в пределах дня. Измерения: номенклатура (СправочникСсылка «Номенклатура»), тип цены (ПеречислениеСсылка «Типы цен»). Ресурсы: Цена (16/2). Реквизиты: единица измерения (СправочникСсылка «Единицы измерения»).

4.	Отчет «Прайс-лист».

Над отчетом поговорим подробнее. В новом отчете добавим новую схему компоновки данных. В макетах у нас появится «ОсновнаяСхемаКомпоновкиДаннах». Откроем ее двойным кликом. На закладке «Наборы данных» добавим новый набор типа «Запрос».
Текст запроса:

ВЫБРАТЬ

	ЦеныНоменклатурыСрезПоследних.Номенклатура,
	
	ЦеныНоменклатурыСрезПоследних.Цена,
	
	ЦеныНоменклатурыСрезПоследних.БазоваяЕдиницаИзмерения КАК ЕдИзм
	
ИЗ

РегистрСведений.ЦеныНоменклатуры.СрезПоследних(, ТипЦены = &ТипЦены) КАК ЦеныНоменклатурыСрезПоследних

В параметрах добавим  параметр «ТипЦены» .

Настройки:
Конструктор настроек, поля: номенклатура, цена, единица измерения;  группировка: номенклатура(только иерархия);

Параметр «ТипЦены» включим в пользовательские настройки.

   В итоге мы имеем готовый отчет без единой строчки кода. Как мы видим  создавать отчеты с помощью конструктора СКД  довольно просто и удобно.
  Но на практике часто бывают случаи когда код все таки требуется писать. Поэтому перейдем сразу к написанию кода.

   Предположим мы хотим отображать наш прайс из формы списка нашого справочника. Создаем форму списка. Создаем команду  «ПоказатьПрайс» и в модуле формы пишем следующий код:


&НаКлиенте

Процедура  ПоказатьПрайс (Команда)

   Результат  =  ВывестиПрайс();
   
   Результат.ОтображатьСетку      =  Ложь;
   
   Результат.ОтображатьЗаголовки  =  Ложь;
   
   Результат.Показать();

КонецПроцедуры


&НаСервереБезКонтекста

Функция  ВывестиПрайс ()

     СхемаКомпоновкиДанных  =  Отчеты.ПрайсЛист.ПолучитьМакет("ОсновнаяСхемаКомпоновкиДанных");
 
      Настройки  =  СхемаКомпоновкиДанных.НастройкиПоУмолчанию;
   
      КомпоновщикМакета  =  Новый КомпоновщикМакетаКомпоновкиДанных;
      
      МакетКомпоновки   = КомпоновщикМакета.Выполнить(СхемаКомпоновкиДанных, Настройки);
 
      ПроцессорКомпоновки  =  Новый ПроцессорКомпоновкиДанных;
      
      ПроцессорКомпоновки.Инициализировать(МакетКомпоновки);
 
      ТабДок  =  Новый  ТабличныйДокумент;

     ПроцессорВывода  =  Новый ПроцессорВыводаРезультатаКомпоновкиДанныхВТабличныйДокумент;
     
     ПроцессорВывода.УстановитьДокумент(ТабДок);
     
     ПроцессорВывода.Вывести(ПроцессорКомпоновки);
  
     Возврат  ТабДок; 
     
КонецФункции

  Как видно из кода, из обработчика команды вызывается функция которая программно создает отчет на основе СКД. Результат отчета выводится в табличный документ и возвращается обратно на клиент. За основу мы берем уже готовый макет СКД : «ОсновнаяСхемаКомпоновкиДанных».

<HEAD> Отчет в СКД полностью на встроенном языке <Head/>
<BODY>
Теперь создадим полностью все с нуля.  Для этого нам понадобится создать новый отчет, а в нем создадим форму отчета.
  В событии формы «ПриСозданииНаСервере» создадим схему компоновки данных, добавим в нее набор данных и опишем в нем запрос и добавим поля набора. На закладке «Параметри» добавим параметр: «ТипЦены».
  Схему поместим во временное хранилище. Адресс хранилища запишем в реквизит формы типа: «Строка», который назовем «Адресс».

Примечание: 

   Добавляя в набор поля ресурсов мы не рассчитываем по ним итоги. Так как ресурсы в данном примере не сворачиваются агрегатной функцией «СУММА» мы будем добавлять их в поля таким же способом каким мы добавляли измерения.
<BODY/>
