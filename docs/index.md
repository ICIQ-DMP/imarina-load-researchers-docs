# imarina-load-researchers

`imarina-load-researchers` produces the researcher-data spreadsheet that ICIQ
loads into [iMarina](https://en.wikipedia.org/wiki/Research_information_system),
its CRIS (Current Research Information System). It merges an A3 HR database
dump with the previous iMarina load into the next iMarina-format load file,
using a Sharepoint/Microsoft List-driven pipeline (`download` → `build` →
`upload` → human review/approval → `publish`) orchestrated by Jenkins and
Power Automate. It's for the ICIQ staff who maintain the researcher list kept
in sync between HR records and iMarina.

## Where to start

| I want to… | Go to… |
|---|---|
| Learn by doing | [Tutorials](tutorials/index.md) |
| Solve a specific problem | [How-To Guides](how-to/index.md) |
| Look up a class or function | [Reference](reference/index.md) |
| Understand the design | [Explanation](explanation/index.md) |
