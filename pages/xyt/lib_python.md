---
title: Accessing data from Python
keywords: python, sample, data access
summary: "This is the official documentation for xyt's Python package."
sidebar: xyt_sidebar
permalink: lib_python.html
folder: xyt
---

Installation
------------

To install package from file:

`$ pip install xyt-hub-1.1.0.tar.gz`

Configuration
-------------

To configure authentication settings set following attributes in the
`ApiSettings` class:

    from xyt import ApiSettings

    ApiSettings.user_id = USERNAME
    ApiSettings.user_password = PASSWORD


Browsing data catalogue
-----------------------

### Data sources

Listing of available sources:

    from xyt import ApiSettings

    ApiSettings.lookup_sources()


### Exchanges lookup

Listing all available exchanges:

    from xyt import lookup_exchanges

    exchanges = lookup_exchanges(source='ACTIV')


#### Input parameters

| Parameter        | Type                   | Required  | Description                                         |
|------------------|------------------------|-----------|-----------------------------------------------------|
| source           | string                 | x         | Data source.                                        |
| exchange_filters | [string]               |           | List of exchanges.                                  |
| first_day        | pd.Timestamp / string  |           | First day that should be taken into account.        |
| last_day         | pd.Timestamp / string  |           | Last day that should be taken into account.         |


#### Output columns

| Column        | Type           | Description                     |
|---------------|----------------|---------------------------------|
| exchange      | string         | Exchange identifier.            |
| name          | string         | Full exchange name.             |
| abbreviation	| string         | Exchange abbreviation.          |
| first_day     | pd.Timestamp   | First available day.            |
| last_day      | pd.Timestamp   | Last available day.             |
| feeds         | [string]       | List of included feeds.         |


### Symbols lookup

Listing all symbols on given data source:

    from xyt import lookup_symbols

    symbols = lookup_symbols(source='ACTIV')


Listing all symbols on given data source by pattern:

    from xyt import lookup_symbols

    symbols = lookup_symbols(source='ACTIV', pattern='apple')


Listing all symbols on given data source by pattern and dates range:

    from xyt import lookup_symbols

    symbols = lookup_symbols(source='ACTIV', pattern='apple',
                             first_day='2016.01.01', last_day='2016.12.31')

#### Advanced searching

More advanced search limiting to specific exchanges on the data source and specific item type:

    from xyt import lookup_symbols, ItemType

    symbols = lookup_symbols(source='ACTIV', pattern='zal', exchange_filters=['XETRA', 'BXTR'],
                             first_day='2016.01.01', last_day='2016.12.31',
                             item_type_filters=[ItemType.INDIVIDUAL_ITEM])


#### Input parameters

| Parameter              | Type                   | Required  | Description                                      |
|------------------------|------------------------|-----------|--------------------------------------------------|
| source                 | string                 | x         | Data source.                                     |
| pattern                | string                 |           | Lookup pattern.                                  |
| exchange_filters       | [string]               |           | Narrow lookup to one or more exchange names.     |
| region_filters         | [string]               |           | Narrow lookup to one or more region names.       |
| entity_filters         | [string]               |           | Narrow lookup to one or more entities.           |
| product_type_filters   | [ProductType]          |           | Narrow lookup to one or more product types.      |
| item_type_filters      | [ItemType]             |           | Narrow lookup to one or more item types.         |
| first_day              | pd.Timestamp / string  |           | First day that should be taken into account.     |
| last_day               | pd.Timestamp / string  |           | Last day that should be taken into account.      |


#### Output columns

| Column           | Type           | Description                |
|------------------|----------------|----------------------------|
| source           | string         | Source identifier.         |
| region           | string         | Region identifier.         |
| exchange         | string         | Exchange identifier.       |
| symbol           | string         | Symbol.                    |
| name             | string         | Full item name.            |
| isin             | string         | ISIN.                      |
| local_code       | string         | Exchange local code.       |
| currency         | string         | Item currency.             |
| market_segment   | string         | Market segment.            |
| entity_class     | string         | Entity class.              |
| item_type        | ItemType       | Item type.                 |
| first_day        | pd.Timestamp   | First available day.       |
| last_day         | pd.Timestamp   | Last available day.        |


