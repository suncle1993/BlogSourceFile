---
abbrlink: 113432121
alias: 2017/05/10/常用SQL集锦/index.html
categories:
- 数据库
date: '2017-05-10T00:57:17'
description: ''
tags:
- SQL
- Oracle数据库
title: 常用Oracle SQL集锦
---









#### 常用dml和query开并行

```sql
--开并行
ALTER SESSION FORCE PARALLEL DML PARALLEL 16;
ALTER SESSION FORCE PARALLEL QUERY PARALLEL 16;
--关并行
ALTER SESSION DISABLE PARALLEL DML ;
ALTER SESSION DISABLE PARALLEL QUERY;
```

#### 索引开并行

```sql
drop index IDX_SB_SBZT_LRRQ;
create index IDX_SB_SBZT_LRRQ on SB_SBZT (LRRQ, CWLX_DM, SWJG_DM)
  tablespace TS_GS_SB_IDX
  pctfree 20
  initrans 2
  maxtrans 255
  storage
  (
    initial 64K
    next 1M
    minextents 1
    maxextents unlimited
  ) parallel 6;

alter index IDX_SB_SBZT_LRRQ noparallel;
```

#### 查看ASM

```sql
-- 查看asm的各个disk group是否超过80%
-- name：name of the disk group
-- total_mb：total capacity of the disk group(in megabytes)
-- free_mb：unused capacity of the disk group(in megabytes)
select t.name,t.TOTAL_MB /1024, round(100 * (t.TOTAL_MB - t.FREE_MB) / t.TOTAL_MB, 2)||'%'
  from v$asm_diskgroup t;
```

<!--more-->

#### 查看数据库表空间

```sql
--查询数据表空间情况
SELECT d.status "Status",
d.tablespace_name "Name",
d.contents "Type",
d.extent_management "Extent Management",
       TO_CHAR(NVL(a.bytes / 1024 / 1024, 0), '99,999,990.900') "Size (M)",
       TO_CHAR(NVL(a.bytes - NVL(f.bytes, 0), 0) / 1024 / 1024,
               '99999999.999') "Used (M)",
       TO_CHAR(NVL((a.bytes - NVL(f.bytes, 0)) / a.bytes * 100, 0),
               '990.00') "Used %"
  FROM sys.dba_tablespaces d,
       (select tablespace_name, sum(bytes) bytes
          from dba_data_files
         group by tablespace_name) a,
       (select tablespace_name, sum(bytes) bytes
          from dba_free_space
         group by tablespace_name) f
 WHERE d.tablespace_name = a.tablespace_name(+)
   AND d.tablespace_name = f.tablespace_name(+)
   AND NOT
        (d.extent_management like 'LOCAL' AND d.contents like 'TEMPORARY')
UNION ALL
SELECT d.status "Status",
d.tablespace_name "Name",
d.contents "Type",
d.extent_management "Extent Management",
       TO_CHAR(NVL(a.bytes / 1024 / 1024, 0), '99,999,990.900') "Size (M)",
       TO_CHAR(NVL(t.bytes, 0) / 1024 / 1024, '99999999.999') "Used (M)",
       TO_CHAR(NVL(t.bytes / a.bytes * 100, 0), '990.00') "Used %"
  FROM sys.dba_tablespaces d,
       (select tablespace_name, sum(bytes) bytes
          from dba_temp_files
         group by tablespace_name) a,
       (select tablespace_name, sum(bytes_cached) bytes
          from v$temp_extent_pool
         group by tablespace_name) t
 WHERE d.tablespace_name = a.tablespace_name(+)
   AND d.tablespace_name = t.tablespace_name(+)
   AND d.extent_management like 'LOCAL'
   AND d.contents like 'TEMPORARY'
   order by "Used %" desc;
```

#### 判断字段中是否包含小写字母

```sql
--gs_cx查询纳税人识别号这个字段中包含小写字母的并且做过工资薪金税费种认定的户数：upper函数
select count(*) from hx_dj.dj_nsrxx a,hx_dj.dj_nsrxx b,hx_rd.rd_sfzrdxxb r
where a.djxh=b.djxh
and a.djxh=r.djxh
and r.zspm_dm='101060100'
and upper(a.nsrsbh) <> a.nsrsbh
and a.nsrsbh like '3%';
```

#### 判断字段中是否包含大写字母

