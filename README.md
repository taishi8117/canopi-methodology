# Canopi's Carbon Accounting Methodology for Bitcoin

## Summary

This repository outlines [Canopi](https://www.canopi.cash)'s open-source carbon accounting methodology for Bitcoin. 

It provides daily estimates of carbon emissions released by holding 1 BTC for a day. The model is primarily based on the [Cambridge Bitcoin Electricity Consumption Index](https://cbeci.org/), as well as various emission factors derived from the energy mix and geographic distribution of hashrate.

Canopi's [Climate API](https://www.canopi.cash/services/developer) and [Crypto Emission Calculator](https://app.canopi.cash/crypto) use this model to offer the ability to calculate the carbon footprint of BTC addresses.

### License

***NON-COMMERCIAL USE ONLY***

### Caveat

We are confident that our model uses the most reliable public data sets and methodologies and produces fair and up-to-date best-guess values. However, we are aware that the limitations and assumptions made in the adopted data sources and methodologies are still applicable in this model. This is why we consider this methodology "work-in-progress" and plan to review and revise on a continuous basis. We welcome comments and suggestions for improvements — please feel free to use Github Issues/PR.

### TL; DR

* your usage of the Bitcoin network = the share of your bitcoin balance out of the total bitcoin in circulation
* carbon emissions of the Bitcoin network = total electricity consumption * emission factor
* carbon emissions of your Bitcoin address = sum of daily (total_electricity_consumption \* emission_factor \* your_balance / total_supply )

### File Structure

* `source/`  \- directory that contains data sources
  + `btc_hashrate_distribution.csv`
  + `cbeci_export.csv`
  + `emission_factor.csv`
  + `total-bitcoins.csv`
* `btc_v0.2.ipynb` - Jupyter notebook with full source code of the methodology
* `1btc_emission_daily.csv` - the output of this model; the daily estimates of carbon emissions released by holding 1BTC for a day, since 2014-07-01.
* `README.md` - this document

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

### Total number of Bitcoins in circulation, *Daily*

The total number of Bitcoins in circulation constantly changes but is publicly available. Canopi uses blockchain.info's API to retrieve this information.

## Methodology

The following outlines Canopi's methodology to calculate daily estimates of carbon emissions released by holding 1 BTC for a day (Step 1-5), and to calculate the total emissions released by a Bitcoin address (Step 6). The Jupyter notebook contains the full source code of Step 1 - 5.

1. **Calculating daily total electricity consumption of the BTC network**

   The daily total is calculated by simply dividing the *yearly* total provided in the CBECI data ( `total_energy_df` ) by 365.

2. **Calculating overall emission factor of BTC mining based on geographic distribution of hashrate and energy mix**

   The emission factor of overall BTC mining is derived from summing the multiplications of the geographic emission factors ( `emission_factor_df` ) and the geographic distribution of hashrate ( `hashrate_dist_df` ).

   The function `btc_emission_factor_bt_ts` returns the emission factor closest to a given timestamp based on this model.

3. **Calculating daily carbon emissions of the Bitcoin network**

   The daily emissions can simply be calculted as the daily total electricity
   consumption multiplied by the emission factor.

4. **Calculating daily supply of Bitcoin**

   Data about the daily supply of Bitcoin (in circulation) is available through
   various blockchain APIs. Canopi uses blockchain.com's API to retrieve this information.

5. **Calculating daily carbon emissions of holding 1 BTC**

   As the primary output of this model, the daily carbon emissions of
   holding 1 BTC for a day are derived from the total emissions (step 3)
   divided by the supply (step 4).

6. **Calculating the carbon footprint of a Bitcoin address**

   Given a BTC address, its historical daily balances can be reconstructed from the full list of its transactions available on the public blockchain.
   Therefore, the carbon footprint of an address can be considered as 
   the sum of daily (emissions * share of your network usage).