Retrieving price data
---------------------

### Trades and/or quotes

Retrieves tick data (trades, quote, trades and quotes) for given symbol and filtering rules.

Retrieving all trades (including non-regular) with trade corrections applied:

    from xyt import tick_data, TickDataType, TickDataFlag

    td = tick_data(source='ACTIV', symbol='ZAL.XE', day='2016.05.05',
                   data_type=TickDataType.TRADES,
                   from_time='09:00:00.000', to_time='09:10:09.999',
                   flags=[TickDataFlag.APPLY_TRADE_CORRECTIONS, TickDataFlag.INCLUDE_NON_REGULAR,
                          TickDataFlag.INCLUDE_TRADE_CONDITION_INFO])


Retrieving quotes:

    from xyt import tick_data, TickDataType, TickDataFlag

    td=tick_data(source='ACTIV', symbol='ZAL.XE', day='2016.05.05',
                 data_type=TickDataFlag.QUOTES,
                 from_time='09:00:00.000', to_time='09:10:09.999')


Retrieving trades and quotes in one call:

    from xyt import tick_data, TickDataType, TickDataFlag

    td = tick_data(source='ACTIV', symbol='ZAL.XE', day='2016.05.05',
                   data_type=TickDataType.TRADES_AND_QUOTES,
                   from_time='09:00:00.000', to_time='09:02:00.000',
                   flags=[TickDataFlag.APPLY_TRADE_CORRECTIONS, TickDataFlag.INCLUDE_NON_REGULAR,
                          TickDataFlag.INCLUDE_TRADE_CONDITION_INFO])


#### Input parameters

| Parameter     | Type                   | Required  | Description                                                     |
|---------------|------------------------|-----------|-----------------------------------------------------------------|
| source        | string                 | x         | Data source.                                                    |
| data_type     | TickDataType           | x         | Specifies if output should contain trades, quotes or both.      |
| symbol        | string                 | x         | Symbol.                                                         |
| day           | pd.Timestamp / string  | x         | Requested day.                                                  |
| from_time     | pd.Timedelta / string  |           | Start of the requested time range.                              |
| to_time       | pd.Timedelta / string  |           | End of the requested time range.                                |
| flags         | [TickDataFlag]         |           | List of flags.                                                  |

Available TickDataType values:

| TickDataType      | Description        |  
|-------------------|--------------------|                                                                                                              
| TRADES            | Trades only.       |
| QUOTES            | Quotes only.       |
| TRADES_AND_QUOTES	| Trades and quotes. |

Available TickDataFlag values:

| TickDataFlag                    | Description                                                                                                                                                                                                                                                          |
|---------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|                                                                                                                                                    |
| USE_MARKET_TS                   | Specifies if exchange timestamp is returned in the output. If flag USE_MARKET_TS is set, result contains exchange timestamp. Otherwise capture timestamp is returned.                                                                                                |
| APPLY_TRADE_CORRECTIONS         | Specifies if trade corrections should be applied to trades. If flag APPLY_TRADE_CORRECTIONS is set, prices and sizes of corrected trades are adjusted accordingly. Usually corrections apply to non-regular trades and are available for limited scope of exchanges. |
| INCLUDE_TRADE_CONDITION_INFO    | Specifies if trade conditions information is returned in the output. It contains mapping of trade conditions to human-readable descriptions, trade categories and markers of continuous session assignment.                                                          |
| INCLUDE_NON_REGULAR             | Specifies if non-regular trades should be included in the output.                                                                                                                                                                                                    |
| INCLUDE_CONTINUOUS_SESSION_ONLY | Specifies if only continuous session messages should be included in the result set. This covers filtering by trade conditions assigned to continuous session for trades and instrument status filtering for quotes.                                                  |
| EXCLUDE_CANCELLED_TRADES 	      | Specifies if canceled trades are removed from the output. Functionality removes only canceled trades that are no corrections.                                                                                                                                        |
| NANOSECONDS_TIMESTAMP	          | Use nanoseconds precision, if available, to represent time.                                                                                                                                                                                                          |

#### Output columns

