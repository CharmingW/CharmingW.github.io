1. 查找最晚入职员工的所有信息
```
CREATE TABLE `employees` (
`emp_no` int(11) NOT NULL,
`birth_date` date NOT NULL,
`first_name` varchar(14) NOT NULL,
`last_name` varchar(16) NOT NULL,
`gender` char(1) NOT NULL,
`hire_date` date NOT NULL,
PRIMARY KEY (`emp_no`));
```
答案
```
select * from employees
order by hire_date desc limit 0,1
```

2. 查找入职员工时间排名倒数第三的员工所有信息
```
CREATE TABLE `employees` (
`emp_no` int(11) NOT NULL,
`birth_date` date NOT NULL,
`first_name` varchar(14) NOT NULL,
`last_name` varchar(16) NOT NULL,
`gender` char(1) NOT NULL,
`hire_date` date NOT NULL,
PRIMARY KEY (`emp_no`));
```
答案
```
select * from employees
order by hire_date desc limit 2,1
```

3. 查找各个部门当前(to_date='9999-01-01')领导当前薪水详情以及其对应部门编号dept_no
```
CREATE TABLE `dept_manager` (
`dept_no` char(4) NOT NULL,
`emp_no` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`dept_no`));
CREATE TABLE `salaries` (
`emp_no` int(11) NOT NULL,
`salary` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`from_date`));
```
答案
```
select salaries.*, dept_manager.dept_no
from salaries, dept_manager
where dept_manager.emp_no = salaries.emp_no
and dept_manager.to_date = '9999-01-01'
and salaries.to_date = '9999-01-01'
```

4. 查找所有已经分配部门的员工的last_name和first_name
```
CREATE TABLE `dept_emp` (
`emp_no` int(11) NOT NULL,
`dept_no` char(4) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`dept_no`));
CREATE TABLE `employees` (
`emp_no` int(11) NOT NULL,
`birth_date` date NOT NULL,
`first_name` varchar(14) NOT NULL,
`last_name` varchar(16) NOT NULL,
`gender` char(1) NOT NULL,
`hire_date` date NOT NULL,
PRIMARY KEY (`emp_no`));
```
答案
```
select employees.last_name, first_name, dept_emp.dept_no
from dept_emp
inner join employees
where dept_emp.emp_no = employees.emp_no;
```

5. 查找所有员工的last_name和first_name以及对应部门编号dept_no，也包括展示没有分配具体部门的员工
```
CREATE TABLE `dept_emp` (
`emp_no` int(11) NOT NULL,
`dept_no` char(4) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`dept_no`));
CREATE TABLE `employees` (
`emp_no` int(11) NOT NULL,
`birth_date` date NOT NULL,
`first_name` varchar(14) NOT NULL,
`last_name` varchar(16) NOT NULL,
`gender` char(1) NOT NULL,
`hire_date` date NOT NULL,
PRIMARY KEY (`emp_no`));
```
答案
```
select employees.last_name, employees.first_name, dept_emp.dept_no
from employees
left join dept_emp
on employees.emp_no = dept_emp.emp_no
```

6. 查找所有员工入职时候的薪水情况，给出emp_no以及salary， 并按照emp_no进行逆序
```
CREATE TABLE `employees` (
`emp_no` int(11) NOT NULL,
`birth_date` date NOT NULL,
`first_name` varchar(14) NOT NULL,
`last_name` varchar(16) NOT NULL,
`gender` char(1) NOT NULL,
`hire_date` date NOT NULL,
PRIMARY KEY (`emp_no`));
CREATE TABLE `salaries` (
`emp_no` int(11) NOT NULL,
`salary` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`from_date`));
```
答案
```
select employees.emp_no, salaries.salary
from employees join salaries
on employees.emp_no = salaries.emp_no and employees.hire_date =  salaries.from_date
order by salaries.emp_no desc
```

7. 查找薪水涨幅超过15次的员工号emp_no以及其对应的涨幅次数t
```
CREATE TABLE `salaries` (
`emp_no` int(11) NOT NULL,
`salary` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`from_date`));
```
答案
```
select emp_no, count(emp_no) as t
from salaries
group by emp_no
having count(emp_no) > 15
```

