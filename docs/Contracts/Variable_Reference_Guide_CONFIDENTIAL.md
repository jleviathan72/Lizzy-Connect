# VARIABLE REFERENCE GUIDE
## Partnership Agreements - Initial Proposal Numbers
### CONFIDENTIAL - For Leviathan Consulting Group LLC Internal Use Only

**Last Updated:** November 29, 2025  
**Purpose:** Private reference for negotiating partnership agreements with Nizex, Inc. and Carl's Mower & Saw Inc.

---

## NIZEX, INC. - API PARTNERSHIP AGREEMENT

### FINANCIAL TERMS

**R&D Cost Recovery:**
- `[RD_CAP]` = **$30,000** (Total R&D cost cap - EXPECT COUNTER OFFER)
- `[INFRA_FEE]` = **$500** (Monthly infrastructure fee after cap reached)

**Revenue Share Tiers:**
- `[TIER_1]` = **$250,000** (First revenue threshold)
- `[TIER_2]` = **$500,000** (Second revenue threshold)
- `[TIER_3]` = **$1,500,000** (Third revenue threshold)
- `[TIER_4]` = **$3,000,000** (Fourth revenue threshold)

**Revenue Share Percentages:**
- `[PCT_1]` = **8%** (Revenue share for $0-$250K)
- `[PCT_2]` = **10%** (Revenue share for $250K-$500K)
- `[PCT_3]` = **12%** (Revenue share for $500K-$1.5M)
- `[PCT_4]` = **15%** (Revenue share for $1.5M-$3M)
- `[PCT_5]` = **18%** (Revenue share for $3M+)

**Note:** With these tiers, R&D cap of $30K is reached at approximately $375K in cumulative revenue:
- First $250K @ 8% = $20,000
- Next $125K @ 10% = $12,500
- **Total = $32,500** (cap reached)

**Customer Referral Terms:**
- `[REFERRAL_PCT]` = **50%** (Percentage of Year 1 recurring revenue both directions)
- `[REFERRAL_TERM]` = **1 year** (Duration of referral commission)
- `[PAYMENT_DAYS]` = **60 days** (Payment timing after quarter end)

### TIME PERIODS

**Partnership Duration:**
- `[EXCLUSIVE_TERM]` = **4 years** (Exclusivity period)
- `[RENEWAL_NOTICE_DAYS]` = **90 days** (Notice for non-renewal)
- `[CONVENIENCE_TERMINATION_DAYS]` = **180 days** (Notice for termination for convenience)
- `[MIN_TERM_YEARS]` = **2 years** (Minimum term before convenience termination allowed)
- `[TAIL_PERIOD_MONTHS]` = **3 months** (Revenue share tail period after termination)
- `[WINDDOWN_MONTHS]` = **6 months** (API access wind-down period)
- `[CURE_PERIOD_DAYS]` = **30 days** (Time to cure material breach)
- `[CONFIDENTIALITY_YEARS]` = **5 years** (Confidentiality survival period)

**Referral & Attribution:**
- `[REFERRAL_REGISTRATION_DAYS]` = **5 days** (Time to register referral prospect)
- `[REFERRAL_ATTRIBUTION_DAYS]` = **180 days** (Referral attribution window)
- `[REFERRAL_MIN_ACTIVE_DAYS]` = **30 days** (Minimum customer active period for commission)

### API DEVELOPMENT

**API Count:**
- `[API_COUNT]` = **7** (Total number of APIs)
- `[API_CRITICAL_COUNT]` = **4** (Number of critical APIs)
- `[API_IMPORTANT_COUNT]` = **3** (Number of important APIs)
- `[API_PHASE1_COUNT]` = **4** (Critical APIs in Phase 1)
- `[API_PHASE2_COUNT]` = **3** (Important APIs in Phase 2)