| Column           | Type           | TRADES request    | QUOTES request   | TRADES_AND_QUOTES request                       |
|------------------|----------------|----------------------------------------------------------------------------------------|
| symbol           | string         | Symbol.           | Symbol.          | Symbol.                                         |
| date             | pd.Timestamp   | Date.             | Date.            | Date.                                           |
| time             | pd.Timedelta   | Time of trade.    | Time of quote.   | Time of trade.                                  |
| trade            | float          | Trade price.      | -                | Price of trade.                                 |
| trade_size       | long           | Trade size.       | -                | Size of trade.                                  |
| trade_condition  | string         | Trade conditions. | -                | Condition of trade.                             |
| ask              | float          | -                 | Ask price.       | Price of most recent ask as of trade time.      |
| ask_size         | long           | -                 | Ask size.        | Size of most recent ask as of trade time.       |
| bid              | float          | -                 | Bid price.       | Price of most recent ask as of trade time.      |
| bid_size         | long           | -                 | Bid size.        | Size of most recent ask as of trade time.       |


### Trades and/or quotes aggregated

Retrieves aggregated tick data (trades, quote, trades and quotes) for given symbol and filtering rules.
The result contains one row for each time-bin in given time range. Time-bins size is determined via `bin_size_in_seconds` parameter.
In case there was no ticks in the market, the result still contains given bin with empty values.

Retrieving hourly aggregated trades:

    from xyt import tick_aggregated, TickDataType, TickDataFlag

    td = tick_aggregated(source='ACTIV', symbol='CL/17U.NXG', day='2017.08.16',
                         bin_size=3600, # hourly
                         data_type=TickDataType.TRADES,
                         flags=[TickDataFlag.APPLY_TRADE_CORRECTIONS, TickDataFlag.INCLUDE_NON_REGULAR,
                                TickDataFlag.USE_MARKET_TS])


Retrieving quotes aggregated in 15 minute intervals:

    from xyt import tick_aggregated, TickDataType, TickDataFlag

    td = tick_aggregated(source='ACTIV', symbol='CL/17U.NXG', day='2017.08.16',
                         bin_size=900, # 15 minutes
                         data_type=TickDataType.QUOTES, flags=[])


Retrieving trades and quotes in one minute bins:

    from xyt import tick_aggregated, TickDataType, TickDataFlag

    td = tick_aggregated(source='ACTIV', symbol='CL/17U.NXG', day='2017.08.16',
                         data_type=TickDataType.TRADES_AND_QUOTES)


#### Input parameters

| Parameter           | Type                   | Required  | Description                                                |
|---------------------|------------------------|-----------|------------------------------------------------------------|
| source              | string                 | x         | Data source.                                               |
| data_type           | TickDataType           | x         | Specifies if output should contain trades, quotes or both. |
| symbol              | string                 | x         | Symbol.                                                    |
| day                 | pd.Timestamp / string  | x         | Requested day.                                             |
| from_time           | pd.Timedelta / string  |           | Start of the requested time range.                         |
| to_time             | pd.Timedelta / string  |           | End of the requested time range.                           |
| bin_size            | long                   |           | Size of the time bins expressed in seconds.                |
| flags               | [TickDataFlag]         |           | List of flags.                                             |

