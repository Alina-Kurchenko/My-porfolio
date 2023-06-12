<code SQL>
select id_client
    , name_city
    , case when gender = 'M' then '1'
    else '0' end as nflag_gender
    , age
    , first_time
    , case when cellphone is not null then '1'
    else '0' end as nflag_cellphone
    , is_active
    , cl_segm
    , amt_loan
    , date_loan::date
    , credit_type
    , sum(amt_loan) over (partition by name_city) as sum_city
    , (amt_loan)::float / sum(amt_loan) over (partition by name_city)*100  as share_credit_city
    , sum(amt_loan) over (partition by credit_type) as sum_credit_type
    , (amt_loan)::float / sum(amt_loan) over (partition by credit_type)*100  as share_credit_type
    , sum(amt_loan) over (partition by name_city, credit_type) as sum_city_tipe
    , (amt_loan)::float / sum(amt_loan) over (partition by name_city, credit_type) as share_city_tipe
    , count(amt_loan) over (partition by name_city) as cnt_loan_city
    , count(amt_loan) over (partition by credit_type) as cnt_credit_type
    , count(amt_loan) over (partition by name_city, credit_type) as cnt_city_tipe
from skybank.late_collection_clients c
join skybank.region_dict d
on c.id_city = d.id_city
</code>
Результат выполнения select'а представлен на картинке "Результат выполнения select.xlsx"
