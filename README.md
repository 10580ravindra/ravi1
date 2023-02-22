  --purchase indent query 2nd screen--
  
  
  
  
  SELECT
    wvf_getmednmstd(medicinename)                                     medicion_name,
     getsubdeptnm(indentfromsubdept)indentfromsubdept,
    indentno                                                          indentno_notot,
    to_date(to_char(h.entrydate, 'dd/mm/yyyy ')|| to_char(h.entrytime, 'hh:mi:ss am'),'dd/mm/yyyy hh:mi:ss am')indentdt,
    to_char(issuedt, 'dd/mm/yyyy hh:mi:ss')                           issue_date_notot,
    to_number(reqqty)                                                 indent_qty_notot,
    to_number(issueqty)                                               issue_qty_notot,
    to_char(nvl(reqqty,0) - nvl(issueqty,0))                                        diff_qty_notot,
  floor ((issuedt - to_date(to_char(h.entrydate, 'dd/mm/yyyy ')|| to_char(h.entrytime, 'hh:mi:ss am'),'dd/mm/yyyy hh:mi:ss am '))*24 )||' hr'diff
FROM
    inventory.insert_pharmasaveindentdtl d,
    inventory.pharmasaveindents          h
WHERE
        d.fkid = h.pkid
        and reqqty!=issueqty
    AND entrydate >= TO_DATE('01/02/2023', 'dd/mm/yyyy')
    AND entrydate <= TO_DATE('20/02/2023', 'dd/mm/yyyy')
    AND indenttosubdept = 'SD000126'
    AND indentno in(1482,1468,1521,1407,1511);
    
    
MERGE COMMAND 📧
-------------
SYNTAX:-MERGE INTO DESTINATION TABLE
USING SOURCE TABLE ON (CONDION)
WHEN MATHED THEN 
STATEMENT
WHEN NOT MATCHED THEN
STATEMENT;

EX:-
merge into emp2 e1
using emp e on (e1.empid=e.empid)
when matched then
update set e1.entrytime=e.entrytime
when not matched then
insert values(e.empid,e.empnm,e.salary,e.deptid,e.deptnm,e.entrydate,e.entrytime);


PL/SQL 🚱
--------
EX1:-INPUT EMPNO AND GET EMPNAME,SALRY:-

DECLARE 
V_ENO NUMBER;
V_ENM VARCHAR2(20);
v_sal number(7,20);
BEGIN
V_ENO:=&EMP_NO;
SELECT EMP_NAME,SALARY INTO V_ENO WHERE EMPNO=V_ENO;
DBMS_OUTPUT.PUT_LINE(V_ENM||' '||V_SAL);
END;
/

EX2:-INPUT EMPNO AND CALCULATE EXPERIANCE OF EMPLOYEE:-

DECLARE 
V_ENO NUMBER;
V_HIRDT DATE;
V_EXP NUMBER;
BEGIN
V_ENO:=&EMP_NO;
SELECT HIREDATE INTO V_HIRDT WHERE EMPNO=V_ENO;
V_EXP:=(SYSDATE-V_HIRDT)/365;
DBMS_OUTPUT.PUT_LINE('EXPERIANCE IN'||V_EXP||'YEARS');
END;
/

EX3:-INPUT EMPNO AND CALUCULATE EXPERIANCE AND DELETE EPERIANCE GREATERTHAN 45 YEARS:-

DECLARE 
V_ENO NUMBER;
V_HIRDT DATE;
V_EXP NUMBER;
BEGIN
V_ENO:=&EMP_NO;
SELECT HIREDATE INTO V_HIRDT WHERE EMPNO=V_ENO;
V_EXP:=(SYSDATE-V_HIRDT)/365;
DBMS_OUTPUT.PUT_LINE('EXPERIANCE IN '||V_EXP||'YEARS');
IF V_EXP>45 THEN
DELETE FROM EMP WHERE EMPNO=V_ENO;
END IF;
END;
/
EX4:-
