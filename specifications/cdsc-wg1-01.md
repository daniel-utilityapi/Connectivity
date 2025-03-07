# CDSC-WG1-01 - Server Metadata

## Abstract <a id="abstract" href="#abstract" class="permalink">🔗</a>

This specification defines how utilities and other entities may provide structured "metadata", such as reference information about them, functionality they offer, and how organizations can interoperate with them.
The defined base metadata endpoint is intended to act as a starting point for organizations to discover details and capabilities of energy utilities and other similar entities.

## Status <a id="status" href="#status" class="permalink">🔗</a>

<b style="color:red">WARNING: This specification is a DRAFT and has not achieved consensus by the working group.</b>

## Copyright <a id="copyright" href="#copyright" class="permalink">🔗</a>

<span style="background-color:yellow">TODO</span>

## Table of Contents <a id="table-of-contents" href="#table-of-contents" class="permalink">🔗</a>

* [1. Introduction](#introduction)  
* [2. Terminology](#terminology)  
* [3. Metadata Endpoint](#metadata-endpoint)  
    * [3.1. Metadata Well-Known URI](#metadata-well-known-uri)  
    * [3.2. Metadata Object Format](#metadata-object-format)  
    * [3.3. Metadata Endpoint Organization](#metadata-endpoint-organization)  
    * [3.4. Server Capabilities](#server-capabilities)
* [4. Coverage Endpoint](#coverage-endpoint)  
    * [4.1. Coverage Listing Format](#coverage-listing-format)  
    * [4.2. Coverage Listing Filters](#coverage-listing-filters)  
    * [4.3. Coverage Entry Format](#coverage-entry-format)  
    * [4.4. Coverage Entry Types](#coverage-entry-types)
    * [4.5. Capability Roles](#capability-roles)
    * [4.6. Infrastructure Types](#infrastructure-types)
    * [4.7. Commodity Types](#commodity-types)
* [5. Server Migration](#server-migration)  
* [6. Extensions](#extensions)  
* [7. Examples](#examples)  
* [8. Security Considerations](#security)  
    * [8.1. Restricted Access](#restricted-access)  
    * [8.2. Rate Limiting](#rate-limiting)  
* [9. References](#references)  
* [10. Acknowledgments](#acknowledgments)  
* [11. Authors' Addresses](#authors-addresses)  

## 1. Introduction <a id="introduction" href="#introduction" class="permalink">🔗</a>

This specification was developed as part of the global effort to combat the climate crisis.
Specifically, in order to scalably measure carbon emissions of organizations and calculate the impact of deploying and operating clean energy technologies, companies need an efficient means to discover the details and capabilities of energy utilities and other similar entities.

There are thousands of utilities serving customers across the world, and each have their own way of organizing and structuring data.
This specification defines a way for these utilities and other entities to provide standardized, structured reference information about them, the functionality they offer, and how organizations can interoperate with them.
By serving "metadata" defined in this specification, utilities and other entities can automatically incorporate their details and capabilities into carbon tracking websites, apps, products, and other clean energy technologies.

This specification is intended to be a starting point for providing basic information about utilities and other similar entities, then extended by other specifications to define additional information and functionality.
For example, this specification may be extended to define how organizations can dynamically register with a utility in order to gain access to customer-authorized energy usage data.

## 2. Terminology <a id="terminology" href="#terminology" class="permalink">🔗</a>

<a id="server" href="#server" class="permalink">🔗</a> **"Server"** - The entity hosting the metadata endpoint.
A Server can be an energy utility or another type of entity that want to provide structured information and capabilities about themselves.
These entities can include, but are not limited to, distribution utilities, grid operators, electric retailers, community choice aggregators, government agencies, data warehouses, and private infrastructure providers.

<a id="client" href="#client" class="permalink">🔗</a> **"Client"** - The entity requesting Server's metadata endpoints.
A Client can be any organization or user seeking the Server's metadata on their details and capabilities.
These entities can include, but are not limited to, carbon tracking applications, electric vehicle companies, clean energy technology providers, commercial utility customers, grid management applications, and energy efficiency organizations.

<a id="referenced-technologies" href="#referenced-technologies" class="permalink">🔗</a> Referenced Technologies:
"[HTTPS](https://www.rfc-editor.org/rfc/rfc9110#name-https-uri-scheme)",
"[Request Methods](https://www.rfc-editor.org/rfc/rfc9110#name-methods)",
"[Status Codes](https://www.rfc-editor.org/rfc/rfc9110#name-status-codes)",
"[JSON](https://www.rfc-editor.org/rfc/rfc8259)",
"[Well-Known URI](https://www.rfc-editor.org/rfc/rfc5785)",
"[ISO8601](https://www.iso.org/iso-8601-date-and-time-format.html)",
"[OAuth](https://oauth.net/specs/)",
"[GeoJSON](https://datatracker.ietf.org/doc/html/rfc7946#section-3)",
<span style="background-color:yellow">TODO: add more as needed</span> are defined by their referenced standards documents.

<a id="string" href="#string" class="permalink">🔗</a> "string" - A series of unicode characters as defined in [RFC 8259](https://www.rfc-editor.org/rfc/rfc8259#section-7).

<a id="datetime" href="#datetime" class="permalink">🔗</a> "datetime" - A string representing date and time in the format of `date-time` as defined by [RFC 3339](https://datatracker.ietf.org/doc/html/rfc3339#section-5.6) (e.g. "2024-01-01T00:00:00Z").

<a id="url" href="#url" class="permalink">🔗</a> "URL" - A string representing resource as defined in [RFC 3986](https://www.rfc-editor.org/rfc/rfc3986.html#section-1.1.3) (e.g. "https://example.com/page1").

<a id="mime-type" href="#mime-type" class="permalink">🔗</a> "MIME type" - A string representing a document media type as defined in [RFC 6838](https://datatracker.ietf.org/doc/html/rfc6838) (e.g. "image/png").

<a id="country-code" href="#country-code" class="permalink">🔗</a> "country code" - A two-character string representing a country as defined in [ISO 3166](https://www.iso.org/iso-3166-country-codes.html) (e.g. "US").


<a id="key-words" href="#key-words" class="permalink">🔗</a> Key Words: "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" are defined in accordance with [BCP 14](https://www.rfc-editor.org/info/bcp14).

## 3. Metadata Endpoint <a id="metadata-endpoint" href="#metadata-endpoint" class="permalink">🔗</a>

Servers supporting this specification SHALL offer one or more HTTPS URLs from which a Client can perform a GET request to retrieve Server metadata, and the Server will respond with the Server's metadata JSON object.

For successful responses, Servers MUST respond with a "200 OK" Status Code and body containing a JSON object that is formatted as defined in [Metadata Object Format](#metadata-object-format).
Cache-optimized responses, such as "304 Not Modified", to Client requests with cached value headers, such as `If-None-Match`, is also acceptable.

For invalid requests and error responses, Servers MUST respond with the appropriate Response Code for the type of issue or error (e.g. "503 Service Unavailable" if the Server is temporarily down for maintenance).

### 3.1. Metadata Well-Known URI <a id="metadata-well-known-uri" href="#metadata-well-known-uri" class="permalink">🔗</a>

If the Server wishes to provide metadata publicly, the Server MUST support a metadata endpoint at the Well-Known URI `"/.well-known/carbon-data-spec.json"` on their hostname (e.g. https://example.com/.well-known/carbon-data-spec.json).

Servers MAY additionally support other public and restricted URLs for special case or private metadata endpoints, as needed.

### 3.2. Metadata Object Format <a id="metadata-object-format" href="#metadata-object-format" class="permalink">🔗</a>

Metadata objects are formatted as JSON objects and contain named values, some of which are required and some of which are optional.
The following values are included in the default list available in metadata objects.

* `cds_metadata_version` - _[string](#string)_ - (REQUIRED) The version of metadata object format this Server follows, which for this version of the specification is `v1`.
* `cds_metadata_url` - _[string](#string)_ - (REQUIRED) A URL link to this metadata object's canonical network location.
* `created` - _[datetime](#datetime)_ - (REQUIRED) When the metadata url was first made available.
* `updated` - _[datetime](#datetime)_ - (REQUIRED) When this metadata object was last modified.
* `name` - _[string](#string)_ - (REQUIRED) The name of the utility or entity (e.g. "Demo Gas & Electric").
* `description` - _[string](#string)_ - (REQUIRED) A brief description about the utility or entity (e.g. "An electric and gas utility serving customers in Northern California").
* `website` - _[URL](#url)_ - (REQUIRED) Where users can learn more about the utility or entity and its capabilities.
* `documentation` - _[URL](#url)_ - (REQUIRED) Where Clients can find the utility or entity's technical reference documentation.
* `support` - _[URL](#url)_ - (REQUIRED) Where Clients can find contact information for technical support on the Server's capabilities.
* `capabilities` - _Array[[ServerCapability](#server-capabilities)]_ - (REQUIRED) What capabilities and functionality this Server supports, which may be an empty array if the Server is not offering any additional capabilities beyond just this metadata object. If the list of capabilities includes `coverage`, this array MUST also include a union of the capabilities listed in the [Coverage Endpoint](#coverage-endpoint) list of [Coverage Entry](#coverage-entry-format) objects' `capabilities` values.
* `coverage` - _[URL](#url)_ - (OPTIONAL) Where to find the [Coverage Endpoint](#coverage-endpoint). This field is REQUIRED if `coverage` is listed in the `capabilities` array.
* `related_metadata` - _Array[[URL](#url)]_ - (OPTIONAL) Where to find other related [Metadata Endpoints](#metadata-endpoint).

### 3.3. Metadata Endpoint Organization <a id="metadata-endpoint-organization" href="#metadata-endpoint-organization" class="permalink">🔗</a>

Many utilities and other similar entities are structured as multiple divisions, operating companies, territories, legal constructs, or other types of groupings.
These organizational complexities and legal structures can make it difficult to publish a single representative metadata endpoint for an entire entity.

When utilities and other similar entities wish to publicly provide multiple metadata objects that more cleanly define the metadata and capabilities for their entity, their Server MUST publish a single metadata endpoint at the [well-known URI](metadata-well-known-uri) and within that metadata object include an array of `related_metadata` URLs that link the Client to the various subdivided or related metadata endpoints.

If capabilities are available by a parent organization across a group of subsidiary utilities or entities (e.g. a holding company offers universal customer data access across many of their utility operating company territories), the parent organization MAY publish a single metatdata object with coverage entries that list the individual subsidiary utility or entity coverage entries.

It is acceptable for the initial well-known metadata endpoint object to include a minimal or empty array of `capabilities`, as it represents the entire entity, and leave enumerating the `capabilities` to the related metadata endpoints.

### 3.4. Server Capabilities <a id="server-capabilities" href="#server-capabilities" class="permalink">🔗</a>

Server capabilities are what specific interoperability or informational capabilities a Server is providing.
The following list of strings are an enumerated set of capabilities that are defined by default in this specification.

* `coverage` - The Server is providing coverage details related to itself as it relates the other listed capabilities.
If included, a `coverage` value is REQUIRED in the metadata object.

Other specifications that define interoperability or informational capabilities for utilities and other entities are encouraged to define an extension that adds their capability as a string to the above list.
See the [Extensions](#Extensions) section for details on how to extend this list of Server capabilities.

## 4. Coverage Endpoint <a id="coverage-endpoint" href="#coverage-endpoint" class="permalink">🔗</a>

If Servers include the `coverage` [Server capability](#server-capabilities), a URL to the utility or entity's coverage MUST be included in the metadata object.

When the URL is requested using the HTTP `GET` method and the request is valid, the Server MUST respond with a "200 OK" Status Code and body containing JSON object formatted according to the [Coverage Listing Format](#coverage-listing-format).
Cache-optimized responses, such as "304 Not Modified", to Client requests with cached value headers, such as `If-None-Match`, is also acceptable.

For invalid requests and error responses, Servers MUST respond with the appropriate Response Code for the type of issue or error (e.g. "503 Service Unavailable" if the Server is temporarily down for maintenance).

### 4.1. Coverage Listing Format <a id="coverage-listing-format" href="#coverage-listing-format" class="permalink">🔗</a>

Coverage objects are formatted as JSON objects and contain the following named values. This object serves mostly as an object wrapper around a list of [coverage entries](#coverage-entry-format), so that the list of coverage entries can be paginated by the Server if too long for one response.

* `coverage_entries` - _Array[[CoverageEntry](#coverage-entry-format)]_ - (REQUIRED) A list of segments this Server covers
* `next` - _`null` or [URL](#url)_ - (REQUIRED) Link to the next page of coverage entries (`null` if no more subsequent pages)
* `previous` - _`null` or [URL](#url)_ - (REQUIRED) Link to the previous page of coverage entries (`null` if no more prior pages)

Listings of Coverage Entry objects MUST be ordered in reverse chronological order by `updated` datetime, where the most recently updated relevant Coverage Entry MUST be first in each listing.

### 4.2. Coverage Listing Filters <a id="coverage-listing-filters" href="#coverage-listing-filters" class="permalink">🔗</a>

Servers MUST support Clients adding any of the following URL parameters to the [Coverage Endpoint](#coverage-endpoint) GET request, which will filter the list of Coverage Entry objects to be the intersection of results for each of the URL parameters filters:

* `ids` - A space-separated list of `id` values for which the Servers MUST filter the Coverage Entries.

### 4.3. Coverage Entry Format <a id="coverage-entry-format" href="#coverage-entry-format" class="permalink">🔗</a>

Coverage entries are formatted as JSON objects and contain named values, some of which are required and some of which are optional.
The following values are included in the default list available in coverage entries.

* `id` - _[string](#string)_ - (REQUIRED) A unique identifier for the coverage entry.
* `created` - _[datetime](#datetime)_ - (REQUIRED) When this coverage entry was first made available.
* `updated` - _[datetime](#datetime)_ - (REQUIRED) When this coverage entry was last modified.
* `entity_name` - _[string](#string)_ - (REQUIRED) A human readable name for the entity being covered (e.g. "Demo Gas & Electric").
  The entity name in a coverage entry MAY be different than the Server metadata `name` value in situations where the Server is operated by a parent or other entity.
* `entity_abbreviation` - _`null` or [string](#string)_ - (REQUIRED) The abbreviation for the entity being covered by the coverage entry (e.g. "DG&E").
  This field is to assist Clients in parsing coverage for entities that are commonly known by their abbreviated names (e.g. "DG&E" rather than "Demo Gas & Electric").
  If no abbreviation is available for the entity being covered, this value is `null`.
* `country` - _[country code](#country-code)_ - (REQUIRED) The country that has jurisdiction over the Coverage Entity.
  If multiple countries have jurisdiction over a piece of coverage, Servers MUST have multiple Coverage Entries, one for each country having jurisdiction.
* `name` - _[string](#string)_ - (REQUIRED) A human readable name for the coverage area or category (e.g. "Upstate New York electric territory").
* `description` - _[string](#string)_ - (OPTIONAL) A brief description the coverage area or category (e.g. "Example Electric's coverage for the northern half of New York state").
* `type` - _[CoverageType](#coverage-entry-types)_ - (REQUIRED) The type of coverage entry.
* `role` - _[CapabilityRole](#capability-roles)_ - (REQUIRED) The role of the Server in offering capabilities for the coverage entry.
* `infrastructure_types` - _Array[[InfrastructureType](#infrastructure-types)]_ - (REQUIRED) A list of infrastructure categories that are applicable to the coverage entry, which may be an empty array if the entry is not any of the possible infrastructure types.
* `commodity_types` - _Array[[CommodityType](#commodity-types)]_ - (REQUIRED) A list of services or commodities that are applicable to the coverage entry, which may be an empty array if the entry does not have an applicable commodity.
* `capabilities` - _Array[[ServerCapability](#server-capabilities)]_ - (REQUIRED) What capabilities and functionality this Server supports for the coverage entry, which may be an empty array if the Server is not offering any capabilities for the coverage entry and is just publishing the coverage entry itself for informational purposes. This list MUST NOT include the `coverage` capability.
* `map_resource` - _[URL](#url)_ - (OPTIONAL) Link to a map of the coverage territory.
* `map_content_type` - _[MIME type](#mime-type)_ - (OPTIONAL) Content-Type format of the `map_resource`.
* `geojson_resource` - _[URL](#url)_ - (OPTIONAL) Link to a GeoJSON object mapping the coverage territory.

When `type` is `geographic`, one or both of `map_resource` and `geojson_resource` MUST be provided.
If `map_resource` is provided, then `map_content_type` MUST also be provided.

### 4.4. Coverage Entry Types <a id="coverage-entry-types" href="#coverage-entry-types" class="permalink">🔗</a>

Coverage entry types describe what group is being detailed in the coverage entry.
Coverage entries MAY represent physical geographic territories (e.g. where a utility provides electric service) or logical groupings (e.g. a certain segment of customers).
Many utilities have different geographic or logical coverage for their different territories and capabilities (e.g. their electric service territory is different from their natural gas service territory), so Servers MAY include multiple coverage entries for different types of service without having to publish multiple metadata endpoints.

The following list of strings are an enumerated set of coverage entry types that are valid `type` values in the coverage entry object.

* `geographic` - The coverage entry represents a geographic area (e.g. electric service territory).
* `logical` - The coverage entry represents a non-geographic logical grouping (e.g. a set of commerical customers).

### 4.5. Capability Roles <a id="capability-roles" href="#capability-roles" class="permalink">🔗</a>

Capability Roles disclose the Server's role in the provision of capabilities covered by the coverage entry.
The following list of strings are an enumerated set of role types that MAY be set as the `role` value in the coverage entry object.

* `authoritative` - The Server represents the authoritative entity for the capabilities covered by the coverage entry (e.g. the utility itselfis providing the capability).
  Only one Server SHOULD mark itself as `authoritative` for the same coverage and capability, to prevent any abiguity for Clients as to a "source of truth".
* `official` - The Server represents a contractually obligated or government mandated entity that has an official connection to the `authoritative` entity and is offering funcational capabilities for the `direct` entity in an official capacity (e.g. a state-wide data hub offering customer data access).
  Multiple Servers could mark themselves as `official` for the same coverage and capability if the `authoritative` entity has multiple implementations of capabilities with officially designated service providers or data hubs.
* `aggregator` - The Server represents an external entity that offers capabilities by proxy and does not have an official agreement with the `direct` entity (e.g. a service provider that aggregates many utility territory capabilities).

### 4.6. Infrastructure Types <a id="infrastructure-types" href="#infrastructure-types" class="permalink">🔗</a>

Infrastructure types are general categories of utility and energy grid entities that are likely to need to provide access to metadata and capabilities under this specification.
The following list of strings are an enumerated set of infrastructure types that MAY be included in the `infrastructure_types` array of values in the coverage entry object.

* `distribution_utility` - A electric, natural gas, and/or water utility that distributes the utility service to the end customer (e.g. "who maintains the wires to your house").
* `metering_provider` - An entity that maintains the utility metering infrastructure and/or reads end customer meters (e.g. "who reads your meter").
* `supplier` - An energy supplier for the end customer (e.g. "who you buy power from").
* `generator` - An entity that operates energy generation assets (e.g. a power plant owner and/or operator).
* `market_operator` - An entity that operates an energy or grid market (e.g. a wholesale energy trading system).
* `distribution_service_operator` - Commonly called a "DSO", this is a control entity manages one or more distribution grids.
* `transmission_service_operator` - Commonly called a "TSO" or Interstate Service Operator ("ISO"), this is a control entity manages one or more distribution grids.
* `service_provider` - An entity that provides customer, utilty, grid, or other services (e.g. a virtual power plant operator).

### 4.7. Commodity Types <a id="commodity-types" href="#commodity-types" class="permalink">🔗</a>

Commodity types are what general categories of services a utility or entity provides to end customers.
The following list of strings are an enumerated set of commodity types that MAY be included in the `commodity_types` array of values in the coverage entry object.

* `electricity` - Has metadata or capabilities around utility electric services
* `natural_gas` - Has metadata or capabilities around utility natural gas services
* `fuel_oil` - Has metadata or capabilities around utility fuel oil services
* `water` - Has metadata or capabilities around utility water services
* `wastewater` - Has metadata or capabilities around utility wastewater services
* `trash` - Has metadata or capabilities around utility trash services
* `distributed_energy` - Has metadata or capabilities around distributed energy resource (DER) services

## 5. Server Migration  <a id="server-migration" href="#server-migration" class="permalink">🔗</a>

Utilities and other entities frequently change their organizational structures, such as buying another utility and merging the utility territory together.
As utilities and other entities evolve, their Server metadata endpoints, including the domains from which they are served, may no longer be valid.

This section describes how Servers MUST transition their [well-known metadata endpoint](#metadata-well-known-uri) when migrating or merging into another Server.

For organizational changes where the utility or other entity is not changing their domain, the Server MUST update their existing well-known metadata endpoint in place to match the new organizational structure.

For organizational changes where the new entity structure will not match the existing domain, the Server MUST create a new well-known metadata endpoint on their new domain and update the metadata contents to match the new organization.
The Server MUST also respond to requests to the previous well-known metadata endpoint with a "301 Moved Permanently" or "302 Found" Status Code with a `Location` header containing the URL of the new updated metadata endpoint so that Clients may be automatically redirected to the overriding.
The Server MUST maintain the redirect response for as long as the Server retains control over the domain or up to 3 years, whichever is less.

## 6. Extensions <a id="extensions" href="#extensions" class="permalink">🔗</a>

Other specifications MAY extend this specification to allow for seamless expansion in Server metadata, capabilities, and coverage.

[Metadata Object](#metadata-object-format), [Coverage Listing](#coverage-listing-format), and [Coverage Entry](#coverage-listing-format) MAY be extended by other specifications to allow for additional values to be possible in their object formats.
When extending the object format, other specifications MUST reference the relevant section in this specification and denote that they are extending the object to add a new named value.
The additional named value MUST be specified with a general description, the value's format, and whether the value is REQUIRED or OPTIONAL.

The enumerated lists for valid [Server Capabilities](#server-capabilities), [Coverage Listing Filters](#coverage-listing-filters), [Coverage Entry Types](#coverage-entry-types), [Capability Roles](#capability-roles), [Infrastructure Types](#infrastructure-types), [Commodity Types](#commodity-types), and  MAY be extended by other specifications to allow for additional strings to be valid.
When extending enumerated list, other specifications MUST reference the relevant section in this specification and denote that they are extending the list to add a new string.
The additional string MUST be specified with a description of what that string means when it is included in the relevant array.

To facilitate forwards compatibility, Clients MUST ignore unknown or undocumented object values and enumerated strings.
If a Client cannot provide adequate functionality based on too many unknown or undocumented object values or enumerated strings, the Client SHOULD refer to the Server's technical documentation (the `documentation` value in the metadata object) or contact the Server's technical support (via the `support` value in the metadata object).

## 7. Examples <a id="examples" href="#examples" class="permalink">🔗</a>

The following is a non-normative example of a Client request and Server response to a metadata endpoint.

```
==Request==
GET /.well-known/carbon-data-spec.json HTTP/1.1
Host: example.com

==Response==
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8

{
    "cds_metadata_version": "v1",
    "cds_metadata_url": "https://example.com/.well-known/carbon-data-spec.json",

    "created": "2022-01-01T00:00:00Z",
    "updated": "2022-06-01T00:00:00Z",

    "name": "Example Data Hub",
    "description": "A fictional regional data hub that offers information about the region's utilities.",
    "website": "https://example.com/data-access",
    "documentation": "https://example.com/data-access/docs",
    "support": "https://example.com/developers/contact",
    "capabilities": [
        "coverage",
    ],

    "coverage": "https://static.example.com/cds-coverage.json"
}
```

The following is a non-normative example of a Client request and Server response to a coverage endpoint.

```
==Request==
GET /cds-coverage.json HTTP/1.1
Host: static.example.com

==Response==
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8

{
    "coverage_entries": [
        {
            "id": "dge_elec_west",

            "created": "2022-06-01T00:00:00.000000Z",
            "updated": "2022-06-01T00:00:00.000000Z",

            "entity_name": "Demo Gas & Electric",
            "entity_abbreviation": "DG&E",

            "name": "Western electric service territory",
            "description": "This is the electric service coverage for the western half of the region.",

            "type": "geographic",
            "role": "official",
            "infrastructure_types": [
                "distribution_utility",
            ],
            "commodity_types": [
                "electricity",
            ],
            "capabilities": [],

            "map_resource": "https://static.example.com/cds-coverage-map.png",
            "map_content_type": "image/png",
            "geojson_resource": "https://static.example.com/cds-coverage-map-geo.json"
        }
    ],
    "next": null,
    "previous": null
}
```

## 8. Security Considerations <a id="security" href="#security" class="permalink">🔗</a>

This specification involves a utility or other entity publishing details about itself.
Server operators MUST review all material published in the metadata and coverage endpoints and make sure it is information that is allowed to be made available to Clients.
This is especially true for [well-known metadata endpoints](#metadata-well-known-uri), as not only are these endpoints public, they are intentionally published with a URL structure that allows them to auto-discovered by Clients.

### 8.1. Restricted Access <a id="restricted-access" href="#restricted-access" class="permalink">🔗</a>

Servers that wish to restrict access to a metadata or coverage endpoint to certain Clients MUST configure well established authentication processes for Clients to ensure that only the approved Clients may access the restricted endpoint.

This specification does not describe specifically how Servers will authenticate Clients, as these restricted access protocols are context dependent.
For example, if a Server is providing the metadata endpoint as part of an existing logged in portal, then they can use that logged in portal's session cookie to authenticate Client requests to the metadata endpoint.

[Well-known metadata endpoints](#metadata-well-known-uri) MUST NOT require Client authentication to access, since these metadata endpoints are intended to be published publicly.

Resources provided as URLs in metadata and coverage objects MUST be accessible under at least the same level of authentication requirements as accessing the metadata or coverage objects themselves.
For example, if a coverage entry object requires cookie-based authentication to access, then and linked the GeoJSON object must also be accessible using the same cookie-based authentication.
Servers MAY also choose to also make those resources available publicly, where any included authentication in the request is ignored.
If the metadata or coverage object is made available publicly, then the linked resource must also be accessible publicly.

### 8.2. Rate Limiting <a id="rate-limiting" href="#rate-limiting" class="permalink">🔗</a>

For [well-known metadata endpoints](#metadata-well-known-uri) and other publicly accessible metadata and coverage endpoints, Serves SHOULD configure rate limiting restrictions so that bots and misconfigured scripts will not flood and overwhelm the endpoints with requests, while still allowing legitimate and low-volume requests have access to the endpoints.

## 9. References <a id="references" href="#references" class="permalink">🔗</a>

<span style="background-color:yellow">TODO</span>

## 10. Acknowledgments <a id="acknowledgments" href="#acknowledgments" class="permalink">🔗</a>

The authors would like to thank the late Shuli Goodman, who was the Executive Director of LFEnergy, for her incredible leadership in initially organizing the Carbon Data Specification.

## 11. Authors' Addresses <a id="authors-addresses" href="#authors-addresses" class="permalink">🔗</a>

Daniel Roesler  
UtilityAPI  
Email: [daniel@utilityapi.com](mailto:daniel@utilityapi.com)  
URI: [https://utilityapi.com/](https://utilityapi.com/)


