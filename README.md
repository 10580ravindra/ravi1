SERVICES INSERT CODE
-----------------------------
BEGIN 
FOR 
I IN (SELECT D.SERVICERATE,H.SERVICEID FROM IP.SERVICESVF H,ADMIN.serviceratesvf_dtl D WHERE H.SERVICEID=D.MEDICINEID)
LOOP
INSERT INTO IP.servicesrtdtlsvf_COPY(SELECT I.SERVICEID ,
ROOMTYPEID , CASE WHEN ROOMTYPEID='CAT00003' THEN I.SERVICERATE *1.3  
 WHEN ROOMTYPEID='CAT00001' THEN I.SERVICERATE*1.2 
 WHEN ROOMTYPEID='CAT00004' THEN I.SERVICERATE*1.3
 WHEN ROOMTYPEID='CAT00005' THEN I.SERVICERATE *1.3
 WHEN ROOMTYPEID='CAT00000' THEN I.SERVICERATE *1.1
 WHEN ROOMTYPEID='CAT00000' THEN I.SERVICERATE *1.1
 WHEN ROOMTYPEID='CAT00009' THEN I.SERVICERATE *1.2
  WHEN ROOMTYPEID='CAT00012' THEN I.SERVICERATE *1.3
   WHEN ROOMTYPEID='CAT00013' THEN I.SERVICERATE *1.3
 WHEN ROOMTYPEID='CAT00000' THEN I.SERVICERATE END ,DOCSHAREPER ,DOCSHAREAMT ,SOCID ,TESTINOROUT ,SNO ,PRVRT ,
USERNM ,USERID ,RIID ,DENTALLABCHARGES ,RATESAI ,CHECKACT ,PCRATE ,PCVALUE ,
1 ,TILFLG ,SCODE_SER ,DEPARTMENT_SER ,COMPANYCODE ,COMPANYNM ,MODRATE ,
MODRATE4  FROM (select * from IP.servicesrtdtlsvf where socid in ('IND00001','IND00015','IND00053')AND SERVICEID='AS007045'));
END LOOP;
END;
ROL AND ROQ UPDATE
------------------------
insert into INVENTORY.dmedmast (mednm,medid,subdeptid,rol,roq,MAXQTY,userid,usernm,MEDCODE,potency,active,genericnm,schedule,unit,
unitsize,unit2,mtypenm,mtypeid,manid,catid,subcatid,invmethod,purexp,saleloose,mitemid)
(select  h.SERVICENM,h.MEDICINEID,h.SDEPT,h.SERVICERATE,h.CODE,h.MXQUTY,'IU001567','10580RAVINDRA',d.MEDCODE,d.potency,d.active,d.genericid,
d.schedule,d.unit,d.unitsize,unit2,d.mtypeid,d.mtypeid,d.manid,d.catid,d.subcatid,d.invmethod,d.purexp,d.saleloose,d.medid
from admin.serviceratesvf_dtl h,
INVENTORY.medmast d where  MEDICINEID=medid );

update admin.serviceratesvf_dtl set MEDICINEID =( select distinct medid from inventory.medmast where upper(mednm)=upper(SERVICENM) and rownum=1);


SELECT SERVICENM,SERVICERATE ROL,CODE ROQ,MXQUTY,GETSUBDEPTNM(SDEPT)SUBDEPTNM FROM serviceratesvf_dtl WHERE medicineid NOT IN (select NVL(MEDID,'A') from  INVENTORY.dmedmast WHERE MEDID=MEDICINEID) ORDER BY SDEPT ASC 


night audit
-------------
select * from (select sum(rcamt)a,sourceid,(select sum(paid)paid from tcvhdr1403202414032024 where locid='L0000002'AND PAID>0 and billid=sourceid
AND BILLID LIKE'%M0%')amt from mrc1403202414032024 where locid='L0000002' AND RCAMT>0 
AND SOURCEID LIKE'%M0%' group by sourceid ORDER BY SOURCEID DESC)where a!=amt  ;




Absenties
----------------
select count(*) from (select EMPNO,EMPNM,GETEMPDESIGNATIONNM(EMPID) DESIGNATIONNM,EMPDATE Absent_date,'Absent' STATUS  FRom 
ATTENDNACEVIEW a where STATUS='A'  And EMPDATE>=To_Date('01/02/2024','dd/mm/yyyy')
   And EMPDATE<=To_Date('29/02/2024','dd/mm/yyyy')
   and empno in (select empcode from payroll.employeeinfo where SDEPTID in(select sdeptid from payroll.subdepts where sdeptid='00000011'))
   and empno is not null  and GETEMPLOCID(empid) in ('L0000002 ') order by empno)  ;
     
