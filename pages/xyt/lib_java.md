---
title: Accessing data from Java
keywords: java, sample, data access
summary: "This is the official documentation for xyt's Java package."
sidebar: xyt_sidebar
permalink: lib_java.html
folder: xyt
---

Installation
------------
In order to use big xyt Cloud Platform Java Client add the following lines to your `pom.xml`:

```
<dependency>
  <groupId>big-xyt</groupId>
  <artifactId>xyt-hub</artifactId>
  <version>1.0.0</version>
 </dependency>
```

Creating client object
-----------------------

```
XytHubClient client = new XytHubClient("your@username.com", "password");
```

Browsing data catalogue
-----------------------

### Data sources lookup

Listing of available sources:

```
Sources sources = client.lookupSources();
```

### Exchanges lookup

Retrieving overview of exchanges on given data source.

Listing all available exchanges:

```
ExchangeData allExchanges = client.lookupExchanges("ACTIV");
```

#### Input parameters

| Parameter       | Type            | Required  | Description                                                     |
|-----------------|-----------------|-----------|-----------------------------------------------------------------|
| source          | String          | x         | Data source.                                                    |
| exchangeFilters | List\<String\>  |           | List of exchanges.                                              |
| firstDay        | Date / String   |           | First day that should be taken into account.                    |
| lastDay         | Date / String   |           | Last day that should be taken into account.                     |


#### Output columns

| Column        | Type       | Description                                                          |
|---------------|------------|----------------------------------------------------------------------|
| exchange      | String     | Exchange identifier.                                                 |
| name          | String     | Full exchange name.                                                  |
| abbreviation	| String     | Exchange abbreviation.                                               |
| first_day     | Date       | First available day.                                                 |
| last_day      | Date       | Last available day.                                                  |
| feeds         | String[]   | List of included feeds.                                              |


### Symbols lookup

Listing all symbols on given data source:

```
SymbolData allSymbols = client.lookupSymbols("ACTIV");
```

Listing all symbols on given data source by pattern:

```
SymbolData symbolsByKeyword = client.lookupSymbols("ACTIV", "apple");
```

Listing all symbols on given data source by pattern and dates range:

```
SymbolData foundSymbols = client.lookupSymbols("ACTIV", "apple", "2016.09.01", "2016.09.02");
```

#### Advanced searching

More advanced search limiting to specific exchanges on the data source and specific item type:

```
String pattern = "zal";
List<String> exchanges = new ArrayList<String>();
exchanges.add("XETRA");
exchanges.add("BXTR");

List<ItemType> itemTypes = new ArrayList<ItemType>();
itemTypes.add(ItemType.INDIVIDUAL_ITEM);

SymbolData foundSymbols = client.lookupSymbols("ACTIV", pattern, exchanges,
                                               new ArrayList<String>(), new ArrayList<String>(), new ArrayList<ProductType>(),
                                               itemTypes, 0, 0, "2016.09.01", "2016.09.02");
```

#### Input parameters

| Parameter            | Type               | Required  | Description                                      |
|----------------------|--------------------|-----------|--------------------------------------------------|
| source               | String             | x         | Data source.                                     |
| pattern              | String             |           | Lookup pattern.                                  |
| exchangeFilters      | List\<String\>     |           | Narrow lookup to one or more exchange names.     |
| regionFilters        | List\<String\>     |           | Narrow lookup to one or more region names.       |
| entityFilters        | List\<String\>     |           | Narrow lookup to one or more entities.           |
| productTypeFilters   | List\<ProductType\>|           | Narrow lookup to one or more product types.      |
| itemTypeFilters      | List\<ItemType\>   |           | Narrow lookup to one or more item types.         |
| firstDay             | Date / String      |           | First day that should be taken into account.     |
| lastDay              | Date / String      |           | Last day that should be taken into account.      |


#### Output columns