```sql
--gs_cx查询纳税人识别号这个字段中包含大写字母的并且做过工资薪金税费种认定的企业户数：lower函数
select count(*) from hx_dj.dj_nsrxx a,hx_dj.dj_nsrxx b,hx_rd.rd_sfzrdxxb r
where a.djxh=b.djxh
and a.djxh=r.djxh
and r.zspm_dm='101060100'
and lower(a.nsrsbh) <> a.nsrsbh
and a.nsrsbh like '3%';
```

#### Lengthb和length

```sql
select length('新奥投资基金管理（北京）有限公司') from dual;
select lengthb('新奥投资基金管理（北京）有限公司') from dual;
--utf-8：一个中文对三个字节
```

#### 按时间段统计数据量

```sql
--sb_kjgrsdsbgb 按五分钟统计数量
select tmp.newTime, count(tmp.newTime)
  from (select a.jylsh,
               to_char(a.lrrq, 'yyyy-mm-dd hh24:mi') oldTime,
               case
                 when substr(to_char(a.lrrq, 'mi'), 2, 1) < 5 then
                  to_char(lrrq, 'yyyy-mm-dd HH24') || ':' ||
                  substr(to_char(a.lrrq, 'mi'), 1, 1) || 0
                 else
                  to_char(lrrq, 'yyyy-mm-dd HH24') || ':' ||
                  substr(to_char(a.lrrq, 'mi'), 1, 1) || 5
               end as newTime
          from gs_cxtj.sb_kjgrsdsbgb a
         where a.lrrq > date '2016-11-16'
         order by newTime desc) tmp
 group by tmp.newtime
 order by tmp.newtime desc;
```

#### Cursor大数据量批量插入

```sql
declare
  cnt integer := 0;
  cursor cur_1 is
    select * from gs_cl.py_dj_nsrxx;
begin
  for icur_1 in cur_1 loop
    insert into xtxj_zdgsxx values (icur_1.DJXH,icur_1.ZGSWJ_DM,icur_1.SWJGMC,icur_1.SJTB_SJ,icur_1.YXBZ,icur_1.XGSJ);
    cnt := cnt + 1;
    if cnt >= 5000 then
      commit;
      cnt := 0;
    end if;
  end loop;
  commit;
exception
  when others then
    rollback;
end;
/
```

#### decode的使用

```sql
--不同渠道扣款数量
select decode(substr(jywysbh, 1, 6),
              'JSDS.N',
              '网厅',
              'SB0603',
              '客户端',
              '大厅') "渠道来源",
 count(case when a.lrrq>to_date(to_char(sysdate, 'yyyy-mm-dd'), 'yyyy-mm-dd')  then   djxh  end)"非0申报当天扣款量",
       count(*) "非0申报扣款数量"
  from sb_sbzt a
 where a.lrrq > date '2016-10-01'
   and a.yrkse_1 > 0
   and cwlx_dm = 'Y'
   and se = yrkse_1
 group by substr(jywysbh, 1, 6)
```

#### 分组数据中每组取前几条数据

```sql
--先得到已分组结果集
select t.nsrsbh, t.nsrmc, t.zgswj_dm, swjg.swjgmc
  from dj_nsrxx t, rd_sfzrdxxb sfz, dm_gy_swjg swjg
 where t.djxh = sfz.djxh
   and t.zgswj_dm = swjg.swjg_dm
   and t.shxydm is null
   and t.nsrzt_dm = '03'
   and sfz.zspm_dm = '101060100'
   and sfz.rdyxqz > date'2016-08-31'
   and substr(t.zgswj_dm, 1, 5) || '000000' in
       (select b.swjg_dm
          from dm_gy_swjg a, dm_gy_swjg b
         where substr(a.swjg_dm, 1, 5) || '000000' = b.swjg_dm
           and b.swjg_dm not in
               ('23200000000', '23299000000', '00000000000')
         group by b.swjg_dm)

--按税务机关分类查询两条记录
select f.ROWN 行号,
       f.djxh 登记序号,
       f.nsrsbh 纳税人识别号,
       f.nsrmc 纳税人名称,
       f.zgswj_dm 主管税务机关代码,
       f.swjgmc 主管税务机关名称,
       substr(f.zgswj_dm, 1, 5) || '000000' 地市税务局名称
  from (SELECT ROW_NUMBER() OVER(PARTITION BY substr(res.zgswj_dm, 1, 5) || '000000' ORDER BY res.zgswj_dm) as ROWN,
               res.*
          FROM （select t.djxh, t.nsrsbh, t.nsrmc, t.zgswj_dm, swjg.swjgmc
          from dj_nsrxx t, rd_sfzrdxxb sfz, dm_gy_swjg swjg
         where t.djxh = sfz.djxh
           and t.zgswj_dm = swjg.swjg_dm
           and t.shxydm is null
           and t.nsrzt_dm = '03'
           and t.nsrmc like '%公司'
           and length(t.nsrsbh) = 15
           and sfz.zspm_dm = '101060100'
           and sfz.rdyxqz > date '2016-08-31' ） res) f
 WHERE f.ROWN <= 20
 order by f.zgswj_dm
```

