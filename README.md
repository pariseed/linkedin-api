# WHY THIS FORK ?
for some reason (maybe my fault) i cannot get this lib in his upstream work for me, so i made some changes in
order to successfully search jobs and read their descriptions, i will provide also some examples.

Feel free to contribute if you find that fork useful.




# Linkedin API for Python


Programmatically send messages, get jobs, and search profiles with a regular Linkedin user account.

No "official" API access required - just use a valid Linkedin account!

**Caution**: This library is not officially supported by LinkedIn. Using it might violate LinkedIn's Terms of Service. Use it at your own risk.

## Installation

> Python >= 3.6 required

To install the linkedin_api package, use the following command:

```bash
pip3 install git+https://github.com/tomquirk/linkedin-api.git
```

### Example profile get

> See all methods on the [documentation website](https://linkedin-api.readthedocs.io/).

Below is a basic example of how to use linkedin_api:

```python
from linkedin_api import Linkedin

# Authenticate using any Linkedin account credentials
api = Linkedin('reedhoffman@linkedin.com', '*******')

# GET a profile
profile = api.get_profile('billy-g')

# GET a profiles contact info
contact_info = api.get_profile_contact_info('billy-g')

# GET 1st degree connections of a given profile
connections = api.get_profile_connections('1234asc12304')
```


### Example job get

```python
from linkedin_api import Linkedin

api = Linkedin('user', 'passwd')
max_results = 90
kw = "devops" 

jobs = api.search_jobs(keywords=kw,location_name="Europe",limit=max_results)


for elem in range(0, len(jobs)):
    title = jobs[elem]["title"]
    job_description = api.get_job(jobs[elem]["entityUrn"].split(':')[3])["description"]["text"]
    company_id = jobs[elem]["companyDetails"]["company"].split(':')[3]
    company_name = api.get_company(public_id=company_id)["name"]
    location = jobs[elem]["formattedLocation"]

    print(title)
    print(job_description)
    print(company_id)
    print(company_name)
    print(location)
```


## Disclaimer

This library is not endorsed or supported by LinkedIn. It is an unofficial library intended for educational purposes and personal use only. By using this library, you agree to not hold the author or contributors responsible for any consequences resulting from its usage.

### Dependencies

- Python 3.7
- A valid Linkedin user account (don't use your personal account, if possible)
- `pipenv` (optional)


#### I keep getting a `CHALLENGE`

Linkedin will throw you a curve ball in the form of a Challenge URL. We currently don't handle this, and so you're kinda screwed. We think it could be only IP-based (i.e. logging in from different location). Your best chance at resolution is to log out and log back in on your browser.

**Known reasons for Challenge** include:

- 2FA
- Rate-limit - "It looks like you’re visiting a very high number of pages on LinkedIn.". Note - n=1 experiment where this page was hit after ~900 contiguous requests in a single session (within the hour) (these included random delays between each request), as well as a bunch of testing, so who knows the actual limit.

Please add more as you come across them.

#### Search problems

- Mileage may vary when searching general keywords like "software" using the standard `search` method. They've recently added some smarts around search whereby they group results by people, company, jobs etc. if the query is general enough. Try to use an entity-specific search method (i.e. search_people) where possible.

## How it works

> Before using this project, please consult the [Terms and Conditions](#terms-and-conditions) and [Legal Notice](#legal).

This project attempts to provide a simple Python interface for the Linkedin API.

> Do you mean the [legit Linkedin API](https://developer.linkedin.com/)?

NO! To retrieve structured data, the [Linkedin Website](https://linkedin.com) uses a service they call **Voyager**. Voyager endpoints give us access to pretty much everything we could want from Linkedin: profiles, companies, connections, messages, etc. - anything that you can see on linkedin.com, we can get from Voyager.

This project aims to provide complete coverage for Voyager.

[How does it work?](#in-depth-overview)

### In-depth overview

Voyager endpoints look like this:

```text
https://www.linkedin.com/voyager/api/identity/profileView/tom-quirk
```

Or, more clearly

```text
 ___________________________________ _______________________________
|             base path             |            resource           |
https://www.linkedin.com/voyager/api /identity/profileView/tom-quirk
```

They are authenticated with a simple cookie, which we send with every request, along with a bunch of headers.

To get a cookie, we POST a given username and password (of a valid Linkedin user account) to `https://www.linkedin.com/uas/authenticate`.

### To find endpoints

We're looking at the Linkedin website and we spot some data we want. What now?

The most reliable method to find the relevant endpoint is to:

1. `view source`
2. `command-f`/search the page for some keyword in the data. This will exist inside of a `<code>` tag.
3. Scroll down to the **next adjacent element** which will be another `<code>` tag, probably with an `id` that looks something like

   ```html
   <code style="display: none" id="datalet-bpr-guid-3900675">
     {"request":"/voyager/api/identity/profiles/tom-quirk/profileView","status":200,"body":"bpr-guid-3900675"}
   </code>
   ```

4. The value of `request` is the url! 🤘

You can also use the `network` tab in you browsers developer tools, but you will encounter mixed results.

### How Clients query Voyager

linkedin.com queries data using the [Rest-li Protocol](https://linkedin.github.io/rest.li/spec/protocol). Rest-li is an internal query language/syntax where clients (like linkedin.com) to specify what data they want (similar to the GraphQL concept).

Here's an example of making a request for an organisation's `name` and `groups` (the Linkedin groups it manages):

```text
/voyager/api/organization/companies?decoration=(name,groups*~(entityUrn,largeLogo,groupName,memberCount,websiteUrl,url))&q=universalName&universalName=linkedin
```

The "querying" happens in the `decoration` parameter, which looks like

```text
(
    name,
    groups*~(entityUrn,largeLogo,groupName,memberCount,websiteUrl,url)
)
```

So here, we request an organisation name, and a list of groups, where for each group we want `largeLogo`, `groupName`, etc.

Different endpoints use different parameters (and perhaps even different syntaxes) to specify these queries. Notice that the above query had a parameter `q` whose value was `universalName`; the query was then specified with the `decoration` parameter.

In contrast, the `/search/cluster` endpoint uses `q=guided`, and specifies its query with the `guided` parameter, whose value is something like

```text
List(v->PEOPLE)
```

It could be possible to document (and implement a nice interface for) this query language - as we add more endpoints to this project, I'm sure it will become more clear if such a thing would be possible (and if it's worth it).