| Column           | Type       | Description                |
|------------------|------------|----------------------------|
| source           | String     | Source identifier.         |
| region           | String     | Region identifier.         |
| exchange         | String     | Exchange identifier.       |
| symbol           | String     | Symbol.                    |
| name             | String     | Full item name.            |
| isin             | String     | ISIN.                      |
| local_code       | String     | Exchange local code.       |
| currency         | String     | Item currency.             |
| market_segment   | String     | Market segment.            |
| entity_class     | String     | Entity class.              |
| item_type        | ItemType   | Item type.                 |
| first_day        | Date       | First available day.       |
| last_day         | Date       | Last available day.        |


Retrieving price data
---------------------

### Trades and/or quotes

Retrieves tick data (trades, quote, trades and quotes) for given symbol and filtering rules.

Retrieving all trades (including non-regular) with trade corrections applied:

```
List<TickDataRequest.Flag> flags = new ArrayList<TickDataRequest.Flag>();
flags.add(TickDataRequest.Flag.APPLY_TRADE_CORRECTIONS);
flags.add(TickDataRequest.Flag.INCLUDE_NON_REGULAR);
flags.add(TickDataRequest.Flag.INCLUDE_TRADE_CONDITION_INFO);

TickData tickData = client.getTickData("ACTIV", "ZAL.XE", "2016.09.02", "09:00:00.000", "09:02:00.000", TickDataRequest.Type.TRADES, flags);
```

Extracting trade condition mapping from trades:

```
TradeConditionData tradeConditions = tickData.getTradeConditions();
```

Retrieving quotes:

```
TickData quoteData = client.getTickData("ACTIV", "ZAL.XE", "2016.09.02", "09:00:00.000", "09:02:00.000", TickDataRequest.Type.QUOTES, new ArrayList<TickDataRequest.Flag>());
```

Retrieving trades and quotes in one call. Since `TickDataRequest.Flag.INCLUDE_TRADE_CONDITION_INFO` is specified in flags trade condition information will be included in the output. It can be retrieved using same logic as in previous example:

```
List<TickDataRequest.Flag> flags = new ArrayList<TickDataRequest.Flag>();
flags.add(TickDataRequest.Flag.APPLY_TRADE_CORRECTIONS);
flags.add(TickDataRequest.Flag.INCLUDE_NON_REGULAR);
flags.add(TickDataRequest.Flag.INCLUDE_TRADE_CONDITION_INFO);

TickData taqData = client.getTickData("ACTIV", "ZAL.XE", "2016.09.02", "09:00:00.000", "09:02:00.000",
                    TickDataRequest.Type.TRADES_AND_QUOTES, flags);
```

#### Input parameters

| Parameter     | Type            | Required  | Description                                                     |
|---------------|-----------------|-----------|-----------------------------------------------------------------|
| source        | String          | x         | Data source.                                                    |
| dataType      | Type            | x         | Specifies if output should contain trades, quotes or both.      |
| symbol        | String          | x         | Symbol.                                                         |
| day           | Date / String   | x         | Requested day.                                                  |
| fromTime      | Time            |           | Start of the requested time range.                              |
| toTime        | Time            |           | End of the requested time range.                                |
| flags         | List\<Flag\>    |           | List of flags.                                                  |

Available Type values:

| Type              | Description        |  
|-------------------|--------------------|                                                                                                              
| TRADES            | Trades only.       |
| QUOTES            | Quotes only.       |
| TRADES_AND_QUOTES	| Trades and quotes. |

Available Flag values:

| Flag                            | Description                                                                                                                                                                                                                                                          |
|---------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|                                                                                                                                                    |
| USE_MARKET_TS                   | Specifies if exchange timestamp is returned in the output. If flag USE_MARKET_TS is set, result contains exchange timestamp. Otherwise capture timestamp is returned.                                                                                                |
| APPLY_TRADE_CORRECTIONS         | Specifies if trade corrections should be applied to trades. If flag APPLY_TRADE_CORRECTIONS is set, prices and sizes of corrected trades are adjusted accordingly. Usually corrections apply to non-regular trades and are available for limited scope of exchanges. |
| INCLUDE_TRADE_CONDITION_INFO    | Specifies if trade conditions information is returned in the output. It contains mapping of trade conditions to human-readable descriptions, trade categories and markers of continuous session assignment.                                                          |
| INCLUDE_NON_REGULAR             | Specifies if non-regular trades should be included in the output.                                                                                                                                                                                                    |
| INCLUDE_CONTINUOUS_SESSION_ONLY | Specifies if only continuous session messages should be included in the result set. This covers filtering by trade conditions assigned to continuous session for trades and instrument status filtering for quotes.                                                  |
| EXCLUDE_CANCELLED_TRADES 	      | Specifies if canceled trades are removed from the output. Functionality removes only canceled trades that are no corrections.                                                                                                                                        |
| NANOSECONDS_TIMESTAMP	          | Use nanoseconds precision, if available, to represent time.                                                                                                                                                                                                          |

#### Output columns

| Column           | Type       | TRADES request    | QUOTES request   | TRADES_AND_QUOTES request                                  |
|------------------|------------|---------------------------------------------------------------------------------------------------|
| symbol           | String     | Symbol.           | Symbol.          | Symbol.                                                    |
| date             | Date       | Date.             | Date.            | Date.                                                      |
| time             | Time       | Time of trade.    | Time of quote.   | Time of trade or quote.                                    |
| trade            | Double     | Trade price.      | -                | Price of trade, empty in case of quote-only row.           |
| trade_size       | Long       | Trade size.       | -                | Size of trade, empty in case of quote-only row.            |
| trade_condition  | String     | Trade conditions. | -                | Condition of trade, empty in case of quote-only row.       |
| bid              | Double     | -                 | Bid price.       | Price of most recent bid as of trade time.                 |
| bid_size         | Long       | -                 | Bid size.        | Size of most recent bid as of trade time.                  |
| ask              | Double     | -                 | Ask price.       | Price of most recent ask as of trade time.                 |
| ask_size         | Long       | -                 | Ask size.        | Size of most recent ask as of trade time.                  |

### Trades and/or quotes aggregated

Retrieves aggregated tick data (trades, quote, trades and quotes) for given symbols and filtering rules.
The result contains one row per symbol for each time-bin in given time range. Time-bins size is determined via `binSizeInSeconds` parameter.
In case there was no ticks in the market, the result still contains given bin with empty values.

Retrieving hourly aggregated trades:

```
List<TickDataRequest.Flag> flags = new ArrayList<TickDataRequest.Flag>();
flags.add(TickDataRequest.Flag.APPLY_TRADE_CORRECTIONS);
flags.add(TickDataRequest.Flag.INCLUDE_NON_REGULAR);
flags.add(TickDataRequest.Flag.USE_MARKET_TS);

// get hourly aggregated data
int binSizeInSeconds = 3600;

TickAggregatedData tickAggregated = client.getTickAggregated("ACTIV", TickDataRequest.Type.TRADES, "CL/17U.NXG", "2017.08.16", binSizeInSeconds, flags);
```

Retrieving quotes aggregated in 15 minute intervals:

```
int binSizeInSeconds = 900; // 15 minutes
TickAggregatedData quotesAggregated = client.getTickAggregated("ACTIV", TickDataRequest.Type.QUOTES, "CL/17U.NXG", "2017.08.16", binSizeInSeconds, new ArrayList<TickDataRequest.Flag>());
```

Retrieving trades and quotes in one minute bins:

```
// we use here default flags for trades(EXCLUDE_CANCELLED_TRADES,USE_MARKT_TS,INCLUDE_NON_REGULAR) and default one-minute binning.
TickAggregatedData tradesAndQuotesAggregated = client.getTickAggregated("ACTIV", TickDataRequest.Type.TRADES_AND_QUOTES, "CL/17U.NXG", "2017.08.16");
```

#### Input parameters