8. 找出所有员工当前(to_date='9999-01-01')具体的薪水salary情况，对于相同的薪水只显示一次,并按照逆序显示
```
CREATE TABLE `salaries` (
`emp_no` int(11) NOT NULL,
`salary` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`from_date`));
```
答案
```
select salary
from salaries
where to_date = '9999-01-01'
group by salary
order by salary desc
```

9. 获取所有部门当前manager的当前薪水情况，给出dept_no, emp_no以及salary，当前表示to_date='9999-01-01'
```
CREATE TABLE `dept_manager` (
`dept_no` char(4) NOT NULL,
`emp_no` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`dept_no`));
CREATE TABLE `salaries` (
`emp_no` int(11) NOT NULL,
`salary` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`from_date`));
```
答案
```
select salary
from salaries
where to_date = '9999-01-01'
group by salary
order by salary desc
```

10. 获取所有非manager的员工emp_no
```
CREATE TABLE `dept_manager` (
`dept_no` char(4) NOT NULL,
`emp_no` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`dept_no`));
CREATE TABLE `employees` (
`emp_no` int(11) NOT NULL,
`birth_date` date NOT NULL,
`first_name` varchar(14) NOT NULL,
`last_name` varchar(16) NOT NULL,
`gender` char(1) NOT NULL,
`hire_date` date NOT NULL,
PRIMARY KEY (`emp_no`));
```
答案
```
select emp_no
from employees
where employees.emp_no not
in(select emp_no from dept_manager)
```

11. 获取所有员工当前的manager，如果当前的manager是自己的话结果不显示，当前表示to_date='9999-01-01'。
结果第一列给出当前员工的emp_no,第二列给出其manager对应的manager_no。
```
CREATE TABLE `dept_emp` (
`emp_no` int(11) NOT NULL,
`dept_no` char(4) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`dept_no`));
CREATE TABLE `dept_manager` (
`dept_no` char(4) NOT NULL,
`emp_no` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`dept_no`));
```
答案
```
select de.emp_no, dm.emp_no as manager_no
from dept_emp de
inner join dept_manager dm
on de.dept_no = dm.dept_no
where de.emp_no != dm.emp_no
and dm.to_date = '9999-01-01' and de.to_date = '9999-01-01'
```

12. 获取所有部门中当前员工薪水最高的相关信息，给出dept_no, emp_no以及其对应的salary
```
CREATE TABLE `dept_emp` (
`emp_no` int(11) NOT NULL,
`dept_no` char(4) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`dept_no`));
CREATE TABLE `salaries` (
`emp_no` int(11) NOT NULL,
`salary` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`from_date`));
```
答案
```
select de.dept_no, de.emp_no, max(s.salary)
from  dept_emp de, salaries s
where de.emp_no = s.emp_no and de.to_date = '9999-01-01' and s.to_date = '9999-01-01'
group by de.dept_no
```

13. 从titles表获取按照title进行分组，每组个数大于等于2，给出title以及对应的数目t。
```
CREATE TABLE IF NOT EXISTS "titles" (
`emp_no` int(11) NOT NULL,
`title` varchar(50) NOT NULL,
`from_date` date NOT NULL,
`to_date` date DEFAULT NULL);
```
答案
```
select title, count(title) as t
from titles
group by title
having count(title) >= 2
```

14. 从titles表获取按照title进行分组，每组个数大于等于2，给出title以及对应的数目t。
注意对于重复的emp_no进行忽略。
```
CREATE TABLE IF NOT EXISTS "titles" (
`emp_no` int(11) NOT NULL,
`title` varchar(50) NOT NULL,
`from_date` date NOT NULL,
`to_date` date DEFAULT NULL);
```
答案
```
select title, count(distinct emp_no) as t
from titles
group by title
having count(title) >= 2
```

15. 查找employees表所有emp_no为奇数，且last_name不为Mary的员工信息，并按照hire_date逆序排列
```
CREATE TABLE `employees` (
`emp_no` int(11) NOT NULL,
`birth_date` date NOT NULL,
`first_name` varchar(14) NOT NULL,
`last_name` varchar(16) NOT NULL,
`gender` char(1) NOT NULL,
`hire_date` date NOT NULL,
PRIMARY KEY (`emp_no`));
```
答案
```
select *
from employees
where emp_no % 2 = 1 and last_name != 'Mary'
order by hire_date desc
```

