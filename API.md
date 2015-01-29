# API

The ParcelBright API allows one to interact with a number of resources listed
below.

Each resource will have its own document nested under the API directory on this
repository.

Here is the table of contents:

- [Liabilities](resources/liabilities.md)
- [Countries](resources/countries.md)
- [Shipments](resources/shipments.md)

As more resources become available, they will be added to this document.

## Types

The type information provided in the resources will describe the types used by
the API internally. However, these are not actual JSON types.

Below you will find the mapping between the types mentioned in the resources and
JSON types:

Type | JSON Type | Description
-----|------|--------------
`string`| String | This is a plain JSON string.
`decimal`| String | This will be a string representing a decimal number with fixed precision, for example '5.67'.
`integer`| Number | An integer, for example 10.
`parcel`| Object | This will be an object with several key/value pairs.
`address`|Object | This will be an object with several key/value pairs.
`date`| String | This will be a string encoding a date with the format YYYY-MM-DD.
`datetime`| String | This will be a string encoding date and time complying with the ISO8601 format.