**Development Timeline:**
- `[SPEC_DELIVERY_DAYS]` = **15 days** (Leviathan delivers specifications)
- `[SPEC_REVIEW_DAYS]` = **10 days** (Nizex reviews specifications)
- `[SPEC_FINALIZE_DAYS]` = **30 days** (Collaborative finalization)
- `[DEV_KICKOFF_DATE]` = **TBD** (Development kickoff - negotiate with Nizex)
- `[PHASE1_COMPLETE_DATE]` = **TBD** (Phase 1 APIs complete - negotiate)
- `[PHASE2_COMPLETE_DATE]` = **TBD** (Phase 2 APIs complete - negotiate)
- `[PRODUCTION_DATE]` = **TBD** (All APIs in production - negotiate)
- `[TOTAL_DEV_WEEKS]` = **14 weeks** (Total estimated development time)

**API Development Time Estimates (for Nizex negotiation):**
- `[API_CUSTOMER_PROFILE_DEV_WEEKS]` = **3 weeks**
- `[API_TRANSACTIONS_DEV_WEEKS]` = **3 weeks**
- `[API_SERVICE_ORDER_DEV_WEEKS]` = **4 weeks**
- `[API_PARTS_ORDER_DEV_WEEKS]` = **4 weeks**
- `[API_SMS_LOG_DEV_WEEKS]` = **2 weeks**
- `[API_FLAG_DEV_WEEKS]` = **2 weeks**
- `[API_STATUS_DEV_WEEKS]` = **2 weeks**

**Acceptance & Enhancement:**
- `[ACCEPTANCE_PERIOD_DAYS]` = **15 days** (Testing and acceptance period)
- `[REMEDY_PERIOD_DAYS]` = **10 days** (Time to fix deficiencies)
- `[ENHANCEMENT_REVIEW_DAYS]` = **10 days** (Time to review enhancement requests)
- `[VERSION_SUPPORT_MONTHS]` = **12 months** (Backward compatibility maintenance)
- `[VERSION_NOTICE_DAYS]` = **90 days** (Advance notice of API changes)
- `[ADVANCE_NOTICE_DAYS]` = **90 days** (General advance notice period)

### SERVICE LEVEL AGREEMENT

**Uptime & Performance:**
- `[UPTIME_PCT]` = **99.5%** (Uptime guarantee)
- `[BUSINESS_HOURS]` = **8am-8pm** (SLA coverage hours)
- `[TIMEZONE]` = **Eastern Time**
- `[BUSINESS_START_TIME]` = **8:00 AM ET**
- `[BUSINESS_END_TIME]` = **8:00 PM ET**
- `[MAINTENANCE_NOTICE_DAYS]` = **7 days** (Advance notice for planned maintenance)

**API Performance Standards:**
- `[AVG_RESPONSE_MS]` = **500ms** (Average API response time)
- `[MAX_RESPONSE_MS]` = **2000ms** (Maximum API response time)
- `[MIN_THROUGHPUT_RPS]` = **100** (Minimum requests per second)
- `[DATA_FRESHNESS_MINUTES]` = **5 minutes** (Maximum data staleness)

**Rate Limits:**
- `[RATE_LIMIT_RPM]` = **1000** (Requests per minute standard)
- `[BURST_LIMIT_RPM]` = **2000** (Burst requests per minute)
- `[DAILY_CAP_REQUESTS]` = **100000** (Daily request cap)

**SLA Credits:**
- `[SLA_CREDIT_TIER1]` = **10%** (Credit for 99.0-99.4% uptime)
- `[SLA_CREDIT_TIER2]` = **25%** (Credit for 98.0-98.9% uptime)
- `[SLA_CREDIT_TIER3]` = **50%** (Credit for <98.0% uptime)
- `[SLA_CLAIM_DAYS]` = **30 days** (Time to claim SLA credits)

### SUPPORT & RESPONSE TIMES

**Critical Issues:**
- `[CRITICAL_RESPONSE_HOURS]` = **1 hour**
- `[CRITICAL_RESOLUTION_HOURS]` = **4 hours**

**High Priority Issues:**
- `[HIGH_RESPONSE_HOURS]` = **4 business hours**
- `[HIGH_RESOLUTION_HOURS]` = **24 business hours**