Over Time
-------------
select count(*) from (SELECT 
	slno,
	approval,
	empnm,
	ot_hrs,
	(case when OTAVAILED=1 then '0' when hrappoved in ('Rejected','Convert to CompOff') then '0' else approvedhrs end )approvedhrs,
	empno,
	transdt,
	to_char(login_tm,'hh:mi:ss am') login_tm,
to_char(logout_tm,'hh:mi:ss am') logout_tm,
	to_char(shift_fromtm,'hh:mi:ss am') shift_fromtm,
	to_char(shift_totm,'hh:mi:ss am')shift_totm,shift,
	empid,
	hrappoved,case when OTAVAILED=1 then 'Availed' else '' end OTAVAILED
FROM
	(
		SELECT
			slno,
			hrappoved                             approval,
			employeename                          empnm,
			ottime                                ot_hrs,
			hrs                                   approvedhrs,
			employeecode                          empno,
			otdate                                transdt,
                  (case when WVF_GetShiftidSGOut(empid,otdate) in (select shiftkeyid from PAYROLL.roastershifts where nightshift = 1) then 
wvf_getattnlogouttmsgns(empid,otdate,fkid)
else wvf_getattnlogouttmsg(empid,otdate,WVF_GetShiftidSG(otapprovalhrdtl.empid, otdate)) end) logout_tm,
                   
                (case when WVF_GetShiftidSG(empid,otdate) in (select shiftkeyid from PAYROLL.roastershifts where nightshift = 1) then 
 wvf_getattnlogintmsgns(empid,otdate) else
 wvf_getattnlogintmsg(empid,otdate,WVF_GetShiftidSG(otapprovalhrdtl.empid, otdate)) end)      login_tm,
			wvf_getattnshiftfrtmintmzoi1(otapprovalhrdtl.empid,
 otdate)   shift_fromtm,
			wvf_getattnshifttotmintmzoi(otapprovalhrdtl.empid,
 otdate)   shift_totm,
 WVF_GetShiftidSG(otapprovalhrdtl.empid, otdate) shift,
			otapprovalhrdtl.empid,decode(HRAPPOVED,'1','Approved','2','Rejected','3','Convert to CompOff') hrappoved	,OTAVAILED
		FROM 
			payroll.otapprovalhrdtl, 
			payroll.otapprovalhr 
		WHERE 
			    pkid=fkid 
			AND otdate>=TO_DATE('01/02/2024','dd/mm/yyyy')
			AND otdate<=TO_DATE('29/02/2024','dd/mm/yyyy')
AND GETPAYROLLEMPSDEPT(EMPID) In ( '00000003')       
			AND empid IN ( select empid from payroll.employeeinfo where locid in ('Loc00001') )
			AND hrs>0        
	) 
order by empno,empnm,transdt               )              
revenue code for accounts
==========================
select TYPE ,BTYPE ,MD ,MODULE ,BILLDT ,BILLID ,BILLNO ,BILLNO2 ,LOCBILLNO ,ACTBILLNO ,NAME ,
LOCID ,REFERTYPE ,BILLUSERID ,nvl(nvl((
                SELECT
                   pamtmode
                FROM
                   ip.pmtmode
                WHERE
                       fkid = substr(tcrvhdr0111202330112023.billid,2)
                    AND ROWNUM = 1
            ),(select  distinct WVF_GETPAMTMODE(paymentmode) from IP.moneyreceipt where patid=billid and rownum=1)),( wvf_getpmtmode_sourceid(billid)))pmtmode,TOTAL ,DISCOUNT ,TAXAMT ,NET ,ADVADJ ,TDS ,DISALLOWAMT ,
