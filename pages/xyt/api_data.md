---
title: Data access API
keywords: api
summary: "Data API Overview."
sidebar: xyt_sidebar
permalink: api_data.html
folder: xyt
---


# <a class="anchor" name="api_data_catalogue.proto"></a> api_data_catalogue.proto
Defines contract file for data retrieval layer API.

Functions from this category cover automatic retrieval of data spread across various tables, unification of result structure per request,
data transformation to ensure user readability and data cleansing.


API version: `1.1.0`


## <a class="anchor" name="api_data_catalogue.proto.types"></a> Types
### <a class="anchor" name="api_data_catalogue.proto.DaysAvailabilityRequest"></a> Message: DaysAvailabilityRequest

Request for service endpoint: `/days/availability`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [RequestHeader](#types.proto.RequestHeader) | 1 | Request header. |
| symbols | repeated  string | 2 | **required** List (capped at 10) of symbols. |
| data_category | [DataCategory](#api_data_catalogue.proto.DaysAvailabilityRequest.DataCategory) | 3 | Data category filter. |

#### <a class="anchor" name="api_data_catalogue.proto.DaysAvailabilityRequest.DataCategory"></a> Enum: DaysAvailabilityRequest.DataCategory


| Value | Id  | Description |
| ----- | --- | ----------- |
| LEVEL_1 | 0 | Level 1 market data: trades and quotes. |
| LEVEL_2 | 1 | Level 2 market data: market by price. |
| LEVEL_3 | 2 | Level 3 market data: market by order. |

### <a class="anchor" name="api_data_catalogue.proto.DaysAvailabilityResponse"></a> Message: DaysAvailabilityResponse

Response from service endpoint: `/days/availability`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [ResponseHeader](#types.proto.ResponseHeader) | 1 | Response header. |
| symbols | repeated  [Symbol](#api_data_catalogue.proto.DaysAvailabilityResponse.Symbol) | 10 |  |

#### <a class="anchor" name="api_data_catalogue.proto.DaysAvailabilityResponse.Symbol"></a> Message: DaysAvailabilityResponse.Symbol


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| symbol | string | 1 | Symbol. |
| first_day | [Date](#types.proto.Date) | 2 | First day when data is available. |
| last_day | [Date](#types.proto.Date) | 3 | Last day when data is available. |
| trading_days | repeated  [Day](#api_data_catalogue.proto.DaysAvailabilityResponse.Day) | 4 | Regular trading days. |
| holidays | repeated  [Date](#types.proto.Date) | 5 | Trading holidays. |

#### <a class="anchor" name="api_data_catalogue.proto.DaysAvailabilityResponse.Day"></a> Enum: DaysAvailabilityResponse.Day


| Value | Id  | Description |
| ----- | --- | ----------- |
| SUNDAY | 0 |  |
| MONDAY | 1 |  |
| TUESDAY | 2 |  |
| WEDNESDAY | 3 |  |
| THURSDAY | 4 |  |
| FRIDAY | 5 |  |
| SATURDAY | 6 |  |

# <a class="anchor" name="api_data_retrieval.proto"></a> api_data_retrieval.proto
Defines contract file for data retrieval layer API.

Functions from this category cover automatic retrieval of data spread across various tables, unification of result structure per request,
data transformation to ensure user readability and data cleansing.


API version: `1.1.0`


## <a class="anchor" name="api_data_retrieval.proto.types"></a> Types
### <a class="anchor" name="api_data_retrieval.proto.TickDataRequest"></a> Message: TickDataRequest

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

Request for service endpoint: `/retrieval/tickData`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [RequestHeader](#types.proto.RequestHeader) | 1 | Request header. |
| type | [Type](#api_data_retrieval.proto.TickDataRequest.Type) | 2 | **required** Type of data to be retrieved. |
| symbols | repeated  string | 3 | **required** List (capped at 10) of symbols. |
| day | [Date](#types.proto.Date) | 4 | **required** Requested day |
| from_time | [Time](#types.proto.Time) | 5 | Start of the requested time range. |
| to_time | [Time](#types.proto.Time) | 6 | End of the requested time range. |
| limit | int32 | 7 | Maximum number of returned entries. |
| flags | repeated  [Flag](#api_data_retrieval.proto.TickDataRequest.Flag) | 15 | List of flags. |

#### <a class="anchor" name="api_data_retrieval.proto.TickDataRequest.Type"></a> Enum: TickDataRequest.Type

Define type of data to be retrieved.

| Value | Id  | Description |
| ----- | --- | ----------- |
| TRADES | 0 | Trades only. |
| QUOTES | 1 | Quotes only. |
| TRADES_AND_QUOTES | 2 | Trades and quotes. |

#### <a class="anchor" name="api_data_retrieval.proto.TickDataRequest.Flag"></a> Enum: TickDataRequest.Flag


| Value | Id  | Description |
| ----- | --- | ----------- |
| USE_MARKET_TS | 0 | Specifies if exchange timestamp is returned in the output. If flag `USE_MARKET_TS` is set, result contains exchange timestamp. Otherwise capture timestamp is returned. |
| APPLY_TRADE_CORRECTIONS | 1 | Specifies if trade corrections should be applied to trades. If flag `APPLY_TRADE_CORRECTIONS` is set, prices and sizes of corrected trades are adjusted accordingly. Usually corrections apply to non-regular trades and are available for limited scope of exchanges. |
| INCLUDE_TRADE_CONDITION_INFO | 5 | Specifies if trade conditions information is returned in the output. It contains mapping of trade conditions to human-readable descriptions, trade categories and markers of continuous session assignment. |
| INCLUDE_NON_REGULAR | 6 | Specifies if non-regular trades should be included in the output. |
| INCLUDE_CONTINUOUS_SESSION_ONLY | 7 | Specifies if only continuous session messages should be included in the result set. This covers filtering by trade conditions assigned to continuous session for trades and instrument status filtering for quotes. |
| EXCLUDE_CANCELLED_TRADES | 10 | Specifies if canceled trades are removed from the output. Functionality removes only canceled trades that are no corrections. |
| NANOSECONDS_TIMESTAMP | 15 | Use nanoseconds precision, if available, to represent time. |

### <a class="anchor" name="api_data_retrieval.proto.TickDataResponse"></a> Message: TickDataResponse

Retrieves binned or tick data (trades, quote, trades and quotes) for given symbols and filtering rules.

Function applies automatic aggregation in case if the size of result set exceeds threshold specified by parameter limit.
In case if aggregation is applied, no quote data are returned in the output. They are included only if raw data are retrieved, which means time interval is small enough
that output entries count is below or equal given limit. Threshold is applied to each requested product separately, therefore in case of multiple products execution overall
count might exceed limit. Number of truncated trades and quotes due to aggregation is returned in the output structure.

In case if selected data type is `TRADES_AND_QUOTES` (trades and quotes), matching of trades and quotes is based on time (exchange or capture timestamp depending if flag `USE_MARKET_TS` is set).

`ResponseHeader` contains warning message in case if particular product does not exist or has no data for requested date and time range.


Response from service endpoint: `/retrieval/tickData`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [ResponseHeader](#types.proto.ResponseHeader) | 1 | Response header. |
| date | [Date](#types.proto.Date) | 2 | Date. |
| chunks | repeated  [Chunk](#api_data_retrieval.proto.TickDataResponse.Chunk) | 10 | Chunk of tick data for single symbol. |
| binning_level | int64 | 3 | Binning level. |
| truncated_trade_rows | int64 | 4 | Number of truncated trade rows. |
| truncated_quote_rows | int64 | 5 | Number of truncated quote rows. |
| trade_conditions | map < string,  [TradeCondition](#api_data_retrieval.proto.TickDataResponse.TradeCondition) >  | 11 | Includes the mapping of trade conditions to human-readable descriptions, trade categories and markers of continuous session assignment. |

#### <a class="anchor" name="api_data_retrieval.proto.TickDataResponse.Chunk"></a> Message: TickDataResponse.Chunk


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| symbol | string | 1 | Chunked symbol. |
| series | repeated  [Series](#api_data_retrieval.proto.TickDataResponse.Series) | 10 | List of tick data. |

#### <a class="anchor" name="api_data_retrieval.proto.TickDataResponse.Series"></a> Message: TickDataResponse.Series


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

#### <a class="anchor" name="api_data_retrieval.proto.TickDataResponse.TradeCondition"></a> Message: TickDataResponse.TradeCondition


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| trade_description | string | 1 | Trade description explaining trade condition in human-readable format. |
| trade_category | string | 2 | Mapping of trade condition to category - one of Dark, Off, Lit, Lit - Auction, Lit - Periodic Auction. |

### <a class="anchor" name="api_data_retrieval.proto.OhlcvRequest"></a> Message: OhlcvRequest

Request for service endpoint: `/retrieval/ohlcv`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [RequestHeader](#types.proto.RequestHeader) | 1 | Request header. |
| symbols | repeated  string | 2 | **required** List (capped at 10) of symbols. |
| first_day | [Date](#types.proto.Date) | 3 | **required** First requested day |
| last_day | [Date](#types.proto.Date) | 4 | **required** Last requested day |
| from_time | [Time](#types.proto.Time) | 5 | Start of the requested time range. |
| to_time | [Time](#types.proto.Time) | 6 | End of the requested time range. |
| limit | int32 | 7 | Maximum number of returned entries. |

### <a class="anchor" name="api_data_retrieval.proto.OhlcvResponse"></a> Message: OhlcvResponse

Response from service endpoint: `/retrieval/ohlcv`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [ResponseHeader](#types.proto.ResponseHeader) | 1 | Response header. |
| chunks | repeated  [Chunk](#api_data_retrieval.proto.OhlcvResponse.Chunk) | 10 | Chunk of tick data for single symbol. |
| binning_level | int64 | 3 | Binning level. |

#### <a class="anchor" name="api_data_retrieval.proto.OhlcvResponse.Chunk"></a> Message: OhlcvResponse.Chunk


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| symbol | string | 1 | Chunked symbol. |
| series | repeated  [Series](#api_data_retrieval.proto.OhlcvResponse.Series) | 10 | List of open, high, low, close, volume series. |

#### <a class="anchor" name="api_data_retrieval.proto.OhlcvResponse.Series"></a> Message: OhlcvResponse.Series


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

### <a class="anchor" name="api_data_retrieval.proto.LookupOtherVenuesRequest"></a> Message: LookupOtherVenuesRequest

Retrieves siblings for given instrument. Identification is done by ISIN search with limitation to particular currency and entity class.

Request for service endpoint: `/lookup/otherVenues`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [RequestHeader](#types.proto.RequestHeader) | 1 | Request header. |
| symbol | string | 2 | **required** Symbol. |
| day | [Date](#types.proto.Date) | 3 | **required** Day that should be taken into account. |
| currency | string | 4 | **required** Currency of affined instruments. |
| entity_classes | repeated  string | 5 | One or more entity classes, omit to include equities only. |

### <a class="anchor" name="api_data_retrieval.proto.LookupOtherVenuesResponse"></a> Message: LookupOtherVenuesResponse

Retrieves siblings for given instrument. Identification is done by ISIN search with limitation to particular currency and entity class.

Response from service endpoint: `/lookup/otherVenues`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [ResponseHeader](#types.proto.ResponseHeader) | 1 | Response header. |
| symbols | repeated  [Symbol](#api_data_retrieval.proto.LookupOtherVenuesResponse.Symbol) | 10 | List of matched symbols. |

#### <a class="anchor" name="api_data_retrieval.proto.LookupOtherVenuesResponse.Symbol"></a> Message: LookupOtherVenuesResponse.Symbol


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

### <a class="anchor" name="api_data_retrieval.proto.OrderDataRequest"></a> Message: OrderDataRequest

Request for retrieval of order stream for given symbol(s), day and selected time range.

Request for service endpoint: `/retrieval/orders`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [RequestHeader](#types.proto.RequestHeader) | 1 | Request header |
| symbols | repeated  string | 2 | **required** List of symbols. |
| day | [Date](#types.proto.Date) | 3 | **required** Requested day. |
| from_time | [Time](#types.proto.Time) | 4 | Start of the requested time range. |
| to_time | [Time](#types.proto.Time) | 5 | End of the requested time range. |

### <a class="anchor" name="api_data_retrieval.proto.OrderDataResponse"></a> Message: OrderDataResponse

Retrieves order stream for given symbol(s) on selected day within given time range.

Response from service endpoint `/retrieval/orders`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [ResponseHeader](#types.proto.ResponseHeader) | 1 | Response header. |
| date | [Date](#types.proto.Date) | 2 | Date. |
| chunks | repeated  [Chunk](#api_data_retrieval.proto.OrderDataResponse.Chunk) | 10 | Chunk of order stream for single symbol. |

#### <a class="anchor" name="api_data_retrieval.proto.OrderDataResponse.Chunk"></a> Message: OrderDataResponse.Chunk


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| symbol | string | 1 | Chunked symbol. |
| series | repeated  [Series](#api_data_retrieval.proto.OrderDataResponse.Series) | 10 | Order stream. |

#### <a class="anchor" name="api_data_retrieval.proto.OrderDataResponse.Series"></a> Message: OrderDataResponse.Series


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

### <a class="anchor" name="api_data_retrieval.proto.AuctionDataRequest"></a> Message: AuctionDataRequest

Request for retrieval of auction data for given symbol(s) on given day.

Request for service endpoint `/retrieval/auctions`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [RequestHeader](#types.proto.RequestHeader) | 1 | Request header. |
| symbols | repeated  string | 2 | **required** List of symbols. |
| day | [Date](#types.proto.Date) | 3 | **required** Requested day. |

### <a class="anchor" name="api_data_retrieval.proto.AuctionDataResponse"></a> Message: AuctionDataResponse

Retrieves auction data for given symbol(s) on given day. Output includes auction information, uncrossing and imbalance information.

Response from service endpoint `/retrieval/auctions`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [ResponseHeader](#types.proto.ResponseHeader) | 1 | Response header. |
| date | [Date](#types.proto.Date) | 2 | Date. |
| chunks | repeated  [Chunk](#api_data_retrieval.proto.AuctionDataResponse.Chunk) | 10 | Chunk of auction data for single symbol. |

#### <a class="anchor" name="api_data_retrieval.proto.AuctionDataResponse.Chunk"></a> Message: AuctionDataResponse.Chunk


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| symbol | string | 1 | Chunked symbol. |
| series | repeated  [Series](#api_data_retrieval.proto.AuctionDataResponse.Series) | 10 | List of instrument status changes. |

#### <a class="anchor" name="api_data_retrieval.proto.AuctionDataResponse.Series"></a> Message: AuctionDataResponse.Series


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

### <a class="anchor" name="api_data_retrieval.proto.InstrumentStatusRequest"></a> Message: InstrumentStatusRequest

Request for instrument status for given symbols(s) on given day on selected exchanges.

Request for service endpoint `/retrieval/instrumentStatus`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [RequestHeader](#types.proto.RequestHeader) | 1 | Request header. |
| symbols | repeated  string | 2 | **required** List of symbols. |
| day | [Date](#types.proto.Date) | 3 | **required** Requested day. |

### <a class="anchor" name="api_data_retrieval.proto.InstrumentStatusResponse"></a> Message: InstrumentStatusResponse

Retrieves instrument status changes for given day on given exchange(s).

Response from service endpoint `retrieval/instrumentStatus`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [ResponseHeader](#types.proto.ResponseHeader) | 1 | Response header. |
| date | [Date](#types.proto.Date) | 2 | Date. |
| chunks | repeated  [Chunk](#api_data_retrieval.proto.InstrumentStatusResponse.Chunk) | 10 | Chunk of instrument status changes for single symbol. |

#### <a class="anchor" name="api_data_retrieval.proto.InstrumentStatusResponse.Chunk"></a> Message: InstrumentStatusResponse.Chunk


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| symbol | string | 1 | Chunked symbol. |
| series | repeated  [Series](#api_data_retrieval.proto.InstrumentStatusResponse.Series) | 10 | List of instrument status changes. |

#### <a class="anchor" name="api_data_retrieval.proto.InstrumentStatusResponse.Series"></a> Message: InstrumentStatusResponse.Series


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| time | [Time](#types.proto.Time) | 1 | Time. |
| seq | int64 | 2 | Sequence number to allow proper alignment to e.g.: trades which can be in same milliseconds. |
| instrument_status | string | 3 | Instrument status. |
| instrument_status_description | string | 4 | Instrument status description. |
| continuous_session | bool | 5 | Continuous session indicator. |

### <a class="anchor" name="api_data_retrieval.proto.TickRulesRequest"></a> Message: TickRulesRequest

Request for retrieval of tick rules for selected symbol on given day on given exchange

Request for service endpoint `/retrieval/tickRules`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [RequestHeader](#types.proto.RequestHeader) | 1 | Request header. |
| symbols | repeated  string | 2 | **required** List of symbol. |
| day | [Date](#types.proto.Date) | 3 | **required** Requested day. |

### <a class="anchor" name="api_data_retrieval.proto.TickRulesResponse"></a> Message: TickRulesResponse

Retrieves tick rules for selected symbol on given day on given exchange. Returns tick sizes for corresponding volume bands.

Response from service endpoint `/retrieval/tickRules`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [ResponseHeader](#types.proto.ResponseHeader) | 1 | Response header. |
| chunks | repeated  [Chunk](#api_data_retrieval.proto.TickRulesResponse.Chunk) | 10 | Chunk of tick rule for single symbol. |

#### <a class="anchor" name="api_data_retrieval.proto.TickRulesResponse.Chunk"></a> Message: TickRulesResponse.Chunk


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| symbol | string | 1 | Chunked symbol. |
| series | repeated  [Series](#api_data_retrieval.proto.TickRulesResponse.Series) | 10 | List of tick rules data. |

#### <a class="anchor" name="api_data_retrieval.proto.TickRulesResponse.Series"></a> Message: TickRulesResponse.Series


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| tick_rule_symbol | string | 1 | Rule identifier. |
| lower_bound | double | 2 | Lower bound for tick size application. |
| tick_size | double | 3 | Tick size |

# <a class="anchor" name="api_analytics.proto"></a> api_analytics.proto
Defines contract file for analytics API layer.

Category includes all analytical functions that derive various indicators e.g. trade impact, fixed size spread and others.


API version: `1.1.0`


## <a class="anchor" name="api_analytics.proto.types"></a> Types
### <a class="anchor" name="api_analytics.proto.AverageDailyVolumeRequest"></a> Message: AverageDailyVolumeRequest

Average Daily Volume returns two types of results:

* daily volume across history of days, which is specified by parameter history_length. Aggregation reflects total traded volume for particular day and one of the following: symbol/symbol and trade category/symbol and trade condition.
* aggregation of the volume by time bars over given window size expressed in days. Aggregation reflects total volume for particular time bar and one of the following: symbol/symbol and trade category/symbol and trade condition. Calculation is based on number of days in the past specified by window size. The result is just a sum of trade sizes with no normalization applied.


In any case `history_length` or `window_size` reflect number of trading days.
Result set contains `trade_size` and `trade_count` to allow seamless aggregation switch between trade condition and instrument level.

`ResponseHeader` contains warning message in case if particular product does not exist or has no data for requested date and time range.


Request for service endpoint: `/analytics/averageDailyVolume`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [RequestHeader](#types.proto.RequestHeader) | 1 | Request header. |
| symbols | repeated  string | 2 | **required** List (capped at 10) of symbols. |
| day | [Date](#types.proto.Date) | 3 | **required** Requested day. |
| minutes_bar | int32 | 4 | Bar size in minutes (applicable only for request type TIME_BARS). |
| window_size | int32 | 5 | Window size (applicable only for request type TIME_BARS). |
| history_length | int32 | 6 | History length (applicable only for request type MOVING_WINDOW). |
| type | [Type](#api_analytics.proto.AverageDailyVolumeRequest.Type) | 10 | Type of measure. |
| flags | repeated  [Flag](#api_analytics.proto.AverageDailyVolumeRequest.Flag) | 15 | List of flags. |

#### <a class="anchor" name="api_analytics.proto.AverageDailyVolumeRequest.Type"></a> Enum: AverageDailyVolumeRequest.Type


| Value | Id  | Description |
| ----- | --- | ----------- |
| MOVING_WINDOW | 0 | Request type `MOVING_WINDOW` calculates daily volumes for history_length number of days. |
| TIME_BARS | 1 | Request type `TIME_BARS` calculates a distribution of volume per given bars based on data from date period (day - window_size number of days; day). |

#### <a class="anchor" name="api_analytics.proto.AverageDailyVolumeRequest.Flag"></a> Enum: AverageDailyVolumeRequest.Flag


| Value | Id  | Description |
| ----- | --- | ----------- |
| USE_MARKET_TS | 0 | Specifies if exchange timestamp is returned in the output. If flag `USE_MARKET_TS` is set, result contains exchange timestamp. Otherwise capture timestamp is returned. |
| APPLY_TRADE_CORRECTIONS | 1 | Specifies if trade corrections should be applied to trades. If flag `APPLY_TRADE_CORRECTIONS` is set, prices and sizes of corrected trades are adjusted accordingly. Usually corrections apply to non-regular trades and are available for limited scope of exchanges. |
| INCLUDE_TRADE_CONDITION_INFO | 5 | Specifies if trade conditions information is returned in the output. It contains mapping of trade conditions to human-readable descriptions, trade categories and markers of continuous session assignment. |
| INCLUDE_NON_REGULAR | 6 | Specifies if non-regular trades should be included in the output. |
| INCLUDE_CONTINUOUS_SESSION_ONLY | 7 | Specifies if only continuous session messages should be included in the result set. This covers filtering by trade conditions assigned to continuous session for trades and instrument status filtering for quotes. |
| EXCLUDE_CANCELLED_TRADES | 10 | Specifies if canceled trades are removed from the output. Functionality removes only canceled trades that are no corrections. |
| NANOSECONDS_TIMESTAMP | 15 | Use nanoseconds precision, if available, to represent time. |

### <a class="anchor" name="api_analytics.proto.AverageDailyVolumeResponse"></a> Message: AverageDailyVolumeResponse

Average Daily Volume returns two types of results:

* daily volume across history of days, which is specified by parameter history_length. Aggregation reflects total traded volume for particular day and one of the following: symbol/symbol and trade category/symbol and trade condition.
* aggregation of the volume by time bars over given window size expressed in days. Aggregation reflects total volume for particular time bar and one of the following: symbol/symbol and trade category/symbol and trade condition. Calculation is based on number of days in the past specified by window size. The result is just a sum of trade sizes with no normalization applied.


In any case `history_length` or `window_size` reflect number of trading days.
Result set contains `trade_size` and `trade_count` to allow seamless aggregation switch between trade condition and instrument level.
`ResponseHeader` contains warning message in case if particular product does not exist or has no data for requested date and time range.


Response from service endpoint: `/analytics/averageDailyVolume`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [ResponseHeader](#types.proto.ResponseHeader) | 1 | Response header. |
| chunks | repeated  [Chunk](#api_analytics.proto.AverageDailyVolumeResponse.Chunk) | 10 | List of aggregations chunked by symbol and date. |
| trade_conditions | map < string,  [TradeCondition](#api_analytics.proto.AverageDailyVolumeResponse.TradeCondition) >  | 11 | Includes the mapping of trade conditions to human-readable descriptions, trade categories and markers of continuous session assignment. |

#### <a class="anchor" name="api_analytics.proto.AverageDailyVolumeResponse.Chunk"></a> Message: AverageDailyVolumeResponse.Chunk


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| symbol | string | 1 | Chunked symbol. |
| date | [Date](#types.proto.Date) | 2 | Chunked date. |
| series | repeated  [Series](#api_analytics.proto.AverageDailyVolumeResponse.Series) | 10 | List of aggregations. |

#### <a class="anchor" name="api_analytics.proto.AverageDailyVolumeResponse.Series"></a> Message: AverageDailyVolumeResponse.Series


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| time | [Time](#types.proto.Time) | 1 | Time (exchange or capture timestamp depending on flag `USE_MARKET_TS` passed in the request). |
| trade_size | double | 2 | Trade size. |
| trade_count | int64 | 3 | Trade count. |
| trade_condition | string | 4 | Trade condition. |

#### <a class="anchor" name="api_analytics.proto.AverageDailyVolumeResponse.TradeCondition"></a> Message: AverageDailyVolumeResponse.TradeCondition


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| trade_description | string | 1 | Trade description explaining trade condition in human-readable format. |
| trade_category | string | 2 | Mapping of trade condition to category - one of Dark, Off, Lit, Lit - Auction, Lit - Periodic Auction. |

### <a class="anchor" name="api_analytics.proto.TradeImpactRequest"></a> Message: TradeImpactRequest

Trade Impact returns advanced analysis of market impact caused by occurring trades.

In general the most activity on the market is concentrated about fundamental news and trade events.
Trade Impact focuses on examination how the trades occurring on the market influences best bid and offer values.

This allows to investigate following market indicators:

* the delay of market participants in reaction on trade events,
* the differences in mid changes depending on market mechanism e.g. lit orderbook, dark orderbook, periodic auction.

Methodology compares the most recent mid valid before trade and mid values following the trade within selected time range given by parameter `max_age` specified in milliseconds.

Market impact for particular trade is defined as the ratio between mid value precedent and following the trade expressed in basis points (several following mid values are possible):
`PriceChange = 10000*(abs MidPriceFollowing-MidPricePrecedent)%MidPricePrecedent.`

Time change corresponding to price change is distance between trade and mid times:
TimeChange = (integer) MidTimeFollowing-TradeTime.

However calculation of price change might slightly differ in case, if any aggregation function is selected:

* `MAX_MID`: instead of calculation price and time change for each mid value following the trade, price change is derived as max of following mid values and time change as median of time distances,
* `TIME_WEIGHTED_AVG_MID`: instead of calculation price and time change for each mid value following the trade, price change is derived as time weighted average mid (weighted by mid duration times) and time change as median of time distances.

In case of multiple trades occurring in one millisecond, trade aggregation by trade size is applied.

Market impact investigation is based on trade and quote data. In order to investigate the impact on market level there are several options available for quote source selection:

* `SAME`: Quote data taken from original symbol. Analysis returns market impact on instrument level.
* `PBBO`: Quote data taken from primary symbol. Analysis returns market impact on exchange level, when comparing instrument from primary exchange and MTF.
* `EBBO`: Quote data are taken from calculated ebbo data. Analysis returns market impact on regional level considering sibling instruments.

Data cleansing: methodology excludes incorrect quote data, which means null or non-positive bid and ask values are ignored in the data processing.


Request for service endpoint: `/analytics/tradeImpact`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [RequestHeader](#types.proto.RequestHeader) | 1 | Request header. |
| symbols | repeated  string | 2 | **required** List (capped at 10) of symbols. |
| day | [Date](#types.proto.Date) | 3 | **required** Requested day. |
| max_age | int64 | 4 | **required** Maximum age. Represents maximum time range of quotes occurring after each trade that are considered in the analysis. |
| from_time | [Time](#types.proto.Time) | 5 | Start of the requested time range. |
| to_time | [Time](#types.proto.Time) | 6 | End of the requested time range. |
| time_offset | int64 | 7 | Number of milliseconds of delay between trade and quote. Default value is 0. Parameter should be specified in case if `bbo_source` is different than `SAME`. |
| aggregation | [Aggregation](#api_analytics.proto.TradeImpactRequest.Aggregation) | 8 | Type of aggregation. Because trade impact analysis might produce large result sets, selection of aggregation function limits the amount of data. |
| bbo_source | [Source](#api_analytics.proto.TradeImpactRequest.Source) | 9 | Bbo source selection - one of SAME, PBBO, EBBO. |
| trade_category_filters | repeated  [TradeCategory](#api_analytics.proto.TradeImpactRequest.TradeCategory) | 10 | One or more trade categories, omit to include all available trade category types. |
| flags | repeated  [Flag](#api_analytics.proto.TradeImpactRequest.Flag) | 15 | List of flags. |

#### <a class="anchor" name="api_analytics.proto.TradeImpactRequest.TradeCategory"></a> Enum: TradeImpactRequest.TradeCategory


| Value | Id  | Description |
| ----- | --- | ----------- |
| DARK | 0 | Include only trades within category of dark trades. |
| LIT | 1 | Include only trades within category of lit trades. |
| AUCTION | 2 | Include only trades within category of lit - auction trades. |
| PERIODIC_AUCTION | 3 | Include only trades within category of lit - periodic auction trades. |

#### <a class="anchor" name="api_analytics.proto.TradeImpactRequest.Aggregation"></a> Enum: TradeImpactRequest.Aggregation


| Value | Id  | Description |
| ----- | --- | ----------- |
| NONE | 0 | No aggregation applied. |
| MAX_MID | 1 | Max mid used as aggregation function. |
| TIME_WEIGHTED_AVG_MID | 2 | Time weighted average mid used as aggregation function. |

#### <a class="anchor" name="api_analytics.proto.TradeImpactRequest.Source"></a> Enum: TradeImpactRequest.Source


| Value | Id  | Description |
| ----- | --- | ----------- |
| SAME | 0 | Same source as input symbol. Quote data taken from original symbol. |
| PBBO | 1 | Primary best bid offer. Quote data taken from primary symbol. |
| EBBO | 2 | European best bid offer. Quote data are taken from calculated ebbo data. |

#### <a class="anchor" name="api_analytics.proto.TradeImpactRequest.Flag"></a> Enum: TradeImpactRequest.Flag


| Value | Id  | Description |
| ----- | --- | ----------- |
| INCLUDE_TRADE_CONDITION_INFO | 0 | Specifies if trade conditions information is returned in the output. It contains mapping of trade conditions to trade categories and markers of continuous session assignment. |
| INCLUDE_SYMBOL_MATCHING_INFO | 1 | Specifies if symbols matching information is returned in the output. This information contains the mapping between requested symbol and the symbol used as quotes source (depending which Source option was selected in the execution). |

### <a class="anchor" name="api_analytics.proto.TradeImpactResponse"></a> Message: TradeImpactResponse

Trade Impact returns advanced analysis of market impact caused by occurring trades.

In general the most activity on the market is concentrated about fundamental news and trade events.
Trade Impact focuses on examination how the trades occurring on the market influences best bid and offer values.

This allows to investigate following market indicators:

* the delay of market participants in reaction on trade events,
* the differences in mid changes depending on market mechanism e.g. lit orderbook, dark orderbook, periodic auction.

Methodology compares the most recent mid valid before trade and mid values following the trade within selected time range given by parameter `max_age` specified in milliseconds.

Market impact for particular trade is defined as the ratio between mid value precedent and following the trade expressed in basis points (several following mid values are possible):
`PriceChange = 10000*(abs MidPriceFollowing-MidPricePrecedent)%MidPricePrecedent.`

Time change corresponding to price change is distance between trade and mid times:
TimeChange = (integer) MidTimeFollowing-TradeTime.

Data cleansing: methodology excludes incorrect quote data, which means null or non-positive bid and ask values are ignored in the data processing.


Response from service endpoint: `/analytics/tradeImpact`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [ResponseHeader](#types.proto.ResponseHeader) | 1 | Response header. |
| chunks | repeated  [Chunk](#api_analytics.proto.TradeImpactResponse.Chunk) | 10 | List of aggregations chunked by symbol. |
| trade_conditions | map < string,  [TradeCondition](#api_analytics.proto.TradeImpactResponse.TradeCondition) >  | 11 | Includes the mapping of trade conditions to human-readable descriptions, trade categories and markers of continuous session assignment. |
| matching_symbols | map < string,  string >  | 12 | Includes the mapping between requested symbol and the symbol used as quotes source (depending which Source option was selected in the execution). |

#### <a class="anchor" name="api_analytics.proto.TradeImpactResponse.Chunk"></a> Message: TradeImpactResponse.Chunk


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| symbol | string | 1 | Chunked symbol. |
| series | repeated  [Series](#api_analytics.proto.TradeImpactResponse.Series) | 10 | List of aggregations. |

#### <a class="anchor" name="api_analytics.proto.TradeImpactResponse.Series"></a> Message: TradeImpactResponse.Series


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| time | [Time](#types.proto.Time) | 1 | Exchange time (trade impact logic is based on exchange time only). |
| trade_size | int64 | 2 | Trade size (aggregated size in case of trades occurring in one millisecond). |
| trade_condition | string | 3 | Trade condition. |
| time_change | double | 4 | Time change. Time distance between trade and precedent mid value. In case if aggregation `MAX_MID` or `TIME_WEIGHTED_AVG_MID` is applied, median of Time changes is returned. |
| price_change | double | 5 | Price change. Defined as the ratio of mid values precedent and following given trade. In case if aggregation `MAX_MID` or `TIME_WEIGHTED_AVG_MID` is applied, mid values are aggregated accordingly. |

#### <a class="anchor" name="api_analytics.proto.TradeImpactResponse.TradeCondition"></a> Message: TradeImpactResponse.TradeCondition


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| trade_description | string | 1 | Trade description explaining trade condition in human-readable format. |
| trade_category | string | 2 | Mapping of trade condition to category - one of Dark, Off, Lit, Lit - Auction, Lit - Periodic Auction. |

### <a class="anchor" name="api_analytics.proto.TradeImpactMeasureRequest"></a> Message: TradeImpactMeasureRequest

Trade Impact Measure returns market impact indicators depending on the distance between trade and following events (trades or quote updates).

There are two types of measures available:

* `MARKET_MOVE_MEASURE`: measure is based on trades only. Market move indicator for each trade checks the number of trades occurring per grid of millisecond distance.
   Market move for given time distance is defined as number of trades that occurred till time distance divided by total number of trades per given symbol and date.
* `PRICE_IMPACT_MEASURE`: measure is based on change of the mid value comparing quotes precedent and following the particular trade.
   Price change value for single trade and chosen precedent mid (x) is defined as:
   `PriceChange_x = 10000*(abs MidPriceFollowing-MidPricePrecedent_x)%MidPricePrecedent_x`.
   and is valid in time range (TimeChange_x; TimeChange_y), where y marks next trade precedent mid change.


For all trades it is possible to identify at each time point z all valid price changes, which form price impact measure after applying averaging:

 * simple average price change
 * average price change weighted by number of trades for given TimeChange distance from trade

Option `PRICE_IMPACT_MEASURE` is strongly related to another API endpoint: Trade Impact.

There are several aggregation methods available:

* `BY_SYMBOL_DATE`: aggregation by symbol and date. The most granular aggregation level.
* `BY_SYMBOL`: aggregation by symbol (ignores date).
* `BY_MARKET`: aggregation by market. This aggregation level covers exchanges as well as various markets inside MTF exchanges (for instance products ADSd.BTE, DBKd.BTE are aggregated to market v.BTEd inside BATS_EUROPE exchange).
* `BY_MARKET_DATE`: aggregation by market and date. Same aggregation rules for market are applied as in option `BY_MARKET`.
* `BY_ALL`: aggregation to one category only (ignores symbol/date/market or exchange).
* `BY_ALL_DATE`: aggregation to one category only with grouping by date.

In any case aggregation does not consider trade condition.

Output grid selection methodology. Two output scales are available:

* `LINEAR`: linear scale depending on parameters maxAge and outputPointsCnt, calculated with using following mathematical approach:
  grid(1 til outputPointsCnt)*(maxAge/outputPointsCnt)
* `LOGARITHMIC`: combined linear - logarithmic scale, calculated by following steps:
    * generate logarithmic scale by the expression: `(integer)(outputPointsCnt/log(maxAge))*log(0.75/(1-exp((-1)/(outputPointsCnt/log(maxAge)))))`
    * remove first `[(integer)(outputPointsCnt/log(maxAge))*log(0.75/(1-exp((-1)/(outputPointsCnt/log(maxAge)))))]` points
    * fill the removed points by linear scale

Market impact investigation is based on trade and quote data. In order to investigate the market impact on market level there are several options available for quote source selection:

* `SAME`: Quote data taken from original symbol. Analysis describes market impact on instrument level.
* `PBBO`: Quote data taken from primary symbol. Analysis describes market impact on exchange level, when comparing instrument from primary exchange and MTF.
* `EBBO`: Quote data are taken from calculated ebbo data. Analysis describes market impact on regional level considering sibling instruments.

Data cleansing: methodology excludes incorrect quote data, which means null or non-positive bid and ask values are ignored in the data processing.


Request for service endpoint: `/analytics/tradeImpactMeasure`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [RequestHeader](#types.proto.RequestHeader) | 1 | Request header. |
| symbols | repeated  string | 2 | **required** List (capped at 10) of symbols. |
| first_day | [Date](#types.proto.Date) | 3 | **required** First day that should be taken into account. |
| last_day | [Date](#types.proto.Date) | 4 | **required** Last day that should be taken into account. |
| max_age | int64 | 5 | **required** Maximum age. |
| from_time | [Time](#types.proto.Time) | 6 | Start of the requested time range. |
| to_time | [Time](#types.proto.Time) | 7 | End of the requested time range. |
| types | repeated  [Type](#api_analytics.proto.TradeImpactMeasureRequest.Type) | 8 | List of measure types. |
| aggregations | repeated  [Aggregation](#api_analytics.proto.TradeImpactMeasureRequest.Aggregation) | 9 | List of aggregations. |
| output_scale | [OutputScale](#api_analytics.proto.TradeImpactMeasureRequest.OutputScale) | 10 | Type of output scale. |
| output_points_count | int64 | 11 | Maximum number of output points for one item. |
| time_offset | int64 | 12 | Number of milliseconds of delay between trade and quote. |
| bbo_source | [Source](#api_analytics.proto.TradeImpactMeasureRequest.Source) | 13 | Bbo source selection - one of SAME, PBBO, EBBO. |
| trade_category_filters | repeated  [TradeCategory](#api_analytics.proto.TradeImpactMeasureRequest.TradeCategory) | 14 | One or more trade categories, omit to include all available trade category types. |
| flags | repeated  [Flag](#api_analytics.proto.TradeImpactMeasureRequest.Flag) | 15 | List of flags. |

#### <a class="anchor" name="api_analytics.proto.TradeImpactMeasureRequest.Type"></a> Enum: TradeImpactMeasureRequest.Type


| Value | Id  | Description |
| ----- | --- | ----------- |
| MARKET_MOVE_MEASURE | 0 | Market move measure. More details included in request description. |
| PRICE_IMPACT_MEASURE | 1 | Price impact measure. More details included in request description. |

#### <a class="anchor" name="api_analytics.proto.TradeImpactMeasureRequest.Aggregation"></a> Enum: TradeImpactMeasureRequest.Aggregation


| Value | Id  | Description |
| ----- | --- | ----------- |
| BY_SYMBOL_DATE | 0 | Aggregation by symbol and date. The most granular aggregation level. |
| BY_SYMBOL | 1 | Aggregation by symbol (ignores date). |
| BY_MARKET | 2 | Aggregation by market. This aggregation level covers exchanges as well as various markets inside MTF exchanges (for instance products ADSd.BTE, DBKd.BTE are aggregated to market v.BTEd inside BATS_EUROPE exchange). |
| BY_MARKET_DATE | 3 | Aggregation by market and date. Same aggregation rules for market are applied as in option `BY_MARKET`. |
| BY_ALL | 4 | Aggregation to one category only (ignores symbol/date/market or exchange). |
| BY_ALL_DATE | 5 | Aggregation to one category only with grouping by date. |

#### <a class="anchor" name="api_analytics.proto.TradeImpactMeasureRequest.OutputScale"></a> Enum: TradeImpactMeasureRequest.OutputScale


| Value | Id  | Description |
| ----- | --- | ----------- |
| LINEAR | 0 | Linear scale depending on parameters maxAge and outputPointsCnt. |
| LOGARITHMIC | 1 | Combined linear - logarithmic scale. More details included in request description. |

#### <a class="anchor" name="api_analytics.proto.TradeImpactMeasureRequest.TradeCategory"></a> Enum: TradeImpactMeasureRequest.TradeCategory


| Value | Id  | Description |
| ----- | --- | ----------- |
| DARK | 0 | Include only trades within category of dark trades. |
| LIT | 1 | Include only trades within category of lit trades. |
| AUCTION | 2 | Include only trades within category of lit - auction trades. |
| PERIODIC_AUCTION | 3 | Include only trades within category of lit - periodic auction trades. |

#### <a class="anchor" name="api_analytics.proto.TradeImpactMeasureRequest.Source"></a> Enum: TradeImpactMeasureRequest.Source


| Value | Id  | Description |
| ----- | --- | ----------- |
| SAME | 0 | Same source as input symbol. |
| PBBO | 1 | Primary best bid offer. |
| EBBO | 2 | European best bid offer. |

#### <a class="anchor" name="api_analytics.proto.TradeImpactMeasureRequest.Flag"></a> Enum: TradeImpactMeasureRequest.Flag


| Value | Id  | Description |
| ----- | --- | ----------- |
| INCLUDE_TRADE_CONDITION_INFO | 0 | Specifies if trade conditions information is returned in the output. It contains mapping of trade conditions to trade categories and markers of continuous session assignment. |
| INCLUDE_SYMBOL_MATCHING_INFO | 1 | Specifies if symbols matching information is returned in the output. This information contains the mapping between requested symbol and the symbol used as quotes source (depending which Source option was selected in the execution). |

### <a class="anchor" name="api_analytics.proto.TradeImpactMeasureResponse"></a> Message: TradeImpactMeasureResponse

Trade Impact Measure returns market impact indicators depending on the distance between trade and following events (trades or quote updates).

There are two types of measures available:

* `MARKET_MOVE_MEASURE`: measure is based on trades only. Market move indicator for each trade checks the number of trades occurring per grid of millisecond distance.
   Market move for given time distance is defined as number of trades that occurred till time distance divided by total number of trades per given symbol and date.
* `PRICE_IMPACT_MEASURE`: measure is based on change of the mid value comparing quotes precedent and following the particular trade.
   Price change value for single trade and chosen precedent mid (x) is defined as:
   `PriceChange_x = 10000*(abs MidPriceFollowing-MidPricePrecedent_x)%MidPricePrecedent_x`.
   and is valid in time range (TimeChange_x; TimeChange_y), where y marks next trade precedent mid change.

For all trades it is possible to identify at each time point z all valid price changes, which form price impact measure after applying averaging:

 * simple average price change
 * average price change weighted by number of trades for given TimeChange distance from trade

Option `PRICE_IMPACT_MEASURE` is strongly related to another API endpoint: Trade Impact.

Data cleansing: methodology excludes incorrect quote data, which means null or non-positive bid and ask values are ignored in the data processing.


Response from service endpoint: `/analytics/tradeImpactMeasure`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [ResponseHeader](#types.proto.ResponseHeader) | 1 | Response header. |
| chunks | repeated  [Chunk](#api_analytics.proto.TradeImpactMeasureResponse.Chunk) | 10 | List of measures. |
| trade_conditions | map < string,  [TradeCondition](#api_analytics.proto.TradeImpactMeasureResponse.TradeCondition) >  | 11 | Includes the mapping of trade conditions to human-readable descriptions, trade categories and markers of continuous session assignment. |
| matching_symbols | map < string,  string >  | 12 | Includes the mapping between requested symbol and the symbol used as quotes source (depending which Source option was selected in the execution). |

#### <a class="anchor" name="api_analytics.proto.TradeImpactMeasureResponse.Chunk"></a> Message: TradeImpactMeasureResponse.Chunk


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| symbol | string | 1 | Chunked symbol. |
| date | [Date](#types.proto.Date) | 2 | Chunked date. |
| series | repeated  [Series](#api_analytics.proto.TradeImpactMeasureResponse.Series) | 10 | List of measures. |

#### <a class="anchor" name="api_analytics.proto.TradeImpactMeasureResponse.Series"></a> Message: TradeImpactMeasureResponse.Series


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| ms_from_trade | int64 | 1 | Number of milliseconds from trade event. |
| trade_condition | string | 2 | Trade condition. |
| market_move | double | 3 | Market move. Defined as trades_per_symbol (valid for particular millisecond) divided by total_trades. |
| avg_price_change | double | 4 | Average price change. Defined as average of price changes valid for particular millisecond. Value can be aggregated depending on aggregation method chosen. |
| lw_avg_price_change | double | 5 | Weighted average price change. Defined as average of price changes valid for particular millisecond weighted by number of trades valid for same time distance. Value can be aggregated depending on aggregation method chosen. |
| trades_per_symbol | int64 | 6 | Trades per symbol. Calculated as number of trades occurring till given millisecond per symbol and date. Value can be aggregated depending on aggregation method chosen. |
| total_trades | int64 | 7 | Total number of trades calculated by symbol and date. Value can be aggregated depending on aggregation method chosen. |

#### <a class="anchor" name="api_analytics.proto.TradeImpactMeasureResponse.TradeCondition"></a> Message: TradeImpactMeasureResponse.TradeCondition


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| trade_description | string | 1 | Trade description explaining trade condition in human-readable format. |
| trade_category | string | 2 | Mapping of trade condition to category - one of Dark, Off, Lit, Lit - Auction, Lit - Periodic Auction. |

### <a class="anchor" name="api_analytics.proto.FixedSizeSpreadRequest"></a> Message: FixedSizeSpreadRequest

Retrieves binned fixed size spread based on orderbook data.

Fixed size spread functionality allows the investigation of the impact on orderbook caused by hypothetical trades with various sizes.
Analysis can be used as indication of possible transaction cost on the market.

Function returns spreads between simulated top of the book (as the result of hypothetical trade) and mid prices.
The result is aggregated by time bars specified by parameter `bar_size` and chosen aggregation method (parameter `aggregation`).

Possible aggregation methods are:

 * `TIME_WEIGHTED`: weighted by spread duration,
 * `VOLUME_WEIGHTED`: weighted by volume,
 * `LAST`: snapshot of bid and ask spread for particular bin.

In case parameter `bar_size` is set to zero, smart binning methodology is applied. The functionality specifies the most adequate time bar from values:

* 100, 200, 500ms (if aggregation on millisecond level is required),
* 1, 2, 5, 10, 15, 20, 30s (if aggregation on seconds level is required),
* 1, 2, 5, 10, 15, 20, 30min (if aggregation on minutes level is required),
* 1, 2, 3, 4, 6h (if aggregation on hourly level is required).

The selection is based on assignment of the result `(toTime - fromTime)/limit` to suitable bar size.

In general result aggregation is based on following steps:

* derivation of the optimal bar size If flag bar_size=0,
* aggregation of data within time bars and application selected aggregation method,
* final grid alignment with bin end times and if required last bin time is truncated to requested time range end,
* last step aligns grid for all products in case if several products are requested and bar size differs for them.

If aggregation is applied, response contains the information about binning level and truncated orderbook entries.

`ResponseHeader` contains warning message in case if particular product does not exist or has no data for requested date and time range.


Request for service endpoint: `/analytics/fixedSizeSpread`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [RequestHeader](#types.proto.RequestHeader) | 1 | Request header. |
| symbols | repeated  string | 2 | **required** List (capped at 10) of symbols. |
| day | [Date](#types.proto.Date) | 3 | **required** Requested day |
| from_time | [Time](#types.proto.Time) | 4 | Start of the requested time range. |
| to_time | [Time](#types.proto.Time) | 5 | End of the requested time range. |
| size | double | 6 | Size of transaction quantity or value (depending on parameter `size_type` chosen). |
| size_type | [SizeType](#api_analytics.proto.FixedSizeSpreadRequest.SizeType) | 7 | Either transaction quantity or value (size times price). |
| aggregation | [Aggregation](#api_analytics.proto.FixedSizeSpreadRequest.Aggregation) | 8 | Aggregation method. |
| bar_size | int32 | 9 | Number of seconds. |
| limit | int32 | 10 | Maximum number of returned entries. |

#### <a class="anchor" name="api_analytics.proto.FixedSizeSpreadRequest.SizeType"></a> Enum: FixedSizeSpreadRequest.SizeType


| Value | Id  | Description |
| ----- | --- | ----------- |
| TRANSACTION_QUANTITY | 0 | Size of orderbook penetration expressed as transaction size. |
| TRANSACTION_VALUE | 1 | Size of orderbook penetration expressed as transaction value (multiplication of price and size). |

#### <a class="anchor" name="api_analytics.proto.FixedSizeSpreadRequest.Aggregation"></a> Enum: FixedSizeSpreadRequest.Aggregation


| Value | Id  | Description |
| ----- | --- | ----------- |
| TIME_WEIGHTED | 0 | Applies time weighted average in spread calculation. |
| VOLUME_WEIGHTED | 1 | Applies volume weighted average in spread calculation. |
| LAST | 2 | Returns snapshot of the spread. |

### <a class="anchor" name="api_analytics.proto.FixedSizeSpreadResponse"></a> Message: FixedSizeSpreadResponse

Retrieves binned fixed size spread based on orderbook data.

Fixed size spread functionality allows the investigation of the impact on orderbook caused by hypothetical trades with various sizes.
Analysis can be used as indication of possible transaction cost on the market.

Function returns spreads between simulated top of the book (as the result of hypothetical trade) and mid prices.
The result is aggregated by time bars specified by parameter `bar_size` and chosen aggregation method (parameter `aggregation`).
If aggregation is applied, response contains the information about binning level and truncated orderbook entries.

`ResponseHeader` contains warning message in case if particular product does not exist or has no data for requested date and time range.


Response from service endpoint: `/analytics/fixedSizeSpread`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [ResponseHeader](#types.proto.ResponseHeader) | 1 | Response header. |
| binning_level | int64 | 2 | Bar size used in the aggregation process. |
| truncated_orderbook_rows | int64 | 3 | Number of truncated orderbook entries in the aggregation process. |
| chunks | repeated  [Chunk](#api_analytics.proto.FixedSizeSpreadResponse.Chunk) | 10 | List of aggregations chunked by symbol and date. |

#### <a class="anchor" name="api_analytics.proto.FixedSizeSpreadResponse.Chunk"></a> Message: FixedSizeSpreadResponse.Chunk


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| symbol | string | 1 | Chunked symbol. |
| date | [Date](#types.proto.Date) | 2 | Chunked date. |
| series | repeated  [Series](#api_analytics.proto.FixedSizeSpreadResponse.Series) | 10 | List of aggregations. |

#### <a class="anchor" name="api_analytics.proto.FixedSizeSpreadResponse.Series"></a> Message: FixedSizeSpreadResponse.Series


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| time | [Time](#types.proto.Time) | 1 | Time (capture timestamp is always used due to no exchange time for in depth orderbook levels). |
| bid_spread | double | 2 | Bid spread in basis points. |
| bid_size | int64 | 3 | Bid size. |
| ask_spread | double | 4 | Ask spread in basis points. |
| ask_size | int64 | 5 | Ask size. |

### <a class="anchor" name="api_analytics.proto.EuropeanBestBidOfferRequest"></a> Message: EuropeanBestBidOfferRequest

Retrieves European best bid offer for the list of symbols.

Ideally symbols should be siblings - means products related to each other by ISIN with limitation to currency (see [LookupOtherVenuesRequest](#lab.proto.LookupOtherVenuesRequest)).

Best bid is defined as maximal bid among given products and particular amount of time.
Best offer is defined as minimal ask among given products and particular amount of time.
Function returns ebbo data for given products together with original bbo data for each symbol.

Aggregation of returned ebbo data is done using following principles:

* bar size is calculated as the most adequate from the values:
    * 100, 200, 500ms (if aggregation on millisecond level is required),
    * 1, 2, 5, 10, 15, 20, 30s (if aggregation on seconds level is required),
    * 1, 2, 5, 10, 15, 20, 30min (if aggregation on minutes level is required),
    * 1, 2, 3, 4, 6h (if aggregation on hourly level is required).
  The selection is based on assignment of the result `(toTime - fromTime)/limit` to suitable bar size,
* calculated ebbo and bbo data of each products are aggregated corresponding to derived bar size (last values are taken),
* final grid is aligned with bin end times and if required last bin time is truncated to requested time range end.

`ResponseHeader` contains warning message in case if particular product does not exist or has no data for requested date and time range.


Request for service endpoint: `/analytics/europeanBestBidOffer`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [RequestHeader](#types.proto.RequestHeader) | 1 | Request header. |
| symbols | repeated  string | 2 | **required** List (capped at 10) of symbols. |
| day | [Date](#types.proto.Date) | 3 | **required** Requested day |
| from_time | [Time](#types.proto.Time) | 4 | Start of the requested time range. |
| to_time | [Time](#types.proto.Time) | 5 | End of the requested time range. |
| limit | int32 | 10 | Maximum number of returned entries. |
| flags | repeated  [Flag](#api_analytics.proto.EuropeanBestBidOfferRequest.Flag) | 15 | List of flags. |

#### <a class="anchor" name="api_analytics.proto.EuropeanBestBidOfferRequest.Flag"></a> Enum: EuropeanBestBidOfferRequest.Flag


| Value | Id  | Description |
| ----- | --- | ----------- |
| USE_MARKET_TS | 0 | Specifies if exchange timestamp is returned in the output. If flag `USE_MARKET_TS` is set, result contains exchange timestamp. Otherwise capture timestamp is returned. |
| NANOSECONDS_TIMESTAMP | 15 | Use nanoseconds precision, if available, to represent time. |

### <a class="anchor" name="api_analytics.proto.EuropeanBestBidOfferResponse"></a> Message: EuropeanBestBidOfferResponse

Retrieves European best bid offer for the list of symbols.

Ideally symbols should be siblings - means products related to each other by ISIN with limitation to currency (see [LookupOtherVenuesRequest](#lab.proto.LookupOtherVenuesRequest)).

Best bid is defined as maximal bid among given products and particular amount of time.
Best offer is defined as minimal ask among given products and particular amount of time.
Function returns ebbo data for given products together with original bbo data for each symbol.

Aggregation of returned ebbo data is done using following principles:

* bar size is calculated as the most adequate from the values:
    * 100, 200, 500ms (if aggregation on millisecond level is required),
    * 1, 2, 5, 10, 15, 20, 30s (if aggregation on seconds level is required),
    * 1, 2, 5, 10, 15, 20, 30min (if aggregation on minutes level is required),
    * 1, 2, 3, 4, 6h (if aggregation on hourly level is required).
  The selection is based on assignment of the result `(toTime - fromTime)/limit` to suitable bar size,
* calculated ebbo and bbo data of each products are aggregated corresponding to derived bar size (last values are taken),
* final grid is aligned with bin end times and if required last bin time is truncated to requested time range end.

`ResponseHeader` contains warning message in case if particular product does not exist or has no data for requested date and time range.


Response from service endpoint: `/analytics/europeanBestBidOffer`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [ResponseHeader](#types.proto.ResponseHeader) | 1 | Response header. |
| binning_level | int64 | 2 | Bar size used in the aggregation process. |
| truncated_orderbook_rows | int64 | 3 | Number of truncated quotes in the aggregation process. |
| chunks | repeated  [Chunk](#api_analytics.proto.EuropeanBestBidOfferResponse.Chunk) | 10 | List of aggregations chunked by symbol and date. |

#### <a class="anchor" name="api_analytics.proto.EuropeanBestBidOfferResponse.Chunk"></a> Message: EuropeanBestBidOfferResponse.Chunk


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| symbol | string | 1 | Chunked symbol. |
| date | [Date](#types.proto.Date) | 2 | Chunked date. |
| series | repeated  [Series](#api_analytics.proto.EuropeanBestBidOfferResponse.Series) | 10 | List of aggregations. |

#### <a class="anchor" name="api_analytics.proto.EuropeanBestBidOfferResponse.Series"></a> Message: EuropeanBestBidOfferResponse.Series


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| time | [Time](#types.proto.Time) | 1 | Time. |
| bid | double | 2 | Bid price. |
| bid_size | int64 | 3 | Bid size. |
| ask | double | 4 | Ask price. |
| ask_size | int64 | 5 | Ask size. |

### <a class="anchor" name="api_analytics.proto.EuropeanBestBidOfferMeasureRequest"></a> Message: EuropeanBestBidOfferMeasureRequest

Retrieves European best bid offer measures for the list of symbols.

Ideally symbols should be siblings - means products related to each other by ISIN with limitation to currency (see [LookupOtherVenuesRequest](#lab.proto.LookupOtherVenuesRequest)).

The purpose of the analytics is identification of the venue with lowest transaction costs (excluding fees).

In case if more than one day is chosen, selection of time range applies to each day separately like a moving window.
There are several aggregation methods of returned measures available:

* by symbol and date: the most granular aggregation level,
* by symbol: aggregation ignores date,
* by all: aggregation ignores products and days,
* by all date: aggregation ignores products.

There are several indicators calculated based on derivation of European best bid/offer:

* average bid/ask,
* time weighted average,
* volume weighted average,
* best bid/ask period overlapping: the amount of time within requested date and time period, when particular product has bid/ask price not worse then other siblings,
* best bid/ask period non-overlapping: the amount of time within requested date and time period, when particular product has the best bid/ask price exclusively among all siblings.

`ResponseHeader` contains warning message in case if particular product does not exist or has no data for requested date and time range.


Request for service endpoint: `/analytics/europeanBestBidOfferMeasure`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [RequestHeader](#types.proto.RequestHeader) | 1 | Request header. |
| symbols | repeated  string | 2 | **required** List (capped at 10) of symbols. |
| first_day | [Date](#types.proto.Date) | 3 | **required** First day that should be taken into account. |
| last_day | [Date](#types.proto.Date) | 4 | **required** Last day that should be taken into account. |
| from_time | [Time](#types.proto.Time) | 6 | Start of the requested time range. |
| to_time | [Time](#types.proto.Time) | 7 | End of the requested time range. |
| types | repeated  [Type](#api_analytics.proto.EuropeanBestBidOfferMeasureRequest.Type) | 8 | List of measure types. |
| aggregation | [Aggregation](#api_analytics.proto.EuropeanBestBidOfferMeasureRequest.Aggregation) | 9 | Type of aggregation. |
| flags | repeated  [Flag](#api_analytics.proto.EuropeanBestBidOfferMeasureRequest.Flag) | 15 | List of flags. |

#### <a class="anchor" name="api_analytics.proto.EuropeanBestBidOfferMeasureRequest.Type"></a> Enum: EuropeanBestBidOfferMeasureRequest.Type


| Value | Id  | Description |
| ----- | --- | ----------- |
| AVERAGE_BID | 0 | Average bid for product/date depending on aggregation type. |
| AVERAGE_ASK | 1 | Average ask for product/date depending on aggregation type. |
| TIME_WEIGHTED_AVERAGE_BID | 2 | Time weighted average bid for product/date depending on aggregation type. |
| TIME_WEIGHTED_AVERAGE_ASK | 3 | Time weighted average ask for product/date depending on aggregation type. |
| VOLUME_WEIGHTED_AVERAGE_BID | 4 | Volume weighted average bid for product/date depending on aggregation type. |
| VOLUME_WEIGHTED_AVERAGE_ASK | 5 | Volume weighted average ask for product/date depending on aggregation type. |
| BEST_BID_PERIOD_OVERLAPPING | 6 | Best bid period overlapping. The amount of time within requested date and time period, when particular product has bid price not worse then other siblings. |
| BEST_ASK_PERIOD_OVERLAPPING | 7 | Best ask period overlapping. The amount of time within requested date and time period, when particular product has ask price not worse then other siblings. |
| BEST_BID_PERIOD_NON_OVERLAPPING | 8 | Best bid period non-overlapping. The amount of time within requested date and time period, when particular product has the best bid price exclusively among all siblings. |
| BEST_ASK_PERIOD_NON_OVERLAPPING | 9 | Best ask period non-overlapping. The amount of time within requested date and time period, when particular product has the best ask price exclusively among all siblings. |

#### <a class="anchor" name="api_analytics.proto.EuropeanBestBidOfferMeasureRequest.Aggregation"></a> Enum: EuropeanBestBidOfferMeasureRequest.Aggregation


| Value | Id  | Description |
| ----- | --- | ----------- |
| BY_SYMBOL_DATE | 0 | Aggregation by symbol and date. |
| BY_SYMBOL | 1 | Aggregation by symbol (ignores date). |
| BY_ALL | 2 | Aggregation by all (ignores any grouping). |
| BY_ALL_DATE | 3 | Aggregation by all and date (ignores products). |

#### <a class="anchor" name="api_analytics.proto.EuropeanBestBidOfferMeasureRequest.Flag"></a> Enum: EuropeanBestBidOfferMeasureRequest.Flag


| Value | Id  | Description |
| ----- | --- | ----------- |
| INCLUDE_TRADE_CONDITION_INFO | 0 | Specifies if trade conditions information is returned in the output. It contains mapping of trade conditions to human-readable descriptions, trade categories and markers of continuous session assignment. |

### <a class="anchor" name="api_analytics.proto.EuropeanBestBidOfferMeasureResponse"></a> Message: EuropeanBestBidOfferMeasureResponse

Retrieves European best bid offer measures for the list of symbols.

Ideally symbols should be siblings - means products related to each other by ISIN with limitation to currency (see [LookupOtherVenuesRequest](#lab.proto.LookupOtherVenuesRequest)).

The purpose of the analytics is identification of the venue with lowest transaction costs (excluding fees).

In case if more than one day is chosen, selection of time range applies to each day separately like a moving window.

`ResponseHeader` contains warning message in case if particular product does not exist or has no data for requested date and time range.


Response from service endpoint: `/analytics/europeanBestBidOfferMeasure`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [ResponseHeader](#types.proto.ResponseHeader) | 1 | Response header. |
| measures | repeated  [Measure](#api_analytics.proto.EuropeanBestBidOfferMeasureResponse.Measure) | 10 | List of measures. |

#### <a class="anchor" name="api_analytics.proto.EuropeanBestBidOfferMeasureResponse.Measure"></a> Message: EuropeanBestBidOfferMeasureResponse.Measure


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| symbol | string | 1 | Symbol of requested product. |
| date | [Date](#types.proto.Date) | 2 | Date requested. |
| average_bid | double | 3 | Average bid for product/date depending on aggregation type. |
| average_ask | double | 4 | Average ask for product/date depending on aggregation type. |
| time_weighted_average_bid | double | 5 | Time weighted average bid for product/date depending on aggregation type. |
| time_weighted_average_ask | double | 6 | Time weighted average ask for product/date depending on aggregation type. |
| volume_weighted_average_bid | double | 7 | Volume weighted average bid for product/date depending on aggregation type. |
| volume_weighted_average_ask | double | 8 | Volume weighted average ask for product/date depending on aggregation type. |
| best_bid_period_overlapping | double | 9 | Best bid period overlapping. The amount of time within requested date and time period, when particular product has bid price not worse then other siblings. |
| best_ask_period_overlapping | double | 10 | Best ask period overlapping. The amount of time within requested date and time period, when particular product has ask price not worse then other siblings. |
| best_bid_period_non_overlapping | double | 11 | Best bid period non-overlapping. The amount of time within requested date and time period, when particular product has the best bid price exclusively among all siblings. |
| best_ask_period_non_overlapping | double | 12 | Best ask period non-overlapping. The amount of time within requested date and time period, when particular product has the best ask price exclusively among all siblings. |

# <a class="anchor" name="api_raw_data.proto"></a> api_raw_data.proto
Defines API contract file for raw data retrieval.


API version: `1.1.0`


## <a class="anchor" name="api_raw_data.proto.types"></a> Types
### <a class="anchor" name="api_raw_data.proto.OverviewExchangesRequest"></a> Message: OverviewExchangesRequest

Retrieves available exchanges with exchange level statistics.
Result is limited to the specified date range and universe subset.

Request for service endpoint: `/raw/overview/exchanges`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [RequestHeader](#types.proto.RequestHeader) | 1 | Request header. |
| exchange_filters | repeated  string | 2 | One or more exchanges names, omit to include all available exchanges. |
| table_filters | repeated  string | 3 | One or more table names, omit to include all available tables. |
| symbol_filters | repeated  string | 4 | One or more symbols, omit to include all available symbols. |
| first_day | [Date](#types.proto.Date) | 5 | First day that should be taken into account. |
| last_day | [Date](#types.proto.Date) | 6 | Last day that should be taken into account. |

### <a class="anchor" name="api_raw_data.proto.OverviewExchangesResponse"></a> Message: OverviewExchangesResponse

Retrieves available exchanges with exchange level statistics.
Result is limited to the specified date range and universe subset.

Response from service endpoint: `/raw/overview/exchanges`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [ResponseHeader](#types.proto.ResponseHeader) | 1 | Response header. |
| exchanges | repeated  [Exchange](#api_raw_data.proto.OverviewExchangesResponse.Exchange) | 10 | Exchange level statistics. |

#### <a class="anchor" name="api_raw_data.proto.OverviewExchangesResponse.Exchange"></a> Message: OverviewExchangesResponse.Exchange


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

### <a class="anchor" name="api_raw_data.proto.OverviewDaysRequest"></a> Message: OverviewDaysRequest

Retrieves available tables with table-day level statistics.
Result is limited to the specified date range and universe subset.

Request for service endpoint: `/raw/overview/days`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [RequestHeader](#types.proto.RequestHeader) | 1 | Request header. |
| exchange_filters | repeated  string | 2 | One or more exchanges names, omit to include all available exchanges. |
| table_filters | repeated  string | 3 | One or more table names, omit to include all available tables. |
| symbol_filters | repeated  string | 4 | One or more symbols, omit to include all available symbols. |
| first_day | [Date](#types.proto.Date) | 5 | First day that should be taken into account. |
| last_day | [Date](#types.proto.Date) | 6 | Last day that should be taken into account. |

### <a class="anchor" name="api_raw_data.proto.OverviewDaysResponse"></a> Message: OverviewDaysResponse

Retrieves available tables with table-day level statistics.
Result is limited to the specified date range and universe subset.

Response from service endpoint: `/raw/overview/days`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [ResponseHeader](#types.proto.ResponseHeader) | 1 | Response header. |
| tables | repeated  [Table](#types.proto.Table) | 10 | Table-day level statistics. |

#### <a class="anchor" name="api_raw_data.proto.OverviewDaysResponse.Table"></a> Message: OverviewDaysResponse.Table


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| exchange | string | 1 | Exchange name. |
| table | string | 2 | Table name. |
| day | [Date](#types.proto.Date) | 3 | Date. |
| rows_count | int64 | 4 | Number of rows in the table-day that match the filtering criteria. |
| symbols_count | int64 | 5 | Number of distinct symbols in the table-day that match the filtering criteria. |
| min_time | [Time](#types.proto.Time) | 8 | Minimal time in the table-day that matches the filtering criteria. |
| max_time | [Time](#types.proto.Time) | 9 | Maximal time in the table-day that matches the filtering criteria. |

### <a class="anchor" name="api_raw_data.proto.OverviewSymbolsRequest"></a> Message: OverviewSymbolsRequest

Retrieves available tables with table-symbol level statistics.

Request for service endpoint: `/raw/overview/symbols`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [RequestHeader](#types.proto.RequestHeader) | 1 | Request header. |
| exchange_filters | repeated  string | 2 | **required** One or more exchanges names. |
| table_filters | repeated  string | 3 | One or more table names, omit to include all available tables. |
| symbol_filters | repeated  string | 4 | One or more symbols, omit to include all available symbols. |
| first_day | [Date](#types.proto.Date) | 5 | First day that should be taken into account. |
| last_day | [Date](#types.proto.Date) | 6 | Last day that should be taken into account. |

### <a class="anchor" name="api_raw_data.proto.OverviewSymbolsResponse"></a> Message: OverviewSymbolsResponse

Retrieves available tables with table-symbol level statistics.

Response from service endpoint: `/raw/overview/symbols`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [ResponseHeader](#types.proto.ResponseHeader) | 1 | Response header. |
| symbols | repeated  [Symbol](#api_raw_data.proto.OverviewSymbolsResponse.Symbol) | 10 | Table-symbol level statistics. |

#### <a class="anchor" name="api_raw_data.proto.OverviewSymbolsResponse.Symbol"></a> Message: OverviewSymbolsResponse.Symbol


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

### <a class="anchor" name="api_raw_data.proto.OverviewTablesRequest"></a> Message: OverviewTablesRequest

Retrieves available tables with table level statistics.
Result is limited to the specified date range and universe subset.

Request for service endpoint: `/raw/overview/tables`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [RequestHeader](#types.proto.RequestHeader) | 1 | Request header. |
| exchange_filters | repeated  string | 2 | One or more exchanges names, omit to include all available exchanges. |
| table_filters | repeated  string | 3 | One or more table names, omit to include all available tables. |
| symbol_filters | repeated  string | 4 | One or more symbols, omit to include all available symbols. |
| first_day | [Date](#types.proto.Date) | 5 | First day that should be taken into account. |
| last_day | [Date](#types.proto.Date) | 6 | Last day that should be taken into account. |

### <a class="anchor" name="api_raw_data.proto.OverviewTablesResponse"></a> Message: OverviewTablesResponse

Retrieves available tables with table level statistics.
Result is limited to the specified date range and universe subset.

Response from service endpoint: `/raw/overview/tables`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [ResponseHeader](#types.proto.ResponseHeader) | 1 | Response header. |
| tables | repeated  [Table](#types.proto.Table) | 10 | Table level statistics. |

#### <a class="anchor" name="api_raw_data.proto.OverviewTablesResponse.Table"></a> Message: OverviewTablesResponse.Table


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

### <a class="anchor" name="api_raw_data.proto.TimeseriesRequest"></a> Message: TimeseriesRequest

Retrieves CSV with tick-by-tick data for specified symbol, table and time range.

Request for service endpoint: `/raw/csv/timeseries`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [RequestHeader](#types.proto.RequestHeader) | 1 | Request header. |
| table | string | 2 | **required** Table name, list of tables can be obtained from `/raw/overview/tables` endpoint. |
| symbol | string | 3 | **required** Requested symbol. |
| day | [Date](#types.proto.Date) | 4 | **required** Requested day. |
| from_time | [Time](#types.proto.Time) | 5 | Start of the requested time range. |
| to_time | [Time](#types.proto.Time) | 6 | End of the requested time range. |
| columns | repeated  string | 7 | List of requested columns or, omit to get all available columns. |

### <a class="anchor" name="api_raw_data.proto.SnapshotsRequest"></a> Message: SnapshotsRequest

Retrieves CSV with snapshot data for specified symbol, table and time range.

Request for service endpoint: `/raw/csv/snapshots`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [RequestHeader](#types.proto.RequestHeader) | 1 | Request header. |
| table | string | 2 | **required** Table name, list of tables can be obtained from `/raw/overview/tables` endpoint. |
| symbol | string | 3 | **required** Requested symbol. |
| day | [Date](#types.proto.Date) | 4 | **required** Requested day. |
| from_time | [Time](#types.proto.Time) | 5 | Start of the requested time range. |
| to_time | [Time](#types.proto.Time) | 6 | End of the requested time range. |
| snapshot_interval | int64 | 7 | Size of the snapshots in milliseconds. |
| columns | repeated  string | 8 | List of requested columns or, omit to get all available columns. |

### <a class="anchor" name="api_raw_data.proto.DataRequest"></a> Message: DataRequest

Request for service endpoint: `/get/{endpoint}`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [RequestHeader](#types.proto.RequestHeader) | 1 | Request header. |
| parameters | map < string,  [Value](#types.proto.Value) >  | 2 | Request parameters. |

### <a class="anchor" name="api_raw_data.proto.DataResponse"></a> Message: DataResponse

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