| Parameter           | Type            | Required  | Description                                                     |
|---------------------|-----------------|-----------|-----------------------------------------------------------------|
| source              | String          | x         | Data source.                                                    |
| dataType            | Type            | x         | Specifies if output should contain trades, quotes or both.      |
| symbol              | String          | x         | Symbol.                                                         |
| day                 | Date / String   | x         | Requested day.                                                  |
| fromTime            | Time            |           | Start of the requested time range.                              |
| toTime              | Time            |           | End of the requested time range.                                |
| binSizeInSeconds    | Long            |           | Size of the time bins expressed in seconds.                     |
| flags               | List\<Flag\>    |           | List of flags.                                                  |

Available Type and Flag values - as described [here](lib_java.html#input-parameters-2).

#### Output columns

| Column           | Type       | TRADES request                     | QUOTES request                       | TRADES_AND_QUOTES request                 |
|------------------|------------|------------------------------------|--------------------------------------|-------------------------------------------|
| symbol           | String     | Symbol.                            | Symbol.                              | Symbol.                                   |
| date             | Date       | Date.                              | Date.                                | Date.                                     |
| time             | Time       | Bin closing time.                  | Bin closing time.                    | Bin closing time.                         |
| open             | Double     | Open price for given bin.          | -                                    | Open price for given bin.                 |
| high             | Double     | High price for given bin.          | -                                    | High price for given bin.                 |
| low              | Double     | Low price for given bin.           | -                                    | Low price for given bin.                  |
| close            | Double     | Close price for given bin.         | -                                    | Close price for given bin.                |
| trade_count      | Long       | Trade count for given bin.         | -                                    | Trade count for given bin.                |
| volume           | Long       | Total trades volume for given bin. | -                                    | Total trades volume for given bin.        |
| trade_condition  | String     | Trade conditions for last trade.   | -                                    | Trade conditions for last trade.          |
| bid              | Double     | -                                  | Price of the last bid as of bin end. | Price of the last bid as of bin end.      |
| bid_size         | Long       | -                                  | Size of the last bid as of bin end.  | Size of the last bid as of bin end.       |
| ask              | Double     | -                                  | Price of the last ask as of bin end. | Price of the last ask as of bin end.      |
| ask_size         | Long       | -                                  | Size of the last ask as of bin end.  | Size of the last ask as of bin end.       |


### Trades and/or quotes snapshot

Retrieves snapshot of tick data (trades, quote, trades and quotes) for given symbols and filtering rules.

Retrieving trade snapshot at 12:00:00.00:

```
List<String> symbols = new ArrayList<String>();
symbols.add("CL/17U.NXG");
List<TickDataRequest.Flag> flags = new ArrayList<TickDataRequest.Flag>();
flags.add(TickDataRequest.Flag.APPLY_TRADE_CORRECTIONS);
flags.add(TickDataRequest.Flag.INCLUDE_NON_REGULAR);
flags.add(TickDataRequest.Flag.USE_MARKET_TS);

TickSnapshotData tradeSnapshot = client.getTickSnapshot("ACTIV", TickDataRequest.Type.TRADES, symbols, "2017.08.16", "12:00:00.000", flags);
```

Retrieving quote snapshot at 09:15:00:

```
List<String> symbols = new ArrayList<String>();
symbols.add("CL/17U.NXG");

List<TickDataRequest.Flag> flags = new ArrayList<TickDataRequest.Flag>();
TickSnapshotData quoteSnapshot = client.getTickSnapshot("ACTIV", TickDataRequest.Type.QUOTES, symbols, "2017.08.16", "09:15:00.000", flags);
```

Retrieving trade and quote snapshot at 10:03:00:

```
List<String> symbols = new ArrayList<String>();
symbols.add("CL/17U.NXG");

List<TickDataRequest.Flag> flags = new ArrayList<TickDataRequest.Flag>();
flags.add(TickDataRequest.Flag.APPLY_TRADE_CORRECTIONS);
flags.add(TickDataRequest.Flag.INCLUDE_NON_REGULAR);
flags.add(TickDataRequest.Flag.USE_MARKET_TS);

TickSnapshotData tradeAndQuoteSnapshot = client.getTickSnapshot("ACTIV", TickDataRequest.Type.TRADES_AND_QUOTES, symbols, "2017.08.16", "10:03:00.000", flags);
```

#### Input parameters

| Parameter           | Type            | Required  | Description                                                     |
|---------------------|-----------------|-----------|-----------------------------------------------------------------|
| source              | String          | x         | Data source.                                                    |
| dataType            | Type            | x         | Specifies if output should contain trades, quotes or both.      |
| symbols             | List\<String\>  | x         | List of symbols.                                                |
| day                 | Date / String   | x         | Requested day.                                                  |
| time                | Time            | x         | Timestamp of the requested snapshot.                            |
| flags               | List\<Flag\>    |           | List of flags.                                                  |

Available Type and Flag values - as described [here](lib_java.html#input-parameters-2).

#### Output columns

| Column           | Type       | TRADES request                                              | QUOTES request                                 | TRADES_AND_QUOTES request                                   |                                                    
|------------------|------------|-------------------------------------------------------------|------------------------------------------------|-------------------------------------------------------------|
| symbol           | String     | Symbol.                                                     | Symbol.                                        | Symbol.                                                     |
| date             | Date       | Date.                                                       | Date.                                          | Date.                                                       |
| time             | Time       | Trade.                                                      | Time.                                          | Time.                                                       |
| trade            | Double     | Price of most recent trade as of requested time.            | -                                              | Price of most recent trade as of requested time.            |
| trade_size       | Long       | Size of most recent trade as of requested time.             | -                                              | Size of most recent trade as of requested time.             |
| trade_condition  | String     | Trade conditions of most recent trade as of requested time. | -                                              | Trade conditions of most recent trade as of requested time. |
| bid              | Double     | -                                                           | Price of most recent bid as of requested time. | Price of most recent bid as of requested time.              |
| bid_size         | Long       | -                                                           | Size of most recent bid as of requested time.  | Size of most recent bid as of requested time.               |
| ask              | Double     | -                                                           | Price of most recent ask as of requested time. | Price of most recent ask as of requested time.              |
| ask_size         | Long       | -                                                           | Size of most recent ask as of requested time.  | Size of most recent ask as of requested time.               |

### Orders

Retrieves orders (Level 3 data) for given symbols and filtering rules:

```
List<String> symbols = new ArrayList<String>();
symbols.add("DBK.XE");
OrderData orderData = client.getOrderData("ACTIV", symbols, "2016.10.04", "09:00:00.000", "09:01:30.000");
```

#### Input parameters

| Parameter     | Type            | Required  | Description                                  |
|---------------|-----------------|-----------|----------------------------------------------|
| source        | String          | x         | Data source.                                 |
| symbols       | List\<String\>  | x         | List of symbols.                             |
| day           | Date / String   | x         | Requested day.                               |
| fromTime      | Time            |           | Start of the requested time range.           |
| toTime        | Time            |           | End of the requested time range.             |

#### Output columns

| Column           | Type       | Description                                                |
|------------------|------------|------------------------------------------------------------|
| symbol           | String     | Symbol.                                                    |
| order_date       | Date       | Date.                                                      |
| time             | Time       | Time of trade or quote.                                    |
| action           | String     | Orderbook entry update action.                             |
| order_id         | String     | Order ID.                                                  |
| order_side       | String     | Order side.                                                |
| order_price      | Double     | Order price.                                               |
| order_size       | Long       | Order size.                                                |
| order_time       | Time       | Order time.                                                |
| trade_id         | String     | Trade ID.                                                  |
| order_type       | Time       | Order type.                                                |


### Auction data

Retrieves auction for given symbols and filtering rules:

```
List<String> symbols = new ArrayList<String>();
symbols.add("DBK.XE");
AuctionData auctionData = client.getAuctionData("ACTIV", symbols, "2016.10.04", "09:00:00.000", "09:01:30.000");
```

#### Input parameters

| Parameter     | Type            | Required  | Description                                   |
|---------------|-----------------|-----------|-----------------------------------------------|
| source        | String          | x         | Data source.                                  |
| symbols       | List\<String\>  | x         | List of symbols.                              |
| day           | Date / String   | x         | Requested day.                                |

#### Output columns

| Column                | Type   | Description                                                |
|-----------------------|--------|------------------------------------------------------------|
| time                  | Time   | Time.                                                      |
| imbalance_buy_volume  | Long   | Imbalance buy volume.                                      |
| imbalance_sell_volume | Long   | Imbalance sell volume.                                     |
| imbalance_volume_time | Time   | Imbalance volume time.                                     |
| cross_type            | String | Cross type.                                                |
| uncrossing_date       | Date   | Uncrossing date.                                           |
| uncrossing_time       | Time   | Uncrossing time.                                           |
| uncrossing_condition  | String | Uncrossing condition.                                      |
| uncrossing_price      | Double | Uncrossing price.                                          |
| uncrossing_volume     | Long   | Uncrossing volume.                                         |
| uncrossing_type       | String | Uncrossing type.                                           |
| auction_date          | Date   | Auction date.                                              |
| auction_time          | Time   | Auction time.                                              |
| auction_status        | Long   | Auction status.                                            |
| auction               | Double | Auction.                                                   |
| auction_volume        | Long   | Auction volume.                                            |
| auction_type          | String | Auction type.                                              |

### End of day

Retrieving end of day data:

```
List<String> symbols = new ArrayList<String>();
symbols.add("DBK.XE");
EndOfDayData eodData = client.getEndOfDayData("ACTIV", symbols, "2016.10.04", "2017.10.04");
```

#### Input parameters

| Parameter     | Type            | Required  | Description                                   |
|---------------|-----------------|-----------|-----------------------------------------------|
| source        | String          | x         | Data source.                                  |
| symbols       | List\<String\>  | x         | List of symbols.                              |
| first_day     | Date / String   | x         | First requested day.                          |
| last_day      | Date / String   | x         | Last requested day.                           |

#### Output columns

| Column           | Type       | Description                   |
|------------------|------------|-------------------------------|
| symbol           | String     | Symbol.                       |
| date             | Date       | Date.                         |
| open             | Double     | Official open price.          |
| high             | Double     | Official high price.          |
| low              | Double     | Official low price.           |
| close            | Double     | Official close price.         |
| trade_count      | Long       | Total trade count.            |
| volume           | Long       | Total trades volume.          |

### Settlement prices

Retrieves settlement prices for given list of derivative symbols:

```
List<String> symbols = new ArrayList<String>();
symbols.add("DBK.XE");
SettlementPricesData prices = client.getSettlementPrices("ACTIV", symbols, "2016.10.04", "2016.11.04");
```

#### Input parameters

| Parameter     | Type            | Required  | Description                                         |
|---------------|-----------------|-----------|-----------------------------------------------------|
| source        | String          | x         | Data source.                                        |
| symbols       | List\<String\>  | x         | List of symbols.                                    |
| first_day     | Date / String   | x         | First requested day.                                |
| last_day      | Date / String   | x         | Last requested day.                                 |

#### Output columns

| Column                     | Type   | Description                                                 |
|----------------------------|--------|-------------------------------------------------------------|
| symbol                     | String | Symbol.                                                     |
| settlement_date            | Date   | Settlement date.                                            |
| settlement_time            | Time   | Settlement time.                                            |
| settlement                 | Double | Settlement.                                                 |
| settlement_status          | Long   | Settlement status.                                          |
| settlement_size            | Long   | Settlement size.                                            |
| capture_date               | Date   | Capture date (date of the update recording - UTC timezone). |
| capture_time               | Time   | Capture time (time of the update recording - UTC timezone). |


Retrieving reference data
-------------------------

### Reference data

Retrieves reference data for given symbols and date.
Note searching by pattern and retrieval of basing reference data is possible via lookupSymbols method.

```
String instrument = "ZALd.BTE"
ReferenceData referenceData = client.getReferenceData("ACTIV", instrument, "2016.09.01");
```

#### Input parameters

| Parameter     | Type            | Required  | Description                                         |
|---------------|-----------------|-----------|-----------------------------------------------------|
| source        | String          | x         | Data source.                                        |
| symbols       | List\<String\>  | x         | List of symbols.                                    |
| day           | Date / String   | x         | Requested day.                                      |

#### Output columns

| Column                    | Type   | Description                                                  |
|---------------------------|--------|--------------------------------------------------------------|
| symbol                    | String | Symbol.                                                      |
| local_code                | String | Local code.                                                  |
| isin                      | String | Isin.                                                        |
| name                      | String | Name.                                                        |
| country_code              | String | Country code.                                                |
| currency                  | String | Currency.                                                    |
| entity_type               | String | Entity type.                                                 |


Additional columns available for equities:

| Column                    | Type   | Description                                                  |
|---------------------------|--------|--------------------------------------------------------------|
| lot_size                  | Long   | Lot size.                                                    |
| market_sector             | String | Market sector.                                               |
| market_segment            | String | Market segment.                                              |
| security_primary_exchange | String | Security primary exchange.                                   |
| sedol_code                | String | Sedol code.                                                  |

Additional columns available for futures:

| Column                               | Type   | Description                                       |
|--------------------------------------|--------|---------------------------------------------------|
| expiration_date                      | Date   | Expiration date.                                  |   
| contract_size                        | Long   | Contract size.                                    |
| currency_multiplier                  | Double | Currency multiplier.                              |
| minimum_tick                         | Double | Minimum tick.                                     |
| point_size                           | Double | point size.                                       |
| underlying_isin                      | String | Underlying isin.                                  |
| underlying_symbol                    | String | Underlying symbol.                                |
| underlying_currency                  | String | Underlying currency.                              |
| underlying_mic                       | String | Underlying mic.                                   |

Additional columns available for future options:

| Column                               | Type   | Description                                       |
|--------------------------------------|--------|---------------------------------------------------|
| expiration_date                      | Date   | Expiration date.                                  |   
| contract_size                        | Long   | Contract size.                                    |
| currency_multiplier                  | Double | Currency multiplier.                              |
| minimum_tick                         | Double | Minimum tick.                                     |
| option_type                          | String | Option type.                                      |
| strike_price                         | Double | Strike price.                                     |
| exercise_style                       | String | Exercise style.                                   |
| strike_price_multiplier              | Double | Strike price multiplier.                          |
| underlying_isin                      | String | Underlying isin.                                  |
| future_symbol                        | String | Future symbol.                                    |
| future_expiration_date               | Date   | Future expiration date.                           |

Additional columns available for options:

| Column                               | Type   | Description                                       |
|--------------------------------------|--------|---------------------------------------------------|
| expiration_date                      | String | Expiration date.                                  |
| contract_size                        | Long   | Contract size.                                    |
| currency_multiplier                  | Double | Option root currency multiplier.                  |
| minimum_tick                         | Double | Option root minimum tick.                         |
| option_type                          | String | Option type.                                      |
| strike_price                         | Double | Strike price.                                     |
| exercise_style                       | String | Exercise style.                                   |
| expiration_type                      | String | Expiration type.                                  |
| contract_multiplier                  | Double | Contract multiplier.                              |
| generation_number                    | String | Generation number.                                |
| strike_price_denominator             | Double | Strike price denominator.                         |
| shares_per_contract                  | Long   | Shares per contract.                              |
| underlying_isin                      | String | Underlying isin.                                  |
| underlying_symbol                    | String | Underlying symbol.                                |
| underlying_currency                  | String | Underlying currency.                              |
| underlying_mic                       | String | Underlying mic.                                   |


### Instrument status

Retrieving instrument status:

```
List<String> symbols = new ArrayList<String>();
symbols.add("DBK.XE");
InstrumentStatusData statuses = client.getInstrumentStatus("ACTIV", symbols, "2016.09.02");
```

#### Input parameters

| Parameter     | Type            | Required  | Description                                         |
|---------------|-----------------|-----------|-----------------------------------------------------|
| source        | String          | x         | Data source.                                        |
| symbols       | List\<String\>  | x         | List of symbols.                                    |
| day           | Date / String   | x         | Requested day.                                      |

#### Output columns

| Column                         | Type    | Description                                            |
|--------------------------------|---------|--------------------------------------------------------|
| symbol                         | String  | Symbol.                                                |
| date                           | Date    | Local code.                                            |
| time                           | Time    | Isin.                                                  |
| instrument_status              | String  | Instrument status.                                     |
| instrument_status_description  | String  | Instrument status description.                         |
| is_continuous_session          | Boolean | Is continuous session.                                 |

### Tick rules

Retrieving tick rules:

```
List<String> symbols = new ArrayList<String>();
symbols.add("DBK.XE");
TickRulesData tickRules = client.getTickRules("ACTIV", symbols, "2016.09.20");
```

#### Input parameters

| Parameter     | Type            | Required  | Description                                         |
|---------------|-----------------|-----------|-----------------------------------------------------|
| source        | String          | x         | Data source.                                        |
| symbols       | List\<String\>  | x         | List of symbols.                                    |
| day           | Date / String   | x         | Requested day.                                      |

#### Output columns

| Column                         | Type   | Description                                             |
|--------------------------------|--------|---------------------------------------------------------|
| symbol                         | String | Symbol.                                                 |
| tick_rule_symbol               | String | Tick rule symbol.                                       |
| lower_bound                    | Double | Lower bound.                                            |
| tick_size                      | Double | Tick size.                                              |

### Option and future chains

Retrieves option or future chain for given chain identifier (either option stem or underlying) and given exchange and day.

Retrieving option chain:

```
ChainData result = client.getChain("ACTIV", "=DJX.W", "OPRA_COMPOSITE", "2017.08.15", ChainType.OPTIONS);
```

Retrieving future chain:

```
ChainData chainMembers = client.getChain("ACTIV", "CL", "NYMEX", "2017.08.15", ChainType.FUTURES);
```

Retrieving future option chain:

```
ChainData chainMembers = client.getChain("ACTIV", "CL", "NYMEX", "2017.08.15", ChainType.FUTURE_OPTIONS);
```


#### Input parameters

| Parameter     | Type            | Required  | Description                                                                |
|---------------|-----------------|-----------|----------------------------------------------------------------------------|
| source        | String          | x         | Data source.                                                               |
| chainType     | ChainType       | x         | Specifies if output should contain options, futures or future options.     |
| symbol        | String          | x         | Stem or underlying symbol.                                                 |
| exchange      | String          | x         | Exchange identifier.                                                       |
| day           | Date / String   | x         | Requested day.                                                             |

#### Output columns

| Column           | Type       | Description                |
|------------------|------------|----------------------------|
| chainSymbol      | String     | Chain identifier.          |
| symbol           | String     | Chain member symbol.       |
| underlying       | String     | Underlying identifier.     |
| name             | String     | Chain name.                |

Available `ChainType` values:

| Type                | Description        |  
|---------------------|--------------------|
| OPTIONS             | Options.           |
| FUTURES             | Futures.           |
| FUTURE_OPTIONS      | Future Options.    |


Other data
----------

### Generic data request

Apart from the data requests described above, it is possible to query for other data sets via generic data request.
The input parameters, query logic and output content is specified by the given `request`.

#### Input parameters

| Parameter     | Type                  | Required  | Description                                                     |
|---------------|-----------------------|-----------|-----------------------------------------------------------------|
| source        | String                | x         | Data source.                                                    |
| request       | String                | x         | Custom-defined request name.                                    |
| parameters    | Map\<String, Object\> | x         | Map with input parameters required for given request.           |

#### Output columns

Output columns are dependent on the selected request.


Troubleshooting
---------------
SSL certificates.

Oracle JDK/JRE prior to 8u101 doesn't trust Let's Encrypt certificates which are used by big xyt Cloud Platform.

In order to remedy this you can:

 * upgrade to latest JDK/JRE,
 * provide your own validator/define a custom keystore that contains the required root CA,
 * import the certificate into the JVM truststore.


License
-------

[Apache License Version 2.0](http://opensource.org/licenses/Apache-2.0)

{% include links.html %}
