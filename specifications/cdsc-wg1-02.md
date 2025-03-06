# CDSC-WG1-02 - Client Registration

## Abstract <a id="abstract" href="#abstract" class="permalink">🔗</a>

This specification defines how utilities and other central entities ("Servers") may allow third parties ("Clients") to register, configure, and maintain connectivity with the utility or other central entity. This specification mostly extends various OAuth specifications ([Dynamic Client Registration](https://www.rfc-editor.org/rfc/rfc7591), [Authorization Server Metadata](https://www.rfc-editor.org/rfc/rfc8414), and more) to create a robust framework for offering secure, streamlined, and standardized connectivity between utilities and third parties.

## Status <a id="status" href="#status" class="permalink">🔗</a>

<b style="color:red">WARNING: This specification is a DRAFT and has not achieved consensus by the working group.</b>

## Copyright <a id="copyright" href="#copyright" class="permalink">🔗</a>

<span style="background-color:yellow">TODO</span>

## Table of Contents <a id="table-of-contents" href="#table-of-contents" class="permalink">🔗</a>

* [1. Introduction](#introduction)  
* [2. Terminology](#terminology)  
* [3. Authorization Server Metadata](#auth-server-metadata)  
    * [3.1. Metadata URL](#auth-server-metadata-url)  
    * [3.2. Metadata Object Format](#auth-server-metadata-format)  
    * [3.3. Scopes Supported](#scopes)  
        * [3.3.1. Client Admin Scope](#scopes-client-admin)  
        * [3.3.2. Grant Admin Scope](#scopes-grant-admin)  
        * [3.3.3. Server-Provided Files Scope](#scopes-server-provided-files)  
    * [3.4. Scope Descriptions Object Format](#scope-descriptions-format)  
    * [3.5. Registration Field Object Format](#registration-field-format)  
    * [3.6. Registration Field Types](#registration-field-types)  
    * [3.7. Registration Field Formats](#registration-field-formats)  
    * [3.8. Authorization Details Field Object Format](#auth-details-fields-format)  
    * [3.9. Authorization Details Field Formats](#auth-details-field-formats)  
    * [3.10. Authorization Details Field Choice Object Format](#auth-details-field-choice-format)  
* [4. Client Registration Process](#client-registration-process)  
    * [4.1. Client Registration Request](#registration-request)  
    * [4.2. Client Registration Response](#registration-response)  
* [5. Clients API](#clients-api)  
    * [5.1. Client Object Format](#client-format)  
    * [5.2. Client Statuses](#client-statuses)  
    * [5.3. Listing Clients](#clients-list)  
    * [5.4. Retrieving Individual Clients](#clients-get)  
    * [5.5. Modifying Clients](#clients-modify)  
* [6. Messages API](#messages-api)  
    * [6.1. Message Object Format](#message-format)  
    * [6.2. Message Types](#message-types)  
    * [6.3. Message Statuses](#message-statuses)  
    * [6.4. Client Update Request Object Format](#client-update-request-format)  
    * [6.5. Listing Messages](#messages-list)  
    * [6.6. Creating Messages](#messages-create)  
    * [6.7. Modifying Messages](#messages-modify)  
* [7. Credentials API](#credentials-api)  
    * [7.1. Credentials Object Format](#credentials-format)  
    * [7.2. Credentials Types](#credentials-types)  
    * [7.3. Listing Credentials](#credentials-list)  
    * [7.4. Retrieving Individual Credentials](#credentials-get)  
    * [7.5. Creating Credentials](#credentials-create)  
    * [7.6. Modifying Credentials](#credentials-modify)  
* [8. Grants API](#grants-api)  
    * [8.1. Grant Object Format](#grant-format)  
    * [8.2. Grant Statuses](#grant-statuses)  
    * [8.3. Listing Grants](#grants-list)  
    * [8.4. Retrieving Individual Grants](#grants-get)  
    * [8.5. Modifying Grants](#grants-modify) 
* [9. Server-Provided Files API](#server-provided-files-api)  
    * [9.1. Server-Provided Files Object Format](#server-provided-files-format)  
    * [9.2. Listing Server-Provided Files](#server-provided-files-list)  
* [10. Extensions](#extensions)  
* [11. Examples](#examples)  
* [12. Security Considerations](#security)  
    * [11.1. Restricted Access](#restricted-access)  
    * [11.2. Rate Limiting](#rate-limiting)  
* [13. References](#references)  
* [14. Acknowledgments](#acknowledgments)  
* [15. Authors' Addresses](#authors-addresses)  

## 1. Introduction <a id="introduction" href="#introduction" class="permalink">🔗</a>

This specification was developed as part of the global effort to combat the climate crisis.
Specifically, in order to scalably measure carbon emissions of organizations and calculate the impact of deploying and operating clean energy technologies, companies need an efficient means to register and interact with energy utilities and other similar entities.

There are thousands of utilities serving customers across the world, and each have their own way of providing access to privileged functionality and data.
This specification defines a way for these utilities and other entities ("Servers") to provide a standardized, structured process for external users and companies ("Clients") to register and obtain secure access to privileged functionality and data.
By offering a standardized Client registration and management protocol, utilities and other entities can enable secure interoperability with other external systems providing climate-change fighting services such as carbon tracking, decarbonization measures, grid flexibility capabilities, and energy benchmarking.

This specification does NOT describe what specific privileged functionality or data is offered to Clients as part of their registration.
Instead, relevant stakeholders are encouraged to write [Extensions](#extensions) to this specification that enable various use cases that need access to privileged functionality or data.
For example, this specification may be extended to define a method for Clients to be able to access customer-authorized energy usage data.

## 2. Terminology <a id="terminology" href="#terminology" class="permalink">🔗</a>

<a id="server" href="#server" class="permalink">🔗</a> **"Server"** - The entity hosting the specified endpoints.
A Server can be an energy utility or another type of entity that want to provide access to privileged functionality or data.
These entities can include, but are not limited to, distribution utilities, grid operators, electric retailers, community choice aggregators, government agencies, data warehouses, and private infrastructure providers.

<a id="client" href="#client" class="permalink">🔗</a> **"Client"** - The entity requesting Server's metadata endpoints.
A Client can be any organization or user seeking to register with a Server for a specific scope of access.
These entities can include, but are not limited to, carbon tracking applications, electric vehicle companies, clean energy technology providers, commercial utility customers, grid management applications, and energy efficiency organizations.

<a id="referenced-technologies" href="#referenced-technologies" class="permalink">🔗</a> Referenced Technologies:
"[HTTPS](https://www.rfc-editor.org/rfc/rfc9110#name-https-uri-scheme)",
"[Request Methods](https://www.rfc-editor.org/rfc/rfc9110#name-methods)",
"[Status Codes](https://www.rfc-editor.org/rfc/rfc9110#name-status-codes)",
"[JSON](https://www.rfc-editor.org/rfc/rfc8259)",
"[OAuth](https://oauth.net/specs/)",
<span style="background-color:yellow">TODO: add more as needed</span> are defined by their referenced standards documents.

<a id="string" href="#string" class="permalink">🔗</a> "string" - A series of unicode characters as defined in [RFC 8259](https://www.rfc-editor.org/rfc/rfc8259#section-7).

<a id="integer" href="#integer" class="permalink">🔗</a> "integer" - A positive integer value as defined by `int` in [RFC 8259](https://www.rfc-editor.org/rfc/rfc8259#section-6).

<a id="decimal" href="#decimal" class="permalink">🔗</a> "decimal" - A decimal value as defined by `number` in [RFC 8259](https://www.rfc-editor.org/rfc/rfc8259#section-6).

<a id="null" href="#null" class="permalink">🔗</a> "null" - The `null` value defined in [RFC 8259](https://www.rfc-editor.org/rfc/rfc8259#section-3).

<a id="boolean" href="#boolean" class="permalink">🔗</a> "boolean" - Either `true` or `false` defined in [RFC 8259](https://www.rfc-editor.org/rfc/rfc8259#section-3).

<a id="array" href="#array" class="permalink">🔗</a> "Array" - A list of objects or values as defined by `Arrays` in [RFC 8259](https://www.rfc-editor.org/rfc/rfc8259#section-5).

<a id="map" href="#map" class="permalink">🔗</a> "Map" - A JSON object as defined by `Objects` in [RFC 8259](https://www.rfc-editor.org/rfc/rfc8259#section-4).

<a id="date" href="#date" class="permalink">🔗</a> "date" - A string representing date in the format of `full-date` as defined by [RFC 3339](https://datatracker.ietf.org/doc/html/rfc3339#section-5.6) (e.g. "2024-01-01").

<a id="datetime" href="#datetime" class="permalink">🔗</a> "datetime" - A string representing date and time in the format of `date-time` as defined by [RFC 3339](https://datatracker.ietf.org/doc/html/rfc3339#section-5.6) (e.g. "2024-01-01T00:00:00Z").

<a id="url" href="#url" class="permalink">🔗</a> "URL" - A string representing resource as defined in [RFC 3986](https://www.rfc-editor.org/rfc/rfc3986.html#section-1.1.3) (e.g. "https://example.com/page1").

<a id="jwk-enc" href="#jwk-enc" class="permalink">🔗</a> "JWK Public Encryption Key" - A JSON object representing a public key as defined in [RFC 7517](https://www.rfc-editor.org/rfc/rfc7517.html#section-4) where the [use](https://www.rfc-editor.org/rfc/rfc7517.html#section-4.2) field MUST contain the value `enc`.

<a id="key-words" href="#key-words" class="permalink">🔗</a> Key Words: "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" are defined in accordance with [BCP 14](https://www.rfc-editor.org/info/bcp14).

## 3. Authorization Server Metadata <a id="auth-server-metadata" href="#auth-server-metadata" class="permalink">🔗</a>

To enable automated Client registration, Servers MUST host an endpoint with structured metadata about how to submit a [registration request](#registration-request), as well as functionality and data capabilities offered by the Server.
The requirements for hosting this metadata endpoint follows [OAuth 2.0 Authorization Server Metadata](https://www.rfc-editor.org/rfc/rfc8414) with several additions for Carbon Data Specification (CDS) functionality described in the following sections.

### 3.1. Metadata URL <a id="auth-server-metadata-url" href="#auth-server-metadata-url" class="permalink">🔗</a>

Following OAuth's [Obtaining Authorization Server Metadata](https://www.rfc-editor.org/rfc/rfc8414#section-3) process, a Server's Authorization Server Metadata Object is retrieved by making an HTTPS request to a specified URL.

To ensure discoverability to the specified URL, this specification extends [CDSC-WG1-01 Server Capabilities](/specs/cdsc-wg1-01/#server-capabilities) to require the following values:

* `oauth` - _[string](#string)_ - (REQUIRED) Including Client Registration via OAuth in the list of a Server's available capabilities

As well as extending the [CDSC-WG1-01 Metadata Object Format](/specs/cdsc-wg1-01/#metadata-object-format) to require the following values:

* `oauth_metadata` - _[URL](#url)_ - (REQUIRED) Where Clients can obtain the Server's Authorization Server Metadata Object

In addition to requiring that the URL be included in the [CDSC-WG1-01 Metadata Object Format](/specs/cdsc-wg1-01/#metadata-object-format), Servers MAY also configure the URL such that it conforms to the default [Well-Known URI](#https://www.rfc-editor.org/rfc/rfc8414#section-7.3) format, which can increase interoperability beyond the above specified CDSC-WG1-01 extension.

### 3.2. Metadata Object Format <a id="auth-server-metadata-format" href="#auth-server-metadata-format" class="permalink">🔗</a>

A Server's Authorization Server Metadata Object follows OAuth's [Authorization Server Metadata](https://www.rfc-editor.org/rfc/rfc8414#section-2) object format with the following modifications from OPTIONAL or RECOMMENDED to REQUIRED:

* `registration_endpoint` - _[URL](#url)_ - (REQUIRED) OAuth's [Dynamic Client Registration](https://www.rfc-editor.org/rfc/rfc7591) functionality is required to enable the [Client Registration Process](#client-registration-process).
* `scopes_supported` - _Array[[string](#string)]_ - (REQUIRED) The scopes in this array MUST represent a union of all scope `id` values contained in the `cds_scope_descriptions` objects.
* `service_documentation` - _[URL](#url)_ - (REQUIRED) Developer documentation is required by the Server to help streamline Client integration.
* `op_policy_uri` - _[URL](#url)_ - (REQUIRED) Policies for Clients registering is required.
* `op_tos_uri` - _[URL](#url)_ - (REQUIRED) Terms of use for Clients registering is required.
* `revocation_endpoint` - _[URL](#url)_ - (REQUIRED) OAuth's [Token Revocation](https://www.rfc-editor.org/rfc/rfc7009) functionality is required.
* `introspection_endpoint` - _[URL](#url)_ - (REQUIRED) OAuth's [Token Introspection](https://www.rfc-editor.org/rfc/rfc7662) functionality is required.
* `code_challenge_methods_supported` - _Array[[string](#string)]_ - (REQUIRED) The code challenge methods in this array MUST represent a union of all `code_challenge_methods_supported` values contained in the `cds_scope_descriptions` objects.

In addition to the standard set of OAuth [Authorization Server Metadata](https://www.rfc-editor.org/rfc/rfc8414#section-2) values, this specification also requires the following OAuth extension capabilities to be included in the metadata object values:

* `authorization_details_types_supported` - _Array[[string](#string)]_ - (REQUIRED) OAuth's [Rich Authorization Requests](https://www.rfc-editor.org/rfc/rfc9396) functionality is required.
  This array of values MUST be the same as `scopes_supported`.
* `pushed_authorization_request_endpoint` - _[URL](#url)_ - (REQUIRED) OAuth's [Pushed Authorization Requests](https://www.rfc-editor.org/rfc/rfc9126) functionality is required.

In addition to the above additionally required set of OAuth [Authorization Server Metadata](https://www.rfc-editor.org/rfc/rfc8414#section-2) values, this specification clarifies use of the following OAuth standard values:

* `response_types_supported` - _Array[[string](#string)]_ - (REQUIRED) The response types in this array MUST represent a union of all `response_types_supported` values contained in the `cds_scope_descriptions` objects.
* `grant_types_supported` - _Array[[string](#string)]_ - (REQUIRED) The response types in this array MUST represent a union of all `grant_types_supported` values contained in the `cds_scope_descriptions` objects.
* `token_endpoint_auth_methods_supported` - _Array[[string](#string)]_ - (REQUIRED) The response types in this array MUST represent a union of all `token_endpoint_auth_methods_supported` values contained in the `cds_scope_descriptions` object.

In addition to OAuth capabilities included in the metadata object, this specification adds the following Carbon Data Specification (CDS) values:

* `cds_oauth_version` - _[string](#string)_ - (REQUIRED) The version of the CDS-WG1-02 Client Registration specification that the Server has implemented, which for this version of the specification is `v1`
* `cds_human_registration` - _[URL](#url)_ - (REQUIRED) Where Clients who do not have the technical capacity to use the `registration_endpoint` can visit to manually register a Client using a user device
* `cds_clients_api` - _[URL](#url)_ - (REQUIRED) The URL for the [Clients API](#clients-api).
* `cds_messages_api` - _[URL](#url)_ - (REQUIRED) The URL for the [Messages API](#messages-api).
* `cds_credentials_api` - _[URL](#url)_ - (REQUIRED) The URL for the [Credentials API](#credentials-api).
* `cds_grants_api` - _[URL](#url)_ - (REQUIRED) The URL for the [Grants API](#grants-api).
* `cds_server_provided_files_api` - _[URL](#url)_ - (OPTIONAL) The URL for the [Server-Provided Files API](#server-provided-files-api).
  This MUST be included if `scopes_supported` includes the `server_provided_files` scope.
* `cds_test_accounts` - _[URL](#url)_ - (OPTIONAL) Where Clients can find developer documentation on what test account credentials may be used for testing functionality.
  This MUST be included if `response_types_supported` is not an empty list (i.e. user authorization is supported for some scopes).
* `cds_scope_descriptions` - _Map[[ScopeDescription](#scope-descriptions-format)]_ - (REQUIRED) An object providing additional information about what Scope values listed in `scopes_supported` mean.
  This object MUST include a key for each Scope listed in `scopes_supported` with a [Scope Description](#scope-descriptions-format) as that key's value.
* `cds_registration_fields` - _Map[[RegistrationField](#registration-field-format)]_ - (REQUIRED) An object providing additional information about Registration Field that are referenced in [Scope Description's](#scope-descriptions-format) `registration_requirements` list.
  This object MUST include a key for each Registration Field `id` included in the `registration_requirements` lists in the metadata object with a [Registration Field](#registration-field-format) as that key's value.
  If all `registration_requirements` lists are empty, this reference object is an empty object (`{}`).

Scope functionality described by [Scope Description](#scope-descriptions-format) objects included in the `cds_scope_descriptions` object MUST be available for all of the scope's coverage, as described by the Scope Description's `coverages_supported` list, up to any limits specified in the scopes authorization details, as described by the Scope Description's `authorization_details_fields_supported` [Authorization Details](#auth-details-fields-format) object's `limit` values.
For situations where the same scope is available for multiple groups of coverages and limits, multiple Scope Description entries with different `id` values MUST be included.
For example, if a Server offers a scope that grants access to historical customer usage data, offering 24 months of usage history for electricity service contracts and and 12 months of history for natural gas service contracts, the Server MUST include two Scope Descriptions with different `id` values that include only the coverage and limits of that scope's functionality.

Other values not mentioned here but listed in specifications for the Authorization Server Metadata specification or other extensions MAY be included as described in their respective specifications.

### 3.3. Scopes Supported <a id="scopes" href="#scopes" class="permalink">🔗</a>

This section defines scopes that MAY be included in a Server's `scopes_supported` list.

Servers MUST include at least the `client_admin` and `grant_admin` scopes in the Server's `scopes_supported` list.

Extensions MAY define and Servers MAY include additional scopes to the above list of defined scopes, so long as they include [Scope Descriptions](#scope-descriptions-format) for the scopes in the `cds_scope_descriptions`.

Extensions are RECOMMENDED to define additional scopes with their specification name or abbreviation as a prefix to prevent namespace collisions with other scopes defined by other extensions and Servers, such as `examplespec_*`.
Server are RECOMMENDED to include their name or other relevant identifier as a prefix when adding additional scopes custom to their server, such as `examplepowercompany_*`.

Clients MUST ignore any scopes in a Server's `scopes_supported` list for which the Client is not compatible.

#### 3.3.1 Client Admin Scope <a id="scopes-client-admin" href="#scopes-client-admin" class="permalink">🔗</a>

This is the main administrative scope that grants access for the Client to manage their registration using the defined APIs in this specification.

The following are how the [Scope Descriptions](#scope-descriptions-format) fields MUST be configured for this scope:

* `id` is `"client_admin"`.
* `name` is `"Client Admin"`.
* `description` is `"This scope grants administrative access to the Client management APIs."`.
* `documentation` is a _[URL](#url)_ for the Server's documentation on the `client_admin` scope.
  Servers MAY set the URL to an online publicly-available copy of this specification as adequate documentation for this scope.
* `registration_requirements` is `[]`.
* `registration_optional` is `[]`.
* `response_types_supported` is `[]`.
* `grant_types_supported` is `["client_credentials"]`.
* `token_endpoint_auth_methods_supported` is `["client_secret_basic"]`.
* `code_challenge_methods_supported` is `[]`.
* `coverages_supported` is `[]`.
* `authorization_details_fields_supported` is `[]`.

#### 3.3.2 Grant Admin Scope <a id="scopes-grant-admin" href="#scopes-grant-admin" class="permalink">🔗</a>

A scope for administratively gaining access to a specific Grant data and functionality.
Since Servers MAY arbitrarily create Grant objects included in the [Grants API](#grants-api), this scope serves as an additional method of gaining access to previously created Grant scopes by Client administrators.

The following are how the [Scope Descriptions](#scope-descriptions-format) fields MUST be configured for this scope:

* `id` is `"grant_admin"`.
* `name` is `"Grant Admin"`.
* `description` is `"This scope grants administrative access to previously created Grants."`.
* `documentation` is a _[URL](#url)_ for the Server's documentation on the `grant_admin` scope.
  Servers MAY set the URL to an online publicly-available copy of this specification as adequate documentation for this scope.
* `registration_requirements` is `[]`.
* `registration_optional` is `[]`.
* `response_types_supported` is `[]`.
* `grant_types_supported` is `["client_credentials"]`.
* `token_endpoint_auth_methods_supported` is `["client_secret_basic"]`.
* `code_challenge_methods_supported` is `[]`.
* `coverages_supported` is `[]`.
* `authorization_details_fields_supported` is a JSON array with the following two entries of [Authorization Details Field Objects](#auth-details-fields-format).
    * Client ID:
        * `id` is `client_id`.
        * `name` is `"Client object identifier"`.
        * `description` is `"The Client object identifier for which the Grant is issued."`.
        * `documentation` is a _[URL](#url)_ for the Server's documentation on the `client_id` authorization details field.
          Servers MAY set the URL to an online publicly-available copy of this specification as adequate documentation for this authorization details field.
        * `format` is `string`.
        * `is_required` is `true`.
        * `default` is not included.
        * `relative_date_limit` is not included.
        * `absolute_date_limit` is not included.
        * `limit` is not included.
        * `choices` is not included.
    * Grant ID:
        * `id` is `grant_id`.
        * `name` is `"Grant identifier"`.
        * `description` is `"The Grant identifier for which the returned access_token will be given access."`.
        * `documentation` is a _[URL](#url)_ for the Server's documentation on the `grant_id` authorization details field.
          Servers MAY set the URL to an online publicly-available copy of this specification as adequate documentation for this authorization details field.
        * `format` is `string`.
        * `is_required` is `true`.
        * `default` is not included.
        * `relative_date_limit` is not included.
        * `absolute_date_limit` is not included.
        * `limit` is not included.
        * `choices` is not included.

Server MUST only approve submissions of this scope to the token endpoint with a single authorization details entry to the token endpoint, ensuring the access token returned is only valid for one `grant_id`.
Servers MUST only approve submissions of this scope to the token endpoint when the `grant_id` is for a Grant that has a matching `client_id`, has a non-empty `enabled_scope` value (i.e. the Grant is not disabled) and neither `client_admin` or `grant_admin` are part of the `scope`.

#### 3.3.3 Server-Provided Files Scope <a id="scopes-server-provided-files" href="#scopes-server-provided-files" class="permalink">🔗</a>

A scope for Servers to grant Client access to arbitrary files that the Server wishes to securely share with the Client.
Grants with this scope MUST be created by the Server and appear on the [Grants API](#grants-api) and include `authorization_details` entries that list the `file_id` values that are being shared for that Grant.

The following are how the [Scope Descriptions](#scope-descriptions-format) fields MUST be configured for this scope:

* `id` is `"server_provided_files"`.
* `name` is `"Server-Provided Files"`.
* `description` is `"This scope grants access to specific files that the Server wants make available to the Client."`.
* `documentation` is a _[URL](#url)_ for the Server's documentation on the `server_provided_files` scope.
  Servers MAY set the URL to an online publicly-available copy of this specification as adequate documentation for this scope.
* `registration_requirements` is `[]`.
* `registration_optional` is `[]`.
* `response_types_supported` is `[]`.
* `grant_types_supported` is `[]`.
* `token_endpoint_auth_methods_supported` is `[]`.
* `code_challenge_methods_supported` is `[]`.
* `coverages_supported` is `[]`.
* `authorization_details_fields_supported` is a JSON array with a single entry [Authorization Details Field Object](#auth-details-fields-format) with the following fields.
    * `id` is `file_id`.
    * `name` is `"File identifier"`.
    * `description` is `"A file provided by the Server that may be accessed by the Client as part of the Grant."`.
    * `documentation` is a _[URL](#url)_ for the Server's documentation on the `file_id` authorization details field.
      Servers MAY set the URL to an online publicly-available copy of this specification as adequate documentation for this authorization details field.
    * `format` is `string`.
    * `is_required` is `true`.
    * `default` is not included.
    * `relative_date_limit` is not included.
    * `absolute_date_limit` is not included.
    * `limit` is not included.
    * `choices` is not included.

Since Grants with the scope are only created by Servers, Client do not receive an authorization code or access token for that Grant with which they may access the files. Instead, to access the files, Clients MUST submit a `client_credentials` grant with the scope of `grant_admin` and an `authorization_details` containing the `client_id` and `grant_id` for the Grant the Client found via the [Grants API](#grants-api).

### 3.4. Scope Descriptions Object Format <a id="scope-descriptions-format" href="#scope-descriptions-format" class="permalink">🔗</a>

Scope Description objects are formatted as JSON objects and contain named values.
The following values are included in the default list available in scope description objects.

* `id` - _[string](#string)_ - (REQUIRED) The unique identifier of the scope.
  This MUST be the same value as the object key the Metadata Object's `cds_scope_descriptions` object.
* `name` - _[string](#string)_ - (REQUIRED) A human-readable name of the scope.
* `description` - _[string](#string)_ - (REQUIRED) A human-readable description of what access or actions the scope will enable.
* `documentation` - _[URL](#url)_ - (REQUIRED) Where developers can find documentation for the scope.
* `registration_requirements` - _Array[[string](#string)]_ - (REQUIRED) A list of any registration fields that MUST be submitted with [Client Registration Requests](#registration-request) or requirements that must be completed after registration before the scope is available to use in the Server's production environment.
  All values in this array MUST be included in the Metadata Object's `cds_registration_fields` object for a complete description of the requirement.
  If no additional requirements are needed, this value is an empty array (`[]`).
* `registration_optional` - _Array[[string](#string)]_ - (REQUIRED) A list of any registration fields that Clients MAY submit with the [Client Registration Requests](#registration-request).
  If no additional fields are available, this value is an empty array (`[]`).
* `response_types_supported` - _Array[[string](#string)]_ - (REQUIRED) The OAuth `response_type` values that can be used with this scope.
  This value follows the same behavior as OAuth's [Metadata object `response_types_supported`](https://www.rfc-editor.org/rfc/rfc8414#section-2), except that this value represents the response types supported for this individual scope and not all scopes for the Client.
  If a scope is not available for OAuth authorization requests (e.g. a `client_credentials`-only scope), this value is an empty array (`[]`).
* `grant_types_supported` - _Array[[string](#string)]_ - (REQUIRED) The OAuth grant type values that can be used with this scope.
  This value follows the same behavior as OAuth's [Metadata object `grant_types_supported`](https://www.rfc-editor.org/rfc/rfc8414#section-2), except that this value represents the grant types supported for this individual scope and not all scopes for the Client.
  This array MUST contain at least one grant type.
* `token_endpoint_auth_methods_supported` - _Array[[string](#string)]_ - (REQUIRED) The OAuth client authentication methods that can be used for granting tokens with this scope.
  This value follows the same behavior as OAuth's [Metadata object `token_endpoint_auth_methods_supported`](https://www.rfc-editor.org/rfc/rfc8414#section-2), except that this value represents the client authentication methods supported for this individual scope and not all scopes for the Client.
  A value of `"none"` this array indicates that the scope may be used for public application without using a client secret.
* `code_challenge_methods_supported` - _Array[[string](#string)]_ - (REQUIRED) For scope descriptions where `grant_types_supported` includes the value `authorization_code`, OAuth's [Proof Key for Code Exchange by OAuth Public Clients](https://www.rfc-editor.org/rfc/rfc7636) functionality is required, and Servers MUST offer `S256` and MUST NOT offer `plain` code verifier methods.
  For scope descriptions where  `grant_types_supported` does not include `authorization_code`, no authorization request is performed, so no code challenge is required and this field value is an empty list (`[]`).
* `coverages_supported` - _Array[[string](#string)]_ - (REQUIRED) A list of [CDSC-WG1-01 Coverage Entry Object](/specs/cdsc-wg1-01/#coverage-entry-format) `id` values for which the scope is available by the Server.
  If the scope provides functionality unrelated to a Server's coverage entries (e.g. the `client_admin` scope), this value is an empty list (`[]`).
* `authorization_details_fields_supported` - _Array[[AuthorizationDetailsField](#auth-details-fields-format)]_ - (REQUIRED) A list of fields that MAY be included in [Rich Authorization Requests](https://www.rfc-editor.org/rfc/rfc9396) the authorization details object for this scope `type`.
  If no extra authorization details fields are available, this value is an empty list (`[]`).

### 3.5. Registration Field Object Format <a id="registration-field-format" href="#registration-field-format" class="permalink">🔗</a>

Registration Field objects are formatted as JSON objects and contain named values.
The following values are included in the default list available in registration fields objects.

* `id` - _[string](#string)_ - (REQUIRED) The unique identifier of the registration field.
  This MUST be the same value as the object key the Metadata Object's `cds_registration_fields` object.
* `type` - _[RegistrationFieldType](#registration-field-types)_ - (REQUIRED) What type of Registration Field this entry is.
* `description` - _[string](#string)_ - (REQUIRED) A human-readable description of what should be submitted or expected for this registration field.
* `documentation` - _[URL](#url)_ - (REQUIRED) Where developers can find more information about this registration field.
* `field_name` - _[string](#string)_ - (OPTIONAL) If type is `registration_field`, this is the name of the field to submit in the [Client Registration Request](#registration-request) and MUST start with `cds_`.
* `format` - _[RegistrationFieldFormats](#registration-field-formats)_ - (OPTIONAL) If type is `registration_field`, this is the data format that MUST be used in the value of the field.
* `default` - _various_ - (OPTIONAL) If type is `registration_field` and the field is optional, this is the default value that will be used in lieu of the Client submitting a value themselves.
  Including this `default` value in the object indicates the registration field is optional.
* `max_length` - _[integer](#integer)_ - (OPTIONAL) If format is one of `string`, `string_or_null`, `url`, `url_or_null`, `email`, or `email_or_null`, this is the maximum length of the submitted value, if not `null`.
* `max_size` - _[integer](#integer)_ - (OPTIONAL) If format is one of `image`, `image_or_null`, `pdf`, or `pdf_or_null`, this is the maximum file size of the submitted value before encoding into [Base64](https://en.wikipedia.org/wiki/Base64), if not `null`.
* `amount` - _[decimal](#decimal)_ - (OPTIONAL) If type is `payment_required`, this is the amount in `currency` that will be required to complete registration.
* `currency` - _[string](#string)_ - (OPTIONAL) If type is `payment_required`, this is the monetary currency in [ISO 4217](https://en.wikipedia.org/wiki/ISO_4217) currency code.

### 3.6. Registration Field Types <a id="registration-field-types" href="#registration-field-types" class="permalink">🔗</a>

Registration Field types define which type of action the Client or Server will take for that registration field.
Registration Fields can be either something that MUST or MAY be submitted as part of the [Client Registration Request](#registration-request), depending on the submitted scope's `registration_requirements` and `registration_optional` values, or can be something that a Client must do after registration in order to be full approved for use in the production environment.

The following list of strings are an enumerated set of registration field types that are valid `type` values in the registration field object.

* `registration_field` - This field should be submitted with the [Client Registration Request](#registration-request) as the `field_name`.
  This type of registration field MUST also have `field_name` and `format` values.
  If this field is optional as part of registration, `default` must also be defined in the registration field object.
* `internal_review` - This field indicates that the Server will review the registration internally before approving it for production use.
  Notifications and communication about the status of this internal review will be conveyed using the [Messages API](#messages-api).
* `payment_required` - This field indicates that a setup payment will be required before the Server will approve the Client for production use.
  Notifications and communication about how to pay and confirmation of payment will be conveyed using the [Messages API](#messages-api).
* `email_verification` - This field indicates that the Client must verify their email before the Server will approve the Client for production use.
  Notifications and communication about how to verify the Client's contact email will be conveyed using the [Messages API](#messages-api).

### 3.7. Registration Field Formats <a id="registration-field-formats" href="#registration-field-formats" class="permalink">🔗</a>

Registration Field formats define the data type of submitted values for these fields in [Client Registration Requests](#registration-request).

The following list of strings are an enumerated set of registration field formats that are valid `format` values in the registration field object.

* `string` - If required, a [string](#string) value MUST be submitted.
* `string_or_null` - Same as `string`, only with `null` being an additional possible value.
* `url` - If required, a [URL](#url) value MUST be submitted.
* `url_or_null` - Same as `url`, only with `null` being an additional possible value.
* `email` - If required, a valid email address string MUST be submitted.
* `email_or_null` - Same as `email`, only with `null` being an additional possible value.
* `boolean` - If required, a [boolean](#boolean) value MUST be submitted.
* `boolean_or_null` - Same as `boolean`, only with `null` being an additional possible value.
* `image` - If required, a valid image file formatted as `image/jpeg` or `image/png` encoded as a string using [Base64](https://en.wikipedia.org/wiki/Base64) MUST be submitted.
* `image_or_null` - Same as `image`, only with `null` being an additional possible value.
* `pdf` - If required, a valid pdf file formatted encoded using [Base64](https://en.wikipedia.org/wiki/Base64) as a string MUST be submitted.
* `pdf_or_null` - Same as `pdf`, only with `null` being an additional possible value.

### 3.8. Authorization Details Field Object Format <a id="auth-details-fields-format" href="#auth-details-fields-format" class="permalink">🔗</a>

Authorization Details Field objects are formatted as JSON objects and contain named values.
The following values are included in the default list available in authorization details field objects.

* `id` - _[string](#string)_ - (REQUIRED) The unique identifier of the authorization details field.
  This value is used as the key for the field when added to `authorization_details` data fields as part of OAuth's [Rich Authorization Requests](https://www.rfc-editor.org/rfc/rfc9396).
* `name` - _[string](#string)_ - (REQUIRED) A human-readable name of the authorization details field.
* `description` - _[string](#string)_ - (REQUIRED) A human-readable description of what submitting values for this authorization details field means.
* `documentation` - _[URL](#url)_ - (REQUIRED) Where developers can find more information about this authorization details field.
* `format` - _[AuthorizationDetailsFieldFormats](#auth-details-field-formats)_ - (REQUIRED) The data format that MUST be used in the value of the field when including it as a data field in `authorization_details` objects.
* `is_required` - _[boolean](#boolean)_ - (REQUIRED) Whether the authorization details field is required or not.
  If required, `authorization_details` MUST be submitted in authorization or token requests with this field included.
* `default` - _various_ - (OPTIONAL) If `is_required` is `false`, this is the default value that will be used in lieu of the Client submitting a value themselves.
  This is also the value that will be used if a basic OAuth `scope` string parameter is used instead of an `authorization_details` parameter.
  If `is_required` is `true`, this is not included.
* `relative_date_limit` - _[integer](#integer)_ - (OPTIONAL) If `format` is `relative_or_absolute_date` and the value is limited by the Server, this is REQUIRED and MUST be the largest relative date range that may be submitted.
  Servers MUST validate both submitted absolute dates and relative dates against the relative date limit, where when comparing to a submitted absolute date, the current Server date in the Server's local timezone is used as the relative point of reference.
  If `format` is not `relative_or_absolute_date`, this is not included.
* `absolute_date_limit` - _[integer](#integer)_ - (OPTIONAL) If `format` is `relative_or_absolute_date` the value is limited by the Server, this is REQUIRED and MUST be the furthest away date that may be submitted.
  Servers MUST validate both submitted absolute dates and relative dates against the absolute date limit, where when comparing to a submitted relative date, the current Server date in the Server's local timezone is used as the relative point of reference.
* `limit` - _[integer](#integer)_ - (OPTIONAL) If `format` is one of `int` or `decimal` and the value is limited by the Server, this is REQUIRED and MUST be the largest value that can be the submitted.
  If `format` is `string` and the length of the value is limited by the Server, this is REQUIRED and MUST be the maximum string length that can be the submitted.
* `choices` - _Array[[AuthorizationDetailsFieldChoice](#auth-details-field-choice-format)]_ - (OPTIONAL) If `format` is `choice`, this is REQUIRED and MUST be a list of one or more available choice objects.

### 3.9. Authorization Details Field Formats <a id="auth-details-field-formats" href="#auth-details-field-formats" class="permalink">🔗</a>

Authorization Details Field formats define the data type of submitted values for these fields in `authorization_details` for OAuth's [Rich Authorization Requests](https://www.rfc-editor.org/rfc/rfc9396).

The following list of strings are an enumerated set of authorization details field formats that are valid `format` values in the [Authorization Details Field objects](#auth-details-fields-format).

* `int` - An [integer](#integer) value.
* `decimal` - A [decimal](#decimal) value, which can have any number of significant units, but MUST NOT be stored or handled as a float value, in order to retain the precision of the value throughout Server and Client processing.
* `string` - A [string](#string) value.
* `string_or_null` - A [string](#string) value or `null`.
* `boolean` - A [boolean](#boolean) value.
* `relative_or_absolute_date` - A relative or absolute date string.
  Relative dates are formatted as defined by `duration` in [RFC 3339 Appendix A](https://datatracker.ietf.org/doc/html/rfc3339#appendix-A).
  For example, `P3Y` represents a relative date range of 3 years.
  Absolute dates are formatted as [date](#date) (`YYYY-MM-DD`).
  For example, `2024-01-02` represents the 2nd of January, 2024.
  Dates are defined as the date from the perspective of the Server's local timezone.
* `choice` - A [string](#string) value from the `id` parameter in one of the listed available `choices` [Authorization Details Field Choice](#auth-details-field-choice-format) objects.
* `jwk_or_null` - A [JWK Public Encryption Key](#jwk-enc) object or `null`.

### 3.10. Authorization Details Field Choice Object Format <a id="auth-details-field-choice-format" href="#auth-details-field-formats" class="permalink">🔗</a>

Authorization Details Field Choice objects are formatted as JSON objects and contain named values.
The following values are included in the default list available in Authorization Details Field Choice objects.

* `id` - _[string](#string)_ - (REQUIRED) The unique identifier of the authorization details field choice.
  This is used as the value for the relvant field when that field is included in an object as part of a `authorization_details` list.
* `name` - _[string](#string)_ - (REQUIRED) A human-readable name of the authorization details field choice.
* `description` - _[string](#string)_ - (REQUIRED) A human-readable description of what submitting this value for the authorizations details field means.
* `documentation` - _[URL](#url)_ - (REQUIRED) Where developers can find more information about this authorization details field choice.

## 4. Client Registration Process <a id="client-registration-process" href="#client-registration-process" class="permalink">🔗</a>

OAuth's [Dynamic Client Registration](https://www.rfc-editor.org/rfc/rfc7591) is used as the basis for initially registering Clients with Servers.
To perform the dynamic registration process, the Client submits a request to the Server's `registration_endpoint` provided in the Server's [Metadata object](#auth-server-metadata-format) as described in [Client Registration Request](#registration-request).
The Server then responds with either an error or a generated Client registration response as described in [Client Registration Response](#registration-response).

### 4.1. Client Registration Request <a id="registration-request" href="#registration-request" class="permalink">🔗</a>

This specification requires Clients and Servers follow the process described in OAuth's [Client Registration Request](https://www.rfc-editor.org/rfc/rfc7591#section-3.1), with the following modifications.

* Clients MAY submit additional named values that are defined as part of scope `registration_requirements` and `registration_optional` arrays in `cds_scope_descriptions` objects, using the registration field reference's `field_name` value as the submitted key in the registration request.
* Servers MUST ignore any submitted `redirect_uris` values.
  Clients MAY later update individual Client object `redirect_uri` values via the [Modifying Clients](#clients-modify) process, when that Client object `response_types` list is non-empty.

### 4.2. Client Registration Response <a id="registration-response" href="#registration-response" class="permalink">🔗</a>

This specification requires Servers follow the process described in OAuth's [Client Registration Response](https://www.rfc-editor.org/rfc/rfc7591#section-3.2), with the following modifications to OAuth's [Client Information Response](https://www.rfc-editor.org/rfc/rfc7591#section-3.2.1) object.

* The reponse object format MUST be the extended [Client object format](#client-format) defined by the Clients API.
* The `scope` value MUST be `client_admin`, indicating that the Client in the response is only used for `client_admin` access.
* The `redirect_uris` value MUST be an empty list (`[]`).
* The `response_types` value MUST be an empty list (`[]`).
* The `grant_types` value MUST contain only the `client_credentials` grant, indicating that the Client details in the response may only be used for `client_admin` client credentials grant.
* The `token_endpoint_auth_method` MUST be set to `client_secret_basic`.
* The `client_secret` MUST be included, despite it not being included in the Client object format returned by the Clients API, and set to the same value as the `client_secret` in the created [Credential object](#credentials-format) for the Client object in the response.

Upon valid registration, servers MUST create a Client object with the scope defined as `"client_admin"`, even if the `client_admin` scope was not included in the Client Registration Request.
This Client object scoped to `client_admin` is the one returned as the main Client object as the Client Registration Response.

Servers MUST also create a Client object with the scope defined as `"grant_admin"`, though it is not returned as part of the Client Registration response and is only available via the [Clients API](#clients-api).

Servers MUST also create Client objects that are configured for any other scopes for which the Client submitted registration and the submission was accepted by the Server.
Servers MAY combine scopes into individual Client objects if the `response_types`, `grant_types`, and `token_endpoint_auth_method` for the scopes are the same.

For each Client object created that has its `token_endpoint_auth_method` value as something other than `none`, at least one [Credential object](#credentials-format) MUST be created and made available on the [Credentials API](#credentials-api) for the Client to use.

For created Client objects that have a non-empty `response_types` list, Servers MUST create a default `redirect_uri` value and and set it as the single value in the created Client object `redirect_uris` list.
The created `redirect_uri` MUST provide functionality such that if specified as part of authorization request parameters, will redirect the user to a receipt of the authorization when the authorization is submitted successfully, or show an error to the user if the authorization is declined or otherwise errors.
This default `redirect_uri` allows Clients to request user authorization without needing to operate their own redirect endpoint.
Clients MAY choose to update the `redirect_uris` to remove this default or add additional redirect URIs using the [Modifying Clients](#clients-modify) functionality.

Servers MUST NOT create Clients that have both `production` and `sandbox` values in the same `cds_status_options`, so that Client objects cannot be used for both testing and production environments.
For situations where Servers do not initially create a Client with `production` as a `cds_status_option`, such as when the Server has a manual review step, Servers MUST create a Client object with `sandbox` as a `cds_status_option` so that Clients MAY being to test their applications immediately after registration.
Then, later if and when the Server decides to allow the Client access to the production environment, the Server MUST create a new Client object with `production` as a `cds_status_option`.

While Servers MAY support OAuth's [Dynamic Client Registration Management Protocol](https://www.rfc-editor.org/rfc/rfc7592) by including the `registration_client_uri` and `registration_access_token` values in their registration response, this specification requires that Servers MUST support the [Clients API](clients-api) by including the `cds_clients_api` value in the [Authorization Server Metadata](#auth-server-metadata-format) as a way for Clients to manage their registrations.
The Clients API is needed beyond the OAuth Dynamic Client Registration Management Protocol because as part of registration, Servers create multiple Clients that group the various registered scopes, and OAuth's Dynamic Client Registration Management Protocol does not support APIs for listing multiple clients or credentials that are created from a single registration request.

## 5. Clients API <a id="clients-api" href="#clients-api" class="permalink">🔗</a>

This specification requires Servers provide a set of Application Programming Interfaces (APIs) allowing Clients to view and edit their Client registrations.
These APIs are authenticated using a Bearer `access_token` obtained by the Client using OAuth's [`client_credentials` grant](https://www.rfc-editor.org/rfc/rfc6749#section-4.4) process, where the `client_id` used is for a Client that includes the `client_admin` scope.

### 5.1. Client Object Format <a id="client-format" href="#client-format" class="permalink">🔗</a>

Client objects are formatted as JSON objects and contain named values.
Client objects are required to follow the same object format as the OAuth Dynamic Client Registration Management Protocol's [Client Metadata](https://www.rfc-editor.org/rfc/rfc7591#section-2), with the following modifications.

The following fields defined in OAuth's [Client Metadata](https://www.rfc-editor.org/rfc/rfc7591#section-2) are REQUIRED to be included with all Client objects:

* `client_id`
* `client_id_issued_at`
* `scope` - If no scope is configured, this value is an empty string (`""`).
* `redirect_uris` - If no redirect URIs are configured, this value is an empty list (`[]`).
* `token_endpoint_auth_method` - If no auth methods are configured, this value is `null`.
* `grant_types` - If no grant types are configured, this value is an empty list (`[]`).
* `response_types` - If no response types are configured, this value is an empty list (`[]`).
* `client_name` - If no value is submitted by the Client, the default value is the `client_id`.
* `contacts` - If no contacts are configured, this value is an empty list (`[]`).

The following fields defined in OAuth's [Rich Authorization Requests](https://www.rfc-editor.org/rfc/rfc9396#name-metadata) are REQUIRED to be included with all Client objects:

* `authorization_details_types`

The following fields MUST NOT be included, except when the Client object is returned as a [Client Registration Response](#registration-response):

* `client_secret`

The following fields MUST NOT be included, since it is moved to the [Credentials API](#credentials-api):

* `client_secret_expires_at`

Other fields defined in the OAuth's [Client Metadata](https://www.rfc-editor.org/rfc/rfc7591#section-2) and its extensions, such as OAuth's [Dynamic Client Registration Management Protocol](https://www.rfc-editor.org/rfc/rfc7592#section-3), remain unmodified in their requirement and behavior.

In addition to the fields defined by OAuth's [Client Metadata](https://www.rfc-editor.org/rfc/rfc7591#section-2) and its extensions, the following fields are additionally defined:

* `cds_created` - _[datetime](#datetime)_ - (REQUIRED) When the Client object was created.
* `cds_modified` - _[datetime](#datetime)_ - (REQUIRED) When the Client object was last modified.
* `cds_client_uri` - _[URL](#url)_ - (REQUIRED) Where to submit modifications using the Clients API [Modifying Clients](#clients-modify) functionality.
  Servers MUST make this URL unique for each Client object.
* `cds_status` - _[ClientStatus](#client-statuses)_ - (REQUIRED) The current status of availability for this Client.
* `cds_status_options` - _Array[[ClientStatus](#client-statuses)]_ - (REQUIRED) What `cds_status` values to which the Client MAY change the Client when [modifying](#clients-modify) it.
  For Client objects that have the `client_admin` scope, this list MUST NOT contain the `disabled` value, and for all other Client objects this list MUST contain at least the `disabled` value, so that Clients MAY opt to disable Client objects as needed.
* `cds_server_metadata` - _[URL](#url)_ - (REQUIRED) Where the Client can find their registration-specific version of the [CDSC-WG1-01 Server Metadata](/specs/cdsc-wg1-01/).
  If the Client's registered scopes have different [Scope Descriptions](#scope-descriptions-format) than what is in the publicly available `oauth_metadata` resource, such as more limited `coverages_supported` or different limits in `authorization_details_fields_supported`, then the `cds_server_metadata` MUST be to a resource that has an updated `oauth_metadata` resource with the Client-specific values.
  If the Client's CDSC server metadata is no different from the public CDSC server metadata, Servers MAY simply link to the public URL.
  If this metadata endpoint requires authentication, Servers MUST authenticate Client requests to this endpoint via Bearer access token obtained using OAuth's `client_credentials` grant with a scope of `client_admin`, and reject unauthenticated requests with a `401 Unauthorized` response code.
  Clients know that they must use a Bearer token when Servers return a `401` response code for this endpoint when the Client makes an unauthenticated request to the endpoint.
* `cds_default_scope` - _[string](#string)_ - (OPTIONAL) The default scope string used when no `scope` parameter is provided as part of an authorization request.
  This MUST be included if `response_types` is not an empty list (i.e. authorization requests are enabled).
  For the initial registration response, Servers MAY choose to configure the default scope string the same as the `scope` field or another subset of default scopes.
* `cds_default_redirect_uri` - _[URL](#url)_ - (OPTIONAL) The default redirect_uri string used when no `redirect_uri` parameter is provided as part of an authorization request.
  This MUST be included if `response_types` is not an empty list (i.e. authorization requests are enabled).
  For the initial registration response, Servers MUST set this value to the value in the `redirect_uris` that was required to be created by the Server that will redirect the user by default to an authorization receipt.
* `cds_default_authorization_details` - _Array[[OAuth AuthorizationDetail](https://www.rfc-editor.org/rfc/rfc9396#section-7.1)]_ - (OPTIONAL) The default authorization details list used when no `authorization_details` parameter is provided as part of an authorization request.
  This MUST be included if `response_types` is not an empty list (i.e. authorization requests are enabled).
  For the initial registration response, Servers MUST set this to an empty list (`[]`).

### 5.2. Client Statuses <a id="client-statuses" href="#client-statuses" class="permalink">🔗</a>

Client object `cds_status` values MUST be one of the following:

* `production` - The Client object MAY be used on the Server's production systems.
  For scopes that require user authorization (e.g. `response_type` values of `code`), this `cds_status` indicates that the Client may request authorization from real users.
* `sandbox` - The Client object MAY be used to request access on the Server's test environment systems.
  For scopes that require user authorization (i.e. `response_type` values of `code`), this `cds_status` indicates that the Client may request authorization from fictional test users, as provided in the Server's `cds_test_accounts` documentation.
* `disabled` - The Client is is disabled.

### 5.3. Listing Clients <a id="clients-list" href="#clients-list" class="permalink">🔗</a>

Clients may request to list Client objects that they have access to by making an HTTPS `GET` request, authenticated with a valid Bearer `access_token` scoped to the `client_admin` scope, to the `cds_clients_api` URL included in the [Authorization Server Metadata](#auth-server-metadata-format).
The Client listing request responses are formatted as JSON objects and contain the following named values.

* `clients` - _Array[[Client](#client-format)]_ - (REQUIRED) A list of Clients to which the requesting `access_token` is scoped to have access.
  If no Clients are accessible, this value is an empty list (`[]`).
  If more than 100 Clients are available to be listed, Servers MAY truncate the list and use the `next` value to link to the next segment of the list of Clients.
* `next` - _[URL](#url) or `null`_ - Where to request the next segment of the list of Clients.
  If no next segment exists (i.e. the requester is at the end of the list), this value is `null`.
* `previous` - _[URL](#url) or `null`_ - Where to request the previous segment of the list of Clients.
  If no previous segment exists (i.e. the requester is at the front of the list), this value is `null`.

Listings of Client objects MUST be ordered in reverse chronological order by `cds_modified` timestamp, where the most recently updated relevant Client MUST be first in each listing.

For Client objects in the listing that are added, removed, or modified by the Server or Client after initial registration, Servers MUST add a message to the [listed Messages](#messages-list) notifiying the Client that a Client object has been added, removed, or modified.
This creates a changelog in the Messages listing for Client object updates.

### 5.4. Retrieving Individual Clients <a id="clients-get" href="#clients-get" class="permalink">🔗</a>

The URL to be used to send `GET` requests for retrieving individual Client objects MUST be the `cds_client_uri` provided in the [Client object](#client-format).
If a Server has optionally implemented OAuth's [Dynamic Client Registration Management Protocol](https://www.rfc-editor.org/rfc/rfc7592), the value of `cds_client_uri` MUST be the same as `registration_client_uri`, and access tokens issued from either a `client_credentials` grant with the scope `client_admin` or the access token provided as the `registration_access_token` MUST be valid access tokens to interact with the client retrieval endpoint (`cds_client_uri`).

### 5.5. Modifying Clients <a id="clients-modify" href="#clients-modify" class="permalink">🔗</a>

This specification requires that the procedure to modify Clients MUST follow OAuth's [Client Update Request](https://www.rfc-editor.org/rfc/rfc7592) section in [Dynamic Client Registration Management Protocol](https://www.rfc-editor.org/rfc/rfc7592).

The URL to be used to send `PUT` requests for updating clients MUST be the `cds_client_uri` provided in the [Client object](#client-format).
If a Server has optionally implemented OAuth's [Dynamic Client Registration Management Protocol](https://www.rfc-editor.org/rfc/rfc7592), the value of `cds_client_uri` MUST be the same as `registration_client_uri`, and access tokens issued from either a `client_credentials` grant with the scope `client_admin` or the access token provided as the `registration_access_token` MUST be valid access tokens to interact with the client endpoint.

Servers MUST allow Clients to intially update the following fields:

* `redirect_uris`
* `client_name`
* `client_uri`
* `logo_uri`
* `scope`
* `contacts`
* `tos_uri`
* `policy_uri`
* `cds_status`
* `cds_default_scope`
* `cds_default_redirect_uri`
* `cds_default_authorization_details`

Servers MUST NOT allow clients to update the following fields:

* `client_id` - This value is immutable from when it was assigned upon Client registration.
* `client_id_issued_at` - This value is immutable from when the Client initially registered.
* `client_secret` - This value is NOT included in any Client object response, except for the initial Client registration request.
  Clients MUST use the [Credentials API](#credentials-api) to manage client secrets.
* `client_secret_expires_at` - This value is NOT included in any Client object response.
  Clients MUST use the [Credentials API](#credentials-api) to discover when a client secret will expire.
* `grant_types` - This list is determined by the Server based on the `grant_types_supported` described in the the [Scope Descriptions](#scope-descriptions-format).
* `response_types` - This list is determined by the Server based on the `response_types_supported` described in the the [Scope Descriptions](#scope-descriptions-format).
* `token_endpoint_auth_method` - This is set by the Server based on the `token_endpoint_auth_methods_supported` described in the the [Scope Descriptions](#scope-descriptions-format).
* `cds_created` - This is a URL set by the Server.
* `cds_modified` - This is a URL set by the Server.
* `cds_client_uri` - This is a URL set by the Server.
* `cds_server_metadata` - This is a URL set by the Server.
* `cds_status_options` - This is a list set by the Server.

For other fields, the Server MUST determine the validity of the submitted Client fields and reject with a 400 Bad Response if not all submitted fields are either the same as previously set, or invalid values for that field.

If a Client does not include a field that is included in the Client object, this indicates that the Client wishes to reset the value of that field to the Server default.

If a Server needs to asynchronously review and approve changes to any submitted Client object fields that have been submitted by the Client and are different from the current values, for valid update requests the Server MUST respond with a `202 Accepted` response, which indicates that the submission was accepted but not fully saved as completed yet.
Any fields that have not been synchronously updated as part of the request and response MUST remain in the response as their previous values, and the Server MUST add one or more entries to the [listed Messages](#messages-list) for the modified fields that need to be asynchronously reviewed and approved.
Clients MAY then use the [Messages API](#messages-api) to track the asynchronous review of the modification request.
If all submitted fields have been synchronously updated as part of the response, Servers MUST respond with a `200 OK` response.

## 6. Messages API <a id="messages-api" href="#messages-api" class="permalink">🔗</a>

To facilitate automated communication and notificatiosn between Servers and Clients, this specification requires that official communication between Servers and Clients be performed using the Client Messages APIs.
Servers MAY implement other means of communications for exchanging messages and notifications, such as email support, but they MUST also mirror any official communications that impact Client or Grant statuses, settings, or access using the Messages API.

The Messages API endpoints are authenticated using a Bearer `access_token` obtained by the Client using OAuth's [`client_credentials` grant](https://www.rfc-editor.org/rfc/rfc6749#section-4.4) process, where the `client_id` used is for a Client that includes the `client_admin` scope.

### 6.1. Message Object Format <a id="message-format" href="#message-format" class="permalink">🔗</a>

Message objects are formatted as JSON objects and contain the following named values:

* `uri` - _[URL](#url)_ - (REQUIRED) Where to retrieve or modify this specific Message object.
* `previous_uri` - _[URL](#url) or `null`_ - (REQUIRED) Where to find the previous Message to which this Message has been created as a reply.
* `type` - _[ClientMessageType](#message-types)_ - (REQUIRED) The type of Message.
* `read` - _boolean_ - (REQUIRED) Whether the object has been marked as read by a Client.
  When the Server creates a Message, this value MUST be `false` by default, so that the Message appears in the [unread](#messages-list) list.
* `creator` - _[string](#string) or `null`_ - (REQUIRED) If the Server created the Message, this value is `null`.
  If the Client created the Message, this value is the Client's `client_id`.
* `created` - _[datetime](#datetime)_ - (REQUIRED) When the Message was created.
* `modified` - _[datetime](#datetime)_ - (REQUIRED) When the Message was last modified.
* `status` - _[ClientMessageStatus](#message-statuses)_ - (REQUIRED) The current status of the Message.
* `name` - _[string](#string)_ - (REQUIRED) A human-readable name of the Message.
  If the Message `type` is `notification`, `private_message`, or `support_request`, this is the message subject.
* `description` - _[string](#string)_ - (REQUIRED) A human-readable description of the Message.
  If the Message `type` is `notification`, `private_message`, or `support_request`, this is the message body.
* `updates_requested` - _Array[[ClientUpdateRequest](#client-update-request-format)]_ - (OPTIONAL) The list of values that a Client has requested to be updated.
  This field is required for Messages with a `type` value of `field_changes` or `server_request`.
* `related_uri` - _[URL](#url) or `null`_ - (OPTIONAL) If the Message `type` is `notification` or `private_message`, this value is where the Client can find more information, if available.
  If the Message `type` is `support_request`, this value is a relevant URL for which the Client is requesting technical support.
  If the Message `type` is `field_changes`, this is where the Client can retrieve the object that has been requested to be modified.
  If the Message `type` is `payment_request`, this is where the Client can submit their payment to the Server or if paid, a link to the payment receipt.
  If the Message `type` is `server_request`, this is where the Client can find more information about what information is being requested by the Server.
* `amount` - _[decimal](#decimal)_ - (OPTIONAL) If the Message `type` is `payment_request`, this amount the Client needs to pay to satisfy the payment request.
* `currency` - _[string](#string)_ - (OPTIONAL) If the Message `type` is `payment_request`, this is the monetary currency for the `amount` in [ISO 4217](https://en.wikipedia.org/wiki/ISO_4217) currency code.

### 6.2. Message Types <a id="message-types" href="#message-types" class="permalink">🔗</a>

Message object `type` values MUST be one of the following:

* `notification` - The Server is sending the Client at notification message that is relevant to the Client.
  This message is considered to be not individually tailored to the Client specifically, but instead directed to all relevant Clients.
* `private_message` - The Server or Client is sending a private message to the opposite party.
  This message is considered to be individually tailored and relevant only to that Client or Server.
* `support_request` - The Client is submitting a technical support request to the Server.
* `field_changes` - This Message represents that the Client has submitted changes to field values in other API objects, listed in the `updates_requested` list, but these changes were not synchronously approved by the Server and need to be reviewed asynchronously by the Server.
  While the Client originally triggered this Message to be created by submitting field changes that need to be asynchronously reviewed, the Server is responsible for creating this Message, so the `creator` value is `null`.
* `server_request` - The Server is requesting the Client submit a new Message of type `client_submission` with the fields listed in `updates_requested`.
* `client_submission` - The Client is submitting a response to a `server_request` to update one or more fields of a specific object.
  Both Clients and Servers MAY create these types of Message requests.
* `payment_request` - The Server is requesting payment by the Client to complete a financial payment before the Client may proceed with either registration approval or completing actions listed in the `updates_requested` list.

### 6.3. Message Statuses <a id="message-statuses" href="#message-statuses" class="permalink">🔗</a>

Message object `status` values MUST be one of the following:

* `complete` - For Messages with `type` values of `notification`, `private_message`, or `client_submission`, the `status` MUST be set as `complete`.
  For Messages with `type` values of `support_request`, `field_changes`, `server_request`, or `payment_request`, this represents the Server has completed and approved or resolved the Client's technical support request, field changes, submission, or payment.
* `open` - For Messages with `type` values of `server_request` or `payment_request`, this represents that the Client has not yet submitted a response to the Server's submission or payment request.
* `pending` - For Messages with `type` values of `support_request`, `field_changes`, `server_request`, or `payment_request`, this represents the Server has not yet completed it's review of the Client's technical support request, field changes, submission, or payment.
* `rejected` - For Messages with `type` values of `field_changes`, `server_request`, or `payment_request`, this represents the Server has completed and rejected the Client's requested field changes, submission, or payment.
* `errored` - For Messages with `type` values of `field_changes`, `server_request`, or `payment_request`, this represents the Server encountered an issue while processing the Client's field changes, submission, or payment.
  The Client is RECOMMENDED to submit a `support_request` Messages with the `related_uri` as the relevant errored Message's `uri`.

### 6.4. Client Update Request Object Format <a id="client-update-request-format" href="#client-update-request-format" class="permalink">🔗</a>

Client Update Request objects are formatted as JSON objects and contain the following named values:

* `field` - _[string](#string)_ - (REQUIRED) The field name of the field that is being requested to be updated.
  For Messages with `type` values of `field_changes`, this is the name of the object's field on the API.
  For Messages with `type` values of `server_request`, this is the Server's identifier for the submission being requested from the client.
* `name` - _[string](#string)_ - (OPTIONAL) For Messages with `type` values of `server_request`, this MUST be a human-readable name of the submission type being requested as the client.
* `description` - _[string](#string)_ - (OPTIONAL) For Messages with `type` values of `server_request`, this MUST be a human-readable description providing the Client with more information about what this submission request item is.
* `submitted_uri` - _[URL](#url)_ - (OPTIONAL) For Messages with `type` values of `client_submission`, this MAY be a URL that the Client is submitting as their response to the Server's `server_request`, when the Server's requested field is for a remote resource, such as an logo or binary file.
* `previous_value` - _various_ - (OPTIONAL) For Messages with `type` values of `field_changes`, this MUST be the value of the field that the is being requested to be changed from.
* `new_value` - _various_ - (OPTIONAL) For Messages with `type` values of `field_changes`, this MUST be the value of the field that the is being requested to be changed to.

### 6.5. Listing Messages <a id="messages-list" href="#messages-list" class="permalink">🔗</a>

Clients may request to list Message objects that they have access to by making an HTTPS `GET` request, authenticated with a valid Bearer `access_token` scoped to the `client_admin` scope, to the `cds_messages_api` URL included in the [Authorization Server Metadata](#auth-server-metadata-format).
The Message listing request responses are formatted as JSON objects and contain the following named values.

* `outstanding` - _Array[[ClientMessage](#message-format)]_ - (REQUIRED) A list of Messages where the `status` is `open` or `pending`.
* `outstanding_next` - _[URL](#url) or `null`_ - Where to request the next segment of the list of outstanding Messages.
  If no next segment exists (i.e. the requester is at the end of the list), this value is `null`.
* `outstanding_previous` - _[URL](#url) or `null`_ - Where to request the previous segment of the list of outstanding Messages.
  If no previous segment exists (i.e. the requester is at the front of the list), this value is `null`.
* `unread` - _Array[[ClientMessage](#message-format)]_ - (REQUIRED) A list of Messages where the `read` is `false`.
* `unread_next` - _[URL](#url) or `null`_ - Where to request the next segment of the list of unread Messages.
  If no next segment exists (i.e. the requester is at the end of the list), this value is `null`.
* `unread_previous` - _[URL](#url) or `null`_ - Where to request the previous segment of the list of unread Messages.
  If no previous segment exists (i.e. the requester is at the front of the list), this value is `null`.
* `read` - _Array[[ClientMessage](#message-format)]_ - (REQUIRED) A list of Messages where the `read` is `true`.
* `read_next` - _[URL](#url) or `null`_ - Where to request the next segment of the list of read Messages.
  If no next segment exists (i.e. the requester is at the end of the list), this value is `null`.
* `read_previous` - _[URL](#url) or `null`_ - Where to request the previous segment of the list of read Messages.
  If no previous segment exists (i.e. the requester is at the front of the list), this value is `null`.

Responses to `outstanding_next`, `outstanding_previous`, `unread_next`, `unread_previous`, `read_next`, or `read_previous` MUST be formatted the same as the initial Message listing, and MUST only include listings for the relevant next segment.
For example, if the Client requests a `unread_next`, the Server's response MUST have `outstanding` and `read` lists be empty lists (`[]`).

Listings of Message objects MUST be ordered in reverse chronological order by `modified` timestamp, where the most recently modified relevant Message MUST be first in each listing.

### 6.6. Creating Messages <a id="messages-create" href="#messages-create" class="permalink">🔗</a>

Clients create new Messages by sending an authenticated HTTPS `POST` request to the `cds_messages_api` endpoint with the body of the request formatted a JSON object.
The fields included in JSON object MUST include the following:

* `previous_uri` - _[URL](#url) or `null`_ - If submitting a Message with a `type` value of `client_submission`, this value MUST be the Message `uri` that this Message is being submitted in response to (i.e. must have a `type` value of `server_request`).
  If submitting a Message with a `type` value of `support_request` or `private_message`, if the Client is responding to a previous Message, this value MUST be the Message `uri` of that Message.
  If submitting a Message with a `type` value of `support_request` or `private_message` that is not responding to another specific Message, this value MUST be `null`.
* `type` - _[ClientMessageType](#message-types)_ - This value MUST be one of `private_message`, `support_request`, or `client_submission`.
* `name` - _[string](#string)_ - If submitting a Message with a `type` value of `client_submission`, this value MUST be an empty string (`""`).
  If submitting a Message with a `type` value of `support_request` or `private_message`, this value MUST be the subject line of the message.
* `description` - _[string](#string)_ - If submitting a Message with a `type` value of `client_submission`, this value MUST be an empty string (`""`).
  If submitting a Message with a `type` value of `support_request` or `private_message`, this value MUST be the body of the message.
* `updates_requested` - _Array[[ClientUpdateRequest](#client-update-request-format)]_ - If submitting a Client Update with a `type` value of `client_submission`, this value MUST be a list of [Client Update Request](#client-update-request-format) objects with `field` values matching the `field` values in the corresponding `server_request` Client Update Request objects, and `description` or `submitted_uri` values being the Client's submission response to the Server's request for that `field`.
* `related_uri` - _[URL](#url) or `null`_ - If submitting a Message with a `type` value of `support_request`, this value MAY be a URL to the relevant API endpoint for the support request.
  If there is no relevant API endpoint, the Client MUST set this value as `null`.

Servers MUST reject requests with a `400 Bad Request` response when a Client submits an incomplete request or the submitted values are invalid.
For valid `POST` requests from Clients, Servers MUST respond with a `201 Created` response with a JSON object of the complete newly created Message object.
When committing Messages created by Clients, Servers MUST populate the following fields in addition to the Client's submitted fields:

* `uri` - _[URL](#url)_ - The endpoint where the Client can retrieve the newly created Message object.
* `read` - _boolean_ - Always set to `true`.
* `creator` - _[string](#string)_ - Always set to the Client's `client_id`.
* `created` - _[datetime](#datetime)_ - Always set to the Server's timestamp for when the Message was created.
* `modified` - _[datetime](#datetime)_ - Always the same as `created`.
* `status` - _[ClientMessageStatus](#message-statuses)_ - For `type` values of `private_message` or `client_submission`, this value MUST be `complete`.
  For `type` values of `support_request`, this value MUST be `pending`.

When Clients submit Messages with `type` value of `client_submission`, if the Message referenced in the `previous_uri` has a `status` of `open`, then the Server MUST update the `status` of that referenced Message to `pending`, which indicates that the Client has submitted a response for Server review.

### 6.7. Modifying Messages <a id="messages-modify" href="#messages-modify" class="permalink">🔗</a>

Clients may modify fields in a Messages object by sending an authenticated HTTPS `PATCH` request to the Message `uri` endpoint with the body of the request formatted a JSON object.
The fields included in JSON object are the fields the Client intends to modify with the submitted fields' values.
If a field is not included in the `PATCH` request, the Server MUST leave the field unmodified from its current value.

The following are fields that MAY be included in the `PATCH` request, and modification MUST be supported by Servers:

* `read` - Servers MUST accept values of `true` and `false` from the Client.

If a Client wishes to modify a previously submitted Message of type `client_submission` they created, the Client MUST [create a new Message](#messages-create) with the same `previous_uri` value as the Message the Client is wishing to supersede.
Severs MUST process newer Messages created by the Client as overriding the Client's previous submission.

If a Client wishes to amend a previously created Message of type `private_message` or `support_request` they created, the Client MUST [create a new Message](#messages-create) with the `previous_uri` value set as the Message `uri` for which they are wanting to amend.
Severs MUST consider newer Messages created by the Client as amending the Client's previous support request or message.

## 7. Credentials API <a id="credentials-api" href="#credentials-api" class="permalink">🔗</a>

To allow Clients to manage `client_secret` values used for authentication to APIs, this specification requires that Server implement a Credentials API.
Servers MAY implement other means of managing Credentials, such as a web interface, but they MUST also mirror any Credentials managed by other means on this required Credentials API.

The Credentials API endpoints are authenticated using a Bearer `access_token` obtained by the Client using OAuth's [`client_credentials` grant](https://www.rfc-editor.org/rfc/rfc6749#section-4.4) process, where the `client_id` used is for a Client that includes the `client_admin` scope.

### 7.1. Credentials Object Format <a id="credentials-format" href="#credentials-format" class="permalink">🔗</a>

Credentials objects are formatted as JSON objects and contain the following named values:

* `credential_id` - _[string](#string)_ - (REQUIRED) The unique identifier for the Credential on the Server's system.
* `uri` - _[URL](#url)_ - (REQUIRED) Where to retrieve, modify, or delete this specific Credential object.
* `client_id` - _[string](#string)_ - (REQUIRED) Which Client this Credential belongs to.
* `created` - _[datetime](#datetime)_ - (REQUIRED) When the Credential was created.
* `modified` - _[datetime](#datetime)_ - (REQUIRED) When the Credential was last modified.
* `type` - _[CredentialType](#credentials-types)_ - (REQUIRED) The type of credential.
* `client_secret` - _[string](#string)_ - (OPTIONAL) A sufficiently random value generated by the Server that can be used by the Client to authenticate themselves when obtaining an `access_token` from the Server's OAuth [token endpoint](https://www.rfc-editor.org/rfc/rfc6749#section-3.2).
  This field MUST be included if the `type` is `client_secret`.
* `client_secret_expires_at` - _[integer](#integer)_ - (OPTIONAL) The timestamp at which the `client_secret` will expire, as defined in OAuth's [Client Information Response](https://www.rfc-editor.org/rfc/rfc7591#section-3.2.1).
  If the Client object refrenced by the `client_id` has a status of `disabled`, this value MUST be set to the time when the Client object was most recently disabled.
  This field MUST be included if the `type` is `client_secret`.

Other specifications MAY extend this set of fields to add additional Credential fields.

### 7.2. Credentials Types <a id="credentials-types" href="#credentials-types" class="permalink">🔗</a>

Credential object `type` values MUST be one of the following:

* `client_secret` - The Credential is a shared secret string string in the `client_secret` value on the Credential object.

Other specifications MAY extend this list to add additional Credential types.

### 7.3. Listing Credentials <a id="credentials-list" href="#credentials-list" class="permalink">🔗</a>

Clients may request to list Credential objects that they have access to by making an HTTPS `GET` request, authenticated with a valid Bearer `access_token` scoped to the `client_admin` scope, to the `cds_credentials_api` URL included in the [Authorization Server Metadata](#auth-server-metadata-format).
The Credential listing request responses are formatted as JSON objects and contain the following named values.

* `credentials` - _Array[[ScopeCredential](#credentials-format)]_ - (REQUIRED) A list of Credentials to which the requesting `access_token` is scoped to have access.
  If no Credentials are accessible, this value is an empty list (`[]`).
  If more than 100 sCredentials are available to be listed, Servers MAY truncate the list and use the `next` value to link to the next segment of the list of Credentials.
* `next` - _[URL](#url) or `null`_ - Where to request the next segment of the list of Credentials.
  If no next segment exists (i.e. the requester is at the end of the list), this value is `null`.
* `previous` - _[URL](#url) or `null`_ - Where to request the previous segment of the list of Credentials.
  If no previous segment exists (i.e. the requester is at the front of the list), this value is `null`.

Servers MUST support Clients adding any of the following URL parameters to the `GET` request, which will filter the list of Credentials to be the intersection of results for each of the URL parameters filters:

* `credential_ids` - A space-separated list of `credential_id` values for which the Servers MUST filter the Credentials.
* `client_ids` - A space-separated list of `client_id` values for which the Servers MUST filter the Credentials.
* `after` - A [datetime](#datetime) for which the Server MUST filter Credentials that were created after or on the datetime.
* `before` - A [datetime](#datetime) for which the Server MUST filter Credentials that were created before or on the datetime.

Listings of Credential objects MUST be ordered in reverse chronological order by `modified` timestamp, where the most recently modified relevant Credential MUST be first in each listing.

For Credential objects in the listing that are added, removed, or modified by the Server or Client after initial registration, Servers MUST add a message to the [listed Messages](#messages-list) notifiying the Client that a Credential object has been added, removed, or modified.
This creates a changelog in the Messages listing for Credential object updates.

### 7.4. Retrieving Individual Credentials <a id="credentials-get" href="#credentials-get" class="permalink">🔗</a>

The URL to be used to send `GET` requests for retrieving individual Credential objects MUST be the Credential `uri` provided in the [Credential object](#credentials-format).

### 7.5. Creating Credentials <a id="credentials-create" href="#credentials-create" class="permalink">🔗</a>

Clients create new Credentials by sending an authenticated HTTPS `POST` request to the `cds_credentials_api` endpoint with the body of the request formatted a JSON object.
The fields included in JSON object MUST include the following:

* `client_id` - _[string](#string)_ - This value is Client object's identifier for which a new Credential object will be created.

Servers MUST reject requests with a `400 Bad Request` response when a Client submits an incomplete request or the submitted values are invalid.
For valid `POST` requests from Clients, Servers MUST respond with a `201 Created` response with an updated JSON object of the created Credential object.

### 7.6. Modifying Credentials <a id="credentials-modify" href="#credentials-modify" class="permalink">🔗</a>

Clients may modify fields in the Credentials API by sending an authenticated HTTPS `PATCH` request to the Credential `uri` endpoint with the body of the request formatted a JSON object.
The fields included in JSON object are the fields the Client intends to update with the submitted fields' values.
If a field is not included in the `PATCH` request, the Server MUST leave the field unmodified from its current value.

The following are fields that MAY be included in the `PATCH` request, and modification MUST be supported by Servers:

* `client_secret_expires_at` - Servers MUST only accept valid timestamp values greater than or equal to the current time and less than or equal to the current value.
  When the current value is `0`, indicating no expiration time, Severs MUST allow Clients to set a value of `0` or any timestamp greater than or equal to the current time.

Servers MUST reject requests with a `400 Bad Request` response when fields are submitted that are not able to be modified by the Client or the submitted values are invalid.
If all submitted fields are valid and have been updated, the Server MUST respond with a `200 OK` response.

If a Client updates the `client_secret_expires_at` of a Credential to be a timestamp equal to or less than the current time, the Server MUST assume the Credential is compromised and synchronously disable the Credential's `client_secret` from being used on the token endpoint and revoke any `access_token` or `refresh_token` values issued by the token endpoint as a result of using the now-disabled Credential.

Servers MAY, based on their own policies or discussion between the Client and Server, modify the `client_secret_expires_at` arbitrarily.

Servers MUST NOT modify `client_secret` values after they are created.
If the `client_secret` is determined to be leaked or compromised, Servers MUST update the `client_secret_expires_at` value to the current time or less, thus expiring the Credential.

## 8. Grants API <a id="grants-api" href="#grants-api" class="permalink">🔗</a>

This specification requires Servers provide an API allowing Clients to view and edit OAuth user authorizations and client credentials grants ("Grants") related to their registration.
These APIs are authenticated using a Bearer `access_token` obtained by the Client using OAuth's [`client_credentials` grant](https://www.rfc-editor.org/rfc/rfc6749#section-4.4) process, where the `client_id` used is for a Client that includes the `client_admin` scope.

### 8.1. Grant Object Format <a id="grant-format" href="#grant-format" class="permalink">🔗</a>

Grant objects are formatted as JSON objects and contain the following named values:

* `grant_id` - _[string](#string)_ - (REQUIRED) The unique identifier for the Grant on the Server's system.
* `uri` - _[URL](#url)_ - (REQUIRED) A link to this specific Grant that can be used to [retrieve](#grants-get) or [modify](#grants-modify) the individual Grant.
* `replacing` - _Array[[URL](#url)]_ - (REQUIRED) If this Grant is superseding another Grant, this is the list of Grant `uri` values of the superseded Grants.
  If this grant is not superseding any other Grants, this value is an empty list (`[]`).
* `replaced_by` - _Array[[URL](#url)]_ - (REQUIRED) If this Grant has been superseded by other Grants, this is the list of Grant `uri` values of the superseding Grants.
  If no other Grants have superseded this Grant, this value is an empty list (`[]`).
* `parent` - _[URL](#url) or `null`_ - (REQUIRED) If this Grant represents an individual sub-authorization that is part of another Grant that required multiple authorizations, this is where to find the parent Grant.
  If this Grant is not a sub-authorization, this value is `null`.
* `children` - _Array[[URL](#url)]_ - (REQUIRED) If this Grant represents a parent Grant that has multiple sub-authorizations underneath it, this is a list of Grant `uri` values for where to find the sub-authorizations.
  If this Grant does not have any sub-authorizations, this value is an empty list (`[]`).
* `created` - _[datetime](#datetime)_ - (REQUIRED) When the Grant was created.
* `modified` - _[datetime](#datetime)_ - (REQUIRED) When the Grant was most recently modified.
  If the Grant has not been modified since creation, this is the same value as `created`.
* `not_before` - _[datetime](#datetime) or `null`_ - (REQUIRED) When a Grant that is for future access, Grant's `status` will be switched from `future` to another value.
  If the Grant is not for future access and access is available immediately upon creation, this value is `null`.
* `not_after` - _[datetime](#datetime) or `null`_ - (REQUIRED) When a Grant has access that will expire in the future, this value indicates when the Grant's `status` will be switched to `expired`.
  If the Grant does not expire, this value is `null`.
* `eta` - _[datetime](#datetime) or `null`_ - (REQUIRED) When a Grant has a `status` with the value `pending`, this value indicates an estimated time for when the status is expected to be switched to another value.
  If the Grant's `status` is not `pending` this value is `null`.
* `expires` - _[datetime](#datetime) or `null`_ - (REQUIRED) When the Grant will expire and access will be removed by the Server.
  If the Grant is to continue indefinitely, this value is `null`.
* `status` - _[GrantStatus](#grant-status)_ - (REQUIRED) The current [Grant Status](#grant-status) of the Grant.
* `client_id` - _[string](#string)_ - (REQUIRED) Which Client for which this Grant is issued.
* `cds_client_uri` - _[URL](#url)_ - (REQUIRED) Where to retrieve using the [Clients API](#clients-get) the Client for which the Grant is issued.
* `scope` - _[string](#string)_ - (REQUIRED) The scopes for which this Grant has issued access.
* `authorization_details` - _Array[[OAuth AuthorizationDetail](https://www.rfc-editor.org/rfc/rfc9396#section-7.1)]_ - (REQUIRED) Any authorization details scopes that are granted in addition to this object's `scope` value.
  If no authorization details scopes are configured in addition to the `scope` string, this value is an empty array (`[]`).
  If the `type` field of an authorization detail object in this array is the same as a value in the `scope` string, the authorization detail object replaces the scope with additional more refined access details for the scope.
* `receipt_confirmations` - _Array[[string](#string)]_ - (REQUIRED) For Grants with scopes that can be obtained via user authorization (`grant_types` contains `authorization_code`), this is a list of receipt confirmation codes that were provided to the end users who authorized the access.
  If no receipt was provided or the Grant did not get issued via `authorization_code` then this value is an empty list (`[]`).
* `enabled_scope` - _[string](#string)_ - For Grants where access has been partially granted, but some access is still disabled, this value is the `scope` that has been enabled by the server.
  For Grants where access has been fully granted, this value is the same as the `scope` value.
  For Grants that have had their access removed (e.g. `disabled`), this value is an empty string (`""`).
* `enabled_authorization_details` - _Array[[OAuth AuthorizationDetail](https://www.rfc-editor.org/rfc/rfc9396#section-7.1)]_ - For Grants where access has been partially granted, but some access is still disabled, this value is the `authorization_details` that has been enabled by the server.
  For Grants where access has been fully granted, this value is the same as the `authorization_details` value.
  For Grants that have had their access removed (e.g. `disabled`), this value is an empty list (`[]`).
* `sub_authorization_scopes` - _Array[[string](#string)]_ - (REQUIRED) For Grants that require sub-authorizations where users must provide additional authorization to enable access, this is the outstanding list of `scope` values that have yet to be authorized individually by users.

### 8.2. Grant Statuses <a id="grant-statuses" href="#grant-statuses" class="permalink">🔗</a>

Grants object `status` values MUST be one of the following:

* `active` - The Grant is active and access for the features and data specified in the `scope` or `authorization_details` is currently enabled.
  For Grants that provide access to data APIs that are being regularly updated, this status also indicates that the Server is currently on schedule to provide the expected ongoing data updates.
* `future` - The Grant will be made in the future, and the Client should check the `not_before` value for when the Grant will have its status switched to `active`.
* `pending` - The Grant has been create and is valid, but the Server is still processing or reviewing the Grant and the status will be changed to another status at approximately the time provided in the Grant's `eta` value.
  While a Grant's status is `pending`, Servers MUST provide access to the granted scope's features or data as the Server processes and the features or data becomes available.
* `partial` - The Grant was submitted originally with the `scope` and `authorization_details` values, but the Server has limited the Client's access a subset of the Grant's `scope`.
  The enabled access is listed in the `enabled_scope` and `enabled_authorization_details` values.
* `needs_authorization` - The Grant has been created, but access is limited to the `enabled_scope` and `enabled_authorization_details` until a user authorizes the listed `scope` and `authorization_details` values using an authorization URL with a `cds_grant_id` parameter of this `grant_id`.
* `needs_sub_authorizations` - The Grant has been created, but access is limited to the `enabled_scope` and `enabled_authorization_details` until user sub-authorizations, listed in the `sub_authorization_scopes` value, are granted using an authorization URL with a `cds_parent_id` parameter of this `grant_id`.
* `disabled` - The Grant has been indefinitely disabled by the Server, and the Client SHOULD NOT expect the Grant's status to be updated in the future.
* `suspended` - The Grant has been temporarily disabled by the Server or user who authorized access, and the Client SHOULD expect the Grant's status to be updated in the future.
* `revoked` - For Grants with scopes that can be obtained via user authorization (`grant_types` contains `authorization_code`), this status means the user has revoked access.
* `closed` - The Grant has been revoked by the Client via the [Modifying Grants](#grants-modify) API.
* `expired` - The Grant has expired and access has been removed.
  The Grant's `not_after` value indicates when the Grant's access was removed and the Grant is considered expired.
* `delayed` - For Grants that provide access to data APIs that are being regularly updated, this status indicates that the data being provided is behind schedule and will be updated when the Server resolves its internal delay issues.
* `stopped` - For Grants that provide access to data APIs that are being regularly updated, this status indicates that the data being provided has stopped being updated either by the Server or the user who originally authorized the access.
  Access to existing data is still available, but the Client should not expect updates to the data being provided.
* `errored` - The Server has encountered an internal error for this Grant, and Client access cannot be confirmed by the Server.

Grant `status` values of `future`, `disabled`, `suspended`, `revoked`, `closed`, and `expired` indicate that access to all features and data that were accessible via this Grant are no longer accessible.

### 8.3. Listing Grants <a id="grants-list" href="#grants-list" class="permalink">🔗</a>

Clients may request to list Grant objects that they have access to by making an HTTPS `GET` request, authenticated with a valid Bearer `access_token` scoped to the `client_admin` scope, to the `cds_grants_api` URL included in the [Authorization Server Metadata](#auth-server-metadata-format).
The Grant listing request responses are formatted as JSON objects and contain the following named values.

* `grants` - _Array[[Grant](#grant-format)]_ - (REQUIRED) A list of Grants to which the requesting `access_token` is scoped to have access.
  If no Grants are accessible, this value is an empty list (`[]`).
  If more than 100 Grants are available to be listed, Servers MAY truncate the list and use the `next` value to link to the next segment of the list of Grants.
* `next` - _[URL](#url) or `null`_ - Where to request the next segment of the list of Grants.
  If no next segment exists (i.e. the requester is at the end of the list), this value is `null`.
* `previous` - _[URL](#url) or `null`_ - Where to request the previous segment of the list of Grants.
  If no previous segment exists (i.e. the requester is at the front of the list), this value is `null`.

Servers MUST support Clients adding any of the following URL parameters to the `GET` request, which will filter the list of Grants to be the intersection of results for each of the URL parameters filters:

* `statuses` - A space-separated list of `status` values for which the Servers MUST filter the Grants.
* `client_ids` - A space-separated list of `client_id` values for which the Servers MUST filter the Grants.
* `cds_client_uris` - A space-separated list of `cds_client_uri` values for which the Servers MUST filter the Grants.
* `scopes` - A space-separated list of `scope` values for which the Server MUST filter the Grants, where the included `scope` values are values within the Grant `scope` (space separated) or as a `type` value in the `authorization_details` list.
* `receipt_confirmations` - A space-separated list of receipt confirmation codes for which the Server MUST filter the Grants.
* `after` - A [datetime](#datetime) for which the Server MUST filter Grants that were created after or on the datetime.
* `before` - A [datetime](#datetime) for which the Server MUST filter Grants that were created before or on the datetime.

Listings of Grant objects MUST be ordered in reverse chronological order by `modified` timestamp, where the most recently updated relevant Grant MUST be first in each listing.

### 8.4. Retrieving Individual Grants <a id="grants-get" href="#grants-get" class="permalink">🔗</a>

The URL to be used to send `GET` requests for retrieving individual Grant objects MUST be the Grant `uri` provided in the [Grant object](#grant-format).

### 8.5. Modifying Grants <a id="grants-modify" href="#grants-modify" class="permalink">🔗</a>

Clients may modify fields in the Grants API by sending an authenticated HTTPS `PATCH` request to the Grant `uri` endpoint with the body of the request formatted a JSON object.
The fields included in JSON object are the fields the Client intends to update with the submitted fields' values.
If a field is not included in the `PATCH` request, the Server MUST leave the field unmodified from its current value.

The following are fields that MAY be included in the `PATCH` request body, and modification MUST be supported by Servers:

* `status` - Servers MUST only accept a value of `closed` from the Client.
* `scope` - Servers MUST evaluate the updated scope for whether another user authorization is required.
  Servers MUST not require another user authorization if the updated scope is a reduction in scope of access (e.g. removal of a scope value from the scope string).
  If a new user authorization or sub-authorizations are required, the Server MUST update the `status` to one of `needs_authorization` or `needs_sub_authorizations` and update the `enabled_scope` to be the value of the current scope for which the Client is allowed access.
* `authorization_details` - Servers MUST evaluate the updated authorization details for whether another user authorization is required.
  Servers MUST not require another user authorization if the updated scope is a reduction in scope of access (e.g. removal of a scope value from the scope string).
  If a new user authorization or sub-authorizations are required, the Server MUST update the `status` to one of `needs_authorization` or `needs_sub_authorizations` and update the `enabled_authorization_details` to be the value of the current authorization details for which the Client is allowed access.

Servers MUST reject requests with a `400 Bad Request` response when fields are submitted that are not able to be modified by the Client or the submitted values are invalid.
For valid `PATCH` requests from Clients, Servers MUST respond with a `200 OK` response with an updated JSON object of the complete current Grant object.

If a Server needs to asynchronously review and approve changes to any submitted Grant object fields that have been submitted by the Client and are different from the current values, for valid modification requests the Server MUST update the Grant's `status` to `pending` in the response.
Additionally, if the `scope` or `authorization_details` has been updated, the Server MUST update the `enabled_scope` and `enabled_authorization_details` to reflect for what the Client currently has access while the Grant is pending.

## 9. Server-Provided Files API <a id="server-provided-files-api" href="#server-provided-files-api" class="permalink">🔗</a>

This specification defines an API by which Servers MAY provide an access to arbitrary files to Clients to download.
These APIs are authenticated using a Bearer `access_token` obtained by the Client using OAuth's [`client_credentials` grant](https://www.rfc-editor.org/rfc/rfc6749#section-4.4) process, where the scope of the access token is `grant_admin` with `authorizatin_details` entries listing `grant_id` values that are for Grants that have the `server_provided_files` scope.

This API is intended to provide a convenient way for Servers to provide secure ad-hoc file access to Clients, such as sharing connectivity-related files (e.g. configs, certificates, secret keys, etc.) or manually created bulk files (e.g. initial backfill raw data, analysis reports, etc.).
This API is NOT intended to be used for automated sharing of structured data (e.g. nightly interval extracts) because the API has limited functionality to convey the appropriate metadata for automated file sharing, such as versioning or schemas.
It is RECOMMENDED that automated sharing of structured datasets be performed using another structured API or transfer method more fit for purpose, which MAY be an extension to this specification.

### 9.1. Server-Provided Files Object Format <a id="server-provided-files-format" href="#server-provided-files-format" class="permalink">🔗</a>

Server-Provided File objects are metadata for arbitrary files made accessible by the Server and are formatted as JSON objects containing the following named values:

* `file_id` - _[string](#string)_ - (REQUIRED) The unique identifier for the Server-Provided File on the Server's system.
* `created` - _[datetime](#datetime)_ - (REQUIRED) When the Server-Provided File was created.
* `modified` - _[datetime](#datetime)_ - (REQUIRED) When the Server-Provided File was most recently modified.
  If the Server-Provided File has not been modified since creation, this is the same value as `created`.
* `mime_type` - _[string](#string)_ - (REQUIRED) The [MIME type](https://www.iana.org/assignments/media-types/media-types.xhtml) of the file.
* `size` - _[integer](#integer)_ - (REQUIRED) The size of the file in bytes.
* `name` - _[string](#string)_ - (REQUIRED) The file name, including any file extension.
* `description` - _[string](#string)_ - (REQUIRED) A description of the file. If no description is provided by the Server, this is an empty string (`""`).
* `download_uri` - _[URL](#url)_ - (REQUIRED) A link to the raw file being made accessible by the Server.
  Servers MUST authenticate this download endpoint using the same authentication requirements as the rest of the [Server-Provided Files API](#server-provided-files-api).

### 9.2. Listing Server-Provided Files <a id="server-provided-files-list" href="#server-provided-files-list" class="permalink">🔗</a>

Clients may request to list Server-Provided File objects that they have access to by making an HTTPS `GET` request, authenticated with a valid Bearer `access_token` scoped to the `client_admin` scope, to the `cds_server_provided_files_api` URL included in the [Authorization Server Metadata](#auth-server-metadata-format).
The Server-Provided File listing request responses are formatted as JSON objects and contain the following named values.

* `files` - _Array[[ServerProvidedFile](#server-provided-files-format)]_ - (REQUIRED) A list of Server-Provided File objects to which the requesting `access_token` is scoped to have access.
  If no Server-Provided File objects are accessible, this value is an empty list (`[]`).
  If more than 100 Server-Provided File objects are available to be listed, Servers MAY truncate the list and use the `next` value to link to the next segment of the list of Server-Provided File objects.
* `next` - _[URL](#url) or `null`_ - Where to request the next segment of the list of Server-Provided File objects.
  If no next segment exists (i.e. the requester is at the end of the list), this value is `null`.
* `previous` - _[URL](#url) or `null`_ - Where to request the previous segment of the list of Server-Provided File objects.
  If no previous segment exists (i.e. the requester is at the front of the list), this value is `null`.

Listings of Server-Provided File objects MUST be ordered in reverse chronological order by `modified` timestamp, where the most recently updated relevant Server-Provided File object MUST be first in each listing.

## 11. Extensions <a id="extensions" href="#extensions" class="permalink">🔗</a>

<span style="background-color:yellow">TODO</span>

## 12. Examples <a id="examples" href="#examples" class="permalink">🔗</a>

<span style="background-color:yellow">TODO</span>

## 13. Security Considerations <a id="security" href="#security" class="permalink">🔗</a>

This specification describes a protocol by which a utility or other entity (a Server) can allow third parties (Clients) to register and obtain privileged access to the Server's offered capabilities and data.
Because the functionality described in this specification enables access to private Server functionality and data, Servers MUST follow industry cybersecurity best practices when securing their implementations of this specification to prevent unintended or inadvertent access to privileged functionality or data to Clients who are not authorized.
These best practices include requiring HTTPS for API endpoints using the latest widely adopted encryption standards, undergoing regular security audits and penetration tests, and internally requiring security-focused process controls and data handling procedures.

<span style="background-color:yellow">TODO: Other security considerations</span>

### 13.1. Restricted Access <a id="restricted-access" href="#restricted-access" class="permalink">🔗</a>

For unauthenticated endpoints ([Authorization Server Metadata](#auth-server-metadata), [Client Registration Process](#client-registration-process)), while Servers can add [rate limiting](#rate-limiting) configurations to protect their systems from being overwhelmed with requests, Servers MUST NOT add anti-bot blocking measures (e.g. captchas) that prevent automated requests from other systems.
The functionality described in this specification is intended to be able to be integrated in other platforms to allow those platforms to automate interactions with Servers on their users' behalf.

If [Authorization Server Metadata](#auth-server-metadata) is referenced in a public [CDSC-WG1-01 Server Metadata endpoint](/specs/cdsc-wg1-01/#metadata-endpoint), then the [Authorization Server Metadata](#auth-server-metadata) and [Client Registration Process](#client-registration-process) must also be public.

Servers that wish to restrict access of by-default unauthenticated endpoints to certain Clients MUST configure well established authentication processes for Clients to ensure that only the approved Clients may access the restricted endpoint.
This specification does not describe specifically how Servers will authenticate Clients for by-default unauthenticated endpoints, as these restricted access protocols are context dependent.
For example, if a Server providing a private Client Registration endpoint as part of an existing logged in portal, then they can use that logged in portal's session cookie to authenticate Client requests to the registration endpoint.

For authenticated endpoints ([Clients API](#clients-api), [Messages API](#messages-api), [Credentials API](#credentials-api), [Grants API](#grants-api)), Servers MUST authenticate requests using OAuth's [Authorization Request Header Field](https://www.rfc-editor.org/rfc/rfc6750#section-2.1) with access tokens obtained using the OAuth's [Issuing an Access Token](https://www.rfc-editor.org/rfc/rfc6749#section-5) process.

### 13.2. Rate Limiting <a id="rate-limiting" href="#rate-limiting" class="permalink">🔗</a>

For unauthenticated endpoints ([Authorization Server Metadata](#auth-server-metadata), [Client Registration Process](#client-registration-process)), Servers SHOULD configure rate limiting restrictions so that bots and misconfigured scripts will not flood and overwhelm the endpoints with requests, while still allowing legitimate and low-volume automated requests have access to the endpoints.

For authenticated endpoints ([Clients API](#clients-api), [Messages API](#messages-api), [Credentials API](#credentials-api), [Grants API](#grants-api)), Servers SHOULD configure rate limiting by Client and Credential to ensure that individual Clients do not overwhelm Servers with authenticated API requests.
Additionally, Servers SHOULD configure rate limiting for unauthenticated or failed authentication requests to authenticated API endpoints to prevent brute force attempts to gain access to authenticated APIs.

## 14. References <a id="references" href="#references" class="permalink">🔗</a>

<span style="background-color:yellow">TODO</span>

## 15. Acknowledgments <a id="acknowledgments" href="#acknowledgments" class="permalink">🔗</a>

The authors would like to thank the late Shuli Goodman, who was the Executive Director of LFEnergy, for her incredible leadership in initially organizing the Carbon Data Specification.

## 16. Authors' Addresses <a id="authors-addresses" href="#authors-addresses" class="permalink">🔗</a>

Daniel Roesler  
UtilityAPI  
Email: [daniel@utilityapi.com](mailto:daniel@utilityapi.com)  
URI: [https://utilityapi.com/](https://utilityapi.com/)


