~~~
student(sid, sname)
course(cid, cname, cplace, ctime, tid)
teacher(tid, tname)
sc(sid, cid)

select cid from sc where sc.sid=?


select a.*,teacher.tname
from (select * from course where cid in (select cid from sc where sc.sid=?)) as a, teacher
where teacher.tid = a.tid
~~~

