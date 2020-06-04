## Considerations for Improvement

- text encoding (Unicode/utf8)
- timezones (UTC)
- SQL abstraction
- html templating
- date math
- localization (I18N)
- ticket/task management
- deployment processes
- unit (& other) testing
- persisent processes for request handling (not reading from disk for every request)
- non-sequential IDs that don't rely on DB to generate

### Application Borders

It's important to consider data within the app as a conistent format, which is not necessarily the same as data outside the app (received or sent to user). Any data conversion is done at the application border; which means the data format within the app is **always** the same. 

The common issues are text encoding and timezones. As a rule of thumb, all text inside the app (stored in db, in files on disk, etc) should be Unicode (commonly utf8); and all timestamps should be UTC. 

For most purposes, the UTF8 encoding doesn't need to change for data leaving the app, and it's easy to ensure all data coming in is converted (if needed) to UTF8. Timezones are a validation & formatting issues; ideally any dates are sent either sent to browser as UTC, and a Javascript plugin can do the localization/formatting for users timezone; or format the date as part of the html template rendering. And the same for date validation; either have a Javascript plugin convert to UTC before submitting, or have server-side validation convert from user's timezone to UTC. 

### SQL Queries

Hand-crafed SQL code is brittle, hard to maintain, and error-prone. Generating dynamic WHERE clauses is **heaps** easier when treating it like a native data structure (arrays & hashes). 

### HTML Templates

Inline HTML code is brittle, hard to maintain, and error-prone. Use of HTML templates brings **lots** of benefits:

- last-minute data formatting (dates, currencies, addresses, etc)
- view logic separated from app logic
- caching & rendering speed (good template engines are quicker than lots of separate print statements)
- and more...

### Date Math

So many good modules for date handling; no good excuse to doing date math by hand (or by sending to db). `DateTime` is the popular choice. Combine that with converting all dates to/from DateTime objects when reading/writing the the DB and trivial date math is always available.

### Localization - AKA I18N

Speaks for itself doesn't it; there are lots of languages other than English; there are enough date and currency formats other than US/AU.

### Persistent HTTP Processes

Except for very lightweight scripts; the extra server load needed to parse/load scripts for *every* request is quite high. Quite substantial hardware savings can be realized by pre-loading scripts into persistent processes. Often a bit more memory is needed, but CPU and disk IO is reduced considerably. DB connections can also be made persistent which lessens the load on database as well. 

### Non-sequential IDs

There are two issues this is addressing; security and scalability/reliability.

From a security standpoint; sequential IDs are a goldmine for hackers. There have been numerous exploits that were possible simply due to hacker attacking the same request, and simply incrementing ID value until he gets a hit. There are also examples of data leakage due to sequential IDs - "propertyid is 5 higher next hour, so site is handling 5 new listings per hour" (bad example, but apply same logic to more sensitive data). 

Sequential IDs are not scalable *at all* if the "app" will ever get used offline reliably; eg agent takes contact details via phone app while off-network, and sends confirmation details back to customer - what ID is generated for that contact - how is that later mapped back to the DB. But using (eg) CUID values for IDs solves the scalability/reliability issues, as well as security issues.  
