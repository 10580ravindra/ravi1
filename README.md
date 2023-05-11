HTML
----------------
<!DOCTYPE html>
<html>
<head>
<title>Page Title</title>
<style>
		a{
			text-decoration: none;
			color: green;
			font-size: 20px;
			padding: 8px;
		}
		a:hover{
			color: blue;
		}
	         h1{
		text-align: center;
		}
	</style>
</head>
 <--<frameset cols="50%,50%">  
    <frame src="https://www.javatpoint.com/html-table">  
    <frame src="https://www.javatpoint.com/css-table">
    <noframes>Sorry! Your browser does not support frames. </noframes>    
   </frameset> -->
<body>

<h1 ALIGN="CENTER">WELCOME TO ZOI HOSPITALS</h1>
<marquee DIRECTION="RIGHT" BEHAVIOR="SCROLL">This is an example of html marquee </marquee> 
<marquee width="100%" behavior="alternate" bgcolor="pink"> This is an example of html marquee </MARQUEE>
<p>Display a gauge:</p>  
<meter value="50" min="1" max="100">30 out of 100</meter><br>  
<meter value="0.9">80%</meter> 
<h3>Navigate Buttons</h3>
<nav>
 	<a href="#">Home</a> |
 	<a href="#">Courses</a> |
 	<a href="#">About-us</a> |
 	<a href="#">Contact-us</a> |
 </nav>
 
 <b>Button(Script)
 <button name="button" value="OK" type="button" onclick="hello()">Click Here</button>  
<script>  
function hello(){  
alert("hello javatpoint user");  
}  
</script>  
<svg width="100" height="100">  
   <circle cx="50" cy="50" r="40" stroke="yellow" stroke-width="4" fill="red" />  
  </svg>   
  <svg height="210" width="500">  
  <polygon points="100,10 40,198 190,78 10,78 160,198"  
  style="fill:red;stroke:yellow;stroke-width:5;fill-rule:nonzero;" />  
</svg>   
<svg width="200" height="100">  
   <rect width="50px" height="30px" stroke="yellow" stroke-width="4" fill="red" />  
  <div class="MagicScroll" data-options="items: [[200,1],[400,2],[600,4]]">
    <img src="example1.jpg">
    <img src="example2.jpg">
    ...
</div>
<div class="MagicScroll" data-options="orientation: vertical; height: 240;">
    <img src="example1.jpg">
    <img src="example2.jpg">
    ...
</div>

<div class="MagicScroll" data-options="orientation: vertical; items: 3;">
    <img src="example1.jpg">
    <img src="example2.jpg">
    ...
</div>
    ...
</div>
<div class="MagicScroll" data-options="autoplay: 1000; step: 1; mode: carousel; height: 275;">
    <img src="example1.jpg">
    <img src="example2.jpg">
    ...
</div>
</body>
</html>

TOMORROW UPDATE QUERIES
--------------------------

SELECT moneyreceipt.NAME,moneyreceipt.OPREGNO ,RECEIPTNO,admin.getagefromdobmoney(moneyreceipt.moneyreceiptid) AGE,IPINFO.IPNO FROM IP.moneyreceipt ,
IP.IPINFO WHERE ipinfo.PATID (+)= MONEYRECEIPT.PATID AND IPINFO.IPNO='1720284' ;

SELECT H.NAME,H.IPNO,H.casesheetnumber, nvl((H.ageyrs|| ' Y '|| H.agemonth|| ' M '|| H.ageday|| ' D '), '') age FROM IP.IPINFO H, ip.dischargetemplate   d         where                                   
    H.patid = d.patid AND H.IPNO='1720284';
ER REGISTRATION AND OP PROCEDURE BILLING DAILY DATA
--------------------------------------------------------
select * from ip.erinitial where entrydate in('06/05/2023','07/05/2023') and locid='Loc00001';

select * from OTS1.nursinghdr_opd h,OTS1.nursingdtl_opd where h.prescid=d.prescid and locid='Loc00001'


ACTIVE EMPLOYEE LIST:-
  ---------------------

SELECT   NVL(E.EMPNM,D.USERNM)NAME ,D.EMPID,(Select subDeptNm  from payroll.subDepts where sDeptID=E.SDEPTID) DEPTNM,
D.USERNM ,D.USERID,D.ADDRESS,D.PHONE,D.PAYROLLEMPID FROM ADMIN.USERS D ,PAYROLL.EMPLOYEEINFO E WHERE D.ACTIVE=0 AND E.EMPID(+)=D.PAYROLLEMPID ORDER BY NAME;

payslip:-

