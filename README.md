Цель проекта: оценить динамику пользовательской базы, активность игроков, финансовые результаты и показатели вирусного роста.

Что делала:
— Изучила исходные данные и провела анализ количества пользователей и игровых сессий.
— Рассчитала долю проблемных сессий по типам устройств и оценила возможные причины аномалий.
— Определила коэффициент вирусности игры и средний размер будущих когорт пользователей.
— Проанализировала динамику выручки и влияние маркетинговой кампании на финансовые показатели.
— Оценила изменения ключевых метрик DAU / WAU / MAU и их соотношение.
— Рассчитала долю лояльных пользователей, протестировав разные критерии определения лояльности.

Используемые инструменты: PostgreSQL (CASE WHEN, агрегации, JOIN-ы, CTE/подзапросы), Excel (постобработка, визуализация).

Результат: подготовлена презентация с основными выводами и визуализациями, демонстрирующая динамику пользовательской активности и влияние маркетинговых факторов на выручку игры.


1. расчета MAU 

SELECT DATE_TRUNC('month', start_session)::date AS "Начало месяца"
  		,COUNT(DISTINCT id_user) AS "MAU"
FROM skygame.game_sessions
GROUP BY 1 
ORDER BY 1 ASC

Вывод: позитивный эффект виден: во время акции DAU вырос примерно на 64%, а WAU — на 88% относительно февраля.
При этом SF Week немного снизился, поэтому нельзя уверенно сказать, что акция улучшила удержание. Скорее, она сильно увеличила общую активность и возвращаемость игроков.

2. Расчет K-factory игры

select 
 sum(ref_reg)/count(distinct us.id_user) as k_faсtor
,sum(ref_reg)/count(distinct date_trunc('month', reg_date)) as future_cohort
from skygame.users us 
  left join skygame.referral r 
    on us.id_user = r.id_user

  3. Расчет критериев лояльных пользователей игры

with loyals1 as 
(
select    id_user
from skygame.monetary m
   join skygame.item_list i
      on m.id_item_buy = i.id_item
   join skygame.log_prices p
		  on m.id_item_buy = p.id_item
      and m.dtime_pay >= p.valid_from
      and m.dtime_pay <= coalesce(valid_to, '3000-01-01')
group by id_user
having sum(cnt_buy * price) >= 1000
),
loyals2 as 
(
select    id_user
from skygame.referral
group by id_user
having count(*) >= 3
  and sum(ref_reg) >= 1
)

select    count(distinct id_user) as cnt_user
, date_trunc('month', start_session) as mm
from skygame.game_sessions
where id_user in (select * from loyals1)
and id_user in (select * from loyals2)
group by mm



