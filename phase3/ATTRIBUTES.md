# Phase 3 customer attributes: definitions, rates, sources

All values are synthetic and calibrated to the cited public benchmarks. Coverage: 100% of each class (no nulls). Other-class columns in the flat `analytics.customers` table read `NOT_APPLICABLE` (text) or 0 (numeric).
Statewide % and segment % are measured on the generated 6,000,000-installation population (`attrs_master.parquet`).


## Residential

### R1. ZCIS_RES_ATTR.ZZ_CLOTHES_WASHER (+ZZ_DRYER_FUEL)
- **Definition:** Clothes washer present in the home (Y/N); dryer fuel ELECTRIC/GAS/NONE (gas only where the premise has gas service).
- **Statewide (headline):** 88.2%
- **By segment:** SFD: 97.7%; SFA: 93.6%; MF24: 69.7%; MF5: 72.9%; MOBILE: 93.8% | Owner: 94.4%; Renter: 73.0% | inc <50k: 78.5%; inc 50-100k: 89.8%; inc 100-150k: 93.7%; inc 150k+: 96.1%
- **Source / calibration:** EIA RECS 2020 microdata, FL weighted (CWASHER 88.2%; DRYRFUEL electric 85.1%); weighted logit fitted on RECS South Atlantic, intercept re-calibrated to FL.
- **Conditional drivers:** structure type (MF 2-4 / 5+ much lower), tenure (renters lower), household income, sqft, year built, householder age
- **Load-shape manifestation:** Washer=Y homes carry the NREL ResStock washer+dryer end-use profile (model's own, or county donor profile if the model had none); washer=N homes have it removed -> evening/weekend laundry peaks appear only in washer homes.

### R2. ZCIS_RES_ATTR.ZZ_POOL_PUMP
- **Definition:** Private swimming pool and pump type: NONE / SINGLE_SPEED / VARIABLE_SPEED.
- **Statewide (headline):** 18.8%
- **By segment:** SFD: 33.9%; SFA: 9.1%; MF24: 0.0%; MF5: 0.0%; MOBILE: 12.2% | Owner: 24.8%; Renter: 4.1% | inc <50k: 11.7%; inc 50-100k: 17.9%; inc 100-150k: 22.9%; inc 150k+: 27.3%
- **Source / calibration:** RECS 2020 FL (SWIMPOOL 18.8%; POOLPUMP variable-speed 9.4% = 50% of pools). Private pools only on single-family/mobile (RECS: not applicable to 2+ unit apartments).
- **Conditional drivers:** structure type (0 in apartments), tenure, income, sqft, year built
- **Load-shape manifestation:** Pool homes carry the ResStock pool-pump profile (x0.40 energy for variable-speed); non-pool homes have it removed. Daytime pump runtime visible year-round (FL).

### R3. ZCIS_RES_ATTR.ZZ_EV_COUNT, ZZ_EV_CHARGER_LEVEL, ZZ_EV_CHARGING_WINDOW (+EANL EV_KWH_YR)
- **Definition:** EVs in household (0/1/2), home charger L1_120V/L2_240V, dominant home-charging window EVENING_ARRIVAL / OVERNIGHT_SCHEDULED / DAYTIME_HOME / MOSTLY_AWAY_PUBLIC.
- **Statewide (headline):** 4.0%
- **By segment:** SFD: 4.9%; SFA: 4.3%; MF24: 1.9%; MF5: 3.0%; MOBILE: 2.8% | Owner: 4.8%; Renter: 2.0% | inc <50k: 1.3%; inc 50-100k: 3.1%; inc 100-150k: 4.9%; inc 150k+: 8.6%
- **Source / calibration:** AFDC/Experian 2024 FL plug-in registrations (334.8k BEV + 70.4k PHEV) / 1.15 EVs per EV household / ACS households -> 4.0%; RECS 2020 ELECVEH & EVCHRGTYPE (L2 ~55%); INL 'Plugged In' (TOU shifts charging to off-peak start).
- **Conditional drivers:** income (strongest), tenure, sqft, age; window from occupancy pattern and tariff (RS-1EV service and RTR-1 -> overnight scheduled)
- **Load-shape manifestation:** Charging sessions are synthesised at the window's start time (evening ~18:15, overnight 00:00, daytime ~12:30) at 7.2 kW (L2) or 1.4 kW (L1), 3.5 or 6 sessions/week, only on occupied days.

### R4. ZCIS_RES_ATTR.ZZ_PV_KW (+ZCIS_PREMISE_ATTR.PV_KW, NET_METERING)
- **Definition:** Rooftop PV system size kW-dc (0 = none).
- **Statewide (headline):** 2.0%
- **By segment:** SFD: 3.5%; SFA: 1.3%; MF24: 0.0%; MF5: 0.0%; MOBILE: 1.6% | Owner: 2.8%; Renter: 0.0% | inc <50k: 0.9%; inc 50-100k: 1.8%; inc 100-150k: 2.5%; inc 150k+: 3.5%
- **Source / calibration:** FPSC Net Metering reports: 292,284 FL customer-owned systems (2024) x FPL 37% share (92,438 of 249,521 in 2023) -> ~2.0% of residential; size LBNL Tracking the Sun (FL median ~8.5 kW).
- **Conditional drivers:** owner-occupied single-family/mobile only; income, householder age, year built
- **Load-shape manifestation:** Delivered channel = load minus clear-sky x cloudiness PV profile (midday dip / export zeroed).

### R5. ZCIS_RES_ATTR.ZZ_BACKUP_POWER
- **Definition:** NONE / PORTABLE_GENERATOR / STANDBY_GENERATOR / BATTERY_STORAGE (priority standby > battery > portable).
- **Statewide (headline):** 24.2%
- **By segment:** SFD: 32.6%; SFA: 7.7%; MF24: 6.2%; MF5: 16.6%; MOBILE: 29.3% | Owner: 28.9%; Renter: 12.7% | inc <50k: 17.5%; inc 50-100k: 23.0%; inc 100-150k: 27.3%; inc 150k+: 33.3%
- **Source / calibration:** RECS 2020 FL BACKUP 22.1% any generator; Generac disclosures: home-standby penetration 6.5% of US SFD owner homes; LBNL Tracking the Sun: FL storage attachment ~10-15% of PV (lower-confidence).
- **Conditional drivers:** structure type, tenure, income, age; standby only SFD owners; battery only PV homes
- **Load-shape manifestation:** No routine interval effect (resilience attribute); battery homes = PV homes (used for outage-restoration and resiliency programme targeting).

### R6. ZCIS_RES_ATTR.ZZ_WATER_HEATER (+ZCIS_PREMISE_ATTR.WATER_HEAT_FUEL)
- **Definition:** ELECTRIC_TANK / ELECTRIC_TANKLESS / HEAT_PUMP_WH / GAS_TANK / GAS_TANKLESS / PROPANE / SOLAR_THERMAL.
- **Statewide (headline):** 91.8%
- **By segment:** SFD: 88.0%; SFA: 91.8%; MF24: 96.1%; MF5: 97.2%; MOBILE: 94.7% | Owner: 90.9%; Renter: 94.1% | inc <50k: 91.8%; inc 50-100k: 91.7%; inc 100-150k: 92.1%; inc 150k+: 91.9%
- **Source / calibration:** Inherited from the assigned NREL ResStock 2025.1 model (RECS-calibrated); RECS 2020 FL electric 87.9% as benchmark.
- **Conditional drivers:** county x structure type x income/tenure-matched ResStock model
- **Load-shape manifestation:** Consistent by construction: the model's hot-water end use (morning/evening draws) is in its load profile only if electric.

### R7. ZCIS_RES_ATTR.ZZ_HVAC_SYSTEM, ZZ_HVAC_SEER, ZZ_HVAC_AGE_BAND
- **Definition:** HVAC system (CENTRAL_HEAT_PUMP, CENTRAL_AC_ELEC_FURNACE, ... ROOM_AC, NO_AC), rated SEER, equipment age band.
- **Statewide (headline):** 44.8%
- **By segment:** SFD: 47.1%; SFA: 44.1%; MF24: 53.6%; MF5: 37.1%; MOBILE: 54.2% | Owner: 45.8%; Renter: 42.2% | inc <50k: 44.0%; inc 50-100k: 44.7%; inc 100-150k: 44.7%; inc 150k+: 46.1%
- **Source / calibration:** Type/efficiency inherited from ResStock model; age band from RECS 2020 FL ACEQUIPAGE by vintage (capped by home age).
- **Conditional drivers:** county, structure type, vintage (newer homes: newer, higher-SEER equipment)
- **Load-shape manifestation:** Cooling/heating end-use of the model drives summer afternoon and winter-morning peaks; low-SEER/old units = larger cooling component.

### R8. ZCIS_RES_ATTR.ZZ_THERMOSTAT
- **Definition:** SMART_CONNECTED / PROGRAMMABLE / MANUAL / NONE (room-AC and no-AC homes = NONE).
- **Statewide (headline):** 12.3%
- **By segment:** SFD: 14.9%; SFA: 14.7%; MF24: 6.3%; MF5: 9.9%; MOBILE: 4.1% | Owner: 15.2%; Renter: 5.0% | inc <50k: 4.5%; inc 50-100k: 10.2%; inc 100-150k: 15.5%; inc 150k+: 24.0%
- **Source / calibration:** RECS 2020 FL TYPETHERM: smart 12.3%, programmable 49.3%.
- **Conditional drivers:** income, tenure, age (older lower), structure type, year built; ducted systems only
- **Load-shape manifestation:** Smart homes pre-cool 12-15h (+10% cooling) and set back 17-20h (-12%) on weekdays -> visible on-peak cooling reduction.

### R9. ZCIS_RES_ATTR.ZZ_OCCUPANCY_PATTERN
- **Definition:** COMMUTER_DAYTIME_AWAY / WORK_FROM_HOME / RETIREE_HOME_DAYTIME / SNOWBIRD_WINTER / SUMMER_SEASONAL.
- **Statewide (headline):** 38.8%
- **By segment:** SFD: 42.2%; SFA: 43.5%; MF24: 30.9%; MF5: 34.4%; MOBILE: 33.6% | Owner: 42.4%; Renter: 30.0% | inc <50k: 27.9%; inc 50-100k: 35.9%; inc 100-150k: 43.7%; inc 150k+: 55.1%
- **Source / calibration:** ACS 2024 B25004 (seasonal), B25007 (householder age by tenure), B08301 (15.7% of FL workers WFH); BLS CPS (80% of 65+ not in labour force); RECS TELLWORK logit.
- **Conditional drivers:** householder age, seasonal status, ZCTA WFH share, income, structure type
- **Load-shape manifestation:** Weekday 9-16h non-HVAC load x0.75 (commuter) / x1.15 (WFH) / x1.20 (retiree), cooling x0.90/1.08; seasonal homes drop to a vacancy base load outside arrival-departure dates.

### R10. ZCIS_RES_ATTR.ZZ_MEDICAL_EQUIPMENT (+ZCIS_BP_ATTR.MEDICAL_CERT)
- **Definition:** NONE / NON_CRITICAL (e.g. CPAP) / LIFE_SUPPORT (oxygen concentrator, ventilator, dialysis...). MEDICAL_CERT (FPL Medically Essential Service) is a 35% subset of LIFE_SUPPORT.
- **Statewide (headline):** 13.8%
- **By segment:** SFD: 15.6%; SFA: 10.9%; MF24: 15.1%; MF5: 9.2%; MOBILE: 23.9% | Owner: 14.6%; Renter: 11.9% | inc <50k: 12.7%; inc 50-100k: 13.8%; inc 100-150k: 14.4%; inc 150k+: 14.9%
- **Source / calibration:** RECS 2020 FL MEDICALDEV 13.8%; HHS emPOWER 2026-03: 192,924 FL electricity-dependent Medicare beneficiaries (~2.2% of households).
- **Conditional drivers:** householder age 65+, income, structure type
- **Load-shape manifestation:** LIFE_SUPPORT adds a constant ~0.35 kW 24x7 base; NON_CRITICAL adds ~0.10 kW overnight (23-07h).


## Commercial

### C1. ZCIS_COM_ATTR.ZZ_OPER_HOURS_WK, ZZ_OPER_HOURS_CLASS
- **Definition:** Weekly operating hours and class STANDARD_LE50H / EXTENDED_51_84H / LONG_85_167H / CONTINUOUS_24X7.
- **Statewide (headline):** 20.7%
- **By segment:** smalloffice: 7.2%; custom_mf_common: 100.0%; outpatient: 2.1%; retailstandalone: 3.7%; retailstripmall: 0.2%; warehouse: 13.9%
- **Source / calibration:** CBECS 2018 South microdata by principal building activity (WKHRS quartiles, OPEN24).
- **Conditional drivers:** building activity (NAICS->ComStock/PBA), size
- **Load-shape manifestation:** Sets the flat (after-hours) fraction of the ComStock shape: 24x7 sites have high night load, standard-hours offices a deep night valley.

### C2. ZCIS_COM_ATTR.ZZ_REFRIG_SHARE_PCT
- **Definition:** Refrigeration share of site electricity (%).
- **Statewide (headline):** 5.8%
- **By segment:** smalloffice: 0.0%; custom_mf_common: 0.0%; outpatient: 0.0%; retailstandalone: 0.1%; retailstripmall: 9.4%; warehouse: 2.9%
- **Source / calibration:** CBECS 2018 South end-use electricity (ELRFBTU/ELBTU) by activity: food sales 55%, food service 24%, lodging 10%...
- **Conditional drivers:** building activity; refrigerated warehouses (NAICS 49312) 45-70%
- **Load-shape manifestation:** Adds flat 24x7 base (compressor load), raising load factor.

### C3. ZCIS_COM_ATTR.ZZ_BAS_PRESENT
- **Definition:** Building automation / EMCS present (Y/N).
- **Statewide (headline):** 14.6%
- **By segment:** smalloffice: 8.9%; custom_mf_common: 42.1%; outpatient: 6.4%; retailstandalone: 14.2%; retailstripmall: 14.3%; warehouse: 12.1%
- **Source / calibration:** CBECS 2018 South EMCS by floor-area class (9% <5k sqft ... 91% >1M sqft) x activity factor; national chains x1.6.
- **Conditional drivers:** floor area, activity, chain
- **Load-shape manifestation:** BAS sites have 20% lower after-hours base (scheduled setback).

### C4. ZCIS_COM_ATTR.ZZ_DEMAND_FLEX_KW
- **Definition:** Estimated curtailable kW in a 2-4 h DR event.
- **Statewide (headline):** 4.2%
- **By segment:** smalloffice: 0.2%; custom_mf_common: 18.9%; outpatient: 0.5%; retailstandalone: 4.0%; retailstripmall: 0.3%; warehouse: 1.6%
- **Source / calibration:** LBNL 2025 California DR Potential Study / DOE GEB: ~5% of peak without controls, +10-15% with BAS, refrigeration float.
- **Conditional drivers:** peak kW, BAS, refrigeration share, 24x7
- **Load-shape manifestation:** Derived from load (peak kW) and controllability; used for DR/TOU targeting.

### C5. ZCIS_COM_ATTR.ZZ_BACKUP_GENERATION, ZZ_BACKUP_GEN_KW
- **Definition:** NONE / DIESEL_STANDBY / NATGAS_STANDBY / TRANSFER_SWITCH_ONLY.
- **Statewide (headline):** 14.6%
- **By segment:** smalloffice: 7.8%; custom_mf_common: 34.4%; outpatient: 8.9%; retailstandalone: 7.0%; retailstripmall: 7.2%; warehouse: 7.0%
- **Source / calibration:** CBECS 2018 South CAPGEN by size and activity (97.5% emergency backup); FL AHCA rules (nursing homes/ALFs, post-2017) and F.S. 526.143 (fuel-station generator readiness); hospitals 100%.
- **Conditional drivers:** size, activity, NAICS (6231/6233, 4471/4571), gas availability
- **Load-shape manifestation:** Resilience attribute (no routine interval effect).

### C6. ZCIS_COM_ATTR.ZZ_EV_PORTS_L2, ZZ_EV_PORTS_DCFC
- **Definition:** On-site EV charging ports.
- **Statewide (headline):** 0.5%
- **By segment:** smalloffice: 0.1%; custom_mf_common: 1.5%; outpatient: 0.0%; retailstandalone: 1.6%; retailstripmall: 0.1%; warehouse: 0.1%
- **Source / calibration:** DOE AFDC Station Locator 2026-09: FL 4,352 public locations, 15,309 ports (5,063 DCFC); FPL ~50% share + workplace/fleet -> ~3,100 sites.
- **Conditional drivers:** NAICS (auto dealers, hotels, grocery, large office), size
- **Load-shape manifestation:** Adds L2 daytime (8-18h) and DCFC (7-22h, 20% utilisation, evening bump) charging load to the site's intervals.

### C7. ZCIS_COM_ATTR.ZZ_OCCUPANCY_SEASONALITY
- **Definition:** YEAR_ROUND / WINTER_PEAK (snowbird) / SUMMER_PEAK (NW beach) / SCHOOL_CALENDAR.
- **Statewide (headline):** 8.1%
- **By segment:** smalloffice: 0.0%; custom_mf_common: 0.0%; outpatient: 26.1%; retailstandalone: 23.7%; retailstripmall: 25.1%; warehouse: 0.0%
- **Source / calibration:** ACS 2024 B25004 seasonal-unit share by county (x3 odds for hospitality/retail/worship/golf); NW beach-county summer tourism; FDOE school calendars.
- **Conditional drivers:** county snowbird share, activity
- **Load-shape manifestation:** Monthly multiplier on the load shape (winter-peak Jan-Mar x1.3, Jul-Sep x0.8; summer-peak mirror).

### C8. ZCIS_COM_ATTR.ZZ_ELECTRIFICATION_READINESS, ZZ_ELECTRIFICATION_MWH_POTENTIAL
- **Definition:** ALL_ELECTRIC / GAS_COOKING / GAS_WATER_OR_SPACE_HEAT / GAS_COOKING_AND_HEAT + MWh/yr if electrified.
- **Statewide (headline):** 6.0%
- **By segment:** smalloffice: 3.6%; custom_mf_common: 4.2%; outpatient: 4.0%; retailstandalone: 3.2%; retailstripmall: 8.4%; warehouse: 2.5%
- **Source / calibration:** EIA Natural Gas Annual 2024: 76,413 FL commercial gas consumers (~6% of commercial accounts); CBECS 2018 South NGUSED by activity for the mix.
- **Conditional drivers:** activity (restaurants, hospitals, hotels), county gas-LDC availability
- **Load-shape manifestation:** Planning attribute; gas sites carry no electric cooking/water-heat load in the shape mix.

### C9. ZCIS_COM_ATTR.ZZ_FLOOR_AREA_SQFT, ZZ_EUI_KWH_SQFT
- **Definition:** Floor area and billed electricity intensity (kWh/sqft/yr, from 2025 billed kWh).
- **Statewide (headline):** median sqft 1050; plan-EUI median 11.4 kWh/sqft
- **By segment:** median sqft 1050; plan-EUI median 11.4 kWh/sqft
- **Source / calibration:** CBECS 2018 South median EUI by activity with customer-level dispersion (lognormal sd 0.35).
- **Conditional drivers:** activity, annual kWh
- **Load-shape manifestation:** EUI is computed from the interval-billed kWh, so it ties to the reads exactly.

### C10. ZCIS_COM_ATTR.ZZ_TENANCY
- **Definition:** OWNER_OCCUPIED / LEASED_SINGLE_TENANT / LEASED_MULTI_TENANT.
- **Statewide (headline):** 70.2%
- **By segment:** smalloffice: 70.9%; custom_mf_common: 94.9%; outpatient: 60.0%; retailstandalone: 55.4%; retailstripmall: 28.1%; warehouse: 63.7%
- **Source / calibration:** CBECS 2018 South OWNOCC by activity; strip malls multi-tenant; chain retail/QSR mostly leased.
- **Conditional drivers:** activity, chain
- **Load-shape manifestation:** Behavioural (split incentive for DR/EE offers).


## Industrial

### I1. ZCIS_IND_ATTR.ZZ_OPERATING_SCHEDULE
- **Definition:** ONE_SHIFT / TWO_SHIFT / THREE_SHIFT_5D / CONTINUOUS_7D / SEASONAL_AG_DAYLIGHT.
- **Statewide (headline):** 8.2%
- **By segment:** agriculture: 10.2%; manufacturing: 6.4%
- **Source / calibration:** Census QPC plant-hours pattern by NAICS3 (distribution table in gen3/attrs.SHIFT_P; QPC-informed, lower-confidence), USDA practice for agriculture.
- **Conditional drivers:** NAICS3, employees (small plants skew one shift)
- **Load-shape manifestation:** Drives the industrial hourly profile (shift windows, weekend level) - replaces the former random shift draw.

### I2. ZCIS_IND_ATTR.ZZ_PROCESS_CONTINUITY
- **Definition:** BATCH_TOLERANT / CONTINUOUS_TOLERANT / CONTINUOUS_CRITICAL.
- **Statewide (headline):** 7.7%
- **By segment:** agriculture: 10.2%; manufacturing: 5.3%
- **Source / calibration:** LBNL ICE Calculator 2.0 sector interruption costs; process-industry NAICS (322/325/326/331/311, poultry/dairy).
- **Conditional drivers:** NAICS3, schedule
- **Load-shape manifestation:** Critical sites have flat 24x7 profiles and low curtailable share.

### I3. ZCIS_IND_ATTR.ZZ_LOAD_CONTROL_PROGRAM, ZZ_CURTAILABLE_KW
- **Definition:** CILC enrolment (from tariff) and curtailable kW.
- **Statewide (headline):** 5.6%
- **By segment:** agriculture: 9.5%; manufacturing: 1.9%
- **Source / calibration:** FPL MFR E-13c 2026 (CILC load-control billing kW ~1,351 kW/bill); LBNL DR potential shares by continuity; irrigation pumps highly shiftable.
- **Conditional drivers:** tariff, continuity, irrigation, peak kW
- **Load-shape manifestation:** Curtailable kW scales with site peak; CILC sites 60-90% of peak.

### I4. ZCIS_IND_ATTR.ZZ_PQ_SENSITIVITY
- **Definition:** LOW / MEDIUM / HIGH power-quality sensitivity.
- **Statewide (headline):** 29.9%
- **By segment:** agriculture: 0.0%; manufacturing: 57.8%
- **Source / calibration:** LBNL ICE 2.0 / EPRI DPQ sector sensitivity (electronics, plastics, printing, food processing high).
- **Conditional drivers:** NAICS3, continuity
- **Load-shape manifestation:** Reliability attribute (no interval effect).

### I5. ZCIS_IND_ATTR.ZZ_ONSITE_GENERATION, ZZ_ONSITE_GEN_KW
- **Definition:** NONE / STANDBY_DIESEL / CHP_COGENERATION / SOLAR_PV.
- **Statewide (headline):** 17.4%
- **By segment:** agriculture: 18.9%; manufacturing: 16.0%
- **Source / calibration:** DOE CHP Installation Database (FL: sugar, paper, chemicals); standby-service (SST) tariff implies on-site generation; poultry ventilation backup practice.
- **Conditional drivers:** tariff (SST/ISST -> CHP), NAICS, size
- **Load-shape manifestation:** CHP sites are the standby-service customers (consistent with tariff).

### I6. ZCIS_IND_ATTR.ZZ_MOTOR_PUMP_SHARE_PCT
- **Definition:** Machine-drive / pumping share of site electricity (%).
- **Statewide (headline):** mean 52.0%
- **By segment:** mean 52.0%
- **Source / calibration:** EIA MECS 2018 Table 5.2 machine-drive share by NAICS3 (e.g. 311 41%, 321 75%, 322 76%, 332 41%); irrigation farms 75%.
- **Conditional drivers:** NAICS3, irrigation
- **Load-shape manifestation:** Motor-heavy sites show flat shift-level plateaus; used for VFD/efficiency targeting.

### I7. ZCIS_IND_ATTR.ZZ_PROCESS_HEAT_FUEL, ZZ_ELECTRIFICATION_MWH_POTENTIAL_IND
- **Definition:** NONE / NATURAL_GAS / PROPANE / ELECTRIC / BIOMASS_STEAM + MWh/yr electrification potential.
- **Statewide (headline):** 48.0%
- **By segment:** agriculture: 0.0%; manufacturing: 92.7%
- **Source / calibration:** EIA MECS 2018 Table 5.2 process-heating fuel by NAICS3; gas-LDC availability by county.
- **Conditional drivers:** NAICS3, county
- **Load-shape manifestation:** Planning attribute (future load growth).

### I8. ZCIS_IND_ATTR.ZZ_IRRIGATION_PUMPING
- **Definition:** NONE / ELECTRIC_WELL_PUMP / MICRO_IRRIGATION_BOOSTER.
- **Statewide (headline):** 25.6%
- **By segment:** agriculture: 53.0%; manufacturing: 0.0%
- **Source / calibration:** USDA 2023 Irrigation & Water Management Survey (FL: electricity is the main pump energy) - rate 70% of crop farms is a lower-confidence anchor.
- **Conditional drivers:** NAICS (111 crop, 112 animal, 115 support)
- **Load-shape manifestation:** Irrigation farms show the spring dry-season (Mar-May) daytime pumping peak; non-irrigated farms are flatter.

### I9. ZCIS_IND_ATTR.ZZ_COLD_STORAGE
- **Definition:** NONE / COOLER / FREEZER / PACKING_HOUSE_PRECOOL / MILK_COOLING.
- **Statewide (headline):** 13.4%
- **By segment:** agriculture: 15.9%; manufacturing: 11.0%
- **Source / calibration:** MECS 2018 process-cooling share (food 26%); USDA dairy/packing practice.
- **Conditional drivers:** NAICS (311/312, dairy 11212, crop packing)
- **Load-shape manifestation:** Adds a flat 24x7 refrigeration base to the site profile.

### I10. ZCIS_IND_ATTR.ZZ_EXPANSION_LIKELIHOOD, ZZ_EXPANSION_SCORE
- **Definition:** LOW / MEDIUM / HIGH three-year expansion likelihood (score 0-100).
- **Statewide (headline):** 21.6%
- **By segment:** agriculture: 0.2%; manufacturing: 41.5%
- **Source / calibration:** BLS QCEW FL private employment growth 2019-2024 by NAICS3 (e.g. 325 +22.7%, 336 +20.3%, 111 -6.3%).
- **Conditional drivers:** NAICS3 growth, firm size
- **Load-shape manifestation:** Planning attribute (load-growth forecasting).


## Lighting

### L1. ZCIS_LGT_ATTR.ZZ_FIXTURE_TECH
- **Definition:** LED / HPS / LED_SIGNAL.
- **Statewide (headline):** LED: 48.9%; HPS: 38.7%; LED_SIGNAL: 12.5%
- **By segment:** LED: 48.9%; HPS: 38.7%; LED_SIGNAL: 12.5%
- **Source / calibration:** FPL MFR E-13d 2026 lighting billing units (LT-1 LED 9.6M vs SL-1 1.2M fixture-months).
- **Conditional drivers:** tariff (LT-1 = LED)
- **Load-shape manifestation:** Fixture watts and the dusk-to-dawn kWh.

### L2. ZCIS_LGT_ATTR.ZZ_CONTROL_TYPE
- **Definition:** PHOTOCELL / SMART_NODE_DIMMABLE / ALWAYS_ON_24X7.
- **Statewide (headline):** SMART_NODE_DIMMABLE: 7.5%; PHOTOCELL: 80.0%; ALWAYS_ON_24X7: 12.5%
- **By segment:** SMART_NODE_DIMMABLE: 7.5%; PHOTOCELL: 80.0%; ALWAYS_ON_24X7: 12.5%
- **Source / calibration:** ANSI C136.41 nodes; DOE MSSLC connected-lighting guidance (~15% networked, lower-confidence); signals 24x7.
- **Conditional drivers:** fixture tech, tariff
- **Load-shape manifestation:** Photocell = astronomical dusk-dawn; metered smart nodes dim 30% 00-05h; signals flat 24x7.

### L3. ZCIS_LGT_ATTR.ZZ_OWNERSHIP
- **Definition:** COMPANY_OWNED_MAINTAINED / CUSTOMER_OWNED_SIGNAL / CUSTOMER_OWNED_METERED.
- **Statewide (headline):** COMPANY_OWNED_MAINTAINED: 84.0%; CUSTOMER_OWNED_SIGNAL: 12.5%; CUSTOMER_OWNED_METERED: 3.5%
- **By segment:** COMPANY_OWNED_MAINTAINED: 84.0%; CUSTOMER_OWNED_SIGNAL: 12.5%; CUSTOMER_OWNED_METERED: 3.5%
- **Source / calibration:** FPL tariff sheets SL-1/LT-1/OL-1 (company-owned), SL-2 (traffic signals), SL-1M (metered).
- **Conditional drivers:** tariff
- **Load-shape manifestation:** Determines billing basis (fixture charge vs metered).

### L4. ZCIS_LGT_ATTR.ZZ_LED_CONVERSION_KWH_YR
- **Definition:** kWh/yr saved by converting remaining HPS fixtures to LED (~55% wattage reduction, 4,100 h).
- **Statewide (headline):** total 147 GWh/yr on 9999 accounts
- **By segment:** total 147 GWh/yr on 9999 accounts
- **Source / calibration:** DOE MSSLC retrofit data; FPL LED conversion programme.
- **Conditional drivers:** fixture tech, watts, count
- **Load-shape manifestation:** Quantifies the HPS->LED opportunity.

## Selection process (scoring note)
- **Candidates:** 97 residential, 96 commercial, 97 industrial and 20 lighting candidates were brainstormed. They are listed in `docs/phase3/attributes/candidates_{res,com,ind,lgt}.csv` and generated by `docs/phase3/attributes/build_candidates.py`.
- **Scoring:** each candidate was scored 1–5 on seven criteria: load-factor relevance (LF), demand-response value (DR), grid planning (GP), revenue/rate design (REV), reliability (REL), measurability (MEAS) and quality of public source (SRC).
  - Composite = 0.5 × mean(LF, DR, GP, REV, REL) + 0.25 × MEAS + 0.25 × SRC.
- **Selection rules, applied in rank order:**
  - Merge duplicates: the dryer folds into the washer, CILC into curtailable kW, SST into on-site generation.
  - Exclude outputs derived from the intervals (load factor, coincident peak) and tariff fields.
  - Exclude candidates that apply only to a small NAICS subset (mining, semiconductors).
  - Require that the attribute can be synthesised for 100% of the class and that it can show up consistently in the load shape.
- **Validation:** `phase3_out/validation/` (`benchmarks.csv`, `segment_benchmarks.csv`, cross-tabs by county, income decile, housing type and tenure, NAICS3 and building type; `validation_attrs.json`).
