![[Pasted image 20231114115457.png]]
```sql
select sno,cno,
case score
when score BETWEEN 0 AND 59
then "不及格"
when score BETWEEN 60 and 69
then "及格"
when score BETWEEN 70 and 79
then "良"
else "优"
end as "成绩"
from score;
```