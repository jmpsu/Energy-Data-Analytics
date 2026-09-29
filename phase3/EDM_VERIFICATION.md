# SAP IS-U EDM interval structures: verification and implementation

Verified 2026-09-29 (06:00–06:20 EDT) against public SAP DDIC mirrors:
- sapdatasheet.org (`/abap/tabl/<name>.html`, which returns HTTP 302 to the home page when the object does not exist)
- tcodesearch.com (`/sap-tables/<NAME>`, which returns 404 when the object does not exist)
- sapstack.com
- the SAP IMG documentation for activity _ISUEDMPRO_000011 (status list)

help.sap.com and leanx.eu were not needed because two independent mirrors agreed on every object.

Conceptual alignment follows the user-supplied SAP Learning lesson **"Understanding the Energy Data Management Components"**. It is part of the course *Configuring Device Management in SAP S/4HANA Utilities*, https://learning.sap.com/courses/configuring-device-management-in-sap-s-4hana-utilities. Local copies: `/workspace/aws-migrate/ref/sap_edm.pdf` and `sap_edm.txt`.

## 1. The user's list compared with the real SAP objects

| User's name | Real SAP table? | Real equivalent / what it actually is | Source |
|---|---|---|---|
| EPROFVALMONTH | **Yes**, "Profile Values for One Month" | Key MANDT, PROFILE, VALUEDAY with columns **VAL01…VAL31 = one value per DAY** of the month (DEC 16). This is a month of *daily* values, **not** a month of 15-minute values. The user's layout ("a month of interval values in one row, with columns for each interval value/status") does not exist: statuses are never stored in the value tables. | https://www.sapdatasheet.org/abap/tabl/eprofvalmonth.html · https://www.tcodesearch.com/sap-tables/EPROFVALMONTH |
| EPROFVALDAY | **No** | Nothing by this name. The one-row-per-day wide 15-minute table is **EPROFVAL15**. Large-interval values (days/months/years) live in **EPROFVALDT** (PROFILE, VALUEDAY, VALUETIME, VALUE). | sapdatasheet 302 / tcodesearch 404 for EPROFVALDAY; https://www.sapdatasheet.org/abap/tabl/eprofvaldt.html |
| EPROFVAL15 | **Yes**, "Profile Values in 15-Minute Intervals" | Key MANDT, PROFILE, VALUEDAY; **96 columns VAL0000, VAL0015 … VAL2345** (DEC 16). This is the real wide layout for 15-minute AMI data. Sister tables: EPROFVAL30 (48 columns), EPROFVAL60, EPROFVAL05_1/_2, EPROFVAL10. | https://www.sapdatasheet.org/abap/tabl/eprofval15.html |
| EPROFVAL30 | **Yes**, "Profile Values in 30-Minute Intervals" | Same key; VAL0000…VAL2330. Not used here because the data is 15-minute. It could be exposed as a roll-up view if a 30-minute product were needed. | https://www.sapdatasheet.org/abap/tabl/eprofval30.html |
| EROP ("profile header") | **No** | The profile header is **EPROFHEAD** ("Profile Header Data"): key MANDT, PROFILE; SPARTE, PROFTYPE, PROFVALCAT, VALUECUM, **INTSIZEID** (interval length), MASS (unit), PROFDECIMALS, OBJNR, FORWARD_ORIENTED, DATEFROM/TIMEFROM, DATETO/TIMETO, REPLACEMETHODGRP, TIME_ZONE, SRCPROFILE, … | sapdatasheet 302 / tcodesearch 404 for EROP; https://www.sapdatasheet.org/abap/tabl/eprofhead.html |
| EPROFUSETYPE | **No** | Nothing by this name. Profile *usage* is modelled by the allocation role: **EPROFASS**-PROFROLE, checked against **EPROFASSROLE** (ROLETYPE 00 not specified / 01 measurement / 02 forecast / 03 settlement; REGISTERASS / INSTLASS flags). Profile *type* is **EPROFTYPE** (PROFTYPE, PROFCATEGORY, PROFHIST 1 historical / 2 forecast / 3 schedule). | sapdatasheet 302 / tcodesearch 404; https://www.sapdatasheet.org/abap/tabl/eprofassrole.html · https://www.sapdatasheet.org/abap/tabl/eproftype.html |
| ETGREG | **No** | Nothing by this name. The register data the user probably means: **ETDZ** (device register: EQUNR, ZWNUMMER, **LOGIKZW**, ZWART, KENNZIFF, …) and **EASTS** (installation structure of registers: ANLAGE, LOGIKZW, BIS, AB). Profiles attach to a register through **EPROFASS** (key MANDT, **LOGIKZW**, PROFROLE, DATETO, TIMETO, ROLENO → PROFILE). | sapdatasheet 302 / tcodesearch 404; https://www.sapdatasheet.org/abap/tabl/eprofass.html · https://www.sapdatasheet.org/abap/tabl/etdz.html · https://www.sapdatasheet.org/abap/tabl/easts.html |
| EPROFVALSTAT | **Yes**, "Profile Value Status" | Key MANDT, PROFILE, DATETO, TIMETO, STAT (J_STATUS CHAR 5); data DATEFROM, TIMEFROM, INACT, CHGNR. It stores **time ranges per status**, not one status per interval. Statuses are the SAP-predefined system statuses IU010–IU023 (for example IU012 valid, IU013 estimated, IU021 interpolated). | https://www.sapdatasheet.org/abap/tabl/eprofvalstat.html · https://www.sapdatasheet.org/abap/cus0/_isuedmpro_000011.html |
| EPROFVALCHGDICT | **No** | Nothing by this name. Status *change history* is **EPROFVALSTATHIST** ("History of Profile Value Status"). The status display customising is **EPROFVALSTATCUST** (STAT, ICON, PRIORITY, COLOUR). Status texts come from **TJ02T** (standard system-status texts). | sapdatasheet 302 / tcodesearch 404; https://www.sapdatasheet.org/abap/tabl/eprofvalstathist.html · https://www.sapdatasheet.org/abap/tabl/eprofvalstatcust.html |
| (EPROFHEAD, the user's belief) | **Yes** | See the EROP row. | as above |
| (EPROFASS, the user's belief) | **Yes**, "Allocation of Profiles" | See the ETGREG row. | as above |

Supporting customising tables also exist and were verified:

| Table | Content | Values used here |
|---|---|---|
| EPROFINTSIZE | Interval lengths: INTSIZEID, INTSIZE, INTSIZETYPE (1 minutes, 2 days, 3 months, 4 years) | 15MIN (INTSIZE 0015, type 1); DAY (INTSIZE 0001, type 2); CALC for unmetered |
| EPROFASSROLE | Allocation roles | CONS and ZDAY |
| EPROFTYPE | Profile types | 01 historical measured (elementary), 02 unmetered lighting (synthetic), 03 daily totals (formula) |

- **Customer namespace:** PROFTYPE, PROFROLE, INTSIZEID, CONCHECKGRP and REPLACEMETHODGRP are *customising values*, so every utility defines its own. The values chosen here are customer-namespace values, flagged as such.
- **Predefined statuses** (IMG _ISUEDMPRO_000011):

| Status | Meaning | Status | Meaning |
|---|---|---|---|
| IU010 | value not available | IU017 | locked |
| IU011 | missing | IU018 | protected |
| IU012 | valid | IU019 | archived |
| IU013 | estimated | IU020 | extrapolated |
| IU014 | implausible | IU021 | interpolated |
| IU015 | changed or entered manually | IU022 | manually changed externally |
| IU016 | released | IU023 | not in validity period |

## 1b. Alignment with the SAP Learning lesson

| Lesson concept | How it is modelled here |
|---|---|
| **Point of delivery (PoD)** with an internal ID (not visible to users) and an external PoD ID linked through **EUITRANS** | `euihead` (INT_UI, UITYPE, EUIROLE_DEREG/TECH, VSTELLE), `euitrans` (INT_UI ↔ EXT_UI with validity dates) and `euiinstln` (INT_UI ↔ ANLAGE with role flags). Fields were verified at sapdatasheet.org (euitrans, euiinstln, euihead). |
| **Deregulation PoD** is generated automatically per installation (the lesson's example: ZP081501 for installation 0815); only one per installation | One per installation. EXT_UI = `ZP<ANLAGE>01` (e.g. ZP500000051201 for installation 5000000512); EUIROLE_DEREG = 'X'. |
| **Technical PoD** is optional. It is needed when the AMR system uses its own number instead of the market PoD ID. | The AMI head-end keys reads by meter number, so every metered installation also gets a technical PoD with EXT_UI = the meter serial (EQUI-SERNR) and EUIROLE_TECH = 'X'. Unmetered lighting has only the deregulation PoD. |
| **Device with registers and register codes** (REG1 active energy 15 min; REG2 reactive energy 15 min) | ETDZ register 001 is active energy (KWH, OBIS 1.8.0, 15MIN) and carries the profiles. 002 is kW demand and 003 is received kWh (PV). **Reactive energy (REG2) is not modelled**, because the AMI store of record has no kvarh channel; this is a documented gap. |
| **Profiles allocated to registers with a profile role**; default role category Measurement / role Consumption; Forecast is a separate role | EPROFASS role **CONS** (Consumption, ROLETYPE 01 Measurement, REGISTERASS 'X') → 15-minute measured profile. Role **ZDAY** (Measurement) → daily-totals profile. No Forecast role, because no forecasts exist. |
| **Profile header** fields: division, status (Active/Inactive/Deletion flag), profile type, interval length, unit, validity dates, day offset, archived-until date | EPROFHEAD fields in the same order: SPARTE '01' (electricity); LOEVM = '' (no deletion flag; the object status via OBJNR is Active); PROFTYPE; INTSIZEID; MASS = KWH; DATEFROM/TIMEFROM–DATETO/TIMETO; DAY_OFFSET 000000; ARCH_DATETO/ARCH_TIMETO = 00000000/000000 (nothing archived). |
| **Five predefined profile *categories*** (elementary, synthetic, formula, day, integral; not customisable) and customisable profile *types* (historical, forecast, schedule, …) | `eproftype` (PROFTYPE, PROFCATEGORY, PROFHIST): 01 = historical measured, category elementary. 02 = unmetered lighting, category synthetic. 03 = daily totals derived from the 15-minute profile (SRCPROFILE), category formula. PROFHIST 01 = historical. The numeric PROFCATEGORY codes (01 elementary, 02 synthetic, 03 formula) follow the lesson's order and are an **assumption**, because the domain values live in value table EPROFCATEGORY and could not be read online. |
| **TOU billing via the RTP interface**: consumption blocks by season, day type and time of day; peak values (max demand) | This is not a new table, and billing is unchanged. The existing billing determinants (`billing_determinants`, ERCH/DBERCHZ) already carry exactly these RTP-interface outputs per billing period: `kwh_onpeak`, `kwh_sdtr_on`, `kwh_optb_on` (consumption blocks by FPL season, time-of-day window and day type, with holidays from ZTOU_HOLIDAY) and `kw_max`, `kw_onpeak`, `kw_sdtr`, `kw_optb` (peak demand). They are computed from the same 15-minute values that EPROFVAL15 exposes, so EDM → RTP → billing reconciles (§4). |

## 2. What was built

The flat AMI Parquet (`utl_local`) stays the storage of record and nothing was regenerated. Code: `edm/edm_layer.py`.

| Glue/Athena object | Kind | Real SAP key / columns | Derivation |
|---|---|---|---|
| `euihead`, `euitrans`, `euiinstln` | views | real PoD fields (INT_UI, EXT_UI, EUIROLE_DEREG/TECH, ANLAGE, validity) | Deregulation PoD `ZP<ANLAGE>01` per installation; technical PoD = meter serial for metered installations |
| `eprofhead` | view | MANDT, PROFILE + the 38 real header fields | One 15-minute profile per register-001 (kWh) and one daily-consumption profile (SRCPROFILE = the 15-minute profile) |
| `eprofass` | view | MANDT, LOGIKZW, PROFROLE, DATETO, TIMETO, ROLENO, DATEFROM, TIMEFROM, PROFILE, CONTEXTCATEGORY, PROFROLECONTEXT | Register → profile (roles CONS = Consumption/Measurement, ZDAY) |
| `easts` | view | MANDT, ANLAGE, LOGIKZW, BIS, AB | Installation → register (ties EDM to EANL/EVER) |
| `eprofval15` | view | MANDT, PROFILE, VALUEDAY, VAL0000…VAL2345 | Pivot of `utl_local`, one row per profile per UTC day. Technical columns `zz_anlage`, `division`, `anlage_bucket` are carried for partition pruning. |
| `eprofvalmonth` | view | MANDT, PROFILE, VALUEDAY (first of month), VAL01…VAL31 | Daily totals of the daily-consumption profile |
| `eprofvalstat` | CTAS Parquet table (148 MB for 3.73M profiles) | MANDT, PROFILE, DATETO, TIMETO, STAT, DATEFROM, TIMEFROM, INACT, CHGNR | Deterministic VEE provenance (see §3) |
| `eprofintsize`, `eprofassrole`, `eproftype`, `tj02t_edm` | views | real fields | Customising and status texts |

**Join path**
- PoD: EUITRANS.EXT_UI (e.g. ZP500000051201) → INT_UI → EUIINSTLN.ANLAGE.
- EANL.ANLAGE → EASTS.LOGIKZW → EPROFASS (PROFROLE ZI15/ZDAY) → EPROFHEAD / EPROFVAL15 / EPROFVALMONTH / EPROFVALSTAT on PROFILE.
- Device: EASTS.LOGIKZW = EGERH.LOGIKNR×1000 + ETDZ.ZWNUMMER, then EQUNR.
- Contract account: EVER.ANLAGE → VKONTO.

**Deterministic keys**
- LOGIKZW = LOGIKNR×1000 + ZWNUMMER.
- PROFILE = EQUNR×10 + 1 for the 15-minute profile and EQUNR×10 + 2 for the daily profile.
- Because EQUNR = `utl_local.meter_id`, interval rows map to their profile without a join.

**Time and units**
- VALUEDAY, VALhhmm and status ranges are in **UTC**, which SAP Community describes as the EDM database format. Display converts through EPROFHEAD-TIME_ZONE (`EST`).
- Every UTC day therefore has exactly 96 values, even on DST days.
- Values are interval-*beginning* labelled (VAL0000 = 00:00–00:15 UTC), with FORWARD_ORIENTED = 'X'.
- Units: kWh per interval (VALUECUM = 'X', MASS = KWH, 3 decimals).
- **Confidence: medium.** The UTC database-storage convention comes from a secondary source. If the target system stores standard time instead, only the `date_add` offset in the views changes.

## 3. Quality statuses (EPROFVALSTAT)

**Store of record has no gaps**
- Phase 3 AMI has no missing intervals; every metered installation has 35,040 reads.
- The values in `utl_local` are therefore treated as the **final, post-VEE (validation-estimation-editing) values that were billed**.
- EPROFVALSTAT records which of those values the meter data management system substituted. Values are unchanged, so interval sums still reconcile exactly to billing.

**Event model** (deterministic hash of PROFILE)
- 0, 1, 2, 3, 4 or 6 events per meter-year, with probabilities 20 / 25 / 20 / 15 / 10 / 10%.
- 70% are short communications drop-outs of 1–4 intervals → **IU021 interpolated**.
- 29% are 2–24 h outages → **IU013 estimated**.
- 1% are manual corrections → **IU015**.
- Everything else is **IU012 valid**.
- Unmetered lighting (register INTSIZEID CALC, `read_quality = 'E'` in AMI) → IU013 for the whole window.

**Results on the current data (3,725,929 profiles)**

| Status | Rows | Profiles | Intervals |
|---|---|---|---|
| IU012 valid | 11,517,158 | 3,725,929 | 130,465,117,290 |
| IU013 estimated | 2,279,574 | 1,617,546 | 758,530,945 (of which 680.8M are the 19,429 unmetered lighting profiles; metered ≈ 77.7M) |
| IU021 interpolated | 5,457,036 | 2,614,746 | 13,502,872 |
| IU015 manual | 78,119 | 77,100 | 193,213 |

- About 0.07% of metered intervals are substituted, and about 99.4% of meter-days are complete. This is consistent with the typical AMI contract SLA of 99.5% of interval reads delivered.
  - Source: Enspiria/EnergyCentral, "Validating Field Performance of AMI Systems", https://electricenergyonline.com/energy/magazine/443/article/validating-field-performance-of-ami-systems.htm
- **Tiling check:** in a 0.1% sample every profile's status ranges tile its window exactly (35,040 intervals, with no gaps or overlaps).

## 4. Reconciliation

For the nine sample customers (sample files `phase3_out/edm/edm_samples.{xlsx,html}` and `edm_sample_*.csv`, including the `point_of_delivery` sheet) (RS1, RS1EV, RTR1, GS1, GSD1, GSDT1, GSLDT1, CILC1D, SL1), the sum of EPROFVALMONTH over the service year equals the sum of the billing-determinant interval kWh **exactly (difference 0.000 kWh)**. It also equals billed HISTKWH to within rounding.

## 5. Known deviations and limits

**Our own model**
- Our generated ETDZ carries a non-standard column INTSIZEID; real ETDZ has none, because interval length belongs in EPROFHEAD.
- The views take LOGIKZW from the formula above. For the v5 rebuild, `gen3/sap3.py` writes LOGIKZW into ETDZ with the same formula.

**Query performance**
- The `eprofval15` and `eprofvalmonth` views group by UTC day, so they cannot prune on the local `year`/`month` partitions.
- Filter on `division`, `anlage_bucket` (= ANLAGE mod 32) and `zz_anlage`: the sample queries scanned 60–340 MB each.
- Full-population scans of these views cost about as much as scanning `utl_local`.

**Storage location**
- The CTAS output sits under the workgroup-enforced `athena-results/tables/<query-id>/` prefix, because the workgroup rejects `external_location`.
- Rerunning after the v5 rebuild needs `DROP TABLE eprofvalstat` and a new CTAS. The old prefix can then be deleted with the user's approval.

**Tables deliberately not modelled**
- EPROFVALSTATHIST (no status history is simulated).
- EPROFVERSSTAT and profile versions.
- EPROFVAL30/60: data is 15-minute only.
- EEDM replacement-value customising tables.
