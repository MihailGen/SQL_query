```sql
/*Напишите запрос, который подсчитает количество студентов в каждом классе. Используйте таблицы Student_in_class и Class*/

select  Class.name as class_name, count(Student_in_class.id) as student_count
from Student_in_class
left join Class on Student_in_class.class = Class.id
group by Class.id

/*Напишите запрос, который выводит количество уроков для каждого преподавателя. Используйте таблицы Teacher и Schedule*/

Select CONCAT_WS(" ", t.first_name, t.middle_name, t.last_name) as teacher, 
count(Schedule.number_pair) as lesson_count from Teacher as t
inner join Schedule on Schedule.teacher = t.id
group by t.id
order by t.last_name

/*Напишите запрос, который определит количество уникальных предметов, которые изучают студенты в каждом классе. Используйте таблицы Schedule и Class*/

select c.name as Class_name,
count(distinct sh.subject) as unique_subjects
from Schedule as sh
inner join class as c on sh.class = c.id
group by c.id

/*Напишите запрос, который определяет количество уникальных предметов, которые преподаватели преподают в каждом классе. Используйте таблицы Teacher, Schedule и Subject*/

Select CONCAT_WS(" ", t.last_name, t.first_name, t.middle_name) as teacher, 
count(distinct s.id) as unique_subjects,
sh.class as Class_ID
from Schedule as sh
inner join teacher as t on t.id = sh.teacher
join Subject as s on sh.subject = s.id
group by sh.class, t.id
order by t.last_name

/*Найдите преподавателей, которые провели больше двух уроков за весь период.
Столбец с количеством уроков назовите lesson_count. Используйте агрегатные функции и оператор HAVING*/

SELECT  
CONCAT_WS(" ", t.last_name, t.first_name, t.middle_name) as teacher,
COUNT(sh.number_pair) AS lesson_count
FROM Schedule as sh
inner join Teacher as t on sh.teacher = t.id
GROUP BY t.id
HAVING COUNT(sh.number_pair) > 2
order by lesson_count;


/*Напишите запрос, который выводит расписание для всех классов на 2019-09-01. Включите информацию о времени начала и окончания пары, имени преподавателя и названии предмета*/

select c.name AS class_name,
s.name AS subject_name,
CONCAT_WS(" ", t.first_name, t.middle_name, t.last_name) as teacher,
DATE_FORMAT(sh.date, '%d.%m.%Y') as date,
tp.start_pair,
tp.end_pair
from Schedule AS sh
INNER JOIN Teacher AS t ON sh.teacher = t.id
INNER JOIN Class AS c ON sh.class = c.id
INNER JOIN Timepair AS tp ON sh.number_pair = tp.id
INNER JOIN Subject AS s on sh.subject = s.id
where sh.date = '2019-09-01'
order by c.id

/*Напишите запрос, который определит максимальное количество уроков, проведенных для каждого класса в один день. Используйте таблицы Schedule и Class*/

WITH count_pair AS (
SELECT
COUNT(number_pair) as p,
c.name as class_name
from Schedule as sh
INNER JOIN Class AS c on sh.class=c.id
GROUP BY c.id, date
ORDER BY c.name )

SELECT MAX(p), class_name
FROM count_pair
GROUP BY class_name

/*Напишите запрос, который определяет максимальное количество уроков, проведенных каждым преподавателем в один день. Используйте таблицы Teacher и Schedule*/

WITH count_pair AS (
SELECT
COUNT(number_pair) as p,
CONCAT_WS(" ", t.last_name, t.first_name, t.middle_name) as teacher
from Schedule as sh
INNER JOIN Teacher AS t on sh.teacher=t.id
GROUP BY t.id, date
ORDER BY t.last_name )

SELECT MAX(p) as max_pairs_count, teacher
FROM count_pair
GROUP BY teacher
ORDER BY max_pairs_count DESC,
teacher

/*Напишите запрос, который выводит список студентов, зарегистрированных в 11 A (english), и информацию обо всех парах, которые у них есть в расписании. Используйте общие табличные выражения (CTE).*/

WITH class_students AS (
SELECT
GROUP_CONCAT(" ", s.last_name," ", s.first_name) as students_11_A_class,
c.id as id
FROM Student_in_class as sc
INNER JOIN Student as s on sc.student = s.id
INNER JOIN Class as c on c.id = sc.class
WHERE c.name = '11 A'
GROUP By c.id
)

select 
s.name,
tp.start_pair,
tp.end_pair,
DATE_FORMAT(sh.date, '%d.%m.%Y') as date,
class_students.students_11_A_class,
CONCAT_WS(" ", t.first_name, t.middle_name, t.last_name) as teacher
FROM Schedule as sh
INNER JOIN Timepair as tp on sh.number_pair = tp.id
INNER JOIN class_students on class_students.id = sh.class
INNER JOIN Subject as s on sh.subject = s.id
INNER JOIN Class as c on sh.class = c.id
INNER JOIN Teacher as t on sh.teacher = t.id
WHERE c.name = '11 A'
ORDER by sh.date, tp.start_pair

/*Напишите запрос, который выводит расписание всех уроков, проводимых в 11 B (english). Используйте подзапрос для фильтрации данных*/

select 
DATE_FORMAT(sh.date, '%d.%m.%Y') as date,
s.name,
tp.start_pair,
tp.end_pair,
CONCAT_WS(" ", t.first_name, t.middle_name, t.last_name) as teacher,
sh.classroom
FROM Schedule as sh
INNER JOIN Timepair as tp on sh.number_pair = tp.id
INNER JOIN Subject as s on sh.subject = s.id
INNER JOIN Teacher as t on sh.teacher = t.id
WHERE sh.class = (select id FROM Class where name = '11 A')
ORDER by sh.date, tp.start_pair
```
