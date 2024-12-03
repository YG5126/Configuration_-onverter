# Общее описание
### Задание
Разработать инструмент командной строки для учебного конфигурационного языка, синтаксис которого приведен далее. Этот инструмент преобразует текст из 
входного формата в выходной. Синтаксические ошибки выявляются с выдачей сообщений. 

Входной текст на **учебном конфигурационном языке** принимается из стандартного ввода. Выходной текст на **языке yaml** попадает в файл, путь к которому задан ключом командной строки. 
### Синтаксис учебного конфигурационного языка
Массивы:
```
#( значение значение значение ... )
```
Имена:
```
[_a-zA-Z]+
```
Значения:
```
• Числа.
• Строки.
• Массивы.
```
Строки:
```
[[Это строка]]
```
Объявление константы на этапе трансляции:
```
set имя = значение
```
Вычисление константного выражения на этапе трансляции (префиксная
форма), пример:
```
$+ имя 1$
```
Результатом вычисления константного выражения является значение.
Для константных вычислений определены операции и функции:
```
1. Сложение.
2. chr().
3. mod().
```
Все конструкции учебного конфигурационного языка (с учетом их возможной вложенности) должны быть покрыты тестами. Необходимо показать 2 примера описания конфигураций из разных предметных областей.
# Реализованный функционал
### grammar
Определяет синтаксические правила:

 - Комментарии
 - Структуры конфигурации
 - Объявление констант
 - Массивы
 - Поддерживаемые операции

### ConfigTransformer
 - Преобразует синтаксическое дерево в словарь Python
 - Управляет объявлениями и вычислениями констант
 - Поддерживает операции:

	• Сложение констант
	• Преобразование числа в символ
	• Вычисление модуля
### parse_config
 - Конвертирует входной текст конфигурации в YAML
 - Обрабатывает синтаксические ошибки
 - Записывает результат в указанный файл
