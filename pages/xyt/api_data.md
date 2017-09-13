---
title: Data access API
keywords: api
summary: "Data API Overview."
sidebar: xyt_sidebar
permalink: api_data.html
folder: xyt
---


# <a class="anchor" name="api_data.proto"></a> api_data.proto
Defines contract file for data retrieval layer API.

Functions from this category cover automatic retrieval of data spread across various tables, unification of result structure per request,
data transformation to ensure user readability and data cleansing.


API version: `1.1.0`


## <a class="anchor" name="api_data.proto.types"></a> Types
### <a class="anchor" name="api_data.proto.DaysAvailabilityRequest"></a> Message: DaysAvailabilityRequest

Request for service endpoint: `/days/availability`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [RequestHeader](#types.proto.RequestHeader) | 1 | Request header. |
| symbols | repeated  string | 2 | **required** List (capped at 10) of symbols. |
| data_category | [DataCategory](#api_data.proto.DaysAvailabilityRequest.DataCategory) | 3 | Data category filter. |

#### <a class="anchor" name="api_data.proto.DaysAvailabilityRequest.DataCategory"></a> Enum: DaysAvailabilityRequest.DataCategory


| Value | Id  | Description |
| ----- | --- | ----------- |
| LEVEL_1 | 0 | Level 1 market data: trades and quotes. |
| LEVEL_2 | 1 | Level 2 market data: market by price. |
| LEVEL_3 | 2 | Level 3 market data: market by order. |

### <a class="anchor" name="api_data.proto.DaysAvailabilityResponse"></a> Message: DaysAvailabilityResponse

Response from service endpoint: `/days/availability`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [ResponseHeader](#types.proto.ResponseHeader) | 1 | Response header. |
| symbols | repeated  [Symbol](#api_data.proto.DaysAvailabilityResponse.Symbol) | 10 |  |

#### <a class="anchor" name="api_data.proto.DaysAvailabilityResponse.Symbol"></a> Message: DaysAvailabilityResponse.Symbol


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| symbol | string | 1 | Symbol. |
| first_day | [Date](#types.proto.Date) | 2 | First day when data is available. |
| last_day | [Date](#types.proto.Date) | 3 | Last day when data is available. |
| trading_days | repeated  [Day](#api_data.proto.DaysAvailabilityResponse.Day) | 4 | Regular trading days. |
| holidays | repeated  [Date](#types.proto.Date) | 5 | Trading holidays. |

#### <a class="anchor" name="api_data.proto.DaysAvailabilityResponse.Day"></a> Enum: DaysAvailabilityResponse.Day


| Value | Id  | Description |
| ----- | --- | ----------- |
| SUNDAY | 0 |  |
| MONDAY | 1 |  |
| TUESDAY | 2 |  |
| WEDNESDAY | 3 |  |
| THURSDAY | 4 |  |
| FRIDAY | 5 |  |
| SATURDAY | 6 |  |

### <a class="anchor" name="api_data.proto.TickDataRequest"></a> Message: TickDataRequest

Retrieves binned or tick data (trades, quote, trades and quotes) for given symbols and filtering rules.

Function applies automatic aggregation in case if the size of result set exceeds threshold specified by parameter limit.
In case if aggregation is applied, no quote data are returned in the output. They are included only if raw data are retrieved, which means time interval is small enough
that output entries count is below or equal given limit. Threshold is applied to each requested product separately, therefore in case of multiple products execution overall
count might exceed limit. Number of truncated trades and quotes due to aggregation is returned in the output structure.

Aggregation of trades is done using following principles:

* bar size is calculated as the most adequate from the values:
    * 100, 200, 500ms (if aggregation on millisecond level is required),
    * 1, 2, 5, 10, 15, 20, 30s (if aggregation on seconds level is required),
    * 1, 2, 5, 10, 15, 20, 30min (if aggregation on minutes level is required),
    * 1, 2, 3, 4, 6h (if aggregation on hourly level is required).
 The selection is based on assignment of the result `(to_time - from_time)/limit` to suitable bar size,
* price is taken as snapshot of most recent price while trade size as total sum of trade quantities in particular bin,
* grouping includes distinction of various trade conditions,
* final grid is aligned with bin end times and if required last bin time is truncated to requested time range end.

In case if selected data type is `TRADES_AND_QUOTES` (trades and quotes), matching of trades and quotes is based on time (exchange or capture timestamp depending if flag `USE_MARKET_TS` is set).

`ResponseHeader` contains warning message in case if particular product does not exist or has no data for requested date and time range.

Request for service endpoint: `/tickData`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [RequestHeader](#types.proto.RequestHeader) | 1 | Request header. |
| type | [Type](#api_data.proto.TickDataRequest.Type) | 2 | **required** Type of data to be retrieved. |
| symbols | repeated  string | 3 | **required** List (capped at 10) of symbols. |
| day | [Date](#types.proto.Date) | 4 | **required** Requested day |
| from_time | [Time](#types.proto.Time) | 5 | Start of the requested time range. |
| to_time | [Time](#types.proto.Time) | 6 | End of the requested time range. |
| limit | int32 | 7 | Maximum number of returned entries. |
| flags | repeated  [Flag](#api_data.proto.TickDataRequest.Flag) | 15 | List of flags. |

#### <a class="anchor" name="api_data.proto.TickDataRequest.Type"></a> Enum: TickDataRequest.Type

Define type of data to be retrieved.

| Value | Id  | Description |
| ----- | --- | ----------- |
| TRADES | 0 | Trades only. |
| QUOTES | 1 | Quotes only. |
| TRADES_AND_QUOTES | 2 | Trades and quotes. |

#### <a class="anchor" name="api_data.proto.TickDataRequest.Flag"></a> Enum: TickDataRequest.Flag


| Value | Id  | Description |
| ----- | --- | ----------- |
| USE_MARKET_TS | 0 | Specifies if exchange timestamp is returned in the output. If flag `USE_MARKET_TS` is set, result contains exchange timestamp. Otherwise capture timestamp is returned. |
| APPLY_TRADE_CORRECTIONS | 1 | Specifies if trade corrections should be applied to trades. If flag `APPLY_TRADE_CORRECTIONS` is set, prices and sizes of corrected trades are adjusted accordingly. Usually corrections apply to non-regular trades and are available for limited scope of exchanges. |
| INCLUDE_TRADE_CONDITION_INFO | 5 | Specifies if trade conditions information is returned in the output. It contains mapping of trade conditions to human-readable descriptions, trade categories and markers of continuous session assignment. |
| INCLUDE_NON_REGULAR | 6 | Specifies if non-regular trades should be included in the output. |
| INCLUDE_CONTINUOUS_SESSION_ONLY | 7 | Specifies if only continuous session messages should be included in the result set. This covers filtering by trade conditions assigned to continuous session for trades and instrument status filtering for quotes. |
| EXCLUDE_CANCELLED_TRADES | 10 | Specifies if canceled trades are removed from the output. Functionality removes only canceled trades that are no corrections. |
| NANOSECONDS_TIMESTAMP | 15 | Use nanoseconds precision, if available, to represent time. |

### <a class="anchor" name="api_data.proto.TickDataResponse"></a> Message: TickDataResponse

Retrieves binned or tick data (trades, quote, trades and quotes) for given symbols and filtering rules.

Function applies automatic aggregation in case if the size of result set exceeds threshold specified by parameter limit.
In case if aggregation is applied, no quote data are returned in the output. They are included only if raw data are retrieved, which means time interval is small enough
that output entries count is below or equal given limit. Threshold is applied to each requested product separately, therefore in case of multiple products execution overall
count might exceed limit. Number of truncated trades and quotes due to aggregation is returned in the output structure.

In case if selected data type is `TRADES_AND_QUOTES` (trades and quotes), matching of trades and quotes is based on time (exchange or capture timestamp depending if flag `USE_MARKET_TS` is set).

`ResponseHeader` contains warning message in case if particular product does not exist or has no data for requested date and time range.


Response from service endpoint: `/tickData`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [ResponseHeader](#types.proto.ResponseHeader) | 1 | Response header. |
| date | [Date](#types.proto.Date) | 2 | Date. |
| chunks | repeated  [Chunk](#api_data.proto.TickDataResponse.Chunk) | 10 | Chunk of tick data for single symbol. |
| binning_level | int64 | 3 | Binning level. |
| truncated_trade_rows | int64 | 4 | Number of truncated trade rows. |
| truncated_quote_rows | int64 | 5 | Number of truncated quote rows. |
| trade_conditions | map < string,  [TradeCondition](#api_data.proto.TickDataResponse.TradeCondition) >  | 11 | Includes the mapping of trade conditions to human-readable descriptions, trade categories and markers of continuous session assignment. |

#### <a class="anchor" name="api_data.proto.TickDataResponse.Chunk"></a> Message: TickDataResponse.Chunk


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| symbol | string | 1 | Chunked symbol. |
| series | repeated  [Series](#api_data.proto.TickDataResponse.Series) | 10 | List of tick data. |

#### <a class="anchor" name="api_data.proto.TickDataResponse.Series"></a> Message: TickDataResponse.Series


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| time | [Time](#types.proto.Time) | 1 | Time (exchange or capture timestamp depending on flag `USE_MARKET_TS` passed in the request). |
| trade | double | 2 | Trade price. |
| trade_size | int64 | 3 | Trade size. |
| trade_condition | string | 4 | Trade condition. |
| bid | double | 5 | Bid price. Returned only if raw data are returned (no binning applied). |
| bid_size | int64 | 6 | Bid size. Returned only if raw data are returned (no binning applied). |
| ask | double | 9 | Ask price. Returned only if raw data are returned (no binning applied). |
| ask_size | int64 | 10 | Ask size. Returned only if raw data are returned (no binning applied). |

#### <a class="anchor" name="api_data.proto.TickDataResponse.TradeCondition"></a> Message: TickDataResponse.TradeCondition


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| trade_description | string | 1 | Trade description explaining trade condition in human-readable format. |
| trade_category | string | 2 | Mapping of trade condition to category - one of Dark, Off, Lit, Lit - Auction, Lit - Periodic Auction. |

### <a class="anchor" name="api_data.proto.OhlcvRequest"></a> Message: OhlcvRequest

Request for service endpoint: `/ohlcv`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [RequestHeader](#types.proto.RequestHeader) | 1 | Request header. |
| symbols | repeated  string | 2 | **required** List (capped at 10) of symbols. |
| first_day | [Date](#types.proto.Date) | 3 | **required** First requested day |
| last_day | [Date](#types.proto.Date) | 4 | **required** Last requested day |
| from_time | [Time](#types.proto.Time) | 5 | Start of the requested time range. |
| to_time | [Time](#types.proto.Time) | 6 | End of the requested time range. |
| limit | int32 | 7 | Maximum number of returned entries. |

### <a class="anchor" name="api_data.proto.OhlcvResponse"></a> Message: OhlcvResponse

Response from service endpoint: `/ohlcv`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [ResponseHeader](#types.proto.ResponseHeader) | 1 | Response header. |
| chunks | repeated  [Chunk](#api_data.proto.OhlcvResponse.Chunk) | 10 | Chunk of tick data for single symbol. |
| binning_level | int64 | 3 | Binning level. |

#### <a class="anchor" name="api_data.proto.OhlcvResponse.Chunk"></a> Message: OhlcvResponse.Chunk


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| symbol | string | 1 | Chunked symbol. |
| series | repeated  [Series](#api_data.proto.OhlcvResponse.Series) | 10 | List of open, high, low, close, volume series. |

#### <a class="anchor" name="api_data.proto.OhlcvResponse.Series"></a> Message: OhlcvResponse.Series


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| date | [Date](#types.proto.Date) | 8 | Date. |
| time | [Time](#types.proto.Time) | 1 | Time. |
| open | double | 2 | Open price. |
| high | double | 3 | Highest price. |
| low | double | 4 | Lowest price. |
| close | double | 5 | Close price. |
| volume | int64 | 6 | Traded volume. |
| trade_count | int64 | 7 | Trades count. |

### <a class="anchor" name="api_data.proto.LookupOtherVenuesRequest"></a> Message: LookupOtherVenuesRequest

Retrieves siblings for given instrument. Identification is done by ISIN search with limitation to particular currency and entity class.

Request for service endpoint: `/lookup/otherVenues`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [RequestHeader](#types.proto.RequestHeader) | 1 | Request header. |
| symbol | string | 2 | **required** Symbol. |
| day | [Date](#types.proto.Date) | 3 | **required** Day that should be taken into account. |
| currency | string | 4 | **required** Currency of affined instruments. |
| entity_classes | repeated  string | 5 | One or more entity classes, omit to include equities only. |

### <a class="anchor" name="api_data.proto.LookupOtherVenuesResponse"></a> Message: LookupOtherVenuesResponse

Retrieves siblings for given instrument. Identification is done by ISIN search with limitation to particular currency and entity class.

Response from service endpoint: `/lookup/otherVenues`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [ResponseHeader](#types.proto.ResponseHeader) | 1 | Response header. |
| symbols | repeated  [Symbol](#api_data.proto.LookupOtherVenuesResponse.Symbol) | 10 | List of matched symbols. |

#### <a class="anchor" name="api_data.proto.LookupOtherVenuesResponse.Symbol"></a> Message: LookupOtherVenuesResponse.Symbol


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| symbol | string | 1 | Symbol of matched product. |
| name | string | 2 | Name of matched product. |
| exchange | string | 3 | Exchange of matched product. |
| currency | string | 4 | Currency of matched product. |
| local_code | string | 5 | Local code of matched product. |
| isin | string | 6 | ISIN code of matched product. |
| market_segment | string | 7 | Market segment of matched product. |
| entity_class | string | 8 | Entity class of matched product. |

### <a class="anchor" name="api_data.proto.OrderDataRequest"></a> Message: OrderDataRequest

Request for retrieval of order stream for given symbol(s), day and selected time range.

Request for service endpoint: `/orders`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [RequestHeader](#types.proto.RequestHeader) | 1 | Request header |
| symbols | repeated  string | 2 | **required** List of symbols. |
| day | [Date](#types.proto.Date) | 3 | **required** Requested day. |
| from_time | [Time](#types.proto.Time) | 4 | Start of the requested time range. |
| to_time | [Time](#types.proto.Time) | 5 | End of the requested time range. |

### <a class="anchor" name="api_data.proto.OrderDataResponse"></a> Message: OrderDataResponse

Retrieves order stream for given symbol(s) on selected day within given time range.

Response from service endpoint `/orders`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [ResponseHeader](#types.proto.ResponseHeader) | 1 | Response header. |
| date | [Date](#types.proto.Date) | 2 | Date. |
| chunks | repeated  [Chunk](#api_data.proto.OrderDataResponse.Chunk) | 10 | Chunk of order stream for single symbol. |

#### <a class="anchor" name="api_data.proto.OrderDataResponse.Chunk"></a> Message: OrderDataResponse.Chunk


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| symbol | string | 1 | Chunked symbol. |
| series | repeated  [Series](#api_data.proto.OrderDataResponse.Series) | 10 | Order stream. |

#### <a class="anchor" name="api_data.proto.OrderDataResponse.Series"></a> Message: OrderDataResponse.Series


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| time | [Time](#types.proto.Time) | 1 | Order time. |
| action | string | 2 | Orderbook operation. |
| seq | int64 | 3 | Sequence number. |
| order_date | [Date](#types.proto.Date) | 4 | Order date. |
| order_time | [Time](#types.proto.Time) | 5 | Order time. |
| order_id | string | 6 | Order id. |
| order_price | double | 7 | Order price. |
| order_size | int64 | 8 | Order size. |
| order_side | string | 9 | Order side. |
| trade_id | string | 10 | Trade id. |
| order_type | int64 | 11 | Order type. |

### <a class="anchor" name="api_data.proto.AuctionDataRequest"></a> Message: AuctionDataRequest

Request for retrieval of auction data for given symbol(s) on given day.

Request for service endpoint `/auctions`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [RequestHeader](#types.proto.RequestHeader) | 1 | Request header. |
| symbols | repeated  string | 2 | **required** List of symbols. |
| day | [Date](#types.proto.Date) | 3 | **required** Requested day. |

### <a class="anchor" name="api_data.proto.AuctionDataResponse"></a> Message: AuctionDataResponse

Retrieves auction data for given symbol(s) on given day. Output includes auction information, uncrossing and imbalance information.

Response from service endpoint `/auctions`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [ResponseHeader](#types.proto.ResponseHeader) | 1 | Response header. |
| date | [Date](#types.proto.Date) | 2 | Date. |
| chunks | repeated  [Chunk](#api_data.proto.AuctionDataResponse.Chunk) | 10 | Chunk of auction data for single symbol. |

#### <a class="anchor" name="api_data.proto.AuctionDataResponse.Chunk"></a> Message: AuctionDataResponse.Chunk


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| symbol | string | 1 | Chunked symbol. |
| series | repeated  [Series](#api_data.proto.AuctionDataResponse.Series) | 10 | List of instrument status changes. |

#### <a class="anchor" name="api_data.proto.AuctionDataResponse.Series"></a> Message: AuctionDataResponse.Series


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| time | [Time](#types.proto.Time) | 1 | Time. |
| seq | int64 | 2 | Seqence number. |
| imbalance_buy_volume | int64 | 3 | Imbalance buy volume. |
| imbalance_sell_volume | int64 | 4 | Imbalance sell volume. |
| imbalance_volume_time | [Time](#types.proto.Time) | 5 | Imbalance volume time. |
| cross_type | string | 6 | Cross type. |
| uncrossing_date | [Date](#types.proto.Date) | 7 | Uncrossing date. |
| uncrossing_time | [Time](#types.proto.Time) | 8 | Uncrossing time. |
| uncrossing_condition | string | 9 | Uncrossing condition. |
| uncrossing_price | double | 10 | Uncrossing price. |
| uncrossing_volume | int64 | 11 | Uncrossing volume. |
| uncrossing_type | string | 12 | Uncrossing type. |
| auction_date | [Date](#types.proto.Date) | 13 | Auction date. |
| auction_time | [Time](#types.proto.Time) | 14 | Auction time. |
| auction_status | int64 | 15 | Auction status. |
| auction | double | 16 | Auction. |
| auction_volume | int64 | 17 | Auction volume. |
| auction_type | string | 18 | Auction type. |

### <a class="anchor" name="api_data.proto.InstrumentStatusRequest"></a> Message: InstrumentStatusRequest

Request for instrument status for given symbols(s) on given day on selected exchanges.

Request for service endpoint `/instrumentStatus`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [RequestHeader](#types.proto.RequestHeader) | 1 | Request header. |
| symbols | repeated  string | 2 | **required** List of symbols. |
| day | [Date](#types.proto.Date) | 3 | **required** Requested day. |

### <a class="anchor" name="api_data.proto.InstrumentStatusResponse"></a> Message: InstrumentStatusResponse

Retrieves instrument status changes for given day on given exchange(s).

Response from service endpoint `/instrumentStatus`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [ResponseHeader](#types.proto.ResponseHeader) | 1 | Response header. |
| date | [Date](#types.proto.Date) | 2 | Date. |
| chunks | repeated  [Chunk](#api_data.proto.InstrumentStatusResponse.Chunk) | 10 | Chunk of instrument status changes for single symbol. |

#### <a class="anchor" name="api_data.proto.InstrumentStatusResponse.Chunk"></a> Message: InstrumentStatusResponse.Chunk


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| symbol | string | 1 | Chunked symbol. |
| series | repeated  [Series](#api_data.proto.InstrumentStatusResponse.Series) | 10 | List of instrument status changes. |

#### <a class="anchor" name="api_data.proto.InstrumentStatusResponse.Series"></a> Message: InstrumentStatusResponse.Series


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| time | [Time](#types.proto.Time) | 1 | Time. |
| seq | int64 | 2 | Sequence number to allow proper alignment to e.g.: trades which can be in same milliseconds. |
| instrument_status | string | 3 | Instrument status. |
| instrument_status_description | string | 4 | Instrument status description. |
| continuous_session | bool | 5 | Continuous session indicator. |

### <a class="anchor" name="api_data.proto.TickRulesRequest"></a> Message: TickRulesRequest

Request for retrieval of tick rules for selected symbol on given day on given exchange

Request for service endpoint `/tickRules`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [RequestHeader](#types.proto.RequestHeader) | 1 | Request header. |
| symbols | repeated  string | 2 | **required** List of symbol. |
| day | [Date](#types.proto.Date) | 3 | **required** Requested day. |

### <a class="anchor" name="api_data.proto.TickRulesResponse"></a> Message: TickRulesResponse

Retrieves tick rules for selected symbol on given day on given exchange. Returns tick sizes for corresponding volume bands.

Response from service endpoint `/tickRules`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [ResponseHeader](#types.proto.ResponseHeader) | 1 | Response header. |
| chunks | repeated  [Chunk](#api_data.proto.TickRulesResponse.Chunk) | 10 | Chunk of tick rule for single symbol. |

#### <a class="anchor" name="api_data.proto.TickRulesResponse.Chunk"></a> Message: TickRulesResponse.Chunk


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| symbol | string | 1 | Chunked symbol. |
| series | repeated  [Series](#api_data.proto.TickRulesResponse.Series) | 10 | List of tick rules data. |

#### <a class="anchor" name="api_data.proto.TickRulesResponse.Series"></a> Message: TickRulesResponse.Series


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| tick_rule_symbol | string | 1 | Rule identifier. |
| lower_bound | double | 2 | Lower bound for tick size application. |
| tick_size | double | 3 | Tick size |

### <a class="anchor" name="api_data.proto.OverviewExchangesRequest"></a> Message: OverviewExchangesRequest

Retrieves available exchanges with exchange level statistics.
Result is limited to the specified date range and universe subset.

Request for service endpoint: `/overview/exchanges`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [RequestHeader](#types.proto.RequestHeader) | 1 | Request header. |
| exchange_filters | repeated  string | 2 | One or more exchanges names, omit to include all available exchanges. |
| table_filters | repeated  string | 3 | One or more table names, omit to include all available tables. |
| symbol_filters | repeated  string | 4 | One or more symbols, omit to include all available symbols. |
| first_day | [Date](#types.proto.Date) | 5 | First day that should be taken into account. |
| last_day | [Date](#types.proto.Date) | 6 | Last day that should be taken into account. |

### <a class="anchor" name="api_data.proto.OverviewExchangesResponse"></a> Message: OverviewExchangesResponse

Retrieves available exchanges with exchange level statistics.
Result is limited to the specified date range and universe subset.

Response from service endpoint: `/overview/exchanges`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [ResponseHeader](#types.proto.ResponseHeader) | 1 | Response header. |
| exchanges | repeated  [Exchange](#api_data.proto.OverviewExchangesResponse.Exchange) | 10 | Exchange level statistics. |

#### <a class="anchor" name="api_data.proto.OverviewExchangesResponse.Exchange"></a> Message: OverviewExchangesResponse.Exchange


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| exchange | string | 1 | Exchange name. |
| name | string | 2 | Full name of the exchange. |
| abbreviation | string | 3 | Abbreviation of the exchange. |
| mic | string | 4 | Market identifier code. |
| rows_count | int64 | 5 | Number of rows in the exchange that match the filtering criteria. |
| days_count | int64 | 6 | Number of days in the table that match the filtering criteria. |
| first_day | [Date](#types.proto.Date) | 7 | First day in the exchange that matches the filtering criteria. |
| last_day | [Date](#types.proto.Date) | 8 | Last day in the exchange that matches the filtering criteria. |
| feeds | repeated  string | 9 | List of the data feeds. |
| tables | repeated  string | 10 | List of tables available in the given exchange. |

### <a class="anchor" name="api_data.proto.OverviewDaysRequest"></a> Message: OverviewDaysRequest

Retrieves available tables with table-day level statistics.
Result is limited to the specified date range and universe subset.

Request for service endpoint: `/overview/days`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [RequestHeader](#types.proto.RequestHeader) | 1 | Request header. |
| exchange_filters | repeated  string | 2 | One or more exchanges names, omit to include all available exchanges. |
| table_filters | repeated  string | 3 | One or more table names, omit to include all available tables. |
| symbol_filters | repeated  string | 4 | One or more symbols, omit to include all available symbols. |
| first_day | [Date](#types.proto.Date) | 5 | First day that should be taken into account. |
| last_day | [Date](#types.proto.Date) | 6 | Last day that should be taken into account. |

### <a class="anchor" name="api_data.proto.OverviewDaysResponse"></a> Message: OverviewDaysResponse

Retrieves available tables with table-day level statistics.
Result is limited to the specified date range and universe subset.

Response from service endpoint: `/overview/days`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [ResponseHeader](#types.proto.ResponseHeader) | 1 | Response header. |
| tables | repeated  [Table](#types.proto.Table) | 10 | Table-day level statistics. |

#### <a class="anchor" name="api_data.proto.OverviewDaysResponse.Table"></a> Message: OverviewDaysResponse.Table


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| exchange | string | 1 | Exchange name. |
| table | string | 2 | Table name. |
| day | [Date](#types.proto.Date) | 3 | Date. |
| rows_count | int64 | 4 | Number of rows in the table-day that match the filtering criteria. |
| symbols_count | int64 | 5 | Number of distinct symbols in the table-day that match the filtering criteria. |
| min_time | [Time](#types.proto.Time) | 8 | Minimal time in the table-day that matches the filtering criteria. |
| max_time | [Time](#types.proto.Time) | 9 | Maximal time in the table-day that matches the filtering criteria. |

### <a class="anchor" name="api_data.proto.OverviewSymbolsRequest"></a> Message: OverviewSymbolsRequest

Retrieves available tables with table-symbol level statistics.

Request for service endpoint: `/overview/symbols`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [RequestHeader](#types.proto.RequestHeader) | 1 | Request header. |
| exchange_filters | repeated  string | 2 | **required** One or more exchanges names. |
| table_filters | repeated  string | 3 | One or more table names, omit to include all available tables. |
| symbol_filters | repeated  string | 4 | One or more symbols, omit to include all available symbols. |
| first_day | [Date](#types.proto.Date) | 5 | First day that should be taken into account. |
| last_day | [Date](#types.proto.Date) | 6 | Last day that should be taken into account. |

### <a class="anchor" name="api_data.proto.OverviewSymbolsResponse"></a> Message: OverviewSymbolsResponse

Retrieves available tables with table-symbol level statistics.

Response from service endpoint: `/overview/symbols`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [ResponseHeader](#types.proto.ResponseHeader) | 1 | Response header. |
| symbols | repeated  [Symbol](#api_data.proto.OverviewSymbolsResponse.Symbol) | 10 | Table-symbol level statistics. |

#### <a class="anchor" name="api_data.proto.OverviewSymbolsResponse.Symbol"></a> Message: OverviewSymbolsResponse.Symbol


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| exchange | string | 1 | Exchange name. |
| table | string | 2 | Table name. |
| symbol | string | 3 | Symbol. |
| rows_count | int64 | 4 | Number of rows for the table-symbol that match the filtering criteria. |
| days_count | int64 | 5 | Number of days for the table-symbol that match the filtering criteria. |
| first_day | [Date](#types.proto.Date) | 6 | First day for the table-symbol that matches the filtering criteria. |
| last_day | [Date](#types.proto.Date) | 7 | Last day for the table-symbol that matches the filtering criteria. |
| min_time | [Time](#types.proto.Time) | 8 | Minimal time for the table-symbol that matches the filtering criteria. |
| max_time | [Time](#types.proto.Time) | 9 | Maximal time for the table-symbol that matches the filtering criteria. |

### <a class="anchor" name="api_data.proto.OverviewTablesRequest"></a> Message: OverviewTablesRequest

Retrieves available tables with table level statistics.
Result is limited to the specified date range and universe subset.

Request for service endpoint: `/overview/tables`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [RequestHeader](#types.proto.RequestHeader) | 1 | Request header. |
| exchange_filters | repeated  string | 2 | One or more exchanges names, omit to include all available exchanges. |
| table_filters | repeated  string | 3 | One or more table names, omit to include all available tables. |
| symbol_filters | repeated  string | 4 | One or more symbols, omit to include all available symbols. |
| first_day | [Date](#types.proto.Date) | 5 | First day that should be taken into account. |
| last_day | [Date](#types.proto.Date) | 6 | Last day that should be taken into account. |

### <a class="anchor" name="api_data.proto.OverviewTablesResponse"></a> Message: OverviewTablesResponse

Retrieves available tables with table level statistics.
Result is limited to the specified date range and universe subset.

Response from service endpoint: `/overview/tables`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [ResponseHeader](#types.proto.ResponseHeader) | 1 | Response header. |
| tables | repeated  [Table](#types.proto.Table) | 10 | Table level statistics. |

#### <a class="anchor" name="api_data.proto.OverviewTablesResponse.Table"></a> Message: OverviewTablesResponse.Table


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| exchange | string | 1 | Exchange name. |
| table | string | 2 | Table name. |
| rows_count | int64 | 3 | Number of rows in the table that match the filtering criteria. |
| days_count | int64 | 4 | Number of days in the table that match the filtering criteria. |
| first_day | [Date](#types.proto.Date) | 5 | First day in the table that matches the filtering criteria. |
| last_day | [Date](#types.proto.Date) | 6 | Last day in the table that matches the filtering criteria. |
| min_time | [Time](#types.proto.Time) | 7 | Mininal time in the table that matches the filtering criteria. |
| max_time | [Time](#types.proto.Time) | 8 | Maximal time in the table that matches the filtering criteria. |
| columns | repeated  string | 9 | List of columns available in the given table. |

### <a class="anchor" name="api_data.proto.TimeseriesRequest"></a> Message: TimeseriesRequest

Retrieves CSV with tick-by-tick data for specified symbol, table and time range.

Request for service endpoint: `/csv/timeseries`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [RequestHeader](#types.proto.RequestHeader) | 1 | Request header. |
| table | string | 2 | **required** Table name, list of tables can be obtained from `/overview/tables` endpoint. |
| symbol | string | 3 | **required** Requested symbol. |
| day | [Date](#types.proto.Date) | 4 | **required** Requested day. |
| from_time | [Time](#types.proto.Time) | 5 | Start of the requested time range. |
| to_time | [Time](#types.proto.Time) | 6 | End of the requested time range. |
| columns | repeated  string | 7 | List of requested columns or, omit to get all available columns. |

### <a class="anchor" name="api_data.proto.SnapshotsRequest"></a> Message: SnapshotsRequest

Retrieves CSV with snapshot data for specified symbol, table and time range.

Request for service endpoint: `/csv/snapshots`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [RequestHeader](#types.proto.RequestHeader) | 1 | Request header. |
| table | string | 2 | **required** Table name, list of tables can be obtained from `/overview/tables` endpoint. |
| symbol | string | 3 | **required** Requested symbol. |
| day | [Date](#types.proto.Date) | 4 | **required** Requested day. |
| from_time | [Time](#types.proto.Time) | 5 | Start of the requested time range. |
| to_time | [Time](#types.proto.Time) | 6 | End of the requested time range. |
| snapshot_interval | int64 | 7 | Size of the snapshots in milliseconds. |
| columns | repeated  string | 8 | List of requested columns or, omit to get all available columns. |

### <a class="anchor" name="api_data.proto.DataRequest"></a> Message: DataRequest

Request for service endpoint: `/get/{endpoint}`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [RequestHeader](#types.proto.RequestHeader) | 1 | Request header. |
| parameters | map < string,  [Value](#types.proto.Value) >  | 2 | Request parameters. |

### <a class="anchor" name="api_data.proto.DataResponse"></a> Message: DataResponse

Response from service endpoint: `/get/{endpoint}`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [ResponseHeader](#types.proto.ResponseHeader) | 1 | Response header. |
| data | [Table](#types.proto.Table) | 2 | Retrieved data. |

# <a class="anchor" name="types.proto"></a> types.proto
Defines common data types used by Cloud Platform APIs.


API version: `1.1.0`


## <a class="anchor" name="types.proto.types"></a> Types
### <a class="anchor" name="types.proto.RequestHeader"></a> Message: RequestHeader

Request header allows client to pass additional information with the request.


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| reference_id | [UUID](#types.proto.UUID) | 1 | Cross reference identifier assigned to the request by the client. |
| nanoseconds_representation | bool | 2 | If set to true, service uses nanosecond time resolution where applicable. |
| page_limit_present | one of |  |  |
| page_limit_present.page_limit | int64 | 3 | Size of the single page. |
| page_start_present | one of |  |  |
| page_start_present.page_start | int64 | 4 | Index of the first search result to be viewed. |

### <a class="anchor" name="types.proto.ResponseHeader"></a> Message: ResponseHeader

Response header allow service to pass additional information with the response.


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| request_id | [UUID](#types.proto.UUID) | 1 | Identifier assigned to the request by the service. |
| reference_id | [UUID](#types.proto.UUID) | 2 | Cross reference identifier assigned to the request by the client. Copied from [RequestHeader](#types.proto.RequestHeader). |
| warning_message | string | 4 | Warning message. |
| total_rows_present | one of |  |  |
| total_rows_present.total_rows | int64 | 3 | Total number of rows available in the data source. |

### <a class="anchor" name="types.proto.UUID"></a> Message: UUID

Represents UUID in human-readable format.


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| value | string | 1 | UUID representation. |

### <a class="anchor" name="types.proto.Date"></a> Message: Date

Represents date.


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| year | int32 | 1 | Year. Accepted values: 0-100000000. |
| month | int32 | 2 | Month. Accepted values: 1-12. |
| day | int32 | 3 | Day of month. Accepted values: 1-31. |

### <a class="anchor" name="types.proto.Time"></a> Message: Time

Represents time during a day as milliseconds.


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| milliseconds | int32 | 1 | Number of milliseconds since midnight. |
| nanoseconds | int32 | 2 | Nanoseconds part. |

### <a class="anchor" name="types.proto.Timestamp"></a> Message: Timestamp

Represents time and date as nanoseconds.


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| nanoseconds | int64 | 1 | Number of nanoseconds from midnight 2000.01.01. |

### <a class="anchor" name="types.proto.Value"></a> Message: Value

Represents a dynamically typed value which can be either:
an integer, a double, a string, a boolean, a time, a date or a list of values.


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| type | one of |  |  |
| type.string_value | string | 1 |  |
| type.integer_value | int64 | 2 |  |
| type.double_value | double | 3 |  |
| type.bool_value | bool | 4 |  |
| type.date_value | [Date](#types.proto.Date) | 5 |  |
| type.time_value | [Time](#types.proto.Time) | 6 |  |
| type.timestamp_value | [Timestamp](#types.proto.Timestamp) | 7 |  |
| type.list | [ValueList](#types.proto.ValueList) | 15 |  |

### <a class="anchor" name="types.proto.MixedList"></a> Message: MixedList

Represents a list of dynamically typed values which can be either:
an integer, a double, a string, a boolean, a time, a date.


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| values | repeated  [Value](#types.proto.Value) | 1 |  |

### <a class="anchor" name="types.proto.ValueList"></a> Message: ValueList

Represents a dynamically typed list.


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| list_type | one of |  |  |
| list_type.string_list | [StringList](#types.proto.StringList) | 1 |  |
| list_type.integer_list | [IntegerList](#types.proto.IntegerList) | 2 |  |
| list_type.double_list | [DoubleList](#types.proto.DoubleList) | 3 |  |
| list_type.bool_list | [BoolList](#types.proto.BoolList) | 4 |  |
| list_type.date_list | [DateList](#types.proto.DateList) | 5 |  |
| list_type.time_list | [TimeList](#types.proto.TimeList) | 6 |  |
| list_type.timestamp_list | [TimestampList](#types.proto.TimestampList) | 7 |  |
| list_type.mixed_list | [MixedList](#types.proto.MixedList) | 15 |  |

### <a class="anchor" name="types.proto.Table"></a> Message: Table

Represents a data table as a list of named columns.


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| columns | map < string,  [ValueList](#types.proto.ValueList) >  | 1 |  |

### <a class="anchor" name="types.proto.StringList"></a> Message: StringList

Represents a list of strings.


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| values | repeated  string | 1 |  |

### <a class="anchor" name="types.proto.IntegerList"></a> Message: IntegerList

Represents a list of integers.


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| values | repeated  int64 | 1 | Null values are represented as Long.MIN_VALUE (-2^63) |

### <a class="anchor" name="types.proto.DoubleList"></a> Message: DoubleList

Represents a list of numeric values.


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| values | repeated  double | 1 | Null values are represented as NaN |

### <a class="anchor" name="types.proto.BoolList"></a> Message: BoolList

Represents a list of boolean values.


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| values | repeated  bool | 1 |  |

### <a class="anchor" name="types.proto.DateList"></a> Message: DateList

Represents a list of date values.


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| values | repeated  [Date](#types.proto.Date) | 1 |  |

### <a class="anchor" name="types.proto.TimeList"></a> Message: TimeList

Represents a list of time values.


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| values | repeated  [Time](#types.proto.Time) | 1 |  |

### <a class="anchor" name="types.proto.TimestampList"></a> Message: TimestampList

Represents a list of timestamp values.


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| values | repeated  [Timestamp](#types.proto.Timestamp) | 1 |  |

### <a class="anchor" name="types.proto.ProductType"></a> Enum: ProductType


| Value | Id  | Description |
| ----- | --- | ----------- |
| TRADES | 0 | Trades only. |
| QUOTES | 1 | Trades and quotes. |
| MARKET_BY_PRICE | 2 | Level 2 market data: market by price. |
| MARKET_BY_ORDER | 3 | Level 3 market data: market by order. |
| OTHER_PRODUCT | 15 | Reserved. |

### <a class="anchor" name="types.proto.ItemType"></a> Enum: ItemType


| Value | Id  | Description |
| ----- | --- | ----------- |
| INDIVIDUAL_ITEM | 0 | Single traded instrument. |
| OPTION_CHAIN | 1 | Options series. |
| FUTURE_CHAIN | 2 | Future series. |
| ENTIRE_EXCHANGE | 3 | Entire exchange. |
| OTHER_ITEM | 15 | Reserved. |

### <a class="anchor" name="types.proto.ChainType"></a> Enum: ChainType


| Value | Id  | Description |
| ----- | --- | ----------- |
| OPTIONS | 0 | Options. |
| FUTURES | 1 | Futures. |
| FUTURE_OPTIONS | 2 | Future options. |
