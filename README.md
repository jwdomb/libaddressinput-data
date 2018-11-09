# Address Data Cache

This repository is a daily cache of [Google’s Address Data Service](https://chromium-i18n.appspot.com/ssl-address/data), which is used by [libaddressinput](https://github.com/googlei18n/libaddressinput). The files contain machine-readable specifications for address data. The specifications seem to be heuristics rather than comprehensive and precise restrictions.

**For ease of comparison, all of the keys in each JSON file are sorted alphabetically by the caching script. The Google service doesn’t seem to use a consistent order.** So, this isn’t a completely faithful cache of the service if, for some reason, you need the exact order of the keys from Google’s service.

At one point, there was a [request](https://docs.google.com/document/u/1/pub?id=1KL6hmksP1D-qnpyI6FpLv_C-rQBEcB_4B18SNCaN39M) to make this data available as part of Unicode’s [Common Locale Data Repository](http://cldr.unicode.org) (CLDR), but that doesn’t appear to have occured.

Contributions to the non-data portions of this repository are welcome via [issues](https://github.com/jwdomb/libaddressinput-data/issues) and [pull requests](https://github.com/jwdomb/libaddressinput-data/pulls) are welcome. For the data portions, please use the [libaddressinput](https://github.com/googlei18n/libaddressinput) repository.

## Method

Approximately once per day, around noon UTC,  the script ```update.sh``` is run, and this repository is updated.

## Sources

This repository tracks the latest data at https://chromium-i18n.appspot.com/ssl-address. The [deprecated](https://groups.google.com/forum/#!msg/libaddressinput-discuss/OgkzUv2C7IY/gzM_WT8_DAAJ), older address (https://i18napis.appspot.com/address) is not tracked.

## Terms

I make no claims whatsoever on this data. Lara Scheidegger from Google [commented to someone else](https://groups.google.com/d/msg/libaddressinput-discuss/kvhArVA2KEQ/jnz1qvEyPR8J):

> We don’t have an official license for it; it comes mostly from IPU and other general research, there are no restrictions on it that anyone involved with it knows about it...

libaddressinput is available under [Apache License 2.0](https://github.com/googlei18n/libaddressinput/blob/master/LICENSE).