--------
select DISTINCT * from ( (SELECT DISTINCT
    d.leaveopenings   open_balance,
    d.leaveavaild     leaves_availed,
    d.leavesavailable leaves_available,
    d.leavecode
FROM
    payroll.empsalhdr_month h,
    payroll.empleave_dtl    d
WHERE
        h.salid = d.fkid
    AND leavecode IN (
        SELECT
            leavecode
        FROM
            payroll.empleave_dtl
    ))
unpivot (no_of_leaves FOR LEAVE_TYPE IN(open_balance,leaves_availed,leaves_available))PIVOT (
    SUM ( no_of_leaves )
    FOR leavecode
    IN ( 'CL',  'SL', 'EL', 'CO' )
))order by LEAVE_TYPE desc;


salary procesessing leave information code:-
----------------------------------------------
SELECT leavenm txtleavenm,    leaveid txtleavenm_id,  leavecode txtLeave_Code_searchid,  leavenm txtleave_type_searchid, leaveid txtleave_type_searchid_id, leavecode txtLeave_Code_searchid_id, (                 CASE WHEN el = 1 AND wvf_getdaysdoj('$txtEmployee_No_searchid.id') > 180 THEN 
               ( nvl(admin.wvf_getavailableleaves('$txtEmployee_No_searchid.id', leaveid), 0) + ( substr(LEAVESCALCULATOR1('$txtEmployee_No_searchid.id',last_day(to_date('01'||'/'||'$selSalary_for_the_Month_Of.value'||'/'||'$selSalary_For_Year.value','dd/mm/yyyy'))),1, 2) * 1.5 ) )               WHEN sl = 1 AND wvf_getdaysdoj('$txtEmployee_No_searchid.id') > 180 THEN 
               ( nvl(admin.wvf_getavailableleaves('$txtEmployee_No_searchid.id', leaveid), 0) + ( substr(LEAVESCALCULATOR1('$txtEmployee_No_searchid.id',last_day(to_date('01'||'/'||'$selSalary_for_the_Month_Of.value'||'/'||'$selSalary_For_Year.value','dd/mm/yyyy'))),1, 2) * 1 ) )               WHEN cl = 1 THEN 
               ( nvl(admin.wvf_getavailableleaves('$txtEmployee_No_searchid.id', leaveid), 0) + ( substr(LEAVESCALCULATORCL_date('$txtEmployee_No_searchid.id',last_day(to_date('01'||'/'||'$selSalary_for_the_Month_Of.value'||'/'||'$selSalary_For_Year.value','dd/mm/yyyy'))),1, 2) * 1 ) )              when leaveid='LV000006' THEN 
 NVL(GET_COMPOFF('$txtEmployee_No_searchid.id',last_day(to_date('01'||'/'||'$selSalary_for_the_Month_Of.value'||'/'||'$selSalary_For_Year.value','dd/mm/yyyy'))),0)+NVL(WVF_GET_COMPOFF_EMP('$txtEmployee_No_searchid.id',last_day(to_date('01'||'/'||'$selSalary_for_the_Month_Of.value'||'/'||'$selSalary_For_Year.value','dd/mm/yyyy'))),0)               ELSE admin.wvf_getavailableleaves('$txtEmployee_No_searchid.id', leaveid) END ) txtleave_openings,                nvl(USED_LEAVES('$txtEmployee_No_searchid.id', leaveid,last_day(to_date('01'||'/'||'$selSalary_for_the_Month_Of.value'||'/'||'$selSalary_For_Year.value','dd/mm/yyyy'))), 0) txtleaves_availed,                  ( nvl(( CASE WHEN el = 1 AND wvf_getdaysdoj('$txtEmployee_No_searchid.id') > 180 THEN 
               (nvl(admin.wvf_getavailableleaves('$txtEmployee_No_searchid.id', leaveid), 0) +(substr(LEAVESCALCULATOR1('$txtEmployee_No_searchid.id',last_day(to_date('01'||'/'||'$selSalary_for_the_Month_Of.value'||'/'||'$selSalary_For_Year.value','dd/mm/yyyy'))),1, 2) * 1.5))WHEN sl = 1 AND wvf_getdaysdoj('$txtEmployee_No_searchid.id') > 180 THEN 
                (nvl(admin.wvf_getavailableleaves('$txtEmployee_No_searchid.id', leaveid), 0) +(substr(LEAVESCALCULATOR1('$txtEmployee_No_searchid.id',last_day(to_date('01'||'/'||'$selSalary_for_the_Month_Of.value'||'/'||'$selSalary_For_Year.value','dd/mm/yyyy'))),1, 2) * 1))WHEN cl = 1 THEN 
               (nvl(admin.wvf_getavailableleaves('$txtEmployee_No_searchid.id', leaveid), 0) +(substr(LEAVESCALCULATORCL_date('$txtEmployee_No_searchid.id',last_day(to_date('01'||'/'||'$selSalary_for_the_Month_Of.value'||'/'||'$selSalary_For_Year.value','dd/mm/yyyy'))),1, 2) * 1))               when leaveid='LV000006' THEN 
 NVL(GET_COMPOFF('$txtEmployee_No_searchid.id',last_day(to_date('01'||'/'||'$selSalary_for_the_Month_Of.value'||'/'||'$selSalary_For_Year.value','dd/mm/yyyy'))),0)+NVL(WVF_GET_COMPOFF_EMP('$txtEmployee_No_searchid.id',last_day(to_date('01'||'/'||'$selSalary_for_the_Month_Of.value'||'/'||'$selSalary_For_Year.value','dd/mm/yyyy'))),0)               ELSE admin.wvf_getavailableleaves('$txtEmployee_No_searchid.id', leaveid) END ), 0)                - nvl(USED_LEAVES('$txtEmployee_No_searchid.id', leaveid,last_day(to_date('01'||'/'||'$selSalary_for_the_Month_Of.value'||'/'||'$selSalary_For_Year.value','dd/mm/yyyy'))), 0) ) txtleaves_available,               0 txtno_of_leaves,               sl txtsl,                  nvl(unpaid, 0)txtlopleave,                  nvl(leave_app_pending('$txtEmployee_No_searchid.id', leaveid), 0) txtl_pending_ap FROM payroll.leavemaster_new WHERE nvl(both, 0) IN ( 1, 2 )     AND nvl(active, 0) = 1      AND leaveid IN ( SELECT leaveid FROM payroll.leave_eligible WHERE empid = '$txtEmployee_No_searchid.id' )ORDER BY disporder 

