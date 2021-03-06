<p align="center">МИНИСТЕРСТВО НАУКИ  И ВЫСШЕГО ОБРАЗОВАНИЯ РОССИЙСКОЙ ФЕДЕРАЦИИ  <br/>
Федеральное государственное автономное образовательное учреждение высшего образования  <br/>
"КРЫМСКИЙ ФЕДЕРАЛЬНЫЙ УНИВЕРСИТЕТ им. В. И. ВЕРНАДСКОГО"  <br/>
ФИЗИКО-ТЕХНИЧЕСКИЙ ИНСТИТУТ  <br/>
Кафедра компьютерной инженерии и моделирования<br/></p>
<br/>

### <p align="center">Отчёт по лабораторной работе № 6<br/> по дисциплине "Программирование"</p>
<br/>

<br/>
​Cтудента 1 курса группы ПИ-б-о-192(2)<br/>
Золотого Александра Витальевича<br/>
направления подготовки 09.03.04 "Программная инженерия"  
<br/>
<br/>

<table>
<tr><td>Научный руководитель<br/> старший преподаватель кафедры<br/> компьютерной инженерии и моделирования</td>
<td>(оценка)</td>
<td>Чабанов В.В.</td>
</tr>
</table>
<br/><br/>

<p align="center">Симферополь, 2020</p>


### Тема: Погодный информер

**Цель:**<br/>
1.Закрепить навыки разработки многофайловыx приложений;<br/>
2.Изучить способы работы с API web-сервиса;<br/>
3.Изучить процесс сериализации/десериализации данных.<br/>

**<p align="center">Ход работы:</p>**
**I. Подготовка серверной части:**<br/>
1.Скачиваем библиотеку для работы с сетью.<br/>
2.Создаем консольное приложение.<br/>
3.В папке с проектом создаем папку include, в ней папку httplib.Из папки cpp-httplib-master  копируем httplib.h в последнюю.<br/>
4.В проекте указываем папку с включаемыми файлами.<br/>
![](img/1.png)
<p align="center">Рис.1 Включение файлов в проект<br/></p>
5. Аналогично примеру получаем:

``` cpp
#include <iostream>
#include <httplib/httplib.h>
using namespace httplib;

void gen_response(const Request& req, Response& res) {
    res.set_content("Hello, World!", "text/plain");
}
int main() {
    Server svr;                    
    svr.Get("/", gen_response);    
    svr.listen("localhost", 3000); 
}
```

6.Проверяем работоспособность проекта.<br/>
![](img/2.png)
<p align="center">Рис.2 Запущенный проект<p/><br/>

**II. Подготовка к работе с сервисом openweathermap.org**<br/>
1.Регистрируемся на сайте и получаем API key.<br/>
07e1054f4a589840494cd54061b1f48d <br/>
2.Составляем и тестируем запрос в браузере: "http://api.openweathermap.org/data/2.5/forecast?id=693805&units=metric&APPID=b0e23cb34e84309d025bc41912f9bb2e". Параметр units=metric отвечает за отображение температуры в градусах цельсия.<br/>
В ответ получаем текст в формате JSON.<br/>
![](img/3.png)
<p align="center">Рисунок 3.Полученный ответ</p><br/>

**III. Подготовка клиента для получения информации от openweathermap.org**<br/>
Модифицирем код клиента, чтобы отправить составленный запрос.<br/>
```cpp
    httplib::Client cli("api.openweathermap.org", 80);
	auto result = cli.Get("/data/2.5/forecast?id=693805&units=metric&APPID=b0e23cb34e84309d025bc41912f9bb2e");   // замени APPID=b0e23cb34e84309d025bc41912f9bb2e на APPID=свой API
	res.set_content(HtmlCode, "text/html");
```