WOAMT ,ORGDISCAMT ,SERVICETAX ,PAID ,DUE ,REFUNDABLE ,PAYABLEBYPAT ,TPAPAYABLE ,BILLSCROLLNO ,
BILLID2 ,TOTALGSTAMT ,REGID ,REGNO ,IDENTIFY  from admin.tcrvhdr0111202330112023
consultation not done
===========================
SELECT
    (dischargedt - regdt)+1 total_days,
    casesheetnumber,
    regdt,
    (
       select count(prescdate) from( SELECT
        distinct  h.PRESCDATE
        FROM
            ots1.dprescmedhdr h,
            ots1.dprescmeddtl d
        WHERE
                h.prescid = d.prescid
            AND ippatid_d = ipinfo.patid)
           
    ) prec_days, (
       select prescdate from( SELECT
        distinct  h.PRESCDATE
        FROM
            ots1.dprescmedhdr h,
            ots1.dprescmeddtl d
        WHERE
                h.prescid = d.prescid
            AND ippatid_d = ipinfo.patid   and rownum=1)
         
    ) prescdt,
    dischargedt,patid,case when (dischargedt - regdt)+1 != (
       select count(prescdate) from( SELECT
        distinct  h.PRESCDATE
        FROM
            ots1.dprescmedhdr h,
            ots1.dprescmeddtl d
        WHERE
                h.prescid = d.prescid
            AND ippatid_d = ipinfo.patid)
           
    ) then 'prescription not given'||' '||to_char(to_char((dischargedt - regdt)+1)-to_char((
       select count(prescdate) from( SELECT
        distinct  h.PRESCDATE
        FROM
            ots1.dprescmedhdr h,
            ots1.dprescmeddtl d
        WHERE
                h.prescid = d.prescid
            AND ippatid_d = ipinfo.patid)
           
    )))||' '||'days' else 'prescription given'||' '||to_char((dischargedt - regdt)+1)||' '||'days' end type
FROM
    ip.ipinfo
    where regdt>=to_date('01/07/2023','dd/mm/yyyy')and regdt<=to_date('30/07/2023','dd/mm/yyyy') and locid='L0000002'

-------------------------------------------------------------------------------------------------------------------------

SELECT EDATE-COUNT(VDATE)FROM (SELECT
DISTINCT H.VDATE, CAST(TO_CHAR(LAST_DAY(H.VDATE), 'DD') AS INT)EDATE,VOUCHERTYPEID,SUBCOMPANYID,D.LOCID
FROM
    ats1.accountshdr H,ATS1.divisionlocation D
WHERE
    H.SUBCOMPANYID=D.DIVISIONID AND H.VDATE>=TRUNC(TO_DATE('10/07/2023','DD/MM/YYYY'),'MM')AND H.VDATE<=LAST_DAY(TO_DATE('10/07/2023','DD/MM/YYYY'))
    AND VOUCHERTYPEID='CAT00022' AND SUBCOMPANYID='SCOM0032' AND D.LOCID='L0000002') GROUP BY EDATE
service master checking code
----------------------------
SELECT
    h.servicenm,
    h.serviceid,
    h.usernm,
    decode(h.docflg, 0, 'Doctor Not Required',1,'Doctor Required') doctorflg,
    d.code
FROM
    ip.servicesvf          h,
    ip.sourceservicedatavf d
WHERE
        h.serviceid = d.serviceid
    AND d.socid IN (
        SELECT
            indexid
        FROM
            ip.tarrifindexmast
        WHERE
            indexid = 'IND00051'
    );
    
    
    nursing tv patent request grid qry
......................................
select  m.servicerequestnm Requested_Service,admin.wvf_getusernm(d.userid)  Room_No, (case when d.accepted = 1 and d.status = 0 then 'Accepted'  when d.accepted = 1 and d.status = 1 then 'Completed' else 'Pending' end)  Status,to_char(d.savedate,'dd/mm/yyyy') Requested_Date, to_char(d.savetime,'HH:MI:SS AM') Requested_Time, d.fkid||d.serreqid txtSerreqid  from admin.patserreqtemp d ,admin.patientservicerequestmaster m where m.pkid = d.serreqid  and d.savedate >= to_Date('$fd','dd/mm/yyyy')  and d.savedate <= to_Date('$td','dd/mm/yyyy')   and (case when d.accepted = 1 and d.status = 0 then 'Accepted' when d.accepted = 1 and d.status = 1 then 'Completed' else 'Pending' end) = 'Pending'  and d.REQUESTTYPE = '3' order by d.savedate desc,d.savetime desc 
purchases sales report query
-------------------------------
SELECT
    invno  invoice_no,
    invdt1 date1,
    suplid agent_name,
    medid item_name,
    quantity bill_quantity,
    free offer_qty,
    d.feededrate rate,
    feededmrp mrp,
    amount amount,
    d.tax tax,
    totalamt total_amount,
    batchno batch_no,
    d.expdt expiry_date,
    purchaseno grnno 
