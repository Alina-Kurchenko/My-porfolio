<code SQL>
with 
 first_payments as 
 (select user_id
    , min (date_trunc ('day', transaction_datetime)) first_payment_date
from skyeng_db.payments
where status_name = 'success'
group by user_id
order by user_id
), 
all_dates as
(select distinct date_trunc('day', class_start_datetime) as dt  
from skyeng_db.classes
where class_start_datetime between '2016-01-01' and '2017-01-01'
order by dt desc
), 
 all_dates_by_user as 
(select user_id, dt
from all_dates
join first_payments on first_payments.first_payment_date <= all_dates.dt),

payments_by_dates as
(select user_id
    , date_trunc('day', transaction_datetime) as payment_date 
    , sum(classes) as transaction_balance_change
from skyeng_db.payments
where status_name = 'success'
group by user_id , payment_date
order by 1 desc 
),
classes_by_dates as
(select    user_id
        , class_start_datetime::date as class_date
        , (count(class_status) * -1) as classes
from skyeng_db.classes
where class_status in ('success','failed_by_student')  
and class_type !=  'trial'
group by user_id, class_date
order by user_id asc
),
payments_by_dates_cumsum as
(select all_dates_by_user.user_id
        , dt
        , transaction_balance_change
        ,sum(coalesce(transaction_balance_change, 0)) over (partition by all_dates_by_user.user_id order by dt rows between unbounded preceding and current row) as transaction_balance_change_sc 
from all_dates_by_user
left join 
payments_by_dates on all_dates_by_user.user_id = payments_by_dates.user_id
and all_dates_by_user.dt = payments_by_dates.payment_date ),
classes_by_dates_dates_cumsum as
(select all_dates_by_user.user_id
        , dt
        , classes
        , sum(coalesce(classes, 0)) over (partition by all_dates_by_user.user_id order by dt rows between unbounded preceding and current row) as classes_cs
from all_dates_by_user
left join 
classes_by_dates on all_dates_by_user.user_id = classes_by_dates.user_id
and all_dates_by_user.dt = classes_by_dates.class_date ),
balances as
(select   payments_by_dates_cumsum.user_id
         , payments_by_dates_cumsum.dt
        , payments_by_dates_cumsum.transaction_balance_change as transaction_balance_change
        , payments_by_dates_cumsum.transaction_balance_change_sc as transaction_balance_change_cs
        , classes_by_dates_dates_cumsum.classes as classes
        , classes_by_dates_dates_cumsum.classes_cs as classes_cs
        , classes_by_dates_dates_cumsum.classes_cs + payments_by_dates_cumsum.transaction_balance_change_sc as balance
from payments_by_dates_cumsum
join
classes_by_dates_dates_cumsum on payments_by_dates_cumsum.user_id = classes_by_dates_dates_cumsum.user_id
and payments_by_dates_cumsum.dt = classes_by_dates_dates_cumsum.dt)
-- select*
-- from balances
-- order by user_id, dt
-- limit 1000
select  dt
        , sum(transaction_balance_change) as sum_transaction_balance_change
        , sum(transaction_balance_change_cs) as sum_transaction_balance_change_cs
        , sum(classes) as sum_classes
        , sum(classes_cs) as sum_classes_cs
        , sum(balance) as sum_balanse
from balances
group by dt
order by dt
</code>
Результат выполнения select'а представлен в файле "Результат выполнения select.xlsx"
Визуализация итогового результата представлена в файле "Визуализация select.png"