**Medium Priority Issues:**
- `[MEDIUM_RESPONSE_HOURS]` = **8 business hours**
- `[MEDIUM_RESOLUTION_DAYS]` = **3 business days**

**Low Priority Issues:**
- `[LOW_RESPONSE_DAYS]` = **5 business days**

### SECURITY & DATA

**Security Standards:**
- `[BREACH_NOTIFICATION_HOURS]` = **24 hours** (Breach notification timeframe)
- `[BREACH_REPORT_DAYS]` = **10 days** (Detailed breach report due)
- `[TOKEN_ROTATION_DAYS]` = **90 days** (API key rotation frequency)

**Data Management:**
- `[DATA_DELETION_DAYS]` = **30 days** (Customer data deletion after termination)
- `[BACKUP_RETENTION_DAYS]` = **90 days** (Backup data retention)
- `[TRANSITION_DAYS]` = **60 days** (Transition assistance period)

### CACHING (API-Specific - Exhibit A)

**API Response Caching:**
- `[CACHE_CUSTOMER_PROFILE_MIN]` = **15 minutes** (Customer profile cache)
- `[CACHE_TRANSACTIONS_MIN]` = **5 minutes** (Transaction history cache)

**API Performance (Exhibit A):**
- `[API_CUSTOMER_PROFILE_RESPONSE_MS]` = **500ms**
- `[API_TRANSACTIONS_RESPONSE_MS]` = **600ms**
- `[API_SERVICE_ORDER_RESPONSE_MS]` = **800ms**
- `[API_PARTS_ORDER_RESPONSE_MS]` = **800ms**
- `[API_SMS_LOG_RESPONSE_MS]` = **300ms**
- `[API_FLAG_RESPONSE_MS]` = **300ms**
- `[API_STATUS_RESPONSE_MS]` = **200ms**

### AUDIT & FINANCIAL CONTROLS

**Audit Rights:**
- `[RECORD_RETENTION_YEARS]` = **7 years** (Financial record retention)
- `[AUDIT_FREQUENCY]` = **1 time** (Maximum audits per year)
- `[AUDIT_NOTICE_DAYS]` = **30 days** (Advance notice for audit)
- `[INTEREST_RATE]` = **1.5%** (APR on underpayments)
- `[UNDERPAYMENT_SETTLEMENT_DAYS]` = **15 days** (Settlement timeframe)
- `[AUDIT_SURVIVAL_YEARS]` = **3 years** (Post-termination audit rights)

**Reporting:**
- `[ANNUAL_RECON_DAYS]` = **60 days** (Annual reconciliation deadline)
- `[SETTLEMENT_DAYS]` = **30 days** (Discrepancy settlement)
- `[FINAL_ACCOUNTING_DAYS]` = **30 days** (Final accounting after termination)
- `[COMMISSION_REPORT_DAYS]` = **45 days** (Quarterly commission reporting)

### TERMINATION

**Termination Fees:**
- `[TERMINATION_FEE_PCT]` = **50%** (Percentage of remaining R&D cap if terminated early)

**Wind-Down:**
- `[EMERGENCY_WINDDOWN_DAYS]` = **30 days** (Emergency wind-down if terminated for cause)
- `[MARKETING_WINDDOWN_DAYS]` = **90 days** (Marketing activity wind-down)
- `[CUSTOMER_NOTICE_DAYS]` = **60 days** (Customer notification before service discontinuation)
- `[TRANSITION_NOTICE_DAYS]` = **180 days** (Notice before pursuing competing products post-exclusivity)
- `[MARKETING_SURVIVAL_MONTHS]` = **6 months** (Continued use of marketing materials)
- `[REMOVAL_DAYS]` = **30 days** (Remove branding from materials upon request)

### CUSTOMER ERP ACCESS

**Access Provisioning:**
- `[ACCESS_PROVISION_DAYS]` = **30 days** (Time to provide ERP access)
- `[ACCESS_EXTENSION_MONTHS]` = **6 months** (Access continuation after exclusive period)
- `[ACCESS_TERMINATION_NOTICE_DAYS]` = **60 days** (Notice to terminate access)
- `[MAX_ACCESS_USERS]` = **5** (Maximum Leviathan employees with access)