16. 统计出当前各个title类型对应的员工当前薪水对应的平均工资。结果给出title以及平均工资avg。
```
CREATE TABLE `salaries` (
`emp_no` int(11) NOT NULL,
`salary` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`from_date`));
CREATE TABLE IF NOT EXISTS "titles" (
`emp_no` int(11) NOT NULL,
`title` varchar(50) NOT NULL,
`from_date` date NOT NULL,
`to_date` date DEFAULT NULL);
```
答案
```
select titles.title, avg(salaries.salary)
from titles inner join salaries
on titles.emp_no = salaries.emp_no
where titles.to_date = '9999-01-01' and salaries.to_date = '9999-01-01'
group by title
```

17. 获取当前（to_date='9999-01-01'）薪水第二多的员工的emp_no以及其对应的薪水salary
```
CREATE TABLE `salaries` (
`emp_no` int(11) NOT NULL,
`salary` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`from_date`));
```
答案
```
select emp_no, salary
from salaries
where to_date = '9999-01-01'
order by salary desc
limit 1, 1
```

18. 查找当前薪水(to_date='9999-01-01')排名第二多的员工编号emp_no、薪水salary、last_name以及first_name，不准使用order by
```
CREATE TABLE `employees` (
`emp_no` int(11) NOT NULL,
`birth_date` date NOT NULL,
`first_name` varchar(14) NOT NULL,
`last_name` varchar(16) NOT NULL,
`gender` char(1) NOT NULL,
`hire_date` date NOT NULL,
PRIMARY KEY (`emp_no`));
CREATE TABLE `salaries` (
`emp_no` int(11) NOT NULL,
`salary` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`from_date`));
```
答案
```
select e.emp_no, max(s.salary), e.last_name, e.first_name
from employees e, salaries s
where e.emp_no = s.emp_no and s.to_date = '9999-01-01' and s.salary != (select max(salary) from salaries)
```

19. 查找所有员工的last_name和first_name以及对应的dept_name，也包括暂时没有分配部门的员工
```
CREATE TABLE `departments` (
`dept_no` char(4) NOT NULL,
`dept_name` varchar(40) NOT NULL,
PRIMARY KEY (`dept_no`));
CREATE TABLE `dept_emp` (
`emp_no` int(11) NOT NULL,
`dept_no` char(4) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`dept_no`));
CREATE TABLE `employees` (
`emp_no` int(11) NOT NULL,
`birth_date` date NOT NULL,
`first_name` varchar(14) NOT NULL,
`last_name` varchar(16) NOT NULL,
`gender` char(1) NOT NULL,
`hire_date` date NOT NULL,
PRIMARY KEY (`emp_no`));
```
答案
```
select e.last_name, e.first_name, d.dept_name
from employees e
left join dept_emp de on e.emp_no = de.emp_no
left join departments d on de.dept_no = d.dept_no
```

20. 查找员工编号emp_now为10001其自入职以来的薪水salary涨幅值growth
```
CREATE TABLE `salaries` (
`emp_no` int(11) NOT NULL,
`salary` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`from_date`));
```
答案
```
select (max(salary) - min(salary))
from salaries
where emp_no = 10001
```

21. 查找所有员工自入职以来的薪水涨幅情况，给出员工编号emp_noy以及其对应的薪水涨幅growth，并按照growth进行升序
```
CREATE TABLE `employees` (
`emp_no` int(11) NOT NULL,
`birth_date` date NOT NULL,
`first_name` varchar(14) NOT NULL,
`last_name` varchar(16) NOT NULL,
`gender` char(1) NOT NULL,
`hire_date` date NOT NULL,
PRIMARY KEY (`emp_no`));
CREATE TABLE `salaries` (
`emp_no` int(11) NOT NULL,
`salary` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`from_date`));
```
答案
```
select t1.emp_no, t1.salary - t2.salary as growth
from (select e.emp_no, s.salary
      from employees e, salaries s
      where e.emp_no = s.emp_no and to_date = '9999-01-01') as t1,
	   (select e.emp_no, s.salary
      from employees e, salaries s
      where e.emp_no = s.emp_no and e.hire_date = s.from_date) as t2
where t1.emp_no = t2.emp_no
order by growth
```

