<code SQL>
select m.*
 , avg(sum_payment) over (order by month_purshase rows between 2 preceding and current row) as ma_3
 , avg(sum_payment) over (order by month_purshase rows between 6 preceding and current row) as ma_7
 , avg(sum_payment) over (order by month_purshase rows between 2 preceding and 2 following) as MA_2side
from
(select date_trunc ('month', date_purchase::date) as month_purshase
  , sum (amt_payment) as sum_payment
from skycinema.client_sign_up
group by month_purshase
) m
</code>
Результат выполнения select'а представлен на картинке "Результат выполнения select.png"