#### case和when的使用

```sql
select * from (select decode(substr(jywysbh, 1, 4),
                      'JSDS',
                      '网厅',
                      '',
                      '大厅',
                      '客户端') "渠道来源",
               count(case
                       when a.lrrq >
                            to_date(to_char(sysdate, 'yyyy-mm-dd'), 'yyyy-mm-dd') and
                            a.yrkse_1 > 0 and se = yrkse_1 then
                        yzpzxh
                     end) "非0申报当天扣款量",
               count(case
                       when a.lrrq >
                            to_date(to_char(sysdate, 'yyyy-mm-dd'), 'yyyy-mm-dd') and
                            se > 0 then
                        yzpzxh
                     end) " 当天非0申报量",
               count(case
                       when a.lrrq >
                            to_date(to_char(sysdate, 'yyyy-mm-dd'), 'yyyy-mm-dd') then
                        yzpzxh
                     end) " 当天申报量",
               count(case
                       when a.yrkse_1 > 0 and se = yrkse_1 then
                        yzpzxh
                     end) "非0申报本月扣款量",
               count(case
                       when se > 0 then
                        yzpzxh
                     end) "本月非0申报量",
               count(case
                       when 1 = 1 then
                        yzpzxh
                     end) "本月总申报量"
          from sb_sbzt a
         where a.lrrq >
               to_date(to_char(sysdate, 'yyyy-mm') || '01', 'yyyy-mm-dd')
           and cwlx_dm = 'Y'
         group by substr(a.jywysbh, 1, 4))
```

#### 查询性能最差和最耗时的SQL

```sql
--性能最差的SQL
select *
  from (select sql_text, disk_reads, buffer_gets, rows_processed
          from v$sqlarea b
         order by disk_reads desc)
 where rownum <= 10;
--最耗时的SQL
select *
  from (select a.SQL_TEXT, a.CPU_TIME, a.PARSING_SCHEMA_NAME
          from v$sql a
         order by cpu_time desc)
 where rownum <= 10;
```

#### 创建数据库表空间

```sql
create tablespace TS_GS_ZM_SSWSZMJLMX_2016_DATA
datafile 'D:\APP\CLG\ORADATA\ORCL\TS_GS_ZM_SSWSZMJLMX_2016_DATA.dbf'
size 50M
AUTOEXTEND ON NEXT 50M;
```

#### 数据库锁表查询及处理

```sql
--以下SQL适用于single instance
----Oracle数据库操作中，会用到锁表查询以及解锁和kill进程等操作
--(1)锁表查询的代码有以下的形式：
select count(*) from v$locked_object;
select * from v$locked_object;
--(2)查看哪个表被锁
select b.owner,b.object_name,a.session_id,a.locked_mode from v$locked_object a,dba_objects b where b.object_id = a.object_id;
--(3)查看是哪个session引起的
select b.username,b.sid,b.serial#,logon_time from v$locked_object a,v$session b where a.session_id = b.sid order by b.logon_time; 
--(4)杀掉对应进程
alter system kill session'587,295';--command下执行，其中587为sid,295为serial#.


--以下SQL适用于RAC和Single instance，查询完成之后即可kill会话或者直接在服务器上kill进程
select o.owner,
       o.object_name,
       l.locked_mode,
       s.username,
       s.sid,
       s.serial#,
       s.logon_time,
       p.spid,
       s.inst_id
  from gv$locked_object l, dba_objects o, gv$session s, gv$process p
 where l.object_id = o.object_id
   and l.session_id = s.sid
   and s.paddr = p.addr
```

#### 修改数据库密码为永不失效

```sql
select * from dba_profiles t where t.PROFILE='DEFAULT' and t.RESOURCE_NAME = 'PASSWORD_LIFE_TIME';

alter profile default limit password_life_time unlimited;
```