--purchase indent query 2nd screen--
  opd billing 1
   union all SELECT  DISTINCT 0 selServiceType_id,nvl(d.doctor_name,r.docid)  txtUnit_or_Doctor_searchid_id,(select doctornm from admin.doctorinfo where doctorinfo.docid=nvl(d.doctor_name,r.docid)) txtUnit_or_Doctor_searchid,case when f.serviceid= 'AS006217' then to_char(WVF_OP_PROCEDURE_CON_RATE( nvl(f.cons,0),'$txtOrgDocFee.value','0',nvl(d.doctor_name,r.docid) ,  '$txtsocid.value' ,'$txtMR_No_searchid.value' ,'$selPatType.id',$selTiming.id ,'$txtdisflg.value','$txtSelect_Source_searchid.id')) else (nvl(admin.WVF_OP_GETSERVICERATESOCIVYStd( f.serviceid,nvl(admin.WVF_OP_GETSOCIDIVYStd( '$txtSelect_Source_searchid.id','$loc','$selType.id',0),'$soc'), '$txtPaytype.value' ),0)) end txtTRate,f.serviceid txtService_name_searchid_id,  servicenm txtService_name_searchid , f.scode txtService_code ,  '0' txtGridDisc ,  d.slno,1 txtQty,D.PRESCID txtbUBBLEPRESCID,nvl(f.cons,0) txtSTflg,nvl(f.docflg,0) txtDrflg , nvl(f.fixedrateflg,0) txtsrfixedvar FROM OTS1.NURSINGHDR_OPD r,OTS1.NURSINGdtl_OPD  d,  IP.SERVICESVF f WHERE r.prescid =d.prescid   AND d.complaintid =f.SERVICEID   AND NVL(d.ISBILLING,0)        =0   AND r.PRESCDaTe =to_date(SYSDATE)   AND r.mrno                    ='$txtMR_No_searchid.value' 
  
  
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
    
 opd consultation area wise query:-
 --------------------------------------
    
    SELECT DISTINCT
    h.name,opmrno,getservicename(serviceid)service_name,
   (select areanm from admin.citys c where c.areaid=h.cityid)area,
    wvf_getdocnm(d.docid) doctorname,
    billdt consul_date,h.address
FROM
    ip.availserviceshdr h,
    ip.availservicesdtl d
