# Modify plugin
It modifies the content for a field or add new field. It works only with strings.
You can provide an unlimited number of config parameters. Each parameter handled as `cfg.FieldSelector`:`cfg.Substitution`.

> Note: When used to add new nested fields, each child field is added step by step, which can cause performance issues.

**Example:**
```yaml
pipelines:
  example_pipeline:
    ...
    actions:
    - type: modify
      my_object.field.subfield: value is ${another_object.value}.
    ...
```

The resulting event could look like:
```
{
  "my_object": {
    "field": {
      "subfield":"value is 666."
    }
  },
  "another_object": {
    "value": 666
  }
```

**Filters**

Sometimes it is required to extract certain data from fields and for that purpose filter chains were added.
Filters are added one after another using pipe '|' symbol and they are applied to the last value in the chain.

For example, in expression `${field|re("(test-pod-\w+)",-1,[1],",")|re("test-pod-(\w+)",-1,[1],",")}` first the value of 'field' is retrieved,
then the data extracted using first regular expression and formed into a new string, then the second regular expression is applied
and its result is formed into a value to be put in modified field.

Currently available filters are:
+ `regex filter` - `re(regex string, limit int, groups []int, separator string)`, filters data using `regex`, extracts `limit` occurrences,
takes regex groups listed in `groups` list, and if there are more than one extracted element concatenates result using `separator`.
Negative value of `limit` means all occurrences are extracted, `limit` 0 means no occurrences are extracted, `limit` greater than 0 means
at most `limit` occurrences are extracted.

Examples:

Example #1:

Data: `{"message:"info: something happened"}`

Substitution: `level: ${message|re("(\w+):.*",-1,[1],",")}`

Result: `{"message:"info: something happened","level":"info"}`

Example #2:

Data: `{"message:"re1 re2 re3 re4"}`

Substitution: `extracted: ${message|re("(re\d+)",2,[1],",")}`

Result: `{"message:"re1 re2 re3 re4","extracted":"re1,re2"}`

Example #3:

Data: `{"message:"service=service-test-1 exec took 200ms"}`

Substitution: `took: ${message|re("service=([A-Za-z0-9_\-]+) exec took (\d+\.?\d*(?:ms|s|m|h))",-1,[2],",")}`

Result: `{"message:"service=service-test-1 exec took 200ms","took":"200ms"}`


<br>*Generated using [__insane-doc__](https://github.com/vitkovskii/insane-doc)*