**IV. Подготовка к работе с JSON**	
1.Скачиваем библиотеку для работы с JSON.<br/>
2.Из папки json-develop\include копируем содержимое в папку include своего проекта.<br/>
3.В главный файл проекта подключаем библиотеку следующим образом: #include <nlohmann/json.hpp>.<br/>
4.Распарсиваем ответ, полученный от сервиса openweathermap.org, в JSON объект.
```cpp
   json j;
	if (result && result->status == 200) j = json::parse(result->body);
	std::string HtmlCode;
	std::ifstream stream("informer_template.html");
	getline(stream, HtmlCode, '\0');
	stream.close();
```
**V. Подготовка шаблона виджета**
1.Скачиваем шаблон виджета informer_template.html и помещаем его в папку с проектом.<br/>
2.В программе загружаем файл informer_template.html в строковую переменную.<br/>
3.Заменяем ряд элементов в виджета.<br/>
```cpp
	replace(HtmlCode, "{city.name}", j["city"]["name"].dump(), 10); // 10 так как Simferopol - 10 символов
	for (int i = 0; i < 5; i++)
	{
		int temp = j["list"][0]["dt"];
		for (int i = 0; i < 40; i++) 
		{
			if (j["list"][i]["dt"] >= temp)
			{
				replace(HtmlCode, "{list.dt}", j["list"][i]["dt_txt"].dump(), 10);  
				replace(HtmlCode, "{list.weather.icon}", j["list"][i]["weather"][0]["icon"].dump(), 3); 
				replace(HtmlCode, "{list.main.temp}", j["list"][i]["main"]["temp"].dump(), 0);
				temp += 86400;
			}
		}
	}
```
**VI. Сборка итогового проекта**
1.В функции gen_response помещаем процесс загрузки шаблона и запроса к openweathermap.org, а также заполняем шаблон данными.
2.Заполненный шаблон передаем первым параметром в функцию set_content, в качестве второго параметра передаем text/html;
3.Код итогового проекта:
```cpp
#include <iostream>
#include <httplib/httplib.h>
#include <nlohmann/json.hpp>
#include <string>

using json = nlohmann::json;

void replace(std::string& str, const std::string from, std::string  to, int key)
{
	if (key != 0) to = to.substr(1, key);
	int start_pos = str.find(from);
	if (start_pos == std::string::npos) return;
	str.replace(start_pos, from.length(), to);
}

void gen_response(const httplib::Request& req, httplib::Response& res)
{
	httplib::Client cli("api.openweathermap.org", 80);
	auto result = cli.Get("/data/2.5/forecast?id=693805&units=metric&APPID=b0e23cb34e84309d025bc41912f9bb2e");   // замени APPID=b3b3215efcb4017b119668066e83bc9e на APPID=свой API
	json j;
	if (result && result->status == 200) j = json::parse(result->body);
	std::string HtmlCode;
	std::ifstream stream("informer_template.html");
	getline(stream, HtmlCode, '\0');
	stream.close();

	replace(HtmlCode, "{city.name}", j["city"]["name"].dump(), 10); // 10 так как Simferopol - 10 символов

	for (int i = 0; i < 5; i++)
	{
		int temp = j["list"][0]["dt"];
		for (int i = 0; i < 40; i++) // 40 так как всего 40 объектов в list(5 дней каждые 3 часа, это 8 в день, всего 40)
		{
			if (j["list"][i]["dt"] >= temp)
			{
				replace(HtmlCode, "{list.dt}", j["list"][i]["dt_txt"].dump(), 10);  // 10 так как в записи  xxxx-xx-xx(дата) 10 символов
				replace(HtmlCode, "{list.weather.icon}", j["list"][i]["weather"][0]["icon"].dump(), 3); // 3 так как в записи, например 01n - 3 символа (для картинки)
				replace(HtmlCode, "{list.main.temp}", j["list"][i]["main"]["temp"].dump(), 0);
				temp += 86400;
			}
		}
	}
	res.set_content(HtmlCode, "text/html");
}

int main()
{
	httplib::Server svr;
	svr.Get("/", gen_response);
	svr.listen("localhost", 3000);
}
```
4.Тестируем работу программы.
![](img/4.png)
<p align="center">Рисунок 4.Тестирование работоспособности<br/></p>

**Вывод:** 
В ходе лабораторной работы я закрепил навыки разработки многофайловыx приложений, изучил способы работы с API web-сервиса и процесс сериализации/десериализации данных.






