WHERE
        h.billid = d.billid 
        AND STFLG=1
    AND h.locid = 'Loc00001'
    AND billdt >= TO_DATE('01/08/2022', 'dd/mm/yyyy')
    AND billdt <= TO_DATE('28/02/2023', 'dd/mm/yyyy')
    and nvl(opflg,0)=1
     ORDER BY billdt desc ;
    
    
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
   
  LOOPS:-
  ----------
  =>wap to print nos from 1 to 20?
  
  DECLARE
  X NUMBER(3):=1;
  BEGIN
  LOOP DBMS_OUTPUT.PUT_LINE(X);
  X:=X+1;
  EXIT WHEN X>20;
  ENDLOOP;
  END;
   
 USING WHILE LOOP:-
 ------------------
 DECLARE
  X NUMBER(3):=1;
  BEGIN
  WHILE(X<=20)
  LOOP DBMS_OUTPUT.PUT_LINE(X);
  X:=X+1;
  END LOOP;
  END;
  USING FOR LOOP:=
  ------------------
  DECLARE
  X NUMBER(3):=1;
  BEGIN
 FOR X IN 1..20
  LOOP DBMS_OUTPUT.PUT_LINE(X);
  END LOOP;
  END;
 =>PRINT 2023 CALENDER DATE AND DAY
 DECLARE 
 D1 DATE;
 D2 DATE;
 BEGIN
 D1:='1-JAN-2023';
 D2:='31-DEC-2023';
 WHILE(D1<D2)
 DBMS_OUTPUT.PUT_LINE(D1||' '||TO_CHAR(D1,'DAY'));
 D1:=D1+1;
 END LOOP;
 END;
 /
 
 =>PRINT SUNDAYS 2023
 
 DECLARE 
 D1 DATE;
 D2 DATE;
 BEGIN
 D1:='01-JAN-23';
 D2:='31-DEC-23';
 D1:=NEXT_DAY(D1-1,'SUNDAY'); 
 WHILE(D1<=D2)
   LOOP
 DBMS_OUTPUT.PUT_LINE(D1||' '||TO_CHAR(D1,'DAY'));
 D1:=D1+7;
 END LOOP;
 END;
 /
   CURSORS 🚱
   ----------
   DECLERATION CURSOR:-
   EX:- cursor <name> is select statement;
   cursor c1 select * from emp;
   Opening Cursor:=
   ------------------
   open <cursor-name>
   open c1;
   1.select stmts submitted to oracle
   2.oracle executes the query and data returned by query is copied to instance variable
   3.cursor c1 points the memory.
   
   Fetching records from cursor:-
   ---------------------------------
   =>"FETCH" stmts is used to featch records from cursor.
   EX:-FETCH c1 into a,b,c......;
   =>a fetch statms fetches one row at atime .So we write fetch inside the loop only.
   cursor attributes:-
   -------------------
   1 %found:-
   TRUE =>if fetch successful
   FALSE => if fetch unsucessful
   2 %notfound:-
   True => if fetch uncessful
   False => if fetch unsuccessful
   3 %rowcount:-
   => returns no of rows fetched successfully.
   4 %isopen:-
   TRUE => if cursor is opened
   FALSE => if cursor is not opened
   
   
   => wap to print all the employee names and salaries ?
   
   DECLARE
   cursor c1 is select emp_name,emp_no,salary from emp;
   v_ename emp.emp_name%Type;
   v_eno emp.emp_no%Type;
   v_sal emp.salary%Type;
   BEGIN 
  open c1;
  LOOP
  fetch c1 into v_ename,v_eno,v_sal;
  exit when c1%notfound;
  dbms_output.put_line(v_ename||' '||v_eno||' '||v_sal);
  END LOOP;
  close c1;
  end;
   
  FOR LOOP CURSOR/CURSOR FOR LOOP:-
  -----------------------------------
   =>Adv of for loop cursor is oening cursor,fetching records from cursor and closing cursor is not required and all these operations performed implicitily. 
   
   => for loop executes no of times  depends on no of rows in cursor
   => every time forloop executes a record is fetched from cursor and assigned to variable "r".
   => loop variable "r" is also declared implicitly as rowtype.
   
   
   DECLARE
   cursor c1 is select emp_name,salary from emp;
   begin
  for r in c1
  loop
  dbms_output.put_line(r.emp_name||' '||r.salary);
  end loop;
  end;
  
  => wap to calculate all the student total,avg,result and insert into result table ?
  
  DECLARE
  cursor c1 is select * from student;
  r result%rowtype;
  begin
  for s in c1
  loop
  r.total:=s.s1+s.s2+s.s3;
  r.avg:=r.total/3;
 r.result:='pass'
 else
 r.result:='fail';
 end if;
 insert into result values(s.sno,r.total,r.avg,r.result);
 end loop;
 commit;
 end;