22. 统计各个部门对应员工涨幅的次数总和，给出部门编码dept_no、部门名称dept_name以及次数sum
```
CREATE TABLE `departments` (
`dept_no` char(4) NOT NULL,
`dept_name` varchar(40) NOT NULL,
PRIMARY KEY (`dept_no`));
CREATE TABLE `dept_emp` (
`emp_no` int(11) NOT NULL,
`dept_no` char(4) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`dept_no`));
CREATE TABLE `salaries` (
`emp_no` int(11) NOT NULL,
`salary` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`from_date`));
```
答案
```
select de.dept_no, d.dept_name, count(e.salary) as sum
from salaries e
inner join dept_emp de on e.emp_no = de.emp_no
inner join departments d on de.dept_no = d.dept_no
group by d.dept_no
```

23. 对所有员工的当前(to_date='9999-01-01')薪水按照salary进行按照1-N的排名，相同salary并列且按照emp_no升序排列
```
CREATE TABLE `salaries` (
`emp_no` int(11) NOT NULL,
`salary` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`from_date`));
```
答案
```
select s1.emp_no,s1.salary,count(distinct s2.salary) as rank
from salaries s1,salaries s2
where s1.salary<=s2.salary and s1.to_date='9999-01-01' and s2.to_date='9999-01-01'
group by s1.emp_no
order by rank
```

24. 获取所有非manager员工当前的薪水情况，给出dept_no、emp_no以及salary ，当前表示to_date='9999-01-01'
```
CREATE TABLE `dept_emp` (
`emp_no` int(11) NOT NULL,
`dept_no` char(4) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`dept_no`));
CREATE TABLE `dept_manager` (
`dept_no` char(4) NOT NULL,
`emp_no` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`dept_no`));
CREATE TABLE `employees` (
`emp_no` int(11) NOT NULL,
`birth_date` date NOT NULL,
`first_name` varchar(14) NOT NULL,
`last_name` varchar(16) NOT NULL,
`gender` char(1) NOT NULL,
`hire_date` date NOT NULL,
PRIMARY KEY (`emp_no`));
CREATE TABLE `salaries` (
`emp_no` int(11) NOT NULL,
`salary` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`from_date`));
```
答案
```
select de.dept_no, s.emp_no, s.salary
from salaries s
join employees e on e.emp_no = s.emp_no
join dept_emp de on de.emp_no = e.emp_no
where s.to_date = '9999-01-01' and de.emp_no not in (select emp_no from dept_manager)
```

25. 获取员工其当前的薪水比其manager当前薪水还高的相关信息，当前表示to_date='9999-01-01',
结果第一列给出员工的emp_no，
第二列给出其manager的manager_no，
第三列给出该员工当前的薪水emp_salary,
第四列给该员工对应的manager当前的薪水manager_salary
```
CREATE TABLE `dept_emp` (
`emp_no` int(11) NOT NULL,
`dept_no` char(4) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`dept_no`));
CREATE TABLE `dept_manager` (
`dept_no` char(4) NOT NULL,
`emp_no` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`dept_no`));
CREATE TABLE `salaries` (
`emp_no` int(11) NOT NULL,
`salary` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`from_date`));
```
答案
```
select t1.emp_no, t2.emp_no as manager_no, t1.salary as emp_salary, t2.salary as manager_salary from
	(select de.dept_no, de.emp_no, s.salary from dept_emp de
    join salaries s on de.emp_no = s.emp_no
    where de.emp_no not in (select emp_no from dept_manager) and s.to_date = '9999-01-01' and de.to_date = '9999-01-01') as t1,
	(select dm.dept_no, dm.emp_no, s.salary from dept_manager dm
    join salaries s on dm.emp_no = s.emp_no
    where s.to_date = '9999-01-01' and dm.to_date = '9999-01-01') as t2
where emp_salary > manager_salary and t1.dept_no = t2.dept_no
```
