# ANALIZA COSTURI

## 5.1 Costuri Lunare Estimative

**Infrastructură lunară (estimare):**
| Componentă | Cost Lunar (USD) | Note |
|------------|------------------|------|
| GKE Regional Cluster | $300 | 3 noduri, e2-standard-2 |
| Cloud SQL | $200 | PostgreSQL HA, 100GB storage |
| Apigee | $400 | Standard tier |
| Cloud Storage | $50 | 500GB storage, accese frecvente |
| Firebase | $25 | Blaze plan (pay-as-you-go) |
| Cloud Functions | $40 | ~1M invocations |
| Networking | $80 | Load Balancer, egress |
| Cloud Monitoring | $70 | Logging, metrics, alerting |
| Total | $1,165 | |

**Total estimat anual: $13,980**

## 5.2 Optimizare Costuri

**Strategii de optimizare:**
- Utilizare GKE Autopilot pentru optimizarea automată a resurselor
- Committed use discounts pentru resurse de lungă durată
- Lifecycle policies pentru stocarea datelor mai vechi pe clase de stocare mai ieftine
- Caching agresiv pentru reducerea costurilor de procesare și networking
- Monitorizare și alertare pentru optimizare continuă