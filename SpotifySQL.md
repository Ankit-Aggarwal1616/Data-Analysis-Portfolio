# HR Analytics
Table Structure (employee):

![image](https://github.com/Ankit-Aggarwal1616/Data-Portfolio/assets/161187358/9cedf701-da19-4e61-8dc9-10034e1c3ab6)




**1**. How many employees are currently active?

```sql
SELECT
    COUNT(*) as NumEmployees
FROM
    employee
WHERE
    EmploymentStatus = 'Active';
```
Output:

![image](https://github.com/Ankit-Aggarwal1616/Data-Portfolio/assets/161187358/c67915e7-df33-40bd-a10c-9c51dd759d6c)

**2**. How many employees have been terminated for any reason?

```sql
SELECT
    COUNT(*) as NumEmployees
FROM
    employee
WHERE
    EmploymentStatus in ('Voluntarily Terminated', 'Terminated for Cause');
```
Output:

![image](https://github.com/Ankit-Aggarwal1616/Data-Portfolio/assets/161187358/b9ff0ea9-cdf1-4fc2-a7de-a18088245ecb)

**3**. What are the most common methods of application?
```sql
SELECT
    RecruitmentSource,
    COUNT(*) as NumEmployees
FROM
    employee
GROUP BY
    RecruitmentSource
ORDER BY
    NumEmployees DESC;
```
Output:

![image](https://github.com/Ankit-Aggarwal1616/Data-Portfolio/assets/161187358/61638572-0ec0-4468-b03a-30ab323cecaa)

**4**. What are the average salary and employee satisfaction categorized by sex for active employees?
```sql
SELECT
    Sex,
    COUNT(*) as NumEmployees,
    ROUND(AVG(Salary), 2) AS AvgSalary,
    ROUND(AVG(EmpSatisfaction), 2) AS AvgEmpSatisfaction
FROM
    employee
WHERE
    EmploymentStatus = 'Active'
GROUP BY
    Sex
ORDER BY
    NumEmployees DESC;
```
Output:

![image](https://github.com/Ankit-Aggarwal1616/Data-Portfolio/assets/161187358/68347d00-0873-4f91-a472-2df2c019d122)

**5**. What are the average salary and employee satisfaction categorized by race for active employees?
```sql
SELECT
    RaceDesc,
    COUNT(*) as NumEmployees,
    ROUND(AVG(Salary), 2) AS AvgSalary,
    ROUND(AVG(EmpSatisfaction), 2) AS AvgEmpSatisfaction
FROM
    employee
WHERE
    EmploymentStatus = 'Active'
GROUP BY
    RaceDesc
ORDER BY
    NumEmployees DESC;
```
Output:

![image](https://github.com/Ankit-Aggarwal1616/Data-Portfolio/assets/161187358/222c5e7b-068c-4b24-92c6-b8238ad10d10)

**6**. Which managers show the highest average employee engagement among active employees?

```sql
SELECT
    ManagerName,
    ROUND(AVG(EngagementSurvey), 2) as AverageEmployeeEngagement,
    COUNT(*) as NumEmployees
FROM
    employee
WHERE
    EmploymentStatus = 'Active'
GROUP BY
    ManagerName
ORDER BY
    AverageEmployeeEngagement DESC;
```
Output:

![image](https://github.com/Ankit-Aggarwal1616/Data-Portfolio/assets/161187358/b7569f1c-b89c-49eb-9e83-fbb88aed70c1)

**7**. What is the average salary and employee satisfaction per department for active employees?
```sql
SELECT
    department,
    COUNT(*) AS NumEmployees,
    ROUND(AVG(salary), 2) AS AverageSalary,
    ROUND(AVG(EmpSatisfaction), 2) AS AverageEmployeeSatisfaction
FROM
    employee
WHERE
    EmploymentStatus = 'Active'
GROUP BY
    department
ORDER BY
    AverageSalary DESC;
```
Output:

![image](https://github.com/Ankit-Aggarwal1616/Data-Portfolio/assets/161187358/67442dfc-5082-4fc2-9687-5d64623e82da)

**8**. What is the average satisfaction for employees who were terminated, and what were the reasons for their departure?
```sql
SELECT
    TermReason,
    COUNT(*) AS NumEmployees,
    ROUND(AVG(EmpSatisfaction), 2) AS AvgEmpSatisfaction
FROM
    employee
WHERE
    TermReason != 'N/A-StillEmployed'
GROUP BY
    TermReason
ORDER BY
    NumEmployees DESC;
```
Output:

![image](https://github.com/Ankit-Aggarwal1616/Data-Portfolio/assets/161187358/6a14b97a-84f6-4bc2-b443-2adee79867f4)

> [!NOTE]
> Among the top eight reasons for termination, individuals who departed citing 'unhappiness' experienced the lowest average employee satisfaction.

**9**. Are employees who left for reasons related to higher pay justified? Compare their salaries with the average for their department.

```sql
SELECT
    e1.Employee_Name,
    e1.Salary,
    e1.Department,
    e1.TermReason,
    AVG(e2.Salary) AS AvgDepartmentSalary
FROM
    employee e1
JOIN
    employee e2 ON e1.Department = e2.Department
WHERE
    e1.TermReason = 'more money'
    AND e2.EmploymentStatus = 'Active'
GROUP BY
    e1.Employee_Name, e1.Salary, e1.Department, e1.TermReason;

```
Output:

![image](https://github.com/Ankit-Aggarwal1616/Data-Portfolio/assets/161187358/2b199180-5e8d-4652-9a16-07b571c16030)

> [!NOTE]
> Six out of the eleven employees who chose to leave in pursuit of higher pay had salaries below the average for their respective departments.

**10**. Who deserves a raise? Identify employees with salaries below the average for their department, high performance scores, and low absences among active employees.
```sql
SELECT
    Employee_Name,
    Salary,
    PerformanceScore,
    Absences,
    EmploymentStatus,
    Department,
    (SELECT AVG(Salary) FROM employee e1 WHERE e1.Department = employee.Department AND e1.EmploymentStatus = 'Active') AS AvgDepartmentSalary
FROM
    employee
WHERE
    Salary < (SELECT AVG(Salary) FROM employee e1 WHERE e1.Department = employee.Department AND e1.EmploymentStatus = 'Active') AND
    PerformanceScore = 'Exceeds' AND
    Absences < (SELECT AVG(Absences) FROM employee WHERE EmploymentStatus = 'Active') AND
    EmploymentStatus = 'Active';
```
Output:

![image](https://github.com/Ankit-Aggarwal1616/Data-Portfolio/assets/161187358/90f1edf3-bdfb-41a7-a2a6-8442fea7c66c)


**11**. Who should be let go? Identify employees with performance scores indicating improvement needs, above-average absences, and salaries higher than the departmental average among active employees.
```sql
WITH EmployeeAnalysis AS (
    SELECT
        Employee_Name,
        Salary,
        PerformanceScore,
        Absences,
        EmploymentStatus,
        Department,
        AVG(Salary) OVER (PARTITION BY Department) AS AvgDepartmentSalary
    FROM
        employee
    WHERE
        EmploymentStatus = 'Active'
)

SELECT
    Employee_Name,
    Salary,
    PerformanceScore,
    Absences,
    EmploymentStatus,
    Department,
    AvgDepartmentSalary
FROM
    EmployeeAnalysis
WHERE
    PerformanceScore IN ('Needs Improvement', 'PIP') AND
    Absences > (SELECT AVG(Absences) FROM EmployeeAnalysis) AND
    Salary > AvgDepartmentSalary;
```
Output:

![image](https://github.com/Ankit-Aggarwal1616/Data-Portfolio/assets/161187358/c8fd570a-07c1-4a62-869a-f8a0132edbf0)

**12**. Who should HR consider rehiring? Identify employees who left voluntarily with exceptional performance scores, below-average absences, and above-average engagement survey scores.
```
SELECT
    Employee_Name,
    Salary,
    PerformanceScore,
    Absences,
    EngagementSurvey,
    EmploymentStatus,
    TermReason,
    Department
FROM
    employee
WHERE
    EmploymentStatus = 'Voluntarily Terminated' AND
    PerformanceScore = 'Exceeds' AND
    Absences < (SELECT AVG(Absences) FROM employee) AND
    EngagementSurvey > (SELECT AVG(EngagementSurvey) FROM employee);
```
Output:

![image](https://github.com/Ankit-Aggarwal1616/Data-Portfolio/assets/161187358/57b2ac89-0c77-488c-9dcc-f7cd2efd50ac)

--
> [!NOTE]
> Jordan Winthrop, who retired, should be excluded from consideration. On the other hand, George Johnson is a candidate worth considering, especially since he earned less than the average salary for his department.
