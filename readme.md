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
$ cat ndjson/* | ./iopan_protist_recalc --occurrence | nd-count --select 'perfect'

```

```ndjson
{"perfect":true,"count":30998}
{"perfect":null,"count":5662}
{"perfect":false,"count":243}

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
