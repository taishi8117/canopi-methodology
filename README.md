# Canopi's Carbon Accounting Methodology for Bitcoin



## Summary

This repository outlines [Canopi](https://www.canopi.cash)'s open-source carbon accounting methodology for Bitcoin. 

It provides daily estimates of carbon emissions released by a single BTC transaction since 2014-07-01. The model is primarily based on the [Cambridge Bitcoin Electricity Consumption Index](https://cbeci.org/), as well as various emission factors derived from the energy mix and geographic distribution of hashrate.

Canopi's [Climate API](https://www.canopi.cash/services/developer) and [Crypto Emission Calculator](https://app.canopi.cash/crypto) use this model to offer the ability to calculate the carbon footprint of BTC addresses. If you want to understand why we attribute emissions on a per-transaction basis, check out our crypto emission [explainer](https://www.canopi.cash/crypto/explainer). 



### License

***NON-COMMERCIAL USE ONLY***

### Caveat

We are confident that our model uses the most reliable public data sets and methodologies and produces fair and up-to-date best-guess values. However, we are aware that the limitations and assumptions made in the adopted data sources and methodologies are still applicable in this model. This is why we consider this methodology "work-in-progress" and plan to review and revise on a continuous basis. We welcome comments and suggestions for improvements — please feel free to use Github Issues/PR.

### TL; DR

- The carbon footprint of your BTC address is derived from the sum of carbon emissions from your address' outgoing transactions.
- The average carbon footprint of a single BTC transaction is adjusted daily and derived from the total electricity consumption of Bitcoin, the geographic distribution of hashrate and the energy mix.

### File Structure

- `source/`  \- directory that contains data sources
  - `btc_hashrate_distribution.csv`
  - `btc_txn_count.csv`
  - `cbeci_export.csv`
  - `emission_factor.csv`
- `btc.ipynb` - Jupyter notebook with full source code of the methodology
- `btc_txn_emissions_daily.csv` - the output of this model; the daily estimates of carbon emissions released by a single BTC transaction since 2014-07-01.
- `README.md` - this document



## Data Sources

The model takes into account the following data sources in calculating the daily average emission of a single BTC transaction.

### Total Electricity Consumption of the BTC Network, *Daily*

The total electricity consumption of the Bitcoin network is derived from the [Cambridge Bitcoin Electricity Consumption Index](https://cbeci.org/) (CBECI). The CBECI model provides a lower bound, an upper bound and a best-guess estimates for the total *yearly (annualized)* electricity consumption of the Bitcoin network, adjusted everyday since 2014-07-01. Canopi uses these numbers to calculate the *daily* total electricity consumption.

The CBECI model is developed by Cambridge Center for Alternative Finance (CCAF) and originally based on a bottom-up approach initially developed by Marc Bevand in 2017 that takes different types of available mining hardware as the starting point. Their methodology, assumptions and limitations are detailed [here](https://cbeci.org/cbeci/methodology).



### Geographic Distribution of BTC Mining Hashrate, *Monthly/Static*

The geographic distribution of hashrate is derived from CCAF's [Bitcoin Mining Map](https://cbeci.org/mining_map), which uses non-public data offered by the BTC.com, Poolin, and ViaBTC pools for research purposes. The data is available monthly between September 2019 and April 2020.

Even though it was last updated more than a year ago, Canopi considers it as the most accurate, publicly available data source as of 06-24-2021, given the transparency of their methodology, assumptions and limitations which are detailed [here](https://cbeci.org/mining_map/methodology). Hence, the April 2020 value is applied statically for calculations after 04-01-2020, and the September 2019 value is used for calculations before 09-30-2019.

We will continue to use this data, but are also seeking partnerships with major mining pool providers to receive up-to-date, fine-grained data.



### Emission Factors with Energy Mix, *Static*

The Canopi model makes an explicit assumption that the energy mix used for mining operations in a region is the same as the average energy mix published for that region.

The emission factors for each country/region identified in Step 2 are derived from two different data sources: [IFI Dataset of Default Grid Factors v.2.0 from July 2019](https://unfccc.int/sites/default/files/resource/Harmonized_Grid_Emission_factor_data_set.xlsx) for all countries except China, and [Xin et al](https://www.sciencedirect.com/science/article/pii/S1876610217361714) for regions in China.



### Number of BTC Transactions, *Daily*

The number of BTC transactions is aggregated daily from a dataset publicly available on Google's BigQuery. The query can be found [here](https://console.cloud.google.com/bigquery?sq=206704257195:4acb2ac67b8c41c190c36a0cb7fa7129).





## Methodology

The following outlines Canopi's methodology to calculate a daily estimate of carbon emissions released by a single BTC transaction (Step 1-4), and to calculate the total emissions released by a Bitcoin address (Step 5). The Jupyter notebook contains the full source code of Step 1 - 4.

1. **Calculating the daily total electricity consumption of the BTC network**

   The daily total is calculated by simply dividing the *yearly* total provided in the CBECI data (`total_energy_df`) by 365.

2. **Calculating the daily average electricity consumption of a single BTC transaction**

   The daily average electricity consumption per transaction is derived from the daily total (Step 1) divided by the number of transaction for each day. (`btc_txn_count_df`)

3. **Calculating the overall emission factor of BTC mining based on geographic distribution of hashrate and energy mix**

   The emission factor of overall BTC mining is derived from summing the multiplications of the geographic emission factors (`emission_factor_df`) and the geographic distribution of hashrate (`hashrate_dist_df`).

   The function `btc_emission_factor_bt_ts` returns the emission factor closest to a given timestamp based on this model.

4. **Calculating the daily average emission of a BTC transaction**

   Finally, the daily average emission of a BTC transaction is derived from the daily average electricity consumption per transaction (Step 2) multiplied by the BTC emission factor (Step 3). The resulting data is exported as a CSV file called `btc_txn_emissions_daily.csv`.

   The function `txn_emission_by_ts` returns the emission per transaction for a given timestamp.

5. **Calculating emissions released by a BTC address**

   Given a BTC address, the full list of outgoing transactions can be fetched from a public blockchain. The emissions of a BTC address, therefore, are calculated as the sum of carbon emissions of these transactions. Specifically, it is the sum of `txn_emission_by_ts(tx.timestamp)` for each transaction `tx`.

   Note that generally, Canopi considers a sender of Bitcoin is responsible for emissions released by the transaction. For this reason, emissions from incoming transactions are not taken into account by default.

   However, we recognize there are certain scenarios in which a receiver should be responsible for the emissions (for example, when you send BTC from your exchange wallet to your hardware wallet). This is why Canopi's Climate API offers the ability to customize this behavior and specify which transactions are used to calculate the emissions of an address.