# Marine protist occurrence data 2009-2017

[Darwin Core](https://dwc.tdwg.org) (DwC) data for [npolar.no]'s [marine-db/milestone/1](https://github.com/npolar/marine-db/milestone/1): [Darwin Core Occurrences](https://dwc.tdwg.org/terms/#occurrence), programmatically compiled from original IOPAN protist spreadsheets.

The occurrences are provided as [NDJSON](./ndjson) and [TSV](./tsv).

The data derives from 1754+ samples from 16 scientific expeditions (see [expeditions-fieldnumbers](./log/expeditions-fieldnumbers.txt) for sample counts per cruise) and amounts to about 37000 counted occurrences of protist cells.

See also

- [input files](./log/line-counts.txt) with line number counts
- [issues of milestone 1](https://github.com/npolar/marine-db/issues?q=is%3Aissue+milestone%3A%22Original+IOPAN+protist+data+2009-2017+converted+to+Darwin+Core%22)
- [GBIF validation report](https://www.gbif.org/tools/data-validator/1635188739144)

## Calculations

## Formulas

In the DwC output, `organismQuantity` (protist cells/l) is provided, along with the number of cells counted (`individualCount`) and the volume (`sampleSizeValue`) in litres. It follows that `organismQuantity` may be (re-)calculated by  
`individualCount/sampleSizeValue`.

The counting/sediment chamber volume in `sampleSizeValue` is given by:

```js
initialVolume *
  dilutionFactor *
  (fieldsInCount / maxFields) *
  (takenVolume / bottleVolume);
```

## Validate calculations

A [deno](https://deno.land) script is [provided](./iopan_protist_recalc) to recalculate and validate Darwin Core occurrence NDJSON from stdin.

### perfect

```sh
$ cat ndjson/* | ./iopan_protist_recalc  | deno run https://deno.land/x/newline@v0.2.0/nd-count.js --select perfect

```

```ndjson
{"perfect":true,"count":30423}
{"perfect":null,"count":5662}
{"perfect":false,"count":818}
```

### ratios

```
$ cat ndjson/* | ./iopan_protist_recalc | nd-count --select volumeRatio
```

```ndjson

{"volumeRatio":1,"count":31161}
{"volumeRatio":null,"count":5662}
{"volumeRatio":0.92,"count":56}
{"volumeRatio":0.1,"count":20}
{"volumeRatio":0.28,"count":4}

```

```sh
cat ndjson/* | ./iopan_protist_recalc | nd-count --select quantityRatio | nd-group 'parseInt(values(d)[0])' | nd-map '[d[0], d[1].map(({count,...r})=>[count,r])]'
```

```ndjson
[{"quantityRatio":1,"count":30499},{"quantityRatio":1.5,"count":2},{"quantityRatio":1.3,"count":2},{"quantityRatio":1.1,"count":2},{"quantityRatio":1.7,"count":1}]
[{"quantityRatio":null,"count":5666}]
[{"quantityRatio":2,"count":18},{"quantityRatio":2.4,"count":1},{"quantityRatio":2.5,"count":1}]
[{"quantityRatio":0.1,"count":597},{"quantityRatio":0.05,"count":1},{"quantityRatio":0.5,"count":13},{"quantityRatio":0.13,"count":2},{"quantityRatio":0.33,"count":10},{"quantityRatio":0.25,"count":6},{"quantityRatio":0.089,"count":1},{"quantityRatio":0.14,"count":2},{"quantityRatio":0.67,"count":2},{"quantityRatio":0.75,"count":1},{"quantityRatio":0.17,"count":2},{"quantityRatio":0.054,"count":1},{"quantityRatio":0.067,"count":1},{"quantityRatio":0.034,"count":1},{"quantityRatio":0.22,"count":2},{"quantityRatio":0.024,"count":1},{"quantityRatio":0.83,"count":1},{"quantityRatio":0.6,"count":1},{"quantityRatio":0.11,"count":2},{"quantityRatio":0.9,"count":1},{"quantityRatio":0.27,"count":2},{"quantityRatio":0.15,"count":1},{"quantityRatio":0.014,"count":1},{"quantityRatio":0.59,"count":1},{"quantityRatio":0.083,"count":1},{"quantityRatio":0.32,"count":1},{"quantityRatio":0.53,"count":1},{"quantityRatio":0.44,"count":1},{"quantityRatio":0.02,"count":18}]
[{"quantityRatio":8,"count":2}]
[{"quantityRatio":4,"count":2},{"quantityRatio":4.5,"count":2},{"quantityRatio":4.3,"count":1}]
[{"quantityRatio":3,"count":7},{"quantityRatio":3.7,"count":1}]
[{"quantityRatio":79,"count":1}]
[{"quantityRatio":9,"count":2}]
[{"quantityRatio":6,"count":3},{"quantityRatio":6.3,"count":1}]
[{"quantityRatio":18,"count":1}]
[{"quantityRatio":5,"count":1},{"quantityRatio":5.8,"count":1},{"quantityRatio":5.5,"count":1}]
[{"quantityRatio":51,"count":1}]
[{"quantityRatio":7.6,"count":1},{"quantityRatio":7,"count":1}]
[{"quantityRatio":36,"count":1}]
[{"quantityRatio":20,"count":1}]
[{"quantityRatio":13000,"count":1}]
[{"quantityRatio":25000,"count":1}]
[{"quantityRatio":6300,"count":1}]
[{"quantityRatio":57000,"count":1}]
[{"quantityRatio":11,"count":1}]

```

## Taxonomy

[464 uniq spellings of scientific names are are used](./log/taxonomy-uniq-counts.txt)

```sh
$ cat ndjson/* | nd-count --select scientificName | nd-filter 'd.scientificName?.length>0' | gbif-species-api --worms --raw | nd-map 'r0=d.results?.[0]??{}, {key,canonicalName,taxonID,issues,synonym,references}=r0, {scientificName}=d, {scientificName, canonicalName,taxonID,issues,key,synonym,references}' > log/taxonomy-gbif-api-worms.ndjson
```

307 names have a known WoRMS LSID

```sh
che@:~/akvaplan-niva/iopan_legacy_protist_dwc$ cat log/taxonomy-gbif-api-worms.ndjson  | nd-count '{ worms: d?.taxonID?.match(/^urn:lsid:marinespecies\.org/) }'
{"count":156}
{"worms":["urn:lsid:marinespecies.org"],"count":307}
```

## Events

The sampling events for this dataset is published in API v2:
https://v2-api.npolar.no/darwin-core/sample/

At the moment, 51 samples spread over 1183 occurrences have a fieldNumber that is not found in the above API.
List: [error-fieldnumber-not-found.txt](./log/error-fieldnumber-not-found.txt)
