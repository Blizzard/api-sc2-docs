# Sc2 Community Web API

This is the documentation for the RESTful APIs exposed through the Starcraft 2
community site as a service to the Starcraft 2 community.

To view the inlined gist json examples you should view this document at
[http://blizzard.github.com/api-sc2-docs/](http://blizzard.github.com/api-sc2-docs/).

## Introduction

The Blizzard Community Platform API provides a number of resources for
developers and Sc2 enthusiasts to gather data about their profile and
ladders. This documentation is primarily for developers and third parties.

Blizzard's epic gaming experiences often take place in game, but can
lead to rewarding and lasting experiences out of game as well. Through
exposing key sets of data, we can enable the community to create
extended communities to continue that epic experience.

## Features

Before getting started with the Community Platform API, programmers
must first understand how the API is organized and how it works. The
following sections provide a high level overview of the features of
this API. It is recommended that the reader have knowledge of the HTTP
protocol as well as a general understanding of web technologies.

### REST

The API is mostly RESTful. Data is exposed in the form of URIs that
represent resources and can be fetched with HTTP clients (like web
browsers). At this time, the API is limited to read-only operations.

### Access and Regions

To access the API, HTTP requests can be made to specific URLs and
resources exposed on the regional Battle.net domains.

The data available through the API is limited to the region that it is
in. Hence, US APIs accessed through us.battle.net will only contain
data within US battlegroups and realms. Support for locales is limited
to those supported on the [Starcraft II community sites](http://battle.net/sc2/).

#### Localization

All of the API resources provided adhere to the practice of providing
localized strings using the locale query string parameter. The locales
supported vary from region to region and align with those supported on
the community sites.

To access a different region just provide the `locale=pt_BR` (for
example) query parameter.

*Example URL for the europe achievement data list localized in French*
```plain
http://eu.battle.net/api/sc2/data/achievements?locale=fr_FR
```

#### Region Host List
<table>
  <tr>
    <th>Region</th>
	<th>Host</th>
	<th>Available Locales</th>
  </tr>
  <tr>
    <td>US</td>
	<td>us.battle.net</td>
	<td>
	  <a href="http://us.battle.net/api/sc2/data/achievements?locale=en_US">en_US</a><br/>
	  <a href="http://us.battle.net/api/sc2/data/achievements?locale=es_MX">es_MX</a><br/>
	  <a href="http://us.battle.net/api/sc2/data/achievements?locale=pt_BR">pt_BR</a>
	</td>
  </tr>
  <tr>
    <td>Europe</td>
	<td>eu.battle.net</td>
	<td>
	  <a href="http://eu.battle.net/api/sc2/data/achievements?locale=en_GB">en_GB</a><br/>
	  <a href="http://eu.battle.net/api/sc2/data/achievements?locale=es_ES">es_ES</a><br/>
	  <a href="http://eu.battle.net/api/sc2/data/achievements?locale=fr_FR">fr_FR</a><br/>
	  <a href="http://eu.battle.net/api/sc2/data/achievements?locale=ru_RU">ru_RU</a><br/>
	  <a href="http://eu.battle.net/api/sc2/data/achievements?locale=de_DE">de_DE</a><br/>
	  <a href="http://eu.battle.net/api/sc2/data/achievements?locale=pt_PT">pt_PT</a><br/>
	  <a href="http://eu.battle.net/api/sc2/data/achievements?locale=pl_PL">pl_PL</a><br/>
	  <a href="http://eu.battle.net/api/sc2/data/achievements?locale=it_IT">it_IT</a>
	</td>
  </tr>
  <tr>
    <td>Korea</td>
	<td>kr.battle.net</td>
	<td>
	  <a href="http://kr.battle.net/api/sc2/data/achievements?locale=ko_KR">ko_KR</a>
	</td>
  </tr>
  <tr>
    <td>Taiwan</td>
	<td>tw.battle.net</td>
	<td>
	  <a href="http://tw.battle.net/api/sc2/data/achievements?locale=zh_TW">zh_TW</a>
	</td>
  </tr>
  <tr>
    <td>China</td>
	<td>www.battlenet.com.cn</td>
	<td>
	  <a href="http://www.battlenet.com.cn/api/sc2/data/achievements?locale=zh_CN">zh_CN</a>
	</td>
  </tr>
</table>

### Throttling

Consumers of the API can make a limited number of requests per day, as
stated in the API Policy and Terms of Use. For anonymous consumers of
the API, the number of requests that can be made per day is set to
3,000. Once that threshold is reached, depending on site activity and
performance, subsequent requests may be denied. High limits are
available to registered applications.

### Authentication

Although most of the application can be accessed without any form of
authentication, we do support a form application registration and
authentication. Application authentication involves creating and
including an application identifier and a request signature and
including those values with the request headers.

The primary benefit to making requests as an authenticated application
is that you can make more requests per day.

#### Application Registration

To send authenticated request you first need to register an
application. Because registration isn't automated, application
registration is limited to those who meet the following criteria:

* You plan on making requests from one or more IP addresses. (e.g. a
  production environment and development environment)

* You can justify making more than 2,000 requests per day from one or
  more IP addresses.

Registering an application is a matter of providing a description of
the application, how you plan on using the API and your contact
information to
[api-support@blizzard.com](mailto:api-support@blizzard.com) with the
subject "Application Registration Request". Once we receive your
request, we will contact you to either provide additional information
or with application keys to use.

#### Authentication Process

**NOTE** The URLs presented below are for the wow api, but the same process
applies for sc2 urls.

To authenticate a request, simple include the "Authorization" header
with your application identifier and the request signature.

*An example authenticated request*

    GET /api/wow/character/Medivh/Thrall HTTP/1.1
    Host: us.battle.net
    Date: Fri, 10 Jun 2011 20:59:24 GMT
    Authorization: BNET c1fbf21b79c03191d:+3fE0RaKc+PqxN0gi8va5GQC35A=

In the above exmple, the value of the Authorization header has three
parts `"BNET"`, `"c1fbf21b79c03191d"` and
`"+3fE0RaKc+PqxN0gi8va5GQC35A="`. The first part is a processing
directive for the Authorization header. The second and third values
are the application public key and the request signature. The
application public key is assigned by Blizzard during the application
registration process. The signature is generated with each request and
is described by the following algorithm.

    UrlPath = <HTTP-Request-URI, from the port to the query string>
    
    StringToSign = HTTP-Verb + "\n" +
        Date + "\n" +
        UrlPath + "\n";
    
    Signature = Base64( HMAC-SHA1( UTF-8-Encoding-Of( PrivateKey ), StringToSign ) );
    
    Header = "Authorization: BNET" + " " + PublicKey + ":" + Signature;

The above process can be seen in action by filling in the blanks:

    UrlPath = "/api/wow/realm/status"
    
    StringToSign = "GET" + "\n" +
        "Fri, 10 Jun 2011 21:37:34 GMT" + "\n" +
        UrlPath + "\n";
    
    Signature = Base64( HMAC-SHA1( UTF-8-Encoding-Of( "examplesecret" ), StringToSign ) );
    
    Header = "Authorization: BNET" + " " + "examplekey" + ":" + Signature;

The date timestamp used in the above algorithm and example is the
value of the Date HTTP header. The two date values, the first being
used to sign the request and the second as sent with the request
headers, must be the same and within 5 minutes of the current GMT
time.

**Important** We strongly advise that client library developers make
secure requests using SSL whenever application authentication is used.

### Formats and Protocols

Data returned in the response message is provided in JSON
format. Please refer to the examples provided with each API section
for additional information.

### Error Handling

Although several of the API resources have specific error responses
that correspond to specific situations, there are several generic
error responses that you should be aware of.

Errors are returned as JSON objects that contain "status" and "reason"
attributes. The value of the "status" attribute will always be
"nok". The reason will be an english string that may be, but is not
limited to, one of the following strings.

<table>
  <tr>
    <th>Code</th>
    <th>Message</th>
    <th>Description</th>
  </tr>
  <tr>
    <td>500</td>
    <td>Invalid Application</td>
    <td>A request was made including application identification information, but either the application key is invalid or missing.</td>
  </tr>
  <tr>
    <td>500</td>
    <td>Invalid application permissions.</td>
	<td>A request was made to an API resource that requires a higher application permission level.</td>
  </tr>
  <tr>
    <td>500</td>
    <td>Access denied, please contact api-support@blizzard.com</td>
	<td>The application or IP address has been blocked from making further requests. This ban may not be permanent.</td>
  </tr>
  <tr>
    <td>404</td>
    <td>When in doubt, blow it up. (page not found)</td>
	<td>A request was made to a resource that doesn't exist.</td>
  </tr>
  <tr>
    <td>500</td>
    <td>If at first you don't succeed, blow it up again. (too many requests)</td>
	<td>The application or IP has been throttled.</td>
  </tr>
  <tr>
    <td>500</td>
    <td>Have you not been through enough? Will you continue to fight what you cannot defeat? (something unexpected happened)</td>
	<td>There was a server error or equally catastrophic exception preventing the request from being fulfilled.</td>
  </tr>
  <tr>
    <td>500</td>
    <td>Invalid authentication header.</td>
	<td>The application authorization information was mallformed or missing when expected.</td>
  </tr>
  <tr>
    <td>500</td>
    <td>Invalid application signature.</td>
	<td>The application request signature was missing or invalid. This will also be thrown if the request date outside of a 15 second window from the current GMT time.</td>
  </tr>
</table>

## Profile API

The Profile API is the primary way to access data about a sc2 profile. This
profile API can be used to fetch a single character profile at a time through an
HTTP GET request.

```plain
URL = Host + "/api/sc2/profile/" + id + "/" + realm + "/" + CharacterName
```

You'll notice that this is identical to the website profile. So if you character
url on the website is http://us.battle.net/sc2/en/profile/999000/1/DayNine/ your
api url will be http://us.battle.net/api/sc2/profile/999000/1/DayNine/

<dl>
  <dt>Example URL</dt>
  <dd>/api/sc2/profile/999000/1/DayNine/</dd>
</dl>

### Ladders

If you add `/ladders` to the profile url you can view ladder information for the
player in question.

<dl>
  <dt>Example URL</dt>
  <dd>/api/sc2/profile/999000/1/DayNine/ladders</dd>
</dl>

### Matches

If you add `/matches` to the profile url you can view the match information for the
last 10 matches.

<dl>
  <dt>Example URL</dt>
  <dd>/api/sc2/profile/999000/1/DayNine/matches</dd>
</dl>

## Ladder API

The ladder API provides data about ladders for the current and previous sc2 seasons.
The ladder API can be accessed by the following HTTP GET request

```plain
URL = Host + "/api/sc2/ladder/" + ladderId
```
The ladderId is provided from the profile ladder information. The result will be
all the members of the particular ladder, with their statistics for that ladder.

<dl>
  <dt>Example URL</dt>
  <dd>/api/sc2/ladder/655</dd>
</dl>

The ladder API also provides the rankings for grandmasters.
There are two queries available. These two provide the current and previous season rankings 
of the grandmaster ladder for the region.

<dl>
  <dt>Current Season Grandmaster</dt>
  <dd>/api/sc2/ladder/grandmaster</dd>
</dl>

<dl>
  <dt>Previous Season Grandmaster</dt>
  <dd>/api/sc2/ladder/grandmaster/last</dd>
</dl>

## Data Resources

The data APIs provide information that can compliment profile
information to provide structure, definition and context.

### Achievements

The achievements data API provides information on the achievements available in
Sc2.

```plain
URL = Host + "/api/sc2/data/achievements"
```

### Rewards

The rewards data API provides information on profile portraits, decals, skins
and animations that are available in Sc2.

```plain
URL = Host + "/api/sc2/data/rewards
```

## Policies and Support
### Support

For questions about the API, please use the Community Platform API
forums as a platform to ask questions and get help.

http://us.battle.net/wow/en/forum/2626217/

You can also email
[api-support@blizzard.com](mailto:api-support@blizzard.com) for
matters that you may not want public discussion for.

### API Policy

With the continued popularity of third-party applications, which are
referred to hereafter as "applications" or "web applications", created
by the community of players for use with our games, Blizzard
Entertainment has created formal guidelines for their design and
distribution. These guidelines have been put in place to ensure the
integrity of our games and to help promote an enjoyable gaming
environment for all of our players.

Thank you for reading these guidelines, and for helping Blizzard
Entertainment continue to deliver high-quality gameplay experiences.

#### Intended Audience

The Third-Party API Usage Policy is for developers developing
applications, where applications include distributed and
non-distributed products and services that at any point engage
Blizzard Entertainment Web API resources. Web API resources include
any data that can be accessed through HTTP requests to URLs on the
Battle.net website that begin with "/api".

Example applications include, but are not limited to:

* Client libraries
* Desktop applications
* Services and deamons such as websites and web services
* Scripts and non-compiled applications and utilities

Blizzard Entertainment reserves the right to change the location and
definition of what constitutes an application and Blizzard
Entertainment Web API resources at any time.

#### Service Availability Notice

Blizzard Entertainment makes no guarantee of the availability of any
data, functionality, or feature provided by or through the API. In
addition, Blizzard Entertainment may at any time revoke access to the
API or disable part or all of the API without any warning or notice.

Applications must abide by the following access guidelines.

The following guidelines have been put in place to ensure that all
users of an API will be able to access it:

* Applications may make up to a total of 10,000 unauthenticated
  requests per day.
* Applications may make up to a total of 50,000 authenticated requests
  per day.
* Applications may not use multiple forms of access, including making
  any combination of unauthenticated and authenticated requests or
  using multiple API keys, to make more requests than permitted by the
  guidelines above. Applications may not use other third-party
  services to make additional requests on their behalf.
* Applications may not sell, share, transfer, or distribute
  application access keys or tokens.

Applications may be classified, solely at Blizzard Entertainment's
discretion, to allow fewer or greater numbers of requests per
day. Blizzard Entertainment also reserves the right to revoke access
to the API completely and without warning.

#### Applications may not charge premiums for features that use the API.

"Premium" versions of applications offering additional for-pay
features are not permitted, nor can players be charged money to
download an application, charged for services related to the
application, or otherwise be required to offer some form of monetary
compensation to download or access an application when those features
use the API. Applications may not include interstitials soliciting
donations before features or functionality becomes available to the
player.

#### Applications must not negatively impact Blizzard Entertainment games, services, or other players.

Applications must perform no function which, in Blizzard
Entertainment's sole discretion, negatively impacts the performance of
Blizzard Entertainment games or services, or otherwise negatively
affects the game for other players.

#### Application code must be completely visible.

The programming code of an application must in no way be hidden or
obfuscated, and must be freely accessible to and viewable by the
general public.

#### Applications may not imply any association with Blizzard Entertainment.

Applications may not imply any association with, or endorsement by,
Blizzard Entertainment.

#### Applications must not contain offensive or objectionable material.

Blizzard Entertainment requires that applications contain no
offensive, obscene, or otherwise objectionable material, as determined
by Blizzard's sole discretion. Applications should contain only
content appropriate for the ESRB rating for the related game(s). For
example, World of Warcraft has been rated "T for Teen" by the ESRB,
and has received similar ratings from other ratings boards around the
world.

#### Blizzard trademarks, titles, or tradenames should not be used when naming an application.

Applications may not use names based on Blizzard's trademarks or taken
from Blizzard's products as the name, or part of the name, of the
application.

### License for Use

Applications that use Blizzard Entertainment intellectual property,
such as Blizzard Web API resources, require a license for that
use. Blizzard Entertainment may at its sole discretion request that
any application that uses its intellectual property be removed and no
longer distributed.

### Policy Compliance Notice

Blizzard Entertainment is committed to maintaining the integrity of
our games and services and to providing safe, fair, and fun gaming
environment for all of our players. As such, failure to abide by the
guidelines in this policy may result in measures up to and including
legal action, when necessary.
