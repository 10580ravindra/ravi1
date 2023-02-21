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
    AND indentno in(1482,1468,1521,1407,1511)
