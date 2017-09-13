---
title: Data lookup API
keywords: api
summary: "Lookup API Overview."
sidebar: xyt_sidebar
permalink: api_lookup.html
folder: xyt
---


# <a class="anchor" name="api_lookup.proto"></a> api_lookup.proto
API contract file for lookup service.

API version: 1.1.0


## <a class="anchor" name="api_lookup.proto.types"></a> Types
### <a class="anchor" name="api_lookup.proto.ProductSearchRequest"></a> Message: ProductSearchRequest

Searches for pattern in the selected columns in table static listing across exchanges.

Request for service endpoint: /product


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [RequestHeader](#types.proto.RequestHeader) | 1 | Request header. |
| pattern | string | 2 | **required** Search pattern in text format. |
| source_filters | repeated  string | 13 | One or more sources names, omit to include all available sources. |
| region_filters | repeated  string | 12 | One or more regions names, omit to include all available regions. |
| exchange_filters | repeated  string | 3 | One or more exchanges names, omit to include all available exchanges. |
| product_type_filters | repeated  [ProductType](#types.proto.ProductType) | 4 | One or more product types, omit to include all available product types. |
| item_type_filters | repeated  [ItemType](#types.proto.ItemType) | 5 | One or more item types, omit to include all item types. |
| entity_class_filters | repeated  string | 6 | One or more entity classes, omit to include all entity classes. |
| first_day | [Date](#types.proto.Date) | 8 | First day that should be taken into account. |
| last_day | [Date](#types.proto.Date) | 9 | Last day that should be taken into account. |
| limit | int32 | 10 | Size of the single page. |
| start | int32 | 11 | Index of the first search result to be viewed. |
| explicit_query | bool | 15 | Suppresses automatic pattern completion and escaping for Lucene query engine. |

### <a class="anchor" name="api_lookup.proto.ProductSearchResponse"></a> Message: ProductSearchResponse

Searches for pattern in the selected columns in table static listing across exchanges.

Response from service endpoint: /product


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [ResponseHeader](#types.proto.ResponseHeader) | 1 | Response header. |
| products | repeated  [Product](#api_lookup.proto.ProductSearchResponse.Product) | 10 | List of matched symbols. |
| total_hits | int32 | 15 |  |

#### <a class="anchor" name="api_lookup.proto.ProductSearchResponse.Product"></a> Message: ProductSearchResponse.Product


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| source | string | 12 | Data source of matched product. |
| region | string | 11 | Region of matched product. |
| symbol | string | 1 | Symbol of matched product. |
| name | string | 2 | Name of matched product. |
| exchange | string | 3 | Exchange of matched product. |
| currency | string | 4 | Currency of matched product. |
| local_code | string | 5 | Local code of matched product. |
| isin | string | 6 | ISIN code of matched product. |
| market_segment | string | 7 | Market segment of matched product. |
| entity_class | string | 8 | Entity class of matched product. |
| product_type | [ProductType](#types.proto.ProductType) | 9 | Product type of matched product. |
| item_type | [ItemType](#types.proto.ItemType) | 10 | Item type of matched product. |
| first_day | [Date](#types.proto.Date) | 13 | First day in the table that matches the filtering criteria. |
| last_day | [Date](#types.proto.Date) | 14 | Last day in the table that matches the filtering criteria. |
| liquidity_index | int64 | 15 | Liquidity index. |

### <a class="anchor" name="api_lookup.proto.OverviewProductsRequest"></a> Message: OverviewProductsRequest

Retrieves available products with product types statistics.

Request for service endpoint: `/overview/products`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [RequestHeader](#types.proto.RequestHeader) | 1 | Request header. |

### <a class="anchor" name="api_lookup.proto.OverviewProductsResponse"></a> Message: OverviewProductsResponse

Retrieves available products with view level statistics.

Response from service endpoint: `/overview/products`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [ResponseHeader](#types.proto.ResponseHeader) | 1 | Response header. |
| views | repeated  [View](#api_lookup.proto.OverviewProductsResponse.View) | 10 | List of views. |

#### <a class="anchor" name="api_lookup.proto.OverviewProductsResponse.View"></a> Message: OverviewProductsResponse.View


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| source | string | 11 | Exchange data source. |
| region | string | 10 | Exchange region. |
| exchange | string | 1 | Exchange name. |
| name | string | 2 | Full name of the exchange. |
| abbreviation | string | 3 | Abbreviation of the exchange. |
| mic | string | 4 | Market identifier code. |
| product_type | [ProductType](#types.proto.ProductType) | 5 | Product type. |
| rows_count | int64 | 6 | Number of rows in the product type. |
| first_day | [Date](#types.proto.Date) | 7 | First day in the product type. |
| last_day | [Date](#types.proto.Date) | 8 | Last day in the product type. |
| feed | string | 9 | Exchange feed. |

### <a class="anchor" name="api_lookup.proto.OverviewDatasetsRequest"></a> Message: OverviewDatasetsRequest

Retrieves available datasets.

Request for service endpoint: `/overview/datasets`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [RequestHeader](#types.proto.RequestHeader) | 1 | Request header. |

### <a class="anchor" name="api_lookup.proto.OverviewDatasetsResponse"></a> Message: OverviewDatasetsResponse

Retrieves available datasets.

Response from service endpoint: `/overview/datasets`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [ResponseHeader](#types.proto.ResponseHeader) | 1 | Response header. |
| datasets | repeated  [Dataset](#api_lookup.proto.OverviewDatasetsResponse.Dataset) | 10 | List of datasets. |

#### <a class="anchor" name="api_lookup.proto.OverviewDatasetsResponse.Dataset"></a> Message: OverviewDatasetsResponse.Dataset


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| source | string | 11 | Data source. |
| region | string | 10 | Region. |
| id | string | 1 | Id. |
| exchange | string | 2 | Exchange. |
| name | string | 3 | Name. |
| title | string | 4 | Title. |
| description | string | 5 | Description. |
| location | string | 6 | Location. |
| symbols | repeated  string | 7 | Symbols. |
| abbreviations | repeated  string | 8 | Abbreviation(s) of the exchange. |
| mics | repeated  string | 9 | Market identifier codes. |
| formats | repeated  string | 12 | Formats. |
| entity_types | repeated  string | 13 | Entity type(s). |
| granularity | string | 14 | Granularity. |
| products | repeated  [Product](#api_lookup.proto.OverviewDatasetsResponse.Product) | 15 | Products. |

#### <a class="anchor" name="api_lookup.proto.OverviewDatasetsResponse.Product"></a> Message: OverviewDatasetsResponse.Product


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| product | string | 1 | Product. |
| first_day | [Date](#types.proto.Date) | 2 | First day. |

### <a class="anchor" name="api_lookup.proto.OverviewEntitiesRequest"></a> Message: OverviewEntitiesRequest

Retrieves available entity types.

Request for service endpoint: `/overview/entities`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [RequestHeader](#types.proto.RequestHeader) | 1 | Request header. |

### <a class="anchor" name="api_lookup.proto.OverviewEntitiesResponse"></a> Message: OverviewEntitiesResponse

Retrieves available entity types.

Response from service endpoint: `/overview/entities`


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| header | [ResponseHeader](#types.proto.ResponseHeader) | 1 | Response header. |
| entities | repeated  [Entity](#api_lookup.proto.OverviewEntitiesResponse.Entity) | 10 | List of entity types. |

#### <a class="anchor" name="api_lookup.proto.OverviewEntitiesResponse.Entity"></a> Message: OverviewEntitiesResponse.Entity


| Field | Type | Id  | Description |
| ----- | ---- | --- | ----------- |
| source | string | 11 | Exchange data source. |
| region | string | 10 | Exchange region. |
| entity_class | string | 1 | Entity class. |
| exchange | string | 2 | Exchange. |
| first_day | [Date](#types.proto.Date) | 5 | First day in the table that matches the filtering criteria. |
| last_day | [Date](#types.proto.Date) | 6 | Last day in the table that matches the filtering criteria. |

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