### main
Выполняется парсинг аргументов командной строки. Считывается конфигурация на учебном конфигурационном языке из стандартного потока ввода. Вызываются поочередно функции получения строки на языке yaml и получения отформатированной строки на языке yaml. Отформатированная строка записывается в файл, указанный ключом командной строки.
# Сборка и запуск проекта
1. Загрузка репозитория на компьютер
```
git clone https://github.com/YG5126/Configuration_Converter
```
2. Преход в директорию репозитория
```
cd Configuration_Converter
```
3. Установка библиотеки парсинга Lark
```
pip install PyYAML
```
4. Запустить main.py с указанием имени yaml файла
```
py main.py <имя_файла.yaml>
```
5. Ввод конфигурации в командную строку. Для завершения ввода использовать ctrl + Z
# Примеры работы программы
### Конфигурация сетевой службы
**Входные данные:**
```
***> Network Service Configuration
set PORT = 8080
set MAX_CONNECTIONS = $+ MAX_CONNECTIONS 10$

NetworkService {
    host = "localhost",
    port = $+ PORT 0$,
    max_connections = MAX_CONNECTIONS,
    allowed_protocols = #(http https),
    description = [[Primary Web Service]]
}
```
**Выходные данные (XML):**
```
<?xml version="1.0" encoding="utf-8"?>
<database>
	<database type="dict">
		<host type="int">19216801</host>
		<port type="int">5432</port>
		<max_connections type="int">100</max_connections>
		<connection_timeout type="int">30</connection_timeout>
	</database>
</database>
```
### Конфигурация игрового персонажа
**Входные данные:**
```
***> RPG Character Configuration
set BASE_HEALTH = 100
set STRENGTH_MODIFIER = $+ BASE_HEALTH 50$

GameCharacter {
    name = [[Warrior]],
    health = BASE_HEALTH,
    strength = $+ STRENGTH_MODIFIER 0$,
    skills = #(sword_fighting archery),
    race = [[Human]],
    special_ability = [[Critical Strike]]
}
```
**Выходные данные (XML):**
```
<?xml version="1.0" encoding="utf-8"?>
<web_config>
	<webserver type="dict">
		<hostname type="int">127001</hostname>
		<port type="int">8080</port>
		<threads type="int">8</threads>
		<routes type="dict">
			<home type="int">1</home>
			<login type="int">2</login>
			<logout type="int">3</logout>
		</routes>
	</webserver>
</web_config>
```
### Конфигурация научного эксперимента
**Входные данные:**
```
***> Experimental Setup
set SAMPLE_SIZE = 50
set TEMPERATURE = $+ 20 $mod(SAMPLE_SIZE, 10)$

ExperimentConfig {
    experiment_type = [[Chemical Reaction]],
    sample_count = SAMPLE_SIZE,
    temperature = $+ 20 $chr(TEMPERATURE)$$,
    chemicals = #(hydrogen oxygen nitrogen),
    safety_level = [[High]],
    notes = [[Precise measurements required]]
}
```
**Выходные данные (XML):**
```
<?xml version="1.0" encoding="utf-8"?>
<monitoring_config>
	<monitoring type="dict">
		<interval type="int">15</interval>
		<retention_days type="int">365</retention_days>
		<services type="dict">
			<first type="int">1</first>
			<second type="int">2</second>
			<third type="int">3</third>
			<fourth type="int">4</fourth>
		</services>
	</monitoring>
</monitoring_config>
```
# Результаты тестирования
### Тест простой конфигурации
```
def test_simple_config(self):
	input_text = ('config {\n'
                      '\tsmth = 13\n'
                      '}\n')
        expected_output = '<config><smth type="int">13</smth></config>'
        self.assertEqual(parse_config(input_text), expected_output)
```
### Тест словаря
```
def test_dict(self):
	input_text = ('config {\n'
                      '\tnames = struct {\n'
                      '\t\tnikita = 1,\n'
                      '\t\tartem = 2\n'
                      '\t}\n'
                      '}\n')
        expected_output = '<config><names type="dict"><nikita type="int">1</nikita><artem type="int">2</artem></names></config>'
        self.assertEqual(parse_config(input_text), expected_output)
```
### Тест константы
```
def test_constant(self):
        input_text = ('def x = 5\n'
                      'config {\n'
                      '\tsmth = [x]\n'
                      '}\n')
        expected_output = '<config><smth type="int">5</smth></config>'
        self.assertEqual(parse_config(input_text), expected_output)
```
### Тест комментария
```
def test_comment(self):
        input_text = ('*> comment\n'
                      'config {\n'
                      '\tsmth = 10\n'
                      '}\n')
        expected_output = '<config><smth type="int">10</smth></config>'
        self.assertEqual(parse_config(input_text), expected_output)
```
### Тест синтаксической ошибки
```
def test_syntax_error(self):
        input_text = ('def x = 5\n'
                      'config {\n'
                      '\tsmth = x\n'
                      '}\n')
        result = parse_config(input_text)
        assert "Unexpected Characters" in result
```
### Тест использования необъявленной константы
```
def test_undefined_constant_error(self):
        input_text = ('config {\n'
                      '\tsmth = [undefined_constant]\n'
                      '}\n')
        result = parse_config(input_text)
        assert "В конфигурации использована неизвестная константа по имени undefined_constant" in result
```
### Тест повторного объявления константы
```
def test_duplicate_constant_error(self):
        input_text = ('def x = 5\n'
                      'def x = 10\n'
                      'config {\n'
                      '\tsmth = [x]\n'
                      '}\n')
        result = parse_config(input_text)
        assert "Константа x уже объявлена" in result
```
### Тест форматированного вывода
```
def test_output_xml(self):
        input_text = ('*> Test comment\n'
                      'def int = 10\n'
                      'main {\n'
                      '\tcombo = struct {\n'
                      '\t\tnumber = 19216801,\n'
                      '\tmax_connections = [int]\n'
                      '\t}\n'
                      '}\n')
        expected_output = ('<?xml version="1.0" encoding="utf-8"?>\n'
                        '<main>\n'
                        '\t<combo type="dict">\n'
                        '\t\t<number type="int">19216801</number>\n'
                        '\t\t<max_connections type="int">10</max_connections>\n'
                        '\t</combo>\n'
                        '</main>\n')
        self.assertEqual(pretty_print_xml(parse_config(input_text)), expected_output)
```
![image](https://github.com/user-attachments/assets/4f180366-0af7-44be-81a5-15770bd454dc)