### IMPLEMENTATION SUPPORT

**Nizex Support Obligations:**
- `[IMPL_SUPPORT_HOURS]` = **8 hours** (Free implementation support per customer)
- `[IMPL_NOTICE_DAYS]` = **14 days** (Advance notice for implementation support)

### MARKETING

**Marketing Commitments:**
- `[PRESS_RELEASE_DAYS]` = **30 days** (Joint press release after effective date)
- `[WEBINAR_COUNT]` = **2** (Webinars per year)
- `[TRADESHOW_COUNT]` = **3** (Trade shows per year)
- `[APPROVAL_DAYS]` = **5 business days** (Marketing material approval)
- `[ANNOUNCEMENT_NOTICE_DAYS]` = **10 days** (Advance notice for public announcements)
- `[CASE_STUDY_COUNT]` = **3** (Customer case studies during exclusive period)
- `[TRAINING_SESSIONS]` = **2** (Sales training sessions per year)
- `[MARKETING_FUND_AMOUNT]` = **TBD** (Optional shared marketing fund - negotiate)

### DISPUTE RESOLUTION

**Resolution Process:**
- `[EXEC_MEETING_DAYS]` = **15 days** (Executive meeting deadline)
- `[NEGOTIATION_DAYS]` = **30 days** (Good faith negotiation period)
- `[MEDIATION_ORG]` = **JAMS** (or AAA - American Arbitration Association)
- `[MEDIATION_LOCATION]` = **Orange County, California**
- `[MEDIATION_PERIOD_DAYS]` = **60 days** (Mediation completion deadline)
- `[ARBITRATION_ORG]` = **JAMS** (or AAA)
- `[ARBITRATION_LOCATION]` = **Orange County, California**
- `[GOVERNING_LAW_STATE]` = **California**
- `[LITIGATION_VENUE]` = **Orange County, California**

### LIABILITY & INSURANCE

**Liability Caps:**
- `[LIABILITY_CAP_AMOUNT]` = **$500,000**
- `[LIABILITY_CAP_MONTHS]` = **12 months** (Lookback period for liability cap)

**Insurance Requirements - Nizex:**
- `[NIZEX_GL_AMOUNT]` = **$1,000,000** (General liability)
- `[NIZEX_EO_AMOUNT]` = **$1,000,000** (E&O insurance)
- `[NIZEX_CYBER_AMOUNT]` = **$1,000,000** (Cyber liability)

**Insurance Requirements - Leviathan:**
- `[LEVIATHAN_GL_AMOUNT]` = **$1,000,000** (General liability)
- `[LEVIATHAN_EO_AMOUNT]` = **$1,000,000** (E&O insurance)
- `[LEVIATHAN_CYBER_AMOUNT]` = **$1,000,000** (Cyber liability)

**Insurance Administration:**
- `[INSURANCE_NOTICE_DAYS]` = **30 days** (Notice of coverage changes)

### FORCE MAJEURE

**Force Majeure Terms:**
- `[FORCE_MAJEURE_NOTICE_DAYS]` = **5 days** (Notification deadline)
- `[FORCE_MAJEURE_TERMINATION_DAYS]` = **90 days** (Termination right if force majeure continues)

### FEE ADJUSTMENTS

**Infrastructure Fee Adjustments:**
- `[USAGE_INCREASE_PCT]` = **50%** (Threshold for fee adjustment)
- `[FEE_ADJUSTMENT_NOTICE_DAYS]` = **60 days** (Advance notice for fee changes)

---

## CARL'S MOWER & SAW INC. - PARTNERSHIP AGREEMENT

### FINANCIAL TERMS

**Revenue Share:**
- `[REVENUE_SHARE_PCT]` = **5%** (Revenue share percentage)
- `[REVENUE_SHARE_TERM]` = **4 years** (Duration of revenue share)
- `[COMMISSION_PCT]` = **50%** (Customer referral commission)
- `[COMMISSION_TERM]` = **1 year** (Duration of commission on referred customer)
- `[PAYMENT_DAYS]` = **60 days** (Payment timing after quarter end)

