WITH revenue_data as
(
SELECT
       sp.continent,
       sum(p.price) as revenue,
       sum(case when device = 'mobile' then p.price end) as revenue_from_mobile,
       sum(case when device = 'desktop' then p.price end) as revenue_from_desktop
FROM `data-analytics-mate.DA.order` o
JOIN `data-analytics-mate.DA.product` p
ON o.item_id = p.item_id
JOIN `data-analytics-mate.DA.session_params`  sp
ON o.ga_session_id = sp.ga_session_id
GROUP BY sp.continent
),


revenue_percent as
(
SELECT
       revenue_data.continent,
       revenue_data.revenue,
       revenue_data.revenue / sum(revenue_data.revenue) OVER () * 100 as revenue_from_total_percent,
       revenue_data.revenue_from_mobile,
       revenue_data.revenue_from_desktop
FROM revenue_data
),


account_data as
(
SELECT
       sp.continent,
       COUNT(acs.account_id) as account_count,
       COUNT(CASE WHEN acc.is_verified = 1 then acc.is_verified end) as verified_account


FROM `data-analytics-mate.DA.session_params` sp
JOIN `data-analytics-mate.DA.account_session` acs
ON sp.ga_session_id = acs.ga_session_id
JOIN `data-analytics-mate.DA.account` acc
ON acs.account_id = acc.id
GROUP BY sp.continent
),


session_data as
(
SELECT
       sp.continent,
       COUNT(sp.ga_session_id) as session_count
FROM `data-analytics-mate.DA.session_params` sp
GROUP BY sp.continent
)


SELECT
       session_data.continent,
       revenue_percent.revenue,
       revenue_percent.revenue_from_mobile,
       revenue_percent.revenue_from_desktop,
       revenue_percent.revenue_from_total_percent,
       account_data.account_count,
       account_data.verified_account,
       session_data.session_count


FROM session_data
LEFT JOIN revenue_percent
ON session_data.continent = revenue_percent.continent
LEFT JOIN account_data
ON session_data.continent = account_data.continent