FROM
    inventory.receiptmedmast h,
    inventory.receiptmeddtls d
WHERE
    h.rcid = d.rcid
    and h.rcdt>=to_date('01/02/2023','dd/mm/yyyy')and h.rcdt<=to_date('30/05/2023','dd/mm/yyyy');
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



 important notes
 =================
select netsal,salid,dtlsal,empnm from (SELECT
    hi.netsal,SALID,empnm,(SELECT SAL FROM(SELECT SUM(SAL)SAL,SALID FROM(SELECT
    SUM(PAIDAMT) SAL,SALID
FROM
    payroll.empsaldtl_month
WHERE
    salid IN (
        SELECT
            hi.salid
        FROM
            payroll.empsalhdr_month hi
        WHERE
                hi.companyid = 'CMP00006'
            AND hi.formonth = '11'AND FORYEAR='2023'
            AND nvl(hi.structure, 'A') != 'SS000010'
            AND hi.foryear = '2023'
    ) AND ADDDED='A'group by  SALID
    UNION ALL 
    SELECT
    SUM(-PAIDAMT),SALID
FROM
    payroll.empsaldtl_month
WHERE
    salid IN (
        SELECT
            hi.salid
        FROM
            payroll.empsalhdr_month hi
        WHERE
                hi.companyid = 'CMP00006'
            AND hi.formonth = '11'AND FORYEAR='2023'
            AND nvl(hi.structure, 'A') != 'SS000010'
            AND hi.foryear = '2023'
    )AND ADDDED='D' group by  SALID) group by SALID)WHERE SALID=hi.SALID)dtlsal
FROM
    payroll.empsalhdr_month hI
   
WHERE HI.companyid = 'CMP00006'--AND SALID='ES022751'
    AND HI.formonth = '11' AND NVL(HI.STRUCTURE,'A')!='SS000010'
    AND HI.foryear = '2023'AND  hi.netsal NOT IN(SELECT SAL FROM(SELECT SUM(SAL)SAL,SALID FROM(SELECT
    SUM(PAIDAMT) SAL,SALID
FROM
    payroll.empsaldtl_month
WHERE
    salid IN (
        SELECT
            hi.salid
        FROM
            payroll.empsalhdr_month hi
        WHERE
                hi.companyid = 'CMP00006'
            AND hi.formonth = '11'AND FORYEAR='2023'
            AND nvl(hi.structure, 'A') != 'SS000010'
            AND hi.foryear = '2023'
    ) AND ADDDED='A'group by  SALID
    UNION ALL 
    SELECT
    SUM(-PAIDAMT),SALID
FROM
    payroll.empsaldtl_month
WHERE
    salid IN (
        SELECT
            hi.salid
        FROM
            payroll.empsalhdr_month hi
        WHERE
                hi.companyid = 'CMP00006'
            AND hi.formonth = '11'AND FORYEAR='2023'
            AND nvl(hi.structure, 'A') != 'SS000010'
            AND hi.foryear = '2023'
    )AND ADDDED='D' group by  SALID) group by SALID)WHERE SALID=SALID))where netsal-dtlsal>10 or dtlsal-netsal>10
=======================
monthly neded qry
=======================
SELECT
    empno,
    empnm,
    getempdesignationnm(empid) designationnm,
    (select sdeptid from payroll.employeeinfo where  empid=a.empid and sdeptid='00000003')deptid,
    (
        SELECT
            subdeptnm
        FROM
            payroll.subdepts     s,
            payroll.employeeinfo e
        WHERE
                s.sdeptid = e.sdeptid
            AND e.empid = a.empid
    )                          subdeptnm,
    empdate                    absent_date,
    'Absent'                   status
FROM
    attendnaceview a
WHERE status='A' AND
         empdate >= TO_DATE('01/01/2024', 'dd/mm/yyyy')
    AND empdate <= TO_DATE('31/01/2024', 'dd/mm/yyyy')
    AND empno IS NOT NULL
    AND getemplocid(empid) IN ( 'L0000008' )and empid in (select empid from payroll.employeeinfo where  empid=a.empid and sdeptid='00000011')
ORDER BY
    empno
