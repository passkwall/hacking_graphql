# Hacking GraphQL
A quick repo with a demo node GraphQL application and pentesting tips, tricks, notes from the field. This is more of a `mindset` guide rather than a technical guide.

I've tried to summarize a lot of information from HackTricks, YouTube, HTB write-ups, disclosed vulnerabilities, and the GraphQL documentation to come up with succinct notes on GraphQL. This way you don't need to be an expert to focus on what's important. 

I'm not claiming to be an expert on GraphQL, but enough to know what I'm trying to accomplish in the field (so be kind please :slightly_smiling_face:).  

## What's in this repo?
- [Sample applicaton `/app` with quickstart](./app)
- [What should we know?](#whatis)
- [Attack Vectors](#vectors)

## What should pentesters know about GraphQL? <a name="whatis" ></a>

> tl;dr - GraphQL is a querying tool.  That's it.

So when trying to distill `what` GraphQL is for our purposes, it's important to call out that GraphQL's purpose is to more efficiently query data sources on the back end of a web application.  Instead of the client sending a bunch of separate API calls to the back-end, one carefully crafted GraphQL query can be sent to retrieve the data that the client needs.  This, in theory, reduces the number of RESTful calls made to the server since the results can be batched together and results "picked" so we're not returning too much information that isn't necessary.

To make the most of GraphQL, there are a variety of drivers out there which make it possible to use GraphQL transactions with other kinds of storage mechanisms.  SQL and No-SQL databases can be queried with GraphQL and, according to some docs, concurrently (neat!) which allows for multiple GraphQL "resolvers" to interact with multiple data sources for a single server. 

To recap, multiple queries :arrow_right:  one GraphQL query. 

<a name="vectors"></a>

## What kind of attack vectors apply to services using GraphQL? 

This is where things get somewhat _not interesting_ as far as general pentesting goes.  I guess that, at the time of this being written, the concept is relatively newer with 2015 being an initial release and 2018 being a stable release.  In short, here are a few attack vectors: 

- [Data Disclosure and Enumeration via Introspection](#introspection)
- [Specific bugs with the package/module implementing GraphQL](#modules)
- [Issues with the application itself](#appissues)


<a name="introspection"></a>

### Data Disclosure and Enumeration via Introspection :eyes:
[Introspection](https://graphql.org/learn/introspection/), as it relates to GraphQL, is a way of asking GraphQL what schemas are supported.  As an attacker, the idea is that if you can query schemas then you can enumerate what else GraphQL returns to you as the end-user.

If you're able to supply a query string to enumerate schemas, consider yourself lucky as a pentester because now you can figure out how to query the rest of the big data behind the service.

Example:
```
query={__schema{types{name,fields{name}}}}
```

[HackTricks](https://book.hacktricks.xyz/pentesting/pentesting-web/graphql#introduction) already covers this so I won't beat the topic to death here.

In addition, since this is a glaring security issue, it's worth noting that some libraries have implementations to [turn off introspection](https://lab.wallarm.com/why-and-how-to-disable-introspection-query-for-graphql-apis/).  If introspection is turned off, you can still invoke error-based enumeration.  If no errors are returned to you, your chances for enumeration have gone from super slim to `none`.

For more information on this technique, there are HackTheBox write-ups for the now-retired ['Help' lab](https://0xrick.github.io/hack-the-box/help/).

<a name="modules"></a>

### Specific bugs with the package/module implementing GraphQL :bug:

As long as there are no misconfigurations with the GraphQL implementation itself, and you can't expose sensitive or privilege information, next up would be to see if the package using GraphQL is vulnerable.  At this time of this writing, there are only 32 CVEs according to [cve.mitre.org](https://cve.mitre.org/cgi-bin/cvekey.cgi?keyword=graphql) and most of them have to do with services using GraphQL and poor implementation or underlying application issues.

My recommendation, stick to tracking down vulnerabilities that directly impact the tech stack you're working with.  For example if you're attacking GraphQL implemented on Express, check for vulnerabilities in packages such as [express-graphql](https://snyk.io/vuln/npm:express-graphql) or [graphql for node](https://snyk.io/vuln/npm:graphql).

<a name="appissues"></a>

### Issues with the application itself :interrobang:

This is where the basics come back into play. Forget the fact that you're using GraphQL as a wrapper, think about the application itself.

Ask yourself questions like:
- What database is on the back-end? Is it susceptible to injection attacks? 
- Who _should_ be making this query? Are there any authentication or authorization checks that can be bypassed?
- How fungible is the query?  Is there poor schema validation where I can request additional information from GraphQL? 

Bring yourself back to OWASP Top 10 territory rather on specific GraphQL attacks at this point.

<a name="resources"></a>

## Additional Resources
- [APISecurity.io - Common GraphQL Vulnerabilities](https://apisecurity.io/issue-82-common-graphql-vulnerabilities-pentesting-insomnia/)
- [Vulners: GraphQL](https://vulners.com/search?query=graphql)
- [GraphQL.org - Introduction](https://graphql.org/learn/)
- [PayloadAllTheThings - GraphQL](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/GraphQL%20Injection/README.md)