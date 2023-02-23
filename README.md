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

ANONYMOUS BLOCKS
----------------
EX1:-INPUT EMPNO AND GET EMPNAME,SALRY:-

DECLARE 
V_ENO NUMBER;
V_ENM VARCHAR2(20);
v_sal number(7,20);
BEGIN
V_ENO:=&EMP_NO;
SELECT EMP_NAME,SALARY INTO V_ENO  FROM EMP1 WHERE EMPNO=V_ENO;
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
SELECT HIREDATE INTO V_HIRDT FROM EMP1 WHERE EMPNO=V_ENO;
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
SELECT HIREDATE INTO V_HIRDT FROM EMP1  WHERE EMPNO=V_ENO;
V_EXP:=(SYSDATE-V_HIRDT)/365;
DBMS_OUTPUT.PUT_LINE('EXPERIANCE IN '||V_EXP||'YEARS');
IF V_EXP>45 THEN
DELETE FROM EMP WHERE EMPNO=V_ENO;
END IF;
END;
/
EX4:-WAP TO INPUT EMPNO AND INCREMENT SALARY  BY SPECIFIC AMOUNT AND AFTER INCEREMENTIF SALARY EXCEDED 5000 THEN CANCEL THE INCEREMENT:- 

DECLARE 
V_ENO NUMBER(4);
V_AMT NUMER(6);
V_SAL NUMBER(7,2);
BEGIN
V_ENO := &EMPNO;
V_AMT := &AMOUNT;
UPDATE EMP1 SET SALARY=SALARY+V_AMT WHERE EMPNO=V_ENO;
SELECT SALARY INTO V_SAL FROM EMP1 WHERE EMPNO=V_ENO;
IF V_SAL > 5000 THEN
ROLLBACK;
ELSE COMMIT;
END IF;
END;
/
EX5:-WAP TO INPUT EMPNO AND INCREMENT EMPLOYEE SALRY AS FOLLOWS
         IF JOB=CLERK INCREMENT SAL BY 10%
                 SALSAMAN              15%
                 MANAGER               20%
                 OTHERS                05%
   DECLARE 
   V_ENO NUMBER(5);
   V_EJOB VARCHAR2(100);
   V_PCT NUMBER(3);
   BEGIN
   V_ENO:=&EMPNO;
   SELECT JOB INTO V_EJOB FROM EMP1 WHRE EMPNO=V_ENO;
   IF V_EJOB='CLERK' THEN
   V_PCT=10;
   ENSIF V_EJOB:='SALSEMAN' THEN
   V_PCT:=15;
   ELSIF V_EJOB:='MANAGER' THEN
   V_PCT:=20;
   ELSE
   V_PCT:=5;
   END IF;
   UPDATE EMP1 SET SALARY=SALARY+(SALARY*V_PCT)/100) WHERE EMPNO=V_ENO;
   COMMIT;
   END;
   /
   
   REFERENCE TYPES:-
   -----------------
   
   1.%TYPE
   2.%ROWTYPE
   
   %TYPE:-
   --------
   =>USED TO REFER DATATYPE OF A COLUMN
   =>WHAREVER DATATYPE AND SIZE DECLARED FOR ENAME COLUMN THE SAME DATATYPE ANS        SIZE TO VARIABLE V_ENAME.
     EX:-V_ENAME EMP1.EMP_NAME%TYPE;
 
 %ROWTYPE:-
   ------------
   =>Used to refer row datatype.
   ex:- e emp.emp_name%type;
   =>a row from emp table can be assigned to variable "e".
   select * from emp where empno=****;
   =>From the rowtype variable induivedual field values are accese by using          rowtypevariable.fieldname
  
   Example:-
   
   DECLARE
   V_eno emp.emp_no%TYPE;
   r emp%ROWTYPE;
   begin
   V_ENO :=&empno;
   select * into r from emp where empno=v_eno;
  -- dbms.output.put_line(r);Error:
   dbms.output.put_line(r.emp_name||r.salary);
   end;
   
   Example:-
   =>wap to input sno and calculate total,avg,result and insert into result          table;
   DECLARE
   V_Sno  STUDENT.SNO%TYPE;
   S STUDENT.%TYPE;
   R RESULT.%TYPE;
   BEGIN
   V_Sno:=$SNO;
   SELECT * INTO S FROM STUDENT WHERE V_Sno=SNO;
   R.TOTAL:=S.S1+S.S2+S.S3;
   R.AVG:=R.TOTAL/3;
   IF S1>=35 AND S2>=35 AND S3>=35 THEN
   R.RESULT:='PASS';
   ELSE
   R.RESULT:='FAIL';
   END IF;
   INSERT INTO RESULT VALUES(V_Sno,R.TOTAL,R.AVG,R.RESULT);
   COMMIT;
   END;
   /
   
  
   
 
    
   
   
