---
name: Home
assetId: bede996d-5025-443d-a7a6-a4c39996a950
type: page
---

# Welcome

This is your new project's homepage. Edit this file to get started.

```sql paid_bookings_mom
WITH monthly AS (
  SELECT
    date_trunc('month', bookingdate) AS month,
    COUNT(*) AS bookings
  FROM first_session_original_receipts_and_paystubs_db_public_acuityappointment
  WHERE type = 'PAID'
    AND appointmentstatus IN ('BOOKED', 'LATE_CANCEL', 'NO_SHOW')
    AND bookingdate IS NOT NULL
    AND date_trunc('month', bookingdate) < date_trunc('month', today())
    AND bookingdate >= date_trunc('month', today()) - INTERVAL 24 MONTH
  GROUP BY 1
)
SELECT
  month,
  ROUND(
    toFloat64(bookings - lagInFrame(bookings) OVER (ORDER BY month))
    / NULLIF(lagInFrame(bookings) OVER (ORDER BY month), 0) * 100, 1
  ) AS mom_growth_pct
FROM monthly
ORDER BY month
```

{% bar_chart
    data="paid_bookings_mom"
    x="month"
    y="mom_growth_pct"
    title="Paid Bookings - MoM Growth %"
    subtitle="Monthly growth rate of paid bookings"
    y_fmt="#,##0.0'%'"
%}
    {% reference_line y=7 label="Goal" color="grey" /%}
{% /bar_chart %}
