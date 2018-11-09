# Address Validation Metadata

This document is from [Google's libaddressinput wiki](https://github.com/googlei18n/libaddressinput/wiki/AddressValidationMetadata). It is available under [Apache License 2.0](https://github.com/googlei18n/libaddressinput/blob/master/LICENSE).

## Introduction

This document explains how to use Google's open-source address metadata to validate addresses.


## Details

**Data location:**

The data is publicly available at this location:

https://chromium-i18n.appspot.com/ssl-address

Data for each country is available by building a key, and appending it to the hyperlink above (possibly applying URI percent escaping where necessary).

**Building a data key:**

Keys are built from least-to-most specific address field, separated by “/”, stopping when no data for a particular level is available. Optionally, there may be a language code appended. This gives keys the generic form “` data/COUNTRY/Admin_Area/Locality/Dependent_Locality--language `”.

_Country Level Data:_

Keys consist of a root called "data", followed by the ISO 3166-2 country code, such as GB for the United Kingdom. In cases where the country has more than one local language, alternative languages can be specified by appending the lower case BCP 47 language code via "` --<language-code> `". Data for the default language is obtained by omitting the language code altogether.

For example, for Canada there are two keys, **data/CA** and **data/CA--fr** which result in the data URLs:

https://chromium-i18n.appspot.com/ssl-address/data/CA

https://chromium-i18n.appspot.com/ssl-address/data/CA--fr

_Checking for Sub-Regions:_

In any JSON response there may be a “sub\_keys” fields which denotes the existence of sub-region data. If the “sub\_keys” field exists, it contains a list of sub-region keys (separated by ‘~’) that can be appended to the key to obtain the sub-region data.

For example, for the CA the ` sub_keys ` value is:

"` AB~BC~MB~NB~NL~NT~NS~NU~ON~PE~QC~SK~YT `"

Correspondingly, data exists for the keys **data/CA/AB** through to **data/CA/YT**.

There will also be data for the French version, such as **data/CA/NB--fr**, where the value for the ` name ` is “` Nouveau-Brunswick `“ instead of ”` New Brunswick `“. Note however that the keys for a sub-region are language specific and it is not safe to assume that sub-region keys for an alternate language will be in the same order, or even the same script (see **data/HK** and **data/HK--en**, which have sub\_keys values "九龍~香港島~新界" and “Hong Kong Island~Kowloon~New Territories” respectively).

Such a hierarchy may be up to four levels deep. For example, for Brazil the key **data/BR/AC/Acrelândia** contains the data for the city of Acrelândia, in the province of Acre (known by its abbreviation AC). Note that the accented character in the key must be percent escaped in the URL (i.e. "https://chromium-i18n.appspot.com/ssl-address/data/BR/AC/Acrelândia").

**Reading the JSON:**

Many fields in the JSON refer to address fields by one-letter abbreviations. These are defined as follows:

N – Name

O – Organisation

A – Street Address Line(s)

D – Dependent locality (may be an inner-city district or a suburb)

C – City or Locality

S – Administrative area such as a state, province, island etc

Z – Zip or postal code

X – Sorting code

For the "S" field, some indication as to the data expected can be read from the “state\_name\_type“ field. If this is missing, it is assumed to be "province" - but it may be overridden with values such as "state", "island" etc.

**Validation using the metadata:**

_Required fields are filled in:_

For every country, we check that all the required fields for that country have a non-null or non-white-space value.

Required address fields are present in the JSON data for a particular key as a string of the one-letter abbreviations, assigned to the field "require". Note that a country field is always required and so is not explicitly listed in the "require" field.

For example, for the US, the value for "require" is "ACSZ". This indicates at least one address line, a city, a state and a postal code (as well as a country) are all required.

_Only fields that are normal for a particular country are used:_

Fields should not be filled in that are unexpected or unnecessary for postal addresses in a particular country. Which fields are expected is determined from the "fmt" string in the JSON data. The format string is a textual template containing literal text, formatting characters and placeholders for the address fields (as identified by their one-letter abbreviations). Formatting characters and address field placeholders are prefixed by a ‘%’ character, while all other text is literal. Formatting characters (such as “%n”) and literal text are ignored for validation purposes.

For example, the US data has a "fmt" value of "%N%n%O%n%A%n%C %S %Z". This means that the allowed fields are N, O, A, C, S and Z, which according to the definitions above correspond to name, organisation, street address lines, city, administrative area and postal code respectively.

For the Guernsey Islands, with “fmt” equal to “%N%n%O%n%A%n%X%n%C%nGUERNSEY%n%Z”, the fields N, O, A, X, C and Z are allowed (the literal text “GUERNSEY” is ignored and the ‘S’ does not imply the need for an administrative area to be present).

_Field values are "known":_

For some fields, there is a list of the permitted values. For example, for the US, the administrative area value must be filled in with either a key from the ”sub\_key“ values or a name from the "sub\_names" value (if present). For countries with latinised equivalents, the latin names ("sub\_lnames") are also permitted.

_Postal code looks sensible:_

The field "zip" in the JSON data is used in two different ways to validate the postal code.

> At the country level, the zip field indicates a Java compatible regular expression corresponding to all postal codes in the country. For example, for the US this is "` \\d{5}([ \\-]\\d{4})? `".

> At other levels, the regular expression indicates the postal code prefix expected for addresses in that region. For example, for data/US/CA the zip field value is "` 9[0-5]|96[01] `" – indicating that any addresses in California need a zip-code beginning with 90, 91, 92, 93, 94, 95, 960 or 961.

The "zipex" field in the data shows some examples of valid postal codes.

**Note:**

If a particular value is missing for a country, the data/ZZ value can be read instead. For example, data/AC has no value for required fields ("require"). This means that the data from data/ZZ should be used instead to answer this question.

**A practical example of validating a whole address:**

Let’s validate this address:

address\_line: "1 My Street"

city: "My City"

admin\_area: "XX"

postal\_code: "3344"

sorting\_code: "123"

country: "US"

  1. First of all, you would look at the data for data/US and see if there was a sub\_keys field.
    * Since there is, you would check to see if “XX” is a valid value there (or, alternatively, if data/US/XX exists).
    * If it doesn’t, and XX is also not a valid value in the sub\_names for the US, this indicates “XX” is not an accepted value for the admin\_area. (If there were no sub\_keys defined, then “XX” would be accepted.)
  1. If this were corrected, such that admin\_area was “CA”, then the next thing to consider would be to check data/US/CA and see if there are any sub\_keys there. Since there are none, “My City” is considered acceptable for the city value.
  1. Next, you could check that all required fields have data, based on the country-level JSON data. For the US, this is ACSZ - all of which have values, so this is fine.
  1. Next, you should check that no values have been filled in for fields that are not usually used. For the US, there should be no value for sorting\_code, since “%X” is not present in the “fmt” string.
  1. Lastly, the postal-code should be verified. Here, “3344” does not match the “zip” value for “data/US” since it is only 4 digits, not 5 digits followed by (optionally) a further 4.
    * If this was corrected such that an extra digit was appended, it would still be invalid, since the zip value for the most specific key that can be constructed from this address, data/US/CA, does not match zip-codes beginning with “3”.


**Programmatically using the data:**

An open-source library in Java, optimised for running on Android, has been written that uses this metadata to provide an input form with country-specific fields, that can be used to enter addresses. It is also able to validate addresses. For more information, visit the website:

https://github.com/googlei18n/libaddressinput/tree/master/android

## Acknowledgement
This document was originally written by lararennie at google.com