**Fractional CTO Services:**
- `[CTO_MONTHLY_FEE]` = **$10,000** (Monthly fee for fractional CTO services)

### TIME PERIODS

**Agreement Duration:**
- `[AGREEMENT_TERM]` = **4 years** (Initial term with annual auto-renewal)
- `[EXCLUSIVITY_TERM]` = **4 years** (Non-compete/exclusivity period)
- `[RENEWAL_NOTICE_DAYS]` = **90 days** (Notice for non-renewal)
- `[CURE_PERIOD_DAYS]` = **30 days** (Time to cure breach)
- `[CONFIDENTIALITY_YEARS]` = **5 years** (Confidentiality survival period)

**Project Milestones:**
- `[PROTOTYPE_DATE]` = **December 1, 2025** (Target prototype completion - VERY SOON!)
- `[MVP_DATE]` = **March 31, 2026** (Target MVP validation)
- `[PRODUCTION_DATE]` = **June 30, 2026** (Target production proving)
- `[ESTIMATED_HOURS]` = **TBD** (Carl's estimated time contribution)

**Termination & Wind-Down:**
- `[TAIL_PERIOD_MONTHS]` = **3 months** (Revenue share tail period)
- `[MARKETING_WINDDOWN_DAYS]` = **90 days** (Marketing wind-down)
- `[EXEC_MEETING_DAYS]` = **15 days** (Executive meeting for disputes)
- `[NEGOTIATION_DAYS]` = **30 days** (Good faith negotiation)

### SUPPORT & RESPONSE TIMES

**Technical Support:**
- `[SUPPORT_RESPONSE_HOURS]` = **4 business hours** (Priority support response)

### DATA & SECURITY

**Data Management:**
- `[DATA_DELETION_DAYS]` = **30 days** (Data deletion after termination)
- `[BACKUP_RETENTION_DAYS]` = **90 days** (Backup retention)
- `[TRANSITION_DAYS]` = **60 days** (Transition assistance)

### FRACTIONAL CTO - STATEMENT OF WORK

**Time Commitment:**
- `[CTO_LIZZY_HOURS_WEEKLY_PHASE12]` = **20 hours/week** (Lizzy Connect Phase 1-2)
- `[CTO_LIZZY_HOURS_WEEKLY_PHASE3]` = **8-12 hours/week** (Lizzy Connect Phase 3+)
- `[CTO_CARLS_HOURS_WEEKLY]` = **8-10 hours/week** (Carl's internal technology)
- `[CTO_TOTAL_HOURS_MONTHLY_PHASE12]` = **~120 hours/month** (Total Phase 1-2)
- `[CTO_TOTAL_HOURS_MONTHLY_PHASE3]` = **~80 hours/month** (Total Phase 3+)

**Deliverables:**
- `[FUNCTIONAL_SYSTEMS_MAP_DAYS]` = **30 days** (Initial systems map delivery)

**Decision Authority:**
- `[MINOR_EXPENDITURE_CAP]` = **$5,000** (CTO can approve without prior approval)
- `[MAJOR_CONTRACT_THRESHOLD]` = **$25,000** (Annual contract value requiring client approval)

### YOUTUBE PERFORMANCE METRICS

**Baseline Variables:**
- `[YOUTUBE_BASELINE_FOLLOWERS]` = **TBD** (Current subscriber count)
- `[YOUTUBE_BASELINE_MONTHLY_VIEWS]` = **TBD** (Average monthly views)
- `[YOUTUBE_BASELINE_GROWTH_RATE]` = **TBD** (Monthly growth rate %)
- `[YOUTUBE_BASELINE_ENGAGEMENT]` = **TBD** (Watch time, likes, comments)
- `[YOUTUBE_BASELINE_CAPTURE_DAYS]` = **30 days** (Deadline to establish baseline)

**Target Metrics:**
- `[YOUTUBE_TARGET_FOLLOWER_GROWTH]` = **TBD** (Target % increase)
- `[YOUTUBE_TARGET_VIEW_GROWTH]` = **TBD** (Target % increase)
- `[YOUTUBE_TARGET_CONVERSION_RATE]` = **TBD** (% SMS to YouTube)
- `[YOUTUBE_SUCCESS_THRESHOLD]` = **TBD** (Minimum growth for success)
- `[YOUTUBE_MEASUREMENT_PERIOD_MONTHS]` = **12 months** (Post-launch measurement)

### DISPUTE RESOLUTION

**Resolution Process:**
- `[EXEC_MEETING_DAYS]` = **15 days**
- `[NEGOTIATION_DAYS]` = **30 days**
- `[MEDIATION_ORG]` = **JAMS** (or AAA)
- `[MEDIATION_LOCATION]` = **Orange County, California**
- `[MEDIATION_PERIOD_DAYS]` = **60 days**
- `[ARBITRATION_ORG]` = **JAMS** (or AAA)
- `[ARBITRATION_LOCATION]` = **Orange County, California**
- `[GOVERNING_LAW_STATE]` = **California**
- `[LITIGATION_VENUE]` = **Orange County, California**

### LIABILITY & INSURANCE

**Liability Caps:**
- `[LIABILITY_CAP_AMOUNT]` = **$500,000**
- `[LIABILITY_CAP_MONTHS]` = **12 months**

**Insurance Requirements - Leviathan:**
- `[LEVIATHAN_GL_AMOUNT]` = **$1,000,000** (General liability)
- `[LEVIATHAN_EO_AMOUNT]` = **$1,000,000** (E&O insurance)
- `[LEVIATHAN_CYBER_AMOUNT]` = **$1,000,000** (Cyber liability)

**Insurance Requirements - Carl's:**
- `[CARLS_GL_AMOUNT]` = **$1,000,000** (General liability)
- `[CARLS_PROPERTY_AMOUNT]` = **TBD** (Business property insurance)

**Insurance Administration:**
- `[INSURANCE_NOTICE_DAYS]` = **30 days**

### FORCE MAJEURE

- `[FORCE_MAJEURE_NOTICE_DAYS]` = **5 days**
- `[FORCE_MAJEURE_TERMINATION_DAYS]` = **90 days**

### REFERRAL ATTRIBUTION

- `[REFERRAL_REGISTRATION_DAYS]` = **5 days** (Time to register referral)
- `[REFERRAL_ATTRIBUTION_DAYS]` = **180 days** (Attribution window)
- `[REFERRAL_MIN_ACTIVE_DAYS]` = **30 days** (Minimum active period)

---

## NEGOTIATION STRATEGY NOTES

### NIZEX - EXPECTED PUSHBACK AREAS

1. **R&D Cap ($30K)** - This is VERY low for 7 enterprise APIs
   - **Expect counter:** $75K-$120K
   - **Your position:** Emphasize strategic value they're getting
   - **Fallback range:** $50K-$75K

2. **Infrastructure Fee ($500/month)** - May want $750-$1,000
   - **Fallback:** $750/month acceptable

3. **Customer ERP Access** - This is unusual and they may resist
   - **Be prepared to:** Offer stronger data security commitments
   - **Fallback:** Aggregated data reports instead of direct access

4. **4-Year Exclusivity** - They may want 5+ years
   - **Fallback:** 5 years acceptable if other terms favorable

### CARL'S MOWER - EXPECTED DISCUSSION AREAS

1. **5% Revenue Share for 4 Years** - Very generous
   - **Total value to them:** $480K (CTO) + revenue share + free usage + commissions + YouTube revenue

2. **$10K/month CTO Fee** - May struggle with cost
   - **Alternative:** Could reduce to $7,500-$8,000 if needed
   - **Justification:** Market rate is $8K-$15K/month

3. **Total Value Proposition: $700K-$900K+**
   - CTO Services: $576K-$720K market value
   - Revenue Share: $185K estimated
   - Free Usage: $14K+ over 4 years
   - Commissions: $10K+
   - YouTube Revenue: $5K-$20K+ annually (unquantified)