Available Type and Flag values - as described [here](lib_python.html#input-parameters-2).

#### Output columns

| Column           | Type         | TRADES request                     | QUOTES request                       | TRADES_AND_QUOTES request                 |
|------------------|--------------|------------------------------------|--------------------------------------|-------------------------------------------|
| symbol           | string       | Symbol.                            | Symbol.                              | Symbol.                                   |
| date             | pd.Timestamp | Date.                              | Date.                                | Date.                                     |
| time             | pd.Timedelta | Bin closing time.                  | Bin closing time.                    | Bin closing time.                         |
| open             | float        | Open price for given bin.          | -                                    | Open price for given bin.                 |
| high             | float        | High price for given bin.          | -                                    | High price for given bin.                 |
| low              | float        | Low price for given bin.           | -                                    | Low price for given bin.                  |
| close            | float        | Close price for given bin.         | -                                    | Close price for given bin.                |
| trade_count      | long         | Trade count for given bin.         | -                                    | Trade count for given bin.                |
| volume           | long         | Total trades volume for given bin. | -                                    | Total trades volume for given bin.        |
| trade_condition  | string       | Trade conditions for last trade.   | -                                    | Trade conditions for last trade.          |
| bid              | float        | -                                  | Price of the last bid as of bin end. | Price of the last bid as of bin end.      |
| bid_size         | long         | -                                  | Size of the last bid as of bin end.  | Size of the last bid as of bin end.       |
| ask              | float        | -                                  | Price of the last ask as of bin end. | Price of the last ask as of bin end.      |
| ask_size         | long         | -                                  | Size of the last ask as of bin end.  | Size of the last ask as of bin end.       |


### Trades and/or quotes snapshot

Retrieves snapshot of tick data (trades, quote, trades and quotes) for given symbols and filtering rules.

Retrieving trade snapshot at 12:00:00.00:

    from xyt import tick_snapshot, TickDataType, TickDataFlag

    td = tick_snapshot(source='ACTIV', symbols=['CL/17U.NXG'], day='2017.08.16',
                       data_type=TickDataType.TRADES,
                       time='12:00:00.000',
                       flags=[TickDataFlag.APPLY_TRADE_CORRECTIONS, TickDataFlag.INCLUDE_NON_REGULAR,
                              TickDataFlag.USE_MARKET_TS])

Retrieving quote snapshot at 09:15:00:

    from xyt import tick_snapshot, TickDataType, TickDataFlag

    td = tick_snapshot(source='ACTIV', symbols=['CL/17U.NXG'], day='2017.08.16',
                       data_type=TickDataType.QUOTES,
                       time='09:15:00.000',
                       flags=[])

Retrieving trade and quote snapshot at 10:03:00:

    from xyt import tick_snapshot, TickDataType, TickDataFlag

    td = tick_snapshot(source='ACTIV', symbols=['CL/17U.NXG'], day='2017.08.16',
                       data_type=TickDataType.TRADES_AND_QUOTES,
                       time='10:03:00.000',
                       flags=[TickDataFlag.APPLY_TRADE_CORRECTIONS, TickDataFlag.INCLUDE_NON_REGULAR,
                              TickDataFlag.USE_MARKET_TS])

#### Input parameters

| Parameter     | Type                   | Required  | Description                                                     |
|---------------|------------------------|-----------|-----------------------------------------------------------------|
| source        | string                 | x         | Data source.                                                    |
| data_type     | TickDataType           | x         | Specifies if output should contain trades, quotes or both.      |
| symbols       | [string]               | x         | List of symbols.                                                |
| day           | pd.Timestamp / string  | x         | Requested day.                                                  |
| time          | pd.Timedelta / string  | x         | Timestamp of the requested snapshot.                            |
| flags         | [TickDataFlag]         |           | List of flags.                                                  |

Available Type and Flag values - as described [here](lib_python.html#input-parameters-2).

#### Output columns

| Column           | Type         | TRADES request                                              | QUOTES request                                 | TRADES_AND_QUOTES request                                   |                                                    
|------------------|--------------|-------------------------------------------------------------|------------------------------------------------|-------------------------------------------------------------|
| symbol           | string       | Symbol.                                                     | Symbol.                                        | Symbol.                                                     |
| date             | pd.Timestamp | Date.                                                       | Date.                                          | Date.                                                       |
| time             | pd.Timedelta | Trade.                                                      | Time.                                          | Time.                                                       |
| trade            | float        | Price of most recent trade as of requested time.            | -                                              | Price of most recent trade as of requested time.            |
| trade_size       | long         | Size of most recent trade as of requested time.             | -                                              | Size of most recent trade as of requested time.             |
| trade_condition  | string       | Trade conditions of most recent trade as of requested time. | -                                              | Trade conditions of most recent trade as of requested time. |
| bid              | float        | -                                                           | Price of most recent bid as of requested time. | Price of most recent bid as of requested time.              |
| bid_size         | long         | -                                                           | Size of most recent bid as of requested time.  | Size of most recent bid as of requested time.               |
| ask              | float        | -                                                           | Price of most recent ask as of requested time. | Price of most recent ask as of requested time.              |
| ask_size         | long         | -                                                           | Size of most recent ask as of requested time.  | Size of most recent ask as of requested time.               |


### Orders

Retrieving orders:

    from xyt import order_data

    orders = order_data(source='ACTIV', symbols=['DBK.XE'], day='2016.10.04',
                        from_time='09:00:00.000', to_time='09:01:30.000')

#### Input parameters

| Parameter     | Type                   | Required  | Description                                   |
|---------------|------------------------|-----------|-----------------------------------------------|
| source        | string                 | x         | Data source.                                  |
| symbols       | [string]               | x         | List of symbols.                              |
| day           | pd.Timestamp / string  | x         | Requested day.                                |
| from_time     | pd.Timedelta / string  |           | Start of the requested time range.            |
| to_time       | pd.Timedelta / string  |           | End of the requested time range.              |

#### Output columns

| Column           | Type          | Description                                                     |
|------------------|---------------|-----------------------------------------------------------------|
| symbol           | string        | Symbol.                                                         |
| order_date       | pd.Timestamp  | Date.                                                           |
| time             | pd.Timedelta  | Time of trade or quote.                                         |
| action           | string        | Orderbook entry update action.                                  |
| order_id         | string        | Order ID.                                                       |
| order_side       | string        | Order side.                                                     |
| order_price      | double        | Order price.                                                    |
| order_size       | long          | Order size.                                                     |
| order_time       | pd.Timedelta  | Order time.                                                     |
| trade_id         | string        | Trade ID.                                                       |
| order_type       | pd.Timedelta  | Order type.                                                     |

### Auction data

Retrieving orders:

    from xyt import auction_data

    auctions = auction_data(source='ACTIV', symbols=['DBK.XE'], day='2016.10.04')


#### Input parameters

| Parameter     | Type                   | Required  | Description                                  |
|---------------|------------------------|-----------|----------------------------------------------|
| source        | string                 | x         | Data source.                                 |
| symbols       | [string]               | x         | List of symbols.                             |
| day           | pd.Timestamp / string  | x         | Requested day.                               |

#### Output columns

| Column                | Type         | Description                                                |
|-----------------------|--------------|------------------------------------------------------------|
| time                  | pd.Timedelta | Time.                                                      |
| imbalance_buy_volume  | long         | Imbalance buy volume.                                      |
| imbalance_sell_volume | long         | Imbalance sell volume.                                     |
| imbalance_volume_time | pd.Timedelta | Imbalance volume time.                                     |
| cross_type            | string       | Cross type.                                                |
| uncrossing_date       | pd.Timestamp | Uncrossing date.                                           |
| uncrossing_time       | pd.Timedelta | Uncrossing time.                                           |
| uncrossing_condition  | string       | Uncrossing condition.                                      |
| uncrossing_price      | float        | Uncrossing price.                                          |
| uncrossing_volume     | long         | Uncrossing volume.                                         |
| uncrossing_type       | string       | Uncrossing type.                                           |
| auction_date          | pd.Timestamp | Auction date.                                              |
| auction_time          | pd.Timedelta | Auction time.                                              |
| auction_status        | long         | Auction status.                                            |
| auction               | float        | Auction.                                                   |
| auction_volume        | long         | Auction volume.                                            |
| auction_type          | string       | Auction type.                                              |


### End of day

Retrieving end of day data:

    from xyt import end_of_day

    eod = end_of_day(source='ACTIV', symbols=['DBK.XE'], first_day='2016.09.01', last_day='2017.08.02')


#### Input parameters

| Parameter     | Type                   | Required  | Description                                 |
|---------------|------------------------|-----------|---------------------------------------------|
| source        | string                 | x         | Data source.                                |
| symbols       | [string]               | x         | List of symbols.                            |
| first_day     | pd.Timestamp / string  | x         | First requested day.                        |
| last_day      | pd.Timestamp / string  | x         | Last requested day.                         |

#### Output columns

| Column           | Type          | Description                   |
|------------------|---------------|-------------------------------|
| symbol           | string        | Symbol.                       |
| date             | pd.Timestamp  | Date.                         |
| open             | float         | Official open price.          |
| high             | float         | Official high price.          |
| low              | float         | Official low price.           |
| close            | float         | Official close price.         |
| trade_count      | long          | Total trade count.            |
| volume           | long          | Total trades volume.          |


### Settlement prices

Retrieving settlement prices for given list of derivative symbols:

    from xyt import settlement_prices

    prices = settlement_prices(source='ACTIV', symbols=['DBK.XE'],
                               first_day='2016.10.04', last_day='2016.11.04')


#### Input parameters

| Parameter     | Type                    | Required  | Description                                 |
|---------------|-------------------------|-----------|---------------------------------------------|
| source        | string                  | x         | Data source.                                |
| symbols       | [string]                | x         | List of symbols.                            |
| first_day     | pd.Timestamp / string   | x         | First requested day.                        |
| last_day      | pd.Timestamp / string   | x         | Last requested day.                         |

#### Output columns

| Column              | Type          | Description                                                 |
|---------------------|---------------|-------------------------------------------------------------|
| symbol              | string        | Symbol.                                                     |
| settlement_date     | pd.Timestamp  | Settlement date.                                            |
| settlement_time     | pd.Timedelta  | Settlement time.                                            |
| settlement          | float         | Settlement.                                                 |
| settlement_status   | long          | Settlement status.                                          |
| settlement_size     | long          | Settlement size.                                            |
| capture_date        | pd.Timestamp  | Capture date (date of the update recording - UTC timezone). |
| capture_time        | pd.Timedelta  | Capture time (time of the update recording - UTC timezone). |


Retrieving reference data
-------------------------

### Reference data

Retrieves reference data for given symbols and date.
Note searching by pattern and retrieval of basing reference data is possible via `lookup_symbols` function.

    from xyt import reference_data

    reference = reference_data(source='ACTIV', symbols=['DBK.XE'], day='2016.09.01')


#### Input parameters

| Parameter     | Type                    | Required  | Description                                 |
|---------------|-------------------------|-----------|---------------------------------------------|
| source        | string                  | x         | Data source.                                |
| symbols       | [string]                | x         | List of symbols.                            |
| day           | pd.Timestamp / string   | x         | Requested day.                              |

#### Output columns

| Column                    | Type   | Description                                                  |
|---------------------------|--------|--------------------------------------------------------------|
| symbol                    | string | Symbol.                                                      |
| local_code                | string | Local code.                                                  |
| isin                      | string | Isin.                                                        |
| name                      | string | Name.                                                        |
| country_code              | string | Country code.                                                |
| currency                  | string | Currency.                                                    |
| entity_type               | string | Entity type.                                                 |


Additional columns available for equities:

| Column                    | Type   | Description                                                  |
|---------------------------|--------|--------------------------------------------------------------|
| lot_size                  | long   | Lot size.                                                    |
| market_sector             | string | Market sector.                                               |
| market_segment            | string | Market segment.                                              |
| security_primary_exchange | string | Security primary exchange.                                   |
| sedol_code                | string | Sedol code.                                                  |

Additional columns available for futures:

| Column                       | Type           | Description                                       |
|------------------------------|----------------|---------------------------------------------------|
| expiration_date              | pd.Timestamp   | Expiration date.                                  |   
| contract_size                | long           | Contract size.                                    |
| currency_multiplier          | float          | Currency multiplier.                              |
| minimum_tick                 | float          | Minimum tick.                                     |
| point_size                   | float          | point size.                                       |
| underlying_isin              | string         | Underlying isin.                                  |
| underlying_symbol            | string         | Underlying symbol.                                |
| underlying_currency          | string         | Underlying currency.                              |
| underlying_mic               | string         | Underlying mic.                                   |

Additional columns available for future options:

| Column                       | Type           | Description                                       |
|------------------------------|----------------|---------------------------------------------------|
| expiration_date              | pd.Timestamp   | Expiration date.                                  |   
| contract_size                | long           | Contract size.                                    |
| currency_multiplier          | float          | Currency multiplier.                              |
| minimum_tick                 | float          | Minimum tick.                                     |
| option_type                  | string         | Option type.                                      |
| strike_price                 | float          | Strike price.                                     |
| exercise_style               | string         | Exercise style.                                   |
| strike_price_multiplier      | float          | Strike price multiplier.                          |
| underlying_isin              | string         | Underlying isin.                                  |
| future_symbol                | string         | Future symbol.                                    |
| future_expiration_date       | pd.Timestamp   | Future expiration date.                           |

Additional columns available for options:

| Column                               | Type   | Description                                       |
|--------------------------------------|--------|---------------------------------------------------|
| expiration_date                      | string | Expiration date.                                  |
| contract_size                        | long   | Contract size.                                    |
| currency_multiplier                  | float  | Option root currency multiplier.                  |
| minimum_tick                         | float  | Option root minimum tick.                         |
| option_type                          | string | Option type.                                      |
| strike_price                         | float  | Strike price.                                     |
| exercise_style                       | string | Exercise style.                                   |
| expiration_type                      | string | Expiration type.                                  |
| contract_multiplier                  | float  | Contract multiplier.                              |
| generation_number                    | string | Generation number.                                |
| strike_price_denominator             | float  | Strike price denominator.                         |
| shares_per_contract                  | long   | Shares per contract.                              |
| underlying_isin                      | string | Underlying isin.                                  |
| underlying_symbol                    | string | Underlying symbol.                                |
| underlying_currency                  | string | Underlying currency.                              |
| underlying_mic                       | string | Underlying mic.                                   |


### Instrument status

Retrieving instrument status:

    from xyt import instrument_status

    status = instrument_status(source='ACTIV', symbols=['DBK.XE', 'ADS.XE'], day='2016.09.02')


#### Input parameters

| Parameter     | Type                   | Required  | Description                                         |
|---------------|------------------------|-----------|-----------------------------------------------------|
| source        | string                 | x         | Data source.                                        |
| symbols       | [string]               | x         | List of symbols.                                    |
| day           | pd.Timestamp / string  | x         | Requested day.                                      |

#### Output columns

| Column                         | Type           | Description                                            |
|--------------------------------|----------------|--------------------------------------------------------|
| symbol                         | string         | Symbol.                                                |
| date                           | pd.Timestamp   | Date.                                                  |
| time                           | pd.Timedelta   | Time.                                                  |
| instrument_status              | string         | Instrument status.                                     |
| instrument_status_description  | string         | Instrument status description.                         |
| is_continuous_session          | bool           | Is continuous session.                                 |


### Tick rules

Retrieving tick rules:

    from xyt import tick_rules

    rules = tick_rules(source='ACTIV', symbols=['DBK.XE'], day='2016.09.02')


#### Input parameters

| Parameter     | Type                    | Required  | Description                                 |
|---------------|-------------------------|-----------|---------------------------------------------|
| source        | string                  | x         | Data source.                                |
| symbols       | [string]                | x         | List of symbols.                            |
| day           | pd.Timestamp / string   | x         | Requested day.                              |

#### Output columns

| Column                         | Type   | Description                                             |
|--------------------------------|--------|---------------------------------------------------------|
| symbol                         | string | Symbol.                                                 |
| tick_rule_symbol               | string | Tick rule symbol.                                       |
| lower_bound                    | float  | Lower bound.                                            |
| tick_size                      | float  | Tick size.                                              |


### Option and future chains

Retrieves option or future chain for given chain identifier (either option stem or underlying) and given exchange and day.

Retrieving option chain:

    from xyt import chain, ChainType

    chains = chain(source='ACTIV', exchange='OPRA_COMPOSITE',
                   symbol='=DJX.W', day='2016.09.02',
                   chain_type=ChainType.OPTIONS)


Retrieving future chain:

    from xyt import chain, ChainType

    chains = chain(source='ACTIV', exchange='NYMEX',
                   symbol='CL', day='2016.09.02',
                   chain_type=ChainType.FUTURES)


Retrieving future option chain:

    from xyt import chain, ChainType

    chains = chain(source='ACTIV', exchange='NYMEX',
                   symbol='CL', day='2016.09.02',
                   chain_type=ChainType.FUTURE_OPTIONS)


#### Input parameters

| Parameter     | Type                   | Required  | Description                                                                |
|---------------|------------------------|-----------|----------------------------------------------------------------------------|
| source        | string                 | x         | Data source.                                                               |
| chain_type    | ChainType              | x         | Specifies if output should contain options, futures or future options.     |
| symbol        | string                 | x         | Stem or underlying symbol.                                                 |
| exchange      | string                 | x         | Exchange identifier.                                                       |
| day           | pd.Timestamp / string  | x         | Requested day.                                                             |

#### Output columns

| Column           | Type       | Description                |
|------------------|------------|----------------------------|
| chain_symbol     | string     | Chain identifier.          |
| symbol           | string     | Chain member symbol.       |
| underlying       | string     | Underlying identifier.     |
| name             | string     | Chain name.                |

Available `ChainType` values:

| Type                | Description        |  
|---------------------|--------------------|
| OPTIONS             | Options.           |
| FUTURES             | Futures.           |
| FUTURE_OPTIONS      | Future options.    |


Other data
----------

### Generic data request

Apart from the data requests described above, it is possible to query for other data sets via generic data request.
The input parameters, query logic and output content is specified by the given `request`.

#### Input parameters

| Parameter     | Type                | Required  | Description                                                     |
|---------------|---------------------|-----------|-----------------------------------------------------------------|
| source        | string              | x         | Data source.                                                    |
| request       | string              | x         | Custom-defined request name.                                    |
| parameters    | **kwargs            |           | Dictionary with input parameters required for given request.    |

#### Output columns

Output columns are dependent on the selected request.


Data representation
-------------------

Retrieved data is represented as instance of class derived from `DataObject`.
`DataObject` is an abstract class, which provides convenience methods for data conversions:

  | Method        | Description                                                                             |
  |---------------|-----------------------------------------------------------------------------------------|
  | to\_pandas()  | Converts retrieved data to `pandas.DataFrame` representation.                           |
  | to\_numpy()   | Converts retrieved data to numpy record array. Wraps: `self.to_pandas().to_records()`.  |
  | to\_csv()     | Converts retrieved data to CSV. Wraps: `self.to_pandas().to_csv()`.                     |

Deserialized protobuf representation is accessible via property: `data`.


Troubleshooting
---------------

### Proxies

If you need to use a proxy, you can configure the `ApiSettings.proxies` property:

    proxies = {
      'http': 'http://10.10.1.10:3128',
      'https': 'http://10.10.1.10:1080',
    }

    ApiSettings.proxies = proxies

You can also configure proxies by setting the environment variables `HTTP_PROXY` and `HTTPS_PROXY`.

Linux:

    $ export HTTP_PROXY="http://10.10.1.10:3128"
    $ export HTTPS_PROXY="http://10.10.1.10:1080"

Windows:

    $ set HTTP_PROXY="http://10.10.1.10:3128"
    $ set HTTPS_PROXY="http://10.10.1.10:1080"


To use HTTP Basic Auth with your proxy, use the `http://user:password@host/` syntax:

    proxies = {'http': 'http://user:pass@10.10.1.10:3128/'}


Development
-----------

### Setup

If you wish to work on local development please clone/fork the git repo
and use: `pip install -r requirements.txt` to setup the project.

### Testing

Package uses following tools for testing:

-   test runner: [nose](https://nose.readthedocs.org/en/latest/),
-   testing against multiple versions of python:
    [tox](https://pypi.python.org/pypi/tox),
-   syntax and style checker:
    [flake8](https://flake8.readthedocs.org/en/latest/),
-   isolating Python runtimes:
    [virtualenv](https://virtualenv.pypa.io/en/latest/).

To run tests:

-   Make sure a version of python 2.7 or python 3.x is installed locally
    in your system.
-   Install [virtualenv](https://virtualenv.pypa.io/en/latest/) and
    [tox](https://pypi.python.org/pypi/tox) using:

`pip install virtualenv tox`

-   Install package locally:

`python setup.py install`

-   Test against all supported Python versions:

`tox -- --tc=source:SOURCE --tc=client_secret:API_SECRET_KEY --tc=user_id:USER --tc=user_password:PASSWORD`


License
-------

[Apache License Version 2.0](http://opensource.org/licenses/Apache-2.0)


{% include links.html %}